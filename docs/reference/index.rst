.. meta::
    :description: Information about architecture, flows, charm integrations, and other reference material for quick lookup.

.. _reference:

Reference
=========

Information about architecture, flows, charm integrations, and other reference material for quick lookup.

Platform Overview
-----------------
General documentation covering the architecture and cross-cutting features of the Identity Platform.

* :doc:`Architecture <canonical-identity-platform-architecture>`: Technical breakdown of core components and communication flows.
* :doc:`Self-service flows <self-service-flows>`: Details on user-initiated actions like recovery and registration.
* :doc:`Observability <canonical-identity-platform-observability>`: Technical specs for metrics, alerts, and dashboards.

Components
----------
Technical reference material for the individual charms that make up the Identity Platform stack.

.. list-table::
    :header-rows: 1
    :widths: 30 70

    * - Component
      - Description
    * - :doc:`Kratos <kratos>`
      - Identity and user management system.
    * - :doc:`Hydra <hydra>`
      - OAuth 2.0 and OpenID Connect (OIDC) server.
    * - :doc:`Login UI <identity-platform-login-ui>`
      - Frontend entry point and identity broker interface.
    * - :doc:`Kratos External IdP Integrator <kratos-external-idp-integrator>`
      - Configuration tool for third-party OIDC providers.
    * - :doc:`OpenFGA <openfga>`
      - Relationship-based access control (ReBAC) engine.
    * - :doc:`OAuth2 Proxy <oauth2-proxy>`
      - Identity and Access Proxy (IAP) for non-OIDC applications.
    * - :doc:`GLAuth <glauth>`
      - Lightweight LDAP interface for the Identity Platform.
    * - :doc:`GLAuth Utils <glauth-utils>`
      - Utility tools for managing the GLAuth interface.

.. toctree::
    :hidden:

    canonical-identity-platform-architecture
    self-service-flows
    canonical-identity-platform-observability
    kratos
    hydra
    openfga
    oauth2-proxy
    identity-platform-login-ui
    kratos-external-idp-integrator
    glauth
    glauth-utils
