.. _kratos-external-idp-integrator-security:

Charmed Kratos External IdP Integrator Security
===============================================

This document provides cryptographic documentation for the Kratos External IdP Integrator charm. Its purpose is to track the exposure of charm code to cryptographic attack vectors.

What is not included in this document and regarded as out of scope:

* Workload code (refer to the workloads’ cryptographic documentation).
* Data at rest encryption.

Sensitive Data Exchange
-----------------------

The charm relies on Juju secrets:

* To pass Kratos client secret that is used for external IdP integrations.

Github secrets are used during development, build, test and deploy phases:

* To get Charmcraft credentials that are used to interact with Charmhub.
* To get a Github token that is used to interact with Github API.

Cryptographic tech and packages in use
--------------------------------------

Kratos External IdP Integrator charm code does not directly rely on cryptographic packages.
