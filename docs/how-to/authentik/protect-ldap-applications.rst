.. meta::
    :description: How to deploy the Charmed Authentik LDAP Outpost and integrate downstream LDAP clients with secure LDAPS.

.. _protect-ldap-applications-authentik:

Protect LDAP applications
=========================

This guide walks you through deploying the **Charmed Authentik LDAP Outpost** and integrating downstream LDAP clients (such as **SSSD** or legacy directory applications) using secure LDAPS (Port 636) and Traefik.

Prerequisites
-------------

This guide assumes you have an active Charmed Authentik deployment matching the topology established in the :doc:`Getting started tutorial </tutorial/authentik/getting-started>`. Specifically, you should have:

* An active ``authentik-server`` deployment integrated with its database and certificates.
* An active Traefik Ingress controller (``traefik-public``) deployed (typically in a shared administrative model, e.g., ``core``).
* Administrative access (``akadmin``) to the Authentik dashboard.

Step 1 (Optional): Deploy the LDAP Outpost and integrate with Server
--------------------------------------------------------------------

.. note::
    This step is **optional** if you used the tutorial Terraform blueprint, which pre-installs and integrates the LDAP Outpost automatically.

The LDAP Outpost acts as a directory gateway. It does not connect directly to PostgreSQL; instead, it communicates with the Authentik Server API over HTTP.

1. **Deploy the Outpost charm**:

   .. code-block:: bash

       juju deploy authentik-ldap-outpost --channel latest/stable --trust

2. **Integrate the Outpost with the core Server**:

   .. code-block:: bash

       juju integrate authentik-ldap-outpost:authentik-server-info authentik-server:authentik-server-info

Step 2: Integrate a downstream LDAP client
------------------------------------------

To connect an LDAP-compliant consumer charm (such as ``sssd``), establish an integration with the Outpost:

.. code-block:: bash

    juju integrate sssd:ldap-client authentik-ldap-outpost:ldap

Managing TLS certificate trust
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Because the Outpost operates over secure LDAPS (Port 636) in production with TLS terminated at Traefik, downstream clients must trust the Certificate Authority (CA) chain that issued Traefik's certificate.

To distribute the CA chain, integrate your consumer client with your certificate authority provider using the ``certificate_transfer`` interface:

.. code-block:: bash

    # Transfer the CA chain to SSSD so it can verify the LDAPS session
    juju integrate sssd:receive-ca-cert self-signed-certificates:send-ca-cert

Behind the scenes: Dynamic bind accounts
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This relation triggers the following automated operations:

1. **Dynamic Account Generation**: The outpost operator calls the Authentik REST API to provision a unique user account (``ldap-client-<app-slug>-<model-uuid-hash>-<relation-id>``) with a randomly generated password.
2. **Access Revocation**: When the relation is removed (``relation-broken``), the user account is automatically deleted from the Authentik database.
3. **Dedicated Bind Flow**: The operator provisions a non-interactive **LDAP Bind Flow** (``default-ldap-bind-flow``) bypassing interactive MFA prompts for programmatic binds.

Step 3: End-to-end query verification
-------------------------------------

To verify directory access using standard command-line tools like ``ldapsearch``:

1. **Retrieve the Dynamic Credentials**:
   Inspect the Juju unit state of your consumer client unit (e.g. ``sssd/0``):

   .. code-block:: bash

       juju show-unit sssd/0 --endpoint ldap-client

   The relation databag contains the connection parameters:

   * ``bind_dn``: The unique bind DN (e.g., ``cn=ldap-client-sssd-...,ou=users,dc=ldap,dc=goauthentik,dc=io``).
   * ``bind_password_secret``: A Juju Secret URI containing the dynamic bind password. The plaintext password is never written directly to the databag and is accessible only to the granted consumer application.
   * ``ldaps_urls``: The secure LDAPS URL (e.g., ``ldaps://outpost.identity.example.com:636``).
   * ``ldaps_enabled``: ``true`` or ``false``, indicating whether Traefik terminates LDAPS.

   .. note::
       Consumer charms should gate on ``ldaps_enabled`` being ``true`` rather than simply checking if ``ldaps_urls`` is non-empty.

2. **Perform an ``ldapsearch`` query**:
   Run the query against the Traefik Ingress VIP, supplying the ``bind_dn`` and the bind password:

   .. code-block:: bash

       ldapsearch -x -H ldaps://outpost.identity.example.com:636 \
         -D "cn=ldap-client-sssd-...,ou=users,dc=ldap,dc=goauthentik,dc=io" \
         -w "<bind-password>" \
         -b "dc=ldap,dc=goauthentik,dc=io" \
         "(objectClass=*)"

   A successful directory setup will return a list of mapped directory objects.

Step 4: Multi-outpost SNI routing and Proxy Protocol
----------------------------------------------------

Configuring Ingress SNI multiplexing
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you deploy multiple independent outposts sharing a single Traefik Ingress controller, you must configure a distinct ``ingress_domain`` for each outpost to avoid routing conflicts on Port 636:

1. **Set unique ingress subdomains**:

   .. code-block:: bash

       juju config outpost-primary ingress_domain="outpost-primary.identity.example.com"
       juju config outpost-secondary ingress_domain="outpost-secondary.identity.example.com"

2. **Integrate both outposts with Traefik**:

   .. code-block:: bash

       # If Traefik is in the same model:
       juju integrate outpost-primary:traefik-route traefik-public:traefik-route
       juju integrate outpost-secondary:traefik-route traefik-public:traefik-route

       # If Traefik is in a different model (e.g., 'core'):
       juju integrate outpost-primary:traefik-route core.traefik-public:traefik-route
       juju integrate outpost-secondary:traefik-route core.traefik-public:traefik-route

.. note::
    Traefik must already hold a valid TLS certificate covering each configured ``ingress_domain``. The ``ingress_domain`` configuration applies to the LDAPS router on Port 636; it has no effect on the cleartext Port 389 router, which uses ``HostSNI("*")``.

Client IP propagation (Proxy Protocol v2)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The ``traefik-route`` relation automatically configures Traefik to prepend Proxy Protocol v2 headers on TCP streams. The Outpost automatically trusts RFC 1918 private subnets to decode client source IPs for rate-limiting and audit logging.

Advanced outpost configurations
-------------------------------

The ``authentik-ldap-outpost`` charm provides configuration parameters to customize directory behavior:

1. ``base_dn`` (Directory Schema Root)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To customize the base Distinguished Name (DN) used for LDAP lookups:

.. code-block:: bash

    juju config authentik-ldap-outpost base_dn="dc=enterprise,dc=local"

2. ``search_mode`` and ``bind_mode`` (Caching and Consistency)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The outpost can serve queries and binds from a local cache or query the core API in real time:

* **``cached``** (Default): Caches read lookups and authentication successes locally, reducing background network requests and lowering server CPU load.
* **``direct``**: Executes real-time REST API requests to the core server for every query and bind.

.. warning::
    Under the default ``cached`` mode, search results can be stale, and password changes or session revocations only take effect after cached bind state refreshes. For immediate consistency, configure both options to ``direct``:

    .. code-block:: bash

        juju config authentik-ldap-outpost search_mode="direct"
        juju config authentik-ldap-outpost bind_mode="direct"

3. ``mfa_support`` (Multi-Factor Authentication)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The charm allows enabling password-appending MFA support on the Authentik LDAP provider:

.. code-block:: bash

    juju config authentik-ldap-outpost mfa_support=true

When enabled on the provider, clients can authenticate by appending their 6-digit TOTP token directly to their passwords (e.g., ``SecretPassword123456``).
