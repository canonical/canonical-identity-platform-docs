.. meta::
    :description: Technical reference for network ports, ingress routing, charm endpoints, and relation databag contracts in Charmed Authentik.

.. _integrations-authentik:

Integrations and contracts
==========================

This document provides the technical reference for network port allocations, charm endpoint definitions, and Juju relation databag contracts across the Charmed Authentik suite.

Network ports and entrypoints
-----------------------------

.. list-table::
    :header-rows: 1
    :widths: 20 10 15 20 35

    * - Component
      - Port
      - Protocol
      - Relation / Scope
      - Description
    * - **``authentik-server``**
      - ``9000``
      - TCP / HTTP
      - ``traefik-route``
      - Primary administrative web UI, user portal, and REST API.
    * -
      - ``9443``
      - TCP / HTTPS
      - Opened on unit
      - Opened container port; unused by the charm's default HTTP traffic routing.
    * -
      - ``9300``
      - TCP / HTTP
      - ``metrics-endpoint``
      - Prometheus metrics listener (scraped pod-to-pod, not opened on unit).
    * - **``authentik-worker``**
      - ``9300``
      - TCP / HTTP
      - ``metrics-endpoint``
      - Prometheus metrics listener (scraped pod-to-pod, not opened on unit).
    * - **``authentik-ldap-outpost``**
      - ``3389``
      - TCP / LDAP
      - Opened on unit
      - Cleartext LDAP listener running in the workload container. Traefik backend target.
    * -
      - ``636``
      - TCP / LDAPS
      - ``traefik-route``
      - External LDAPS entrypoint terminated at Traefik with TLS.
    * -
      - ``389``
      - TCP / LDAP
      - ``traefik-route``
      - Optional cleartext LDAP entrypoint (exposed when ``expose_ldap_ingress=true``).
    * -
      - ``9300``
      - TCP / HTTP
      - ``metrics-endpoint``
      - Prometheus metrics listener (scraped pod-to-pod, not opened on unit).

SNI routing and ingress behavior
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* **LDAPS (Port 636)**: Traefik routes incoming TLS traffic to the outpost backend on port 3389. When ``ingress_domain`` is configured, Traefik matches via ``HostSNI("<domain>")``. If unset, it defaults to ``HostSNI("*")``, which causes collisions if multiple outposts share the same Traefik instance.
* **Plain LDAP (Port 389)**: When enabled, Traefik routes cleartext TCP using ``HostSNI("*")``. The ``ingress_domain`` setting does not affect Port 389.
* **StartTLS**: Opportunistic StartTLS on Port 389 is not supported (``starttls=False`` is hardcoded). All encrypted directory access must use implicit LDAPS on Port 636.

Charm endpoint inventory
------------------------

Authentik Server (``authentik-server``)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. list-table::
    :header-rows: 1
    :widths: 20 15 25 15 25

    * - Endpoint
      - Direction
      - Interface
      - Limit / Optional
      - Description
    * - ``pg-database``
      - Requires
      - ``postgresql_client``
      - Limit: 1, Mandatory
      - PostgreSQL database relation.
    * - ``traefik-route``
      - Requires
      - ``traefik_route``
      - Limit: 1, Mandatory
      - Ingress integration for web UI and API routing.
    * - ``logging``
      - Requires
      - ``loki_push_api``
      - Optional
      - Log forwarding to Loki.
    * - ``tracing``
      - Requires
      - ``tracing``
      - Limit: 1, Optional
      - OTLP trace forwarding to Tempo.
    * - ``receive-ca-cert``
      - Requires
      - ``certificate_transfer``
      - Optional
      - Outbound CA trust certificate transfer.
    * - ``smtp``
      - Requires
      - ``smtp``
      - Limit: 1, Optional
      - Outbound email notification integration.
    * - ``authentik-cluster``
      - Provides
      - ``authentik_cluster``
      - Mandatory
      - Cluster key and database credentials for worker units.
    * - ``authentik-server-info``
      - Provides
      - ``authentik_server_info``
      - Optional
      - Server URL and API automation token for outposts.
    * - ``oauth``
      - Provides
      - ``oauth``
      - Optional
      - Dynamic OIDC / OAuth2 integration for consumer apps.
    * - ``metrics-endpoint``
      - Provides
      - ``prometheus_scrape``
      - Optional
      - Prometheus application metrics scraping.
    * - ``grafana-dashboard``
      - Provides
      - ``grafana_dashboard``
      - Optional
      - Built-in Grafana dashboard provisioning.
    * - ``authentik-peers``
      - Peer
      - ``authentik_peers``
      - N/A
      - Unit clustering and state coordination.

