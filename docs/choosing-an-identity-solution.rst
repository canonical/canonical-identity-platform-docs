.. meta::
    :description: Comparison and selection guide to choose between Charmed Authentik, Canonical Identity Platform, and GLAuth for enterprise and consumer identity use cases.

.. _choosing-an-identity-solution:

Choosing an identity solution
=============================

Canonical Identity offers identity and access management solutions tailored to enterprise identity and consumer identity architectures.

Overview of options
-------------------

.. list-table::
    :header-rows: 1
    :widths: 50 50

    * - Use case & requirements
      - Recommended solution
    * - **Enterprise Identity**: Unified Single Sign-On (SSO), LDAP and SAML protocol bridging, Active Directory synchronization, and built-in administration console
      - :doc:`Charmed Authentik <authentik/index>`
    * - **Consumer Identity**: Cloud-native high-scale OIDC and OAuth 2.0 workloads, customizable login journeys, and independent component scaling
      - :doc:`Canonical Identity Platform <identity-platform/index>` (Ory stack)
    * - **Development & Testing**: Lightweight read-only LDAP endpoint for local prototyping and staging environments
      - :doc:`GLAuth </identity-platform/reference/charms/glauth>` (Identity Platform component)

Enterprise Identity with Charmed Authentik
------------------------------------------

:doc:`Charmed Authentik <authentik/index>` is the recommended solution for enterprise identity and access management. It provides an all-in-one identity provider (IdP) and directory gateway uniting modern OpenID Connect (OIDC) authentication with legacy LDAP protocols and Active Directory synchronization under a single control plane.

Authentik is managed through a built-in administrative console where authentication flows, stages, policies, and property mappings can be configured without code. This provides operators with a straightforward deployment footprint and a rich day-2 operations interface out of the box.

Choose Charmed Authentik when you need to serve internal business applications, provide SSO for services like Grafana, synchronize existing Active Directory domains, or bridge legacy LDAP-dependent systems (such as Linux PAM or SSSD).

Consumer Identity with Canonical Identity Platform
--------------------------------------------------

The :doc:`Canonical Identity Platform <identity-platform/index>` (built on the Ory open-source stack including Kratos and Hydra) is engineered for high-scale, cloud-native consumer identity and API authentication workloads. Each component is a stateless service that scales horizontally and independently, allowing high-traffic authentication endpoints to scale without over-provisioning other subsystems.

The platform provides granular control over user-facing authentication flows. Its login interface is developed by Canonical and exposes extensible self-service hooks, allowing end-to-end customization of branding, registration, verification, and multi-factor authentication steps.

Choose the Canonical Identity Platform when building customer-facing web and mobile applications that require horizontal scalability for large user bases, API-first architecture, or bespoke authentication journeys tailored beyond standard configuration templates.

Development LDAP with GLAuth
----------------------------

:doc:`GLAuth </identity-platform/reference/charms/glauth>` is a lightweight, read-only LDAP server interface included as a component of the Canonical Identity Platform stack. It provides a simple LDAP endpoint suitable for local development, integration testing, and lightweight staging environments.

GLAuth is not intended as a production directory bridge. For enterprise directory integration, protocol bridging, and unified user management, use Charmed Authentik.
