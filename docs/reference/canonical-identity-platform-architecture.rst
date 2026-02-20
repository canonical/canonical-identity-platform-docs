.. _canonical-identity-platform-architecture:

Architecture overview
=====================

The Canonical Identity Platform is a comprehensive identity solution that serves two primary functions:

1. **Identity Provider:** It acts as a standalone source of truth for user identities, allowing you to store and manage user credentials, profiles, and sessions directly within the platform. This native identity management is powered by **Charmed Kratos**.
2. **Identity Broker:** It federates external identity providers (Microsoft Entra ID, Okta, Google, GitHub, etc.) to downstream service providers (Grafana, Kafka, and other workloads), creating a unified sign-on experience.

The diagram below illustrates the high-level architecture and dependencies of the solution:

.. image:: https://raw.githubusercontent.com/canonical/canonical-identity-platform-docs/main/Diagram_sources/arch-diagram.png
   :alt: Canonical Identity Platform Architecture
   :target: https://raw.githubusercontent.com/canonical/canonical-identity-platform-docs/main/Diagram_sources/arch-diagram.png

The Canonical Identity Platform orchestrates several interoperable charmed operators to deliver this functionality.

Core components
---------------

The solution consists of the following modular components:

* `Charmed Kratos <https://github.com/canonical/kratos-operator>`_ – The core **Identity Provider** engine. It handles user management, registration, login flows, and secure storage of local credentials (passwords, MFA).
* `Charmed Hydra <https://github.com/canonical/hydra-operator>`_ – The **OAuth 2.0 and OpenID Connect (OIDC) server**. It acts as the secure interface for applications to authenticate users, regardless of whether the identity comes from Kratos (local) or an external provider.
* `Charmed Identity Platform Login UI <https://github.com/canonical/identity-platform-login-ui-operator>`_ – The user interface for login, registration, settings, and account recovery screens.
* `Charmed Kratos External IDP Integrator <https://github.com/canonical/kratos-external-idp-integrator>`_ – A helper operator that facilitates the **Identity Broker** role by configuring Charmed Kratos to trust and exchange identities with external providers.
* `Charmed OpenFGA <https://github.com/canonical/openfga-operator>`_ *(Optional)* – An authorization engine based on Fine-Grained Authorization (FGA). It allows applications to define flexible permission models and offload complex access control logic from their code.
* `Charmed PostgreSQL <https://github.com/canonical/postgresql-k8s-operator>`_ – The database provider for Kratos and Hydra.
* `Charmed Traefik <https://github.com/canonical/traefik-k8s-operator>`_ – Ingress for public-facing endpoints and internal admin APIs.

Integration points
------------------

The Canonical Identity Platform leverages Juju relations and configs to simplify the integration of Single Sign-On (SSO) across your infrastructure. There are three main integration points:

* **oauth relation interface**:
    This is the standard interface for connecting applications (Service Providers) to the platform.

    * **Charmed Applications:** When you relate a charm to Charmed Hydra via ``oauth``, the platform automatically registers an OAuth client and manages its lifecycle.
    * **External Workloads:** Non-charmed applications that support OIDC can also be integrated using Charmed Hydra's `actions <https://charmhub.io/hydra/actions>`_.

* **openfga relation interface**:
    This interface connects applications to the **Authorization** engine.

    * Services can relate to Charmed OpenFGA to query permissions (e.g., "Can User A view Document B?") and store relationship tuples, decoupling authorization logic from application code.

* **Kratos External IdP Integrator** (via juju config):
    This interface manages the upstream connections when the platform acts as an identity broker.

    * By deploying an instance of the Integrator charm and setting the ``juju config``, you can dynamically register external identity providers.
    * Multiple providers can be supported simultaneously by deploying multiple instances of the Integrator charm.

Interested in learning how to integrate your application with the Canonical Identity Platform? Check our :doc:`how-to guides </how-to/index>`.
