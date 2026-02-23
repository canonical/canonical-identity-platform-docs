.. _hydra-security:

Charmed Hydra Security
======================

This document provides cryptographic documentation for the Hydra charm. Its purpose is to track the exposure of charm code to cryptographic attack vectors.

What is not included in this document and regarded as out of scope:

* Workload code (refer to the workloads’ cryptographic documentation).
* Data at rest encryption.

Sensitive Data Exchange
-----------------------

The charm relies on Juju secrets:

* To pass Hydra client secret that is used to access Hydra API.
* To pass Hydra system secret that is used to encrypt Hydra's database.

Github secrets are used during development, build, test and deploy phases:

* To get Charmcraft credentials that are used to interact with Charmhub.
* To get a Github token that is used to interact with Github API.

Cryptographic tech and packages in use
--------------------------------------

Hydra charm uses the following cryptography packages:

* Python secrets built-in library is used to generate random strings for the cookie and database encryption/signing.
