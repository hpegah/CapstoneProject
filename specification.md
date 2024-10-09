# Automated NDN CERT Specification

## 1. Introduction
The NDN testbed consists of multiple nodes, which are used to announce prefixes. Additionally, some nodes can be added to the testbed as routers. When a new testbed node is added, in order for it to announce prefixes, it needs to be authenticated and issued an NDN certificate. Currently the NDNCERT protocol is used for issuing certificates. This is a manual process that requires human authentication of the testbed nodes and issuance of certificates to the nodes. Additionally, in the case of a key getting compromised, the certificate has to be manually revoked and the node has to be issued a new one. This process is a reactive approach to security rather than a preventative approach of frequently rotating the certificates to avoid the issue of key compromise.
## 2.

## 2. Proof of Possession
### 2.1 Specification
