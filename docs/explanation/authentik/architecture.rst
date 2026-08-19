.. meta::
    :description: Conceptual architecture, component topology, communication boundaries, and security design for Charmed Authentik.

.. _architecture-authentik:

Charmed Authentik architecture and security design
==================================================

This document provides a conceptual overview of the system architecture, relationship topology, communication boundaries, and security designs governing Charmed Authentik.

Charmed Authentik is an open-source Identity Provider (IdP) delivering unified authentication, user management, and authorization. It is modeled in Juju as a cooperative suite of three independent Kubernetes operator charms:

1. **``authentik-server-operator``**: The core API server, admin dashboard, and web interface.
2. **``authentik-worker-operator``**: The asynchronous Dramatiq task worker handling directory synchronizations, email dispatch, outpost management, and background maintenance.
3. **``authentik-ldap-outpost-operator``**: A directory gateway providing LDAP and LDAPS interfaces for downstream consumers.

Component relationship topology
-------------------------------

The Charmed Authentik suite integrates with PostgreSQL database services, Traefik Ingress controllers, and the Canonical Observability Stack (COS).

.. image:: https://raw.githubusercontent.com/canonical/canonical-identity-platform-docs/main/Diagram_sources/authentik-relation-topology.png
   :alt: Charmed Authentik Relation Topology
   :align: center

Core relations overview
~~~~~~~~~~~~~~~~~~~~~~~

* **``pg-database`` (PostgreSQL client)**: The ``authentik-server`` charm connects to PostgreSQL to run migrations and manage database state. The server acts as the central authority for the database configuration.
* **``authentik-cluster`` (Server → Worker)**: The server shares the cluster secret key (``AUTHENTIK_SECRET_KEY``) and database connection credentials with the ``authentik-worker`` over the ``authentik-cluster`` relation. The worker has no direct ``pg-database`` relation; all database parameters and credentials are exchanged via Juju Secrets over this interface.
* **``authentik-server-info`` (Server → Outpost)**: Exposes the core server's API URL and an administrative API token to the LDAP Outpost. The token published in the granted Juju secret is the ``akadmin`` bootstrap token, which the outpost uses to provision its provider, application, and outpost objects during initialization. The bootstrap password stays server-local and is never shared over the relation.
* **``traefik-route`` (Ingress integration)**: Enables the server and outpost to declare custom ingress endpoints, route HTTP and TCP traffic, manage TLS termination, and configure Proxy Protocol header parsing.

Security boundaries and trust systems
-------------------------------------

Deploying Charmed Authentik in enterprise environments requires structured security isolation, non-interactive authentication logic, and high-fidelity traffic propagation.

Dynamic relation-driven LDAP accounts
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Sharing a single administrative credential across multiple downstream applications represents a critical security risk. To enforce the principle of least privilege, the Charmed Authentik LDAP Outpost implements isolated accounts:

1. **Unique Account Generation**: On a ``relation-joined`` event with a consuming client (such as SSSD), the outpost charm leader calls the Authentik REST API to provision a unique user account named ``ldap-client-<app-name-slug>-<model-uuid-hash>-<relation-id>`` with a strong, randomly generated password.
2. **Access Revocation**: When the relation is severed (``relation-broken``), the corresponding user account is deleted from the Authentik database, preventing credential leakage.
3. **Resource Efficiency**: A single Outpost application deployment in the Juju model serves all integrated downstream LDAP clients concurrently via their respective secure accounts.

.. note::
    LDAP bind accounts are created as standard active Authentik users with read capability granted via an object-scoped RBAC role (``search_full_directory`` scoped strictly to the LDAP provider).

Automated non-interactive bind flow resolution
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Standard LDAP bind operations are non-interactive. The default Authentik authentication flow includes interactive multi-factor authentication (MFA) stages, which would ordinarily block command-line or machine-level LDAP binds.

* To address this, the charm automatically provisions a dedicated, non-interactive **LDAP Bind Flow** (``default-ldap-bind-flow``) containing only the ``identification``, ``password``, and ``login`` stages.
* This flow is configured as the ``authentication_flow`` and ``authorization_flow`` of the LDAP Provider, allowing machine-level clients to authenticate without interactive MFA challenges.

Directory access and TLS termination (Port 636 and 389)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* **Implicit LDAPS (Port 636)**: Charmed Authentik standardizes on implicit LDAPS for directory access. TLS is terminated at the Traefik Ingress level (**Zero-Certificate Outpost** design). The ``authentik-ldap-outpost`` workload container runs a cleartext LDAP listener on port 3389 and does not manage local certificates or trust stores.
* **Plain LDAP (Port 389)**: Cleartext LDAP on port 389 is disabled by default and can be optionally exposed via Traefik (using ``expose_ldap_ingress=true``). Opportunistic StartTLS on port 389 is not supported; encryption must be handled via implicit LDAPS on port 636.

Proxy Protocol v2 and client IP propagation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For auditing, rate limiting, and brute-force lockout protection, Authentik must inspect the original client source IP rather than the internal ephemeral Traefik Pod IP.

* **Layer 4 Limitations**: Because LDAP/LDAPS traffic operates over raw TCP (Layer 4), Traefik cannot inject standard Layer 7 HTTP headers (such as ``X-Forwarded-For``). Traefik uses **Proxy Protocol v2** to propagate client source IPs over TCP connections.
* **Trusted CIDR Ranges**: The outpost pre-seeds ``AUTHENTIK_LISTEN__TRUSTED_PROXY_CIDRS`` with standard private address ranges (``127.0.0.1/32``, ``10.0.0.0/8``, ``172.16.0.0/12``, ``192.168.0.0/16``) and dynamically discovered subnet ranges from relation data to parse incoming Proxy Protocol headers reliably.

.. warning::
    Traefik entrypoints for LDAP and LDAPS are configured with ``proxyProtocol.insecure: true``. Consequently, any client within the trusted private network ranges can supply Proxy Protocol headers, which affects the source IP attribution for auditing.

Secret exchange and bootstrap flow
----------------------------------

Sensitive materials (encryption keys, database passwords, and bootstrap tokens) are managed via Juju Secrets rather than plain-text databag entries.

.. image:: https://raw.githubusercontent.com/canonical/canonical-identity-platform-docs/main/Diagram_sources/authentik-secret-bootstrap-flow.png
   :alt: Charmed Authentik Secret Bootstrap Flow
   :align: center

1. **Cluster Key and Database Password**: The ``authentik-server`` charm creates an app-owned Juju secret containing ``secret-key`` and ``db-password``, grants access to the ``authentik-worker`` charm, and passes the secret ID in the ``authentik-cluster`` databag.
2. **API Token**: The server creates an app-owned Juju secret containing ``api-token`` (the bootstrap admin token), grants access to the ``authentik-ldap-outpost`` charm, and passes the secret ID in the ``authentik-server-info`` databag.
3. **Outpost Token Provisioning**: The outpost uses the API token to register its provider, application, and outpost entities in Authentik, retrieves its own dedicated outpost token from the Authentik API, and uses that outpost token to run its service.
