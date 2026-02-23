.. _glauth-utils-reference:

Charmed GLAuth Utility K8s
==========================

``GLAuth-utils`` is a Kubernetes charmed operator designed to supplement the `glauth-k8s <https://charmhub.io/glauth-k8s>`_ charmed operator with administrative and maintenance features.

Key Features
------------

Currently, the utility charm facilitates the following operations:

* **LDIF Application:** Use the LDAP Data Interchange Format (LDIF) to apply data changes (adds, deletes, and modifications) directly into the LDAP directory.

Technical Usage
---------------

The charm provides a Juju action to manage directory data without requiring direct access to the backend database:

.. code-block:: shell

    # Example: Applying an LDIF file
    juju run glauth-utils/0 apply-ldif path=/path/to/data.ldif

Project and community
---------------------

The ``glauth-utils`` charmed operator is a member of the Ubuntu family. It’s an open source project that warmly welcomes community projects, contributions, suggestions, fixes and constructive feedback.

* `Code of conduct <https://ubuntu.com/community/code-of-conduct>`_
* `Join the Discourse community forum <https://discourse.charmhub.io/tag/identity>`_
* `Get support <https://github.com/canonical/glauth-utils/issues>`_
* `Contribute <https://github.com/canonical/glauth-utils/blob/main/CONTRIBUTING.md>`_

Thinking about using ``glauth-utils`` for your next project? `Get in touch with the team! <https://matrix.to/#/!3xNKp3PFtjTOKbwJykMRzruCbSEMkXb8ZRrDKk1vknI?via=ubuntu.com>`_
