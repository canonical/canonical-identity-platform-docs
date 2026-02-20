.. _identity-platform-login-ui-security:

Charmed Login UI Security
=========================

This document provides cryptographic documentation for the Login UI charm. Its purpose is to track the exposure of charm code to cryptographic attack vectors.

What is not included in this document and regarded as out of scope:

* Workload code (refer to the workloads’ cryptographic documentation).
* Data at rest encryption.

Sensitive Data Exchange
-----------------------

Github secrets are used during development, build, test and deploy phases:

* To get Charmcraft credentials that are used to interact with Charmhub.
* To get a Github token that is used to interact with Github API.

Cryptographic tech and packages in use
--------------------------------------

Login UI charm code does not directly rely on cryptographic packages.
