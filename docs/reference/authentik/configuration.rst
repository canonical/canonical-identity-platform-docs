.. meta::
    :description: Configuration options, Juju actions, and resource sizing reference for Charmed Authentik.

.. _configuration-authentik:

Configuration reference
=======================

This document provides the technical reference for configuration options, Juju actions, and resource sizing requirements across the Charmed Authentik operator suite.

Juju configuration options
--------------------------

Authentik Server (``authentik-server``)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. list-table::
    :header-rows: 1
    :widths: 25 15 15 20 25

    * - Option
      - Type
      - Default
      - Environment Variable
      - Description
    * - ``log_level``
      - string
      - ``info``
      - ``AUTHENTIK_LOG_LEVEL``
      - Log level. Values: ``debug``, ``info``, ``warning``, ``error``, ``trace``. (Note: ``trace`` logs may include session cookies).
    * - ``http_proxy``
      - string
      - empty
      - ``HTTP_PROXY``
      - HTTP proxy URL for outbound requests.
    * - ``https_proxy``
      - string
      - empty
      - ``HTTPS_PROXY``
      - HTTPS proxy URL for outbound requests.
    * - ``no_proxy``
      - string
      - empty
      - ``NO_PROXY``
      - Comma-separated list of domains to bypass proxying.
    * - ``cpu``
      - string
      - unset
      - Kubernetes limit
      - Kubernetes CPU limit (e.g. ``500m`` or ``1``). Cannot be lower than the 500m request floor.
    * - ``memory``
      - string
      - unset
      - Kubernetes limit
      - Kubernetes memory limit (e.g. ``1Gi``). Cannot be lower than the 1Gi request floor.
    * - ``web_workers``
      - int
      - ``2``
      - ``AUTHENTIK_WEB__WORKERS``
      - Number of gunicorn worker processes for the web server.
    * - ``postgresql_disable_server_side_cursors``
      - boolean
      - ``false``
      - ``AUTHENTIK_POSTGRESQL__DISABLE_SERVER_SIDE_CURSORS``
      - Set to ``true`` when using connection poolers (PgBouncer) to prevent cursor breakage.
    * - ``postgresql_conn_health_checks``
      - boolean
      - ``false``
      - ``AUTHENTIK_POSTGRESQL__CONN_HEALTH_CHECKS``
      - Proactively tests and discards stale pooled database connections.
    * - ``postgresql_conn_max_age``
      - int
      - ``0``
      - ``AUTHENTIK_POSTGRESQL__CONN_MAX_AGE``
      - Maximum database connection lifetime in seconds. ``0`` forces immediate recycling.
    * - ``postgresql_use_pgbouncer``
      - boolean
      - ``false``
      - None (shared via relation)
      - Declares that the database endpoint is a PgBouncer in transaction pooling mode.

Authentik Worker (``authentik-worker``)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. list-table::
    :header-rows: 1
    :widths: 25 15 15 20 25

    * - Option
      - Type
      - Default
      - Environment Variable
      - Description
    * - ``log_level``
      - string
      - ``info``
      - ``AUTHENTIK_LOG_LEVEL``
      - Log level. Values: ``info``, ``debug``, ``warning``, ``error``, ``critical``.
    * - ``http_proxy``
      - string
      - empty
      - ``HTTP_PROXY``
      - HTTP proxy URL (injected when non-empty).
    * - ``https_proxy``
      - string
      - empty
      - ``HTTPS_PROXY``
      - HTTPS proxy URL (injected when non-empty).
    * - ``no_proxy``
      - string
      - empty
      - ``NO_PROXY``
      - Proxy bypass domain list.
    * - ``cpu``
      - string
      - unset
      - Kubernetes limit
      - Kubernetes CPU limit. Cannot be lower than the 100m request floor.
    * - ``memory``
      - string
      - unset
      - Kubernetes limit
      - Kubernetes memory limit. Cannot be lower than the 200Mi request floor.
    * - ``worker_processes``
      - int
      - ``1``
      - ``AUTHENTIK_WORKER__PROCESSES``
      - Number of Dramatiq worker processes started in the container.
    * - ``worker_threads``
      - int
      - ``2``
      - ``AUTHENTIK_WORKER__THREADS``
      - Number of execution threads running per worker process.
    * - ``task_max_retries``
      - int
      - ``5``
      - ``AUTHENTIK_WORKER__TASK_MAX_RETRIES``
      - Maximum retry attempts before a failing task is abandoned.
    * - ``task_default_time_limit``
      - int
      - ``600``
      - ``AUTHENTIK_WORKER__TASK_DEFAULT_TIME_LIMIT``
      - Default execution timeout for background tasks in seconds.
    * - ``task_expiration_days``
      - int
      - ``30``
      - ``AUTHENTIK_WORKER__TASK_EXPIRATION``
      - Retention period in days for completed task metadata.
    * - ``consumer_listen_timeout``
      - int
      - ``30``
      - ``AUTHENTIK_WORKER__CONSUMER_LISTEN_TIMEOUT``
      - Worker listener polling timeout in seconds. Set to 5-10s in pooled environments.
    * - ``postgresql_disable_server_side_cursors``
      - boolean
      - ``false``
      - ``AUTHENTIK_POSTGRESQL__DISABLE_SERVER_SIDE_CURSORS``
      - Set to ``true`` when using connection poolers.
    * - ``postgresql_conn_health_checks``
      - boolean
      - ``false``
      - ``AUTHENTIK_POSTGRESQL__CONN_HEALTH_CHECKS``
      - Proactively tests pooled connections.
    * - ``postgresql_conn_max_age``
      - int
      - ``0``
      - ``AUTHENTIK_POSTGRESQL__CONN_MAX_AGE``
      - Connection recycling age in seconds.

