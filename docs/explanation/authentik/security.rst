.. _authentik-security:

Charmed Authentik Security Architecture
=======================================

.. meta::
   :description: Security boundaries, secret lifecycle, authentication flows, compliance posture, and accepted risks in Charmed Authentik.

This document provides a technical overview of the security architecture, cryptographic boundaries, trust model, and operational limitations of the Charmed Authentik suite (``authentik-server``, ``authentik-worker``, and ``authentik-ldap-outpost``).

Secret management and cryptography
----------------------------------

Charmed Authentik follows a zero-configuration secret generation model:

* **No secret-typed configuration options**: Administrators never pass raw passwords or secret keys through Juju configuration. The charms generate and manage all secrets internally using Python's ``secrets`` standard library.
* **Juju Secret URI exchange**: Secrets are stored as Juju-native Secret objects owned by the respective application and are referenced across relations exclusively via Secret URIs. Secret fields are excluded from plain relation databags using Pydantic's ``Field(exclude=True)``.
* **Revocation on teardown**: When a relation is broken, secret access granted to related applications or units is revoked by the Juju controller.

Charm-generated secrets
~~~~~~~~~~~~~~~~~~~~~~~

The suite manages six distinct Juju secrets:

.. list-table::
   :header-rows: 1
   :widths: 30 25 25 20

   * - Secret Label / ID
     - Stored Keys
     - Owner Application
     - Shared With
   * - ``authentik-server-secrets``
     - ``secret-key``, ``bootstrap-token``, ``bootstrap-password``
     - ``authentik-server``
     - Server-internal (not shared)
   * - ``authentik-secret-key``
     - ``secret-key``, ``db-password``
     - ``authentik-server``
     - Related ``authentik-worker`` applications
   * - ``authentik-api-token``
     - ``api-token``
     - ``authentik-server``
     - Related ``authentik-ldap-outpost`` applications
   * - ``client_secret_<relation-id>``
     - ``secret``
     - ``authentik-server``
     - Consuming OIDC application on the ``oauth`` relation
   * - ``authentik-ldap-outpost-token-<identity>``
     - ``token``
     - ``authentik-ldap-outpost``
     - Outpost-internal (not shared)
   * - ``relation-<relation-id>-bind-account-secret``
     - ``password``
     - ``authentik-ldap-outpost``
     - Consuming LDAP client on the ``ldap`` relation

Cryptographic entropy
~~~~~~~~~~~~~~~~~~~~~

Random strings are generated using cryptographically secure random bytes (``secrets.token_urlsafe``):

* **Secret key and bootstrap token**: ``token_urlsafe(50)`` (~300 bits of entropy).
* **Bootstrap password and OIDC client secret**: ``token_urlsafe(32)`` (~192 bits of entropy).
* **OIDC client ID and LDAP bind password**: ``token_urlsafe(16)`` (~96 bits of entropy).

.. note::
   Juju secrets created by the charms currently have no automated rotation or expiration policy attached.

Non-interactive LDAP bind flow
------------------------------

Authentik's standard browser authentication flows incorporate interactive multi-factor authentication (MFA) prompts. Because programmatic LDAP clients cannot handle interactive web challenges, the LDAP Outpost charm configures a dedicated non-interactive bind pipeline during initialization:

* **Flow Slug**: ``default-ldap-bind-flow``
* **Designation**: ``authentication``
* **Bound Stages**:
  
  1. ``default-authentication-identification`` (Order: 10)
  2. ``default-authentication-password`` (Order: 20)
  3. ``default-authentication-login`` (Order: 100)

* **Policy Bindings**: None.

.. warning::
   **Authorization behavior**:
   The charm sets ``default-ldap-bind-flow`` as both the ``authentication_flow`` and the ``authorization_flow`` on the LDAP Provider. Because no authorization policies are attached to this flow, any valid user identity that successfully verifies its password is treated as authorized to bind.

Dynamic bind accounts and directory RBAC
----------------------------------------

To prevent shared credential reuse across LDAP clients, the LDAP Outpost charm generates unique, per-relation bind accounts:

Account structure
~~~~~~~~~~~~~~~~~

When an LDAP client relates via the ``ldap`` interface:

1. The outpost provisions an active Authentik user named ``ldap-client-<app-slug>-<model-uuid-hash>-<relation-id>``.
2. The account is an ordinary Authentik user account (payload: ``{username, name, path: "users", is_active: True}``), not a special service account object.
3. The generated password is encrypted into a Juju Secret and granted to the consuming unit.

Search permissions and scoping
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Directory read access is governed by Authentik's Role-Based Access Control (RBAC):