Authentik Worker (``authentik-worker``)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. list-table::
    :header-rows: 1
    :widths: 20 15 25 15 25

    * - Endpoint
      - Direction
      - Interface
      - Limit / Optional
      - Description
    * - ``authentik-cluster``
      - Requires
      - ``authentik_cluster``
      - Optional
      - Receives cluster secret key and DB config from server.
    * - ``logging``
      - Requires
      - ``loki_push_api``
      - Optional
      - Log forwarding to Loki.
    * - ``tracing``
      - Requires
      - ``tracing``
      - Limit: 1, Optional
      - OTLP trace forwarding to Tempo.
    * - ``metrics-endpoint``
      - Provides
      - ``prometheus_scrape``
      - Optional
      - Prometheus metrics scraping.
    * - ``grafana-dashboard``
      - Provides
      - ``grafana_dashboard``
      - Optional
      - Built-in Grafana dashboard provisioning.

Authentik LDAP Outpost (``authentik-ldap-outpost``)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. list-table::
    :header-rows: 1
    :widths: 20 15 25 15 25

    * - Endpoint
      - Direction
      - Interface
      - Limit / Optional
      - Description
    * - ``authentik-server-info``
      - Requires
      - ``authentik_server_info``
      - Optional
      - Receives server API URL and automation token.
    * - ``traefik-route``
      - Requires
      - ``traefik_route``
      - Limit: 1, Optional
      - Ingress routing for LDAPS and LDAP entrypoints.
    * - ``logging``
      - Requires
      - ``loki_push_api``
      - Optional
      - Log forwarding to Loki.
    * - ``tracing``
      - Requires
      - ``tracing``
      - Limit: 1, Optional
      - OTLP trace forwarding to Tempo.
    * - ``ldap``
      - Provides
      - ``ldap``
      - Mandatory
      - LDAP directory service relation for consumer apps.
    * - ``metrics-endpoint``
      - Provides
      - ``prometheus_scrape``
      - Optional
      - Prometheus metrics scraping.
    * - ``grafana-dashboard``
      - Provides
      - ``grafana_dashboard``
      - Optional
      - Built-in Grafana dashboard provisioning.
    * - ``authentik-ldap-peers``
      - Peer
      - ``authentik_ldap_peers``
      - N/A
      - Outpost clustering and coordination.

Relation databag contracts
--------------------------

``authentik-cluster`` relation contract
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* **Interface**: ``authentik_cluster``
* **Provider**: ``authentik-server`` | **Requirer**: ``authentik-worker``
* **Application Databag Fields**:

  .. list-table::
      :header-rows: 1
      :widths: 30 70

      * - Field
        - Description
      * - ``secret_key_secret_id``
        - Juju Secret ID holding ``secret-key`` and ``db-password``.
      * - ``db_host``
        - Primary database host address.
      * - ``db_port``
        - Primary database port.
      * - ``db_user``
        - Database username.
      * - ``db_name``
        - The server requests the database ``authentik`` and forwards the name to workers unchanged.
      * - ``db_read_replicas``
        - Comma-separated ``host:port`` read replica addresses (empty if none).
      * - ``db_use_pgbouncer``
        - ``"true"`` when using PgBouncer in transaction pooling mode.
      * - ``server_version``
        - Authentik server workload version string.

``authentik-server-info`` relation contract
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* **Interface**: ``authentik_server_info``
* **Provider**: ``authentik-server`` | **Requirer**: ``authentik-ldap-outpost``
* **Application Databag Fields**:

  .. list-table::
      :header-rows: 1
      :widths: 30 70

      * - Field
        - Description
      * - ``authentik_host``
        - In-cluster HTTP API URL of the Authentik server (e.g. ``http://authentik-server:9000``).
      * - ``authentik_token_secret_id``
        - Juju Secret ID holding the administrative API token (key: ``api-token``).

``ldap`` relation contract
~~~~~~~~~~~~~~~~~~~~~~~~~~

* **Interface**: ``ldap``
* **Provider**: ``authentik-ldap-outpost`` | **Requirer**: Downstream LDAP clients (e.g., ``sssd``)
* **Application Databag Fields**:

  .. list-table::
      :header-rows: 1
      :widths: 30 70

      * - Field
        - Description
      * - ``urls``
        - JSON-formatted list with a single cleartext LDAP URL (e.g. ``["ldap://outpost.example.com:389"]`` or in-cluster ``["ldap://<app>.<model>.svc:3389"]``).
      * - ``ldaps_urls``
        - JSON-formatted list with the LDAPS URL (e.g. ``["ldaps://outpost.example.com:636"]``). Empty list when LDAPS is not active.
      * - ``base_dn``
        - Base Distinguished Name (default: ``dc=ldap,dc=goauthentik,dc=io``).
      * - ``bind_dn``
        - Unique dynamically generated user DN (e.g. ``cn=ldap-client-...,ou=users,dc=ldap,...``).
      * - ``bind_password_secret``
        - Juju Secret URI granting access to the secret containing the ``password`` key.
      * - ``starttls``
        - Always ``False``.
      * - ``auth_method``
        - Always ``"simple"``.
      * - ``ldaps_enabled``
        - ``"true"`` or ``"false"``, indicating whether LDAPS is terminated and active at Traefik.
