.. meta::
    :description: Comparison and selection guide to choose between the Canonical Identity Platform, Charmed Authentik, and GLAuth.

.. _choosing-an-identity-solution:

Choosing an identity solution
=============================

Canonical Identity offers multiple identity and access management solutions to address different architectural requirements, protocol needs, and deployment scales.

Overview of options
-------------------

.. list-table::
    :header-rows: 1
    :widths: 50 50

    * - If you need
      - Recommended solution
    * - OIDC or OAuth2 only
      - :doc:`Canonical Identity Platform <identity-platform/index>` (Ory stack)
    * - The highest scalability
      - :doc:`Canonical Identity Platform <identity-platform/index>` (Ory stack)
    * - A deeply customizable login experience or custom authentication flow steps
      - :doc:`Canonical Identity Platform <identity-platform/index>` (Ory stack)
    * - An all-in-one identity provider with a built-in administrative console
      - :doc:`Charmed Authentik <authentik/index>`
    * - LDAP, SAML and OIDC from one control plane
      - :doc:`Charmed Authentik <authentik/index>`
    * - Lightweight LDAP for development and testing
      - :doc:`GLAuth </identity-platform/reference/charms/glauth>` (Identity Platform component)

Canonical Identity Platform
---------------------------

The :doc:`Canonical Identity Platform <identity-platform/index>` (built on the Ory open-source stack including Kratos and Hydra) is Canonical's flagship identity solution. It is designed for cloud-native environments requiring high scalability, granular self-service user flows, and enterprise-grade OpenID Connect (OIDC) and OAuth 2.0 authentication and authorization. Each component is a stateless service that scales horizontally and independently, so the parts of the platform under load can be scaled without scaling the rest.

The platform is also the more customizable of the two solutions. Its login interface is developed by Canonical rather than inherited from an upstream product, and the underlying self-service flows expose integration hooks, so the login journey can be tailored end to end — from branding to entirely custom authentication and verification steps.

Choose the Canonical Identity Platform when your infrastructure relies primarily on modern web and API authentication protocols (OIDC and OAuth 2.0), needs to scale horizontally to accommodate large user bases and high request volumes, or requires a login experience tailored beyond what configuration alone can achieve.

Charmed Authentik
-----------------

:doc:`Charmed Authentik <authentik/index>` provides an all-in-one identity provider (IdP) and directory gateway. While the Canonical Identity Platform focuses on OIDC and OAuth 2.0, it does not provide an integrated LDAP bridge. Charmed Authentik fills this gap by providing both an LDAP provider (via the Authentik LDAP Outpost) and an LDAP source for directory synchronization.

Authentik is configured through its built-in administrative console: authentication flows, stages, policies, and property mappings can all be changed without code. This makes it highly configurable and gives operators a rich day-2 interface out of the box, with a simpler deployment footprint than the platform. Deep changes to the login interface itself are limited to theming, since that interface is part of upstream Authentik.

Choose Charmed Authentik when you need to serve both modern OIDC applications and legacy LDAP-oriented consumers (such as Linux PAM, SSSD, or legacy enterprise software) from a single unified control plane, or when integrating Active Directory as an upstream directory source.

GLAuth (LDAP for development and testing)
-----------------------------------------

:doc:`GLAuth </identity-platform/reference/charms/glauth>` is a lightweight, read-only LDAP server interface included as a component of the Canonical Identity Platform stack. It provides a simple LDAP endpoint suitable for local development, integration testing, and lightweight staging environments.

GLAuth is not intended as a full-featured production LDAP bridge. For production directory integration and unified protocol support, use Charmed Authentik.