* The charm creates a deployment-scoped RBAC role (``ldap-search-<identity>``) assigned the ``authentik_providers_ldap.search_full_directory`` permission.
* This permission is strictly scoped to the single LDAP Provider object; the charm verifies this assignment and explicitly rejects any matching global directory grants.
* **Shared read scope**: All bind users created by an outpost deployment share the same search role and can read the entire directory subtree defined under the application-wide ``base_dn``.
* **Adopt-only groups**: If a consuming charm requests specific group memberships during relation negotiation, the outpost attempts to attach them. If a requested group does not exist in Authentik, the request is logged and skipped without modifying the account's existing directory search grants.

Teardown and orphan cleanup
~~~~~~~~~~~~~~~~~~~~~~~~~~~

When a client relation is removed (``relation-broken``):

* The outpost leader reconciles active relations against known users and executes a ``DELETE /api/v3/core/users/<pk>/`` call against the Authentik API.
* Transient network or API errors trigger retries on subsequent reconcile cycles.
* The application, provider, and outpost objects remain intact to serve remaining clients; only the per-relation user account is deleted.

Network architecture and TLS termination
----------------------------------------

The Charmed Authentik workloads operate in cleartext within the Kubernetes pod network and rely on Traefik for boundary TLS termination:

.. list-table::
   :header-rows: 1
   :widths: 20 25 25 30

   * - Path
     - External Protocol & Port
     - Internal Protocol & Port
     - TLS Termination Point
   * - Web UI / OIDC API
     - HTTPS (Port 443)
     - HTTP (Port 9000)
     - Traefik Ingress Controller
   * - LDAPS Directory
     - LDAPS (Port 636)
     - LDAP (Port 3389)
     - Traefik Ingress Controller
   * - Cleartext LDAP
     - LDAP (Port 389)
     - LDAP (Port 3389)
     - Cleartext TCP forward via Traefik

Trust propagation
~~~~~~~~~~~~~~~~~

* **Inbound traffic**: Consuming applications (OIDC clients, SSSD, browsers) must trust the Root/Intermediate CA that issued Traefik's TLS certificate.
* **Outbound upstream trust**: The ``authentik-server`` charm provides a ``receive-ca-cert`` endpoint implementing the ``certificate_transfer`` interface. Integrating this endpoint allows Authentik to trust self-signed or enterprise CAs when connecting to upstream identity providers (e.g. corporate Active Directory or Keycloak instances).

Compliance and workload hardening
---------------------------------

Charmed Authentik enforces several security hardening baselines:

* **FIPS 140 Cryptography**: Both ``authentik-server`` and ``authentik-ldap-outpost`` run with the ``GOFIPS=1`` environment variable enabled, enforcing standard cryptographic primitives.
* **Telemetry and update checks disabled**: Error reporting (Sentry) and automated update checks are explicitly disabled (``AUTHENTIK_ERROR_REPORTING__ENABLED=false``, ``AUTHENTIK_DISABLE_UPDATE_CHECK=true``), ensuring no outbound telemetry escapes air-gapped environments.
* **Non-root container execution**: Workloads run as a fixed non-root user and group (``UID 584792``, ``GID 584792``).

Known limitations and accepted risks
------------------------------------

Administrators should consider the following known architectural limitations and accepted risks:

1. **Bootstrap Token Propagation (Accepted Risk S-1)**:
   The ``authentik-server`` charm shares its administrative ``akadmin`` bootstrap token with related LDAP Outpost applications over the ``authentik-server-info`` relation. The outpost requires this privilege to self-register its provider and RBAC configuration upon first deployment.
2. **Action Credential Exposure (Accepted Risk L-4)**:
   The ``get-bootstrap-admin-credentials`` Juju action returns the ``akadmin`` bootstrap token in plaintext in the action output. Juju action results are stored in the Juju controller state and are readable by users with model administration access.
3. **In-Cluster TLS Bypass**:
   The LDAP Outpost disables TLS verification against the core server (``AUTHENTIK_INSECURE = "true"``) because communication flows over in-cluster pod IPs.
4. **Database Wire Encryption**:
   Server-to-PostgreSQL communication passes unencrypted inside the cluster network.
5. **PROXY Protocol Trust**:
   Traefik entrypoints for Authentik configure ``proxyProtocol.insecure: true`` and trust private RFC 1918 CIDR blocks (``10.0.0.0/8``, ``172.16.0.0/12``, ``192.168.0.0/16``). Any pod or workload on the internal network sending spoofed PROXY headers could misrepresent its source IP address.
6. **Container Process Credential Access**:
   Workload credentials and encryption keys are injected into container processes via Pebble environment layers, making them visible to any process running inside the application container.
7. **Web UI Login Capability for Bind Accounts**:
   Because LDAP bind accounts are provisioned as standard Authentik users, credentials could theoretically be used to authenticate to the Authentik web portal unless explicit access restrictions are configured upstream.
