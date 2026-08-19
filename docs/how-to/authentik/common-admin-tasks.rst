.. meta::
    :description: Routine operations, horizontal scaling, worker concurrency tuning, and database connection pooling for Charmed Authentik.

.. _common-admin-tasks-authentik:

Common operational tasks
========================

This document describes how to execute routine maintenance, performance tuning, and scaling operations on your Charmed Authentik deployment.

Horizontal scaling
------------------

To handle increases in authentication requests, background task processing, or directory service traffic, you can scale each charm independently.

Scale out the Web and API tier (Server)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    juju scale-application authentik-server 3

Vertical process scaling (Web workers)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If horizontal pod scaling is constrained by Kubernetes cluster quotas, you can increase vertical request-handling capacity by tuning the number of gunicorn web worker processes per server unit:

.. code-block:: bash

    # Increase web worker processes per unit (Default: 2)
    juju config authentik-server web_workers=4

Scale out the background processing tier (Worker)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    juju scale-application authentik-worker 2

Scale out the directory gateway tier (LDAP Outpost)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    juju scale-application authentik-ldap-outpost 2

Worker concurrency and thread tuning
------------------------------------

You can tune the runtime processing limits of background task worker processes via Juju configuration settings.

Performance formula
~~~~~~~~~~~~~~~~~~~

The background task concurrency capacity of your deployment is governed by:

.. math::

    \text{Total Task Concurrency} = \text{Worker Units} \times \text{worker\_processes} \times \text{worker\_threads}

* **``worker_processes``** (Default: ``1``): Number of Dramatiq worker processes started in the container.
* **``worker_threads``** (Default: ``2``): Number of Dramatiq execution threads running per process.

Configuration commands
~~~~~~~~~~~~~~~~~~~~~~

To tune background synchronization throughput:

.. code-block:: bash

    # Set 2 concurrent worker processes per unit
    juju config authentik-worker worker_processes=2

    # Set 8 execution threads per worker process
    juju config authentik-worker worker_threads=8

Background task lifecycle and housekeeping
------------------------------------------

To prevent background tasks (such as large upstream directory syncs) from exhausting threads or causing database storage bloat, tune execution constraints and audit metadata lifetimes:

* **Task Retries**: Limit execution retries for failing tasks:

  .. code-block:: bash

      # Abandon failing tasks after 3 retries (Default: 5)
      juju config authentik-worker task_max_retries=3

* **Time Limits**: Limit how long any background task can run before it is aborted:

  .. code-block:: bash

      # Limit tasks to 5 minutes (Default: 600 seconds)
      juju config authentik-worker task_default_time_limit=300

* **Database Housekeeping**: Keep database storage optimized by reducing retention of completed task metadata:

  .. code-block:: bash

      # Delete completed task execution metadata after 14 days (Default: 30)
      juju config authentik-worker task_expiration_days=14

Database connection tuning (Connection pooling)
-----------------------------------------------

When using an external database connection pooler (such as **PgBouncer** or **Pgpool-II**) in **transaction pooling** mode, configure socket lifecycles and disable server-side cursors on **both** the server and worker charms:

.. code-block:: bash

    # Prevent server-side cursors from breaking across multiplexed connections
    juju config authentik-server postgresql_disable_server_side_cursors=true
    juju config authentik-worker postgresql_disable_server_side_cursors=true

    # Enable connection health checks on every request
    juju config authentik-server postgresql_conn_health_checks=true
    juju config authentik-worker postgresql_conn_health_checks=true

    # Recycle connections immediately (default is 0)
    juju config authentik-server postgresql_conn_max_age=0
    juju config authentik-worker postgresql_conn_max_age=0

    # Reduce worker task polling timeout fallback (replaces PostgreSQL LISTEN/NOTIFY)
    juju config authentik-worker consumer_listen_timeout=5

.. note::
    For full configuration option descriptions, see the :doc:`Configuration reference </reference/authentik/configuration>`.

Observability integration
-------------------------

For forwarding logs to Loki, metrics to Prometheus, dashboards to Grafana, and traces to Tempo, see :doc:`Integrate with the Canonical Observability Stack (COS) <integrate-with-cos>`.
