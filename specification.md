# NDNCERT Capstone Specification

## 1. Introduction
The NDN testbed consists of multiple nodes, which are used to announce prefixes. Additionally, some nodes can be added to the testbed as routers. When a new testbed node is added, in order for it to announce prefixes, it needs to be authenticated and issued an NDN certificate. Currently the NDNCERT protocol is used for issuing certificates. This is a manual process that requires human authentication of the testbed nodes and issuance of certificates to the nodes. Additionally, in the case of a key getting compromised, the certificate has to be manually revoked and the node has to be issued a new one. This process is a reactive approach to security rather than a preventative approach of frequently rotating the certificates to avoid the issue of key compromise.

## 2. Proposal
This project proposes an automated version of the NDNCERT protocol in which the certificate authority will be contacted by the testbed node (client), then authenticate the client through some challenge response, receive a certificate signing request (CSR) from client, and issue the certificate. The certificates will be generated with a specified lifetime so that clients must be issued new certificates frequently to prevent the issue of a compromised key.

This protocol uses proof of possession to ensure the client owns the private key corresponding to the public key signed in their x.509 certificate. This protocol assumes that the client has the CA installed in its list of trusted CAs, has generated a key value pair, and already possesses an x.509 certificate. The client then selects the CA and contacts the CA to receive the ownership challenge. The CA issues a random number (nonce), which the client then signs using its private key. This allows the CA to verify the client’s ownership of the private key. Along with the signed nonce, the client also creates a CSR to request a certificate from the CA. This CSR is signed with the client’s private key so the CA can verify the request. The CA then creates a certificate with the public key of the client and issues the certificate to the client.

Additionally, since there are some testbed nodes that have permission to act as routers in the testbed, there need to be certificates specific for these clients. These router nodes are specified on an access control list (ACL). This protocol will utilize another CA under the /router prefix which will authorize these specific clients using the ACL and issue them router certificates.

## 3. x.509 Proof Of Possession Challenge Specification

The propsed x.509 Certificate Proof of Possession challenge uses the existing [NDNCERT Protocol](https://github.com/named-data/ndncert/wiki/NDNCERT-Protocol-0.3) specification. The x.509 Certificate Proof of Possession specification will be an additional challenge type and is based of the [NDNCERT Proof of Possession challenge type](https://github.com/named-data/ndncert/wiki/NDNCERT-Protocol-0.3-Challenges).

### 3.1 Overview

* Challenge ID: `509-possession`
* Description: The CA requires the requester to prove their ownership of an x.509 certificate issued by the same or a different CA.
* Required round trips: 2
* Require out-of-band operations: no
* Mutual Verification: no
* Time limit: 60 seconds
* Allowed number of attempts: 1

### 3.2 Challenge Specification

1. The requester transmits a CHALLENGE request with the following payload, which contains the entire x.509 certificate chain:

   * selected-challenge: `509-possession`
   * parameter-key: `issued-cert`
   * parameter-value: Chain of X.509 certificates, starting with the entity cert and ending with the root cert, encoded as a base64 string
     
2. The CA generates a 128-bit random number, and then responds with a CHALLENGE response with payload:

   * status: `1` (challenge in progress)
   * challenge-status: `need-proof`
   * remaining-tries
   * remaining-time
   * parameter-key: `nonce`
   * parameter-value: the 128-bit random number, in `16OCTET` format

3. The requester signs the 128-bit random number with the private key corresponding to the existing certificate, and then transmits a CHALLENGE request with the following payload:

   * selected-challenge: `possession`
   * parameter-key: `proof`
   * parameter-value: an ASN.1 signature over the 128-bit random number

4. The CA validates the provided certificate against its policy, and checks the proof signature against the public key enclosed in the provided certificate.

   The CA fails the challenge if any of these applies:

   * The provided certificate is malformed.
   * The provided certificate has expired.
   * The provided certificate has been revoked.
   * The provided certificate is not trusted according to the CA's trust schema.
   * The provided certificate is not authorized to obtain the requested certificate according to the CA's policy.
   * The proof signature cannot be validated against the enclosed public key.

   If the challenge fails, the CA responds with a CHALLENGE response with payload:

   * status: `4` (failed)

   Note that the CA cannot leak any information about why the challenge has failed.
   Even if some of these validations could have occurred in step 2, the CA cannot return an error at that step.

   If the challenge succeeds, the CA continues to certificate issuance step as defined in the main NDNCERT protocol.
