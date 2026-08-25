.. _openfga-security:

Charmed OpenFGA Security
========================

This document provides cryptographic documentation for the OpenFGA charm. Its purpose is to track the exposure of charm code to cryptographic attack vectors.

What is not included in this document and regarded as out of scope:

* Workload code (refer to the workloads’ cryptographic documentation).
* Data at rest encryption.

Sensitive Data Exchange
-----------------------

The charm relies on Juju secrets:

* To pass an OpenFGA token to the requirer charm. Implemented in `lib/charms/openfga_k8s/v1/openfga.py <https://github.com/canonical/openfga-operator/blob/ef27bcec0699a6f67ed6ae9130d7b78a6c79c821/lib/charms/openfga_k8s/v1/openfga.py>`_.
* To get OpenFGA datastore credentials. Implemented in `lib/charms/data_platform_libs/v0/data_interfaces.py <https://github.com/canonical/openfga-operator/blob/ef27bcec0699a6f67ed6ae9130d7b78a6c79c821/lib/charms/data_platform_libs/v0/data_interfaces.py>`_.

Github secrets are used during development, build, test and deploy phases:

* To get Charmcraft credentials that are used to interact with Charmhub.
* To get a Github token that is used to interact with Github API.

Cryptographic tech and packages in use
--------------------------------------

OpenFGA charm uses the following cryptography packages:

* Python secrets built-in library is used to create an OpenFGA token.

OpenFGA charm `supports <https://github.com/canonical/openfga-operator?tab=readme-ov-file#tls-certificates-interface>`_ TLS encryption on internal and external connections.  Security considerations related to TLS encryption:

* It is recommended against using self-signed certificates for production clusters.
* It is strongly recommended to use TLS v1.3, as it is more secure than v1.2.
