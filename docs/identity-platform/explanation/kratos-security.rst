.. _kratos-security:

Charmed Kratos Security
=======================

This document provides cryptographic documentation for the Kratos charm. Its purpose is to track the exposure of charm code to cryptographic attack vectors.

What is not included in this document and regarded as out of scope:

* Workload code (refer to the workloads’ cryptographic documentation).
* Data at rest encryption.

Sensitive Data Exchange
-----------------------

The charm relies on Juju secrets:

* To store Kratos admin password.
* To store Kratos cookie secret that is used to encrypt session cookies.

Github secrets are used during development, build, test and deploy phases:

* To get Charmcraft credentials that are used to interact with Charmhub.
* To get a Github token that is used to interact with Github API.

Cryptographic tech and packages in use
--------------------------------------

Kratos charm uses the following cryptography packages:

* Python secrets built-in library is used to generate Kratos cookie secret.

Kratos depends on `ca-certificates dpkg package <https://code.launchpad.net/ubuntu/+source/ca-certificates>`_,
which is `based <https://git.launchpad.net/ubuntu/+source/ca-certificates/tree/mozilla/Makefile>`_ on Mozilla certificate bundle.