Authentik LDAP Outpost (``authentik-ldap-outpost``)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. list-table::
    :header-rows: 1
    :widths: 25 15 15 20 25

    * - Option
      - Type
      - Default
      - Applied To
      - Description
    * - ``log_level``
      - string
      - ``info``
      - Container env var
      - Log level. Values: ``info``, ``debug``, ``warning``, ``error``, ``critical``.
    * - ``http_proxy``
      - string
      - empty
      - Container env var
      - HTTP proxy URL.
    * - ``https_proxy``
      - string
      - empty
      - Container env var
      - HTTPS proxy URL.
    * - ``no_proxy``
      - string
      - empty
      - Container env var
      - Proxy bypass list.
    * - ``cpu``
      - string
      - unset
      - Kubernetes limit
      - Kubernetes CPU limit. Cannot be lower than the 100m request floor.
    * - ``memory``
      - string
      - unset
      - Kubernetes limit
      - Kubernetes memory limit. Cannot be lower than the 200Mi request floor.
    * - ``base_dn``
      - string
      - ``dc=ldap,dc=goauthentik,dc=io``
      - Authentik LDAP Provider
      - Base DN for directory queries. Configured directly on the Authentik provider object.
    * - ``search_mode``
      - string
      - ``cached``
      - Authentik LDAP Provider
      - LDAP search query mode: ``cached`` or ``direct``.
    * - ``bind_mode``
      - string
      - ``cached``
      - Authentik LDAP Provider
      - LDAP authentication bind mode: ``cached`` or ``direct``.
    * - ``mfa_support``
      - boolean
      - ``false``
      - Authentik LDAP Provider
      - Enables password-appending MFA support on the provider.
    * - ``ingress_domain``
      - string
      - empty
      - Traefik Router
      - Domain name for TLS SNI multiplexing on Port 636.
    * - ``expose_ldap_ingress``
      - boolean
      - ``false``
      - Traefik Entrypoint
      - Exposes cleartext LDAP on Port 389 via Traefik.

Juju actions
------------

Actions are available on the ``authentik-server`` charm only:

``get-bootstrap-admin-credentials``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Retrieves initial administrator credentials stored in Juju secrets.

* **Parameters**: None
* **Outputs**:
  - ``username``: Always ``akadmin``
  - ``password``: Initial generated bootstrap password
  - ``bootstrap-token``: Initial cluster-wide API token
  - ``warning``: Stale credential warnings

``create-recovery-link``
~~~~~~~~~~~~~~~~~~~~~~~~

Generates a temporary, single-use administrative recovery link bypassing standard login stages.

* **Parameters**:
  - ``username`` (string, default: ``akadmin``): Account to recover.
  - ``duration`` (integer, default: ``10``): Link validity in minutes.
* **Outputs**:
  - ``url``: Full recovery URL
  - ``path``: Relative recovery flow path
  - ``status``: Link generation status

Resource requirements and sizing
--------------------------------

Resource requests represent fixed minimum floors enforced by the charms. Setting limits lower than the request floor is rejected or adjusted by the operator framework.

.. list-table::
    :header-rows: 1
    :widths: 30 25 25 20

    * - Component
      - Hardcoded Request Floor
      - Default Limit
      - Scaling Guidance
    * - **``authentik-server``**
      - 500m CPU / 1 GiB RAM
      - Unset (no limit)
      - 1–3 units depending on web/API traffic
    * - **``authentik-worker``**
      - 100m CPU / 200 MiB RAM
      - Unset (no limit)
      - 1–2 units depending on background task volume
    * - **``authentik-ldap-outpost``**
      - 100m CPU / 200 MiB RAM
      - Unset (no limit)
      - CPU scales with LDAP query and bind volume

.. note::
    Under LDAP-dominated traffic loads, the LDAP Outpost requires substantial CPU headroom. Load testing indicates outpost CPU consumption can peak above 1000m during intensive search and bind operations, so setting restrictive CPU limits on the outpost container is not recommended.
