.. _kratos-reference:

Charmed Kratos
==============

``kratos`` is a `Kubernetes charm <https://documentation.ubuntu.com/juju/3.6/explanation/charms-vs-kubernetes-operators/>`_ for `Ory Kratos <https://github.com/ory/kratos>`_, an API-first, headless identity and user management system. It serves as the primary identity provider (IdP) within the `Canonical Identity Platform <https://charmhub.io/topics/canonical-identity-platform>`_.

The ``kratos`` charmed operator automates the deployment, scaling, and lifecycle management of the Kratos service on `Juju <https://juju.is/>`_. It manages user identities, credentials, and profile data, while orchestrating essential self-service flows such as registration, login, and multi-factor authentication (MFA). 

Core Responsibilities
---------------------

* **Identity Management:** Storing and managing user profiles based on customizable JSON schemas.
* **Self-Service Flows:** Providing secure APIs for registration, login, account recovery, settings, and verification.
* **Multi-Factor Authentication (MFA):** Supporting secure second-factor methods like TOTP, WebAuthn and backup codes.
* **Identity Brokering:** Integrating with external OIDC providers (Google, GitHub, Azure AD) via the Kratos External IdP Integrator. 

Project and community
---------------------

Kratos is a member of the Ubuntu family. It’s an open source project that warmly welcomes community projects, contributions, suggestions, fixes and constructive feedback.

* `Code of conduct <https://ubuntu.com/community/code-of-conduct>`_
* `Join the Discourse community forum <https://discourse.charmhub.io/tag/identity>`_
* `Join the Matrix community chat <https://matrix.to/#/!3xNKp3PFtjTOKbwJykMRzruCbSEMkXb8ZRrDKk1vknI?via=ubuntu.com>`_
* `Contribute on GitHub <https://github.com/canonical/kratos-operator>`_

Thinking about using Kratos for your next project? `Get in touch <https://matrix.to/#/!3xNKp3PFtjTOKbwJykMRzruCbSEMkXb8ZRrDKk1vknI?via=ubuntu.com>`_ with the team!
