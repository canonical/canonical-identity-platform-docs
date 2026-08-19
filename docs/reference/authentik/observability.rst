.. _authentik-observability:

Observability in Charmed Authentik
==================================

.. meta::
   :description: Reference for alert rules, Grafana dashboards, tracing, and metric definitions in Charmed Authentik.

Charmed Authentik provides comprehensive integration with the `Canonical Observability Stack (COS) <https://charmhub.io/topics/canonical-observability-stack>`_ across all three operators (``authentik-server``, ``authentik-worker``, and ``authentik-ldap-outpost``).

This reference details the metrics, Prometheus alert rules, Loki log alert rules, Grafana dashboards, and OpenTelemetry tracing endpoints provided by the charms.

Integration endpoints
---------------------

All three Authentik operator charms provide identical integration endpoint names for observability:

.. list-table::
   :header-rows: 1
   :widths: 25 20 20 35

   * - Endpoint
     - Role
     - Interface
     - Description
   * - ``metrics-endpoint``
     - Provider
     - ``prometheus_scrape``
     - Exposes Prometheus scrape metrics on port 9300 at the default ``/metrics`` path via in-cluster pod-to-pod networking.
   * - ``grafana-dashboard``
     - Provider
     - ``grafana_dashboard``
     - Forwards charm-specific, built-in Grafana dashboards to Grafana.
   * - ``logging``
     - Requirer
     - ``loki_push_api``
     - Ships container and service logs to Loki.
   * - ``tracing``
     - Requirer
     - ``tracing``
     - Streams distributed tracing spans to Tempo using the OTLP/HTTP protocol (limit: 1 relation).

.. note::
   All four observability integrations are optional. Tracing environment variable injection differs across components:
   
   * ``authentik-server`` and ``authentik-worker`` inject ``OTEL_EXPORTER_OTLP_ENDPOINT``.
   * ``authentik-ldap-outpost`` injects ``AUTHENTIK_OUTPOST__DISCOVER__OTLP_TRACES_ENDPOINT``.

Prometheus alert rules
----------------------

The Charmed Authentik suite defines nine charm-specific Prometheus alert rules across six rule files:

.. list-table::
   :header-rows: 1
   :widths: 28 12 25 10 10 15

   * - Alert Name
     - Charm
     - Trigger Condition
     - Duration
     - Severity
     - Impact
   * - ``AuthentikServerUnavailable-multiple``
     - Server
     - Ratio of ``up`` units < 0.7
     - 5m
     - ``error``
     - More than 30% of Authentik server units are unreachable.
   * - ``AuthentikServerUnavailable-all``
     - Server
     - Sum of ``up`` units == 0
     - 5m
     - ``fatal``
     - All Authentik server units are down; total loss of authentication and API access.
   * - ``AuthentikServerHigh5xxRate``
     - Server
     - 5xx responses > 5% of total HTTP requests
     - 10m
     - ``error``
     - Web or API requests are failing due to internal server errors.
   * - ``AuthentikWorkerUnavailable-multiple``
     - Worker
     - Ratio of ``up`` units < 0.7
     - 5m
     - ``error``
     - More than 30% of Authentik worker units are unreachable.
   * - ``AuthentikWorkerUnavailable-all``
     - Worker
     - Sum of ``up`` units == 0
     - 5m
     - ``fatal``
     - All worker units are down; background tasks and directory syncs are halted.
   * - ``AuthentikWorkerTaskBacklog``
     - Worker
     - De-duplicated ``authentik_tasks_queued`` > 100
     - 15m
     - ``warning``
     - Tasks are accumulating in the queue and worker capacity may be saturated.
   * - ``AuthentikLdapOutpostUnavailable-multiple``
     - Outpost
     - Ratio of ``up`` units < 0.7
     - 5m
     - ``error``
     - More than 30% of LDAP Outpost units are unreachable.
   * - ``AuthentikLdapOutpostUnavailable-all``
     - Outpost
     - Sum of ``up`` units == 0
     - 5m
     - ``fatal``
     - All LDAP Outpost units are down; LDAP authentication is unavailable.
   * - ``AuthentikLdapOutpostDisconnected``
     - Outpost
     - ``authentik_outpost_connection == 0``
     - 5m
     - ``error``
     - The outpost lost its WebSocket connection to the server; policy evaluation and configuration sync will fail.

.. note::
   Each charm also includes the generic application alert group appended automatically by the ``prometheus_scrape`` charm library.

Loki log alert rules
--------------------

All three operator charms include an identical Loki log alert rule:

* **Rule Name**: ``HighFrequencyHighSeverityLog``
* **Data Source**: Loki
* **Query Expression**: Counts log entries with severity ``error``, ``fatal``, or ``critical`` within a 5-minute sliding window across JSON-structured logs (tolerating non-JSON lines via ``__error__ != "JSONParserErr"``).
* **Threshold & Duration**: Fires when the count exceeds 100 lines in 5 minutes (no ``for`` duration clause).
* **Severity**: ``error``
* **Description**: Alerts administrators when an application unit experiences an unusually high volume of high-severity error logs, indicating misconfiguration, unhandled exceptions, or crash loops.

Grafana dashboards
------------------

Each charm exports a dedicated, pre-configured Grafana dashboard tagged with ``authentik`` and ``identity platform``. The dashboards include template variables for Juju topology filtering (``juju_model``, ``juju_application``, ``juju_unit``) and data source selectors (``prometheusds``, ``lokids``):

1. **Charmed Authentik Server Operator Dashboard** (``authentik-server-dashboard``)
   
   * **Availability**: Fractional unit availability gauge and list of available units.
   * **Logging**: High-severity log entries time series (error, fatal, critical) grouped by level.
   * **HTTP**: Response rate by HTTP status code and 90th percentile request latency by Django view.
   * **Flows & Policies**: 90th percentile flow plan duration by flow slug, 90th percentile stage execution duration by stage type, and 90th percentile policy engine evaluation duration by object type.
   * **Platform**: Connected task workers (by version and match state), connected outposts, and queued background tasks.

2. **Charmed Authentik Worker Operator Dashboard** (``authentik-worker-dashboard``)
   
   * **Availability**: Fractional unit availability gauge and list of available units.
   * **Logging**: High-severity log entries time series (error, fatal, critical) grouped by level.
   * **Background Tasks**: Queued tasks by actor, task completion throughput by actor, and connected task workers.
   * **Task Performance**: 90th percentile task execution duration by actor, and 90th percentile policy binding execution duration by binding target type.

3. **Charmed Authentik LDAP Outpost Operator Dashboard** (``authentik-ldap-outpost-dashboard``)
   
   * **Availability**: Fractional unit availability gauge and list of available units.
   * **Logging**: High-severity log entries time series (error, fatal, critical) grouped by level.
   * **LDAP**: Request rate by operation type, 90th percentile request duration by operation type, rejected requests by reason, and WebSocket connection state to the Authentik server (``authentik_outpost_connection``).

Metric inventory
----------------

The following Prometheus metrics are scraped from the Authentik workload on port 9300:

.. list-table::
   :header-rows: 1
   :widths: 35 20 45

   * - Metric Name
     - Component
     - Description & Key Labels
   * - ``up``
     - All
     - Standard Prometheus reachability metric (1 = available, 0 = unreachable).
   * - ``django_http_responses_total_by_status_total``
     - Server
     - Total HTTP responses partitioned by ``status`` code (e.g. 200, 400, 500).
   * - ``django_http_requests_latency_seconds_by_view_method_bucket``
     - Server
     - HTTP request latency histogram partitioned by Django ``view`` and HTTP method.
   * - ``authentik_flows_plan_time_bucket``
     - Server
     - Histogram of the time taken to plan an execution flow, partitioned by ``flow_slug``.
   * - ``authentik_flows_execution_stage_time_bucket``
     - Server
     - Duration histogram of individual flow stages, partitioned by ``stage_type``.
   * - ``authentik_policies_engine_time_total_seconds_bucket``
     - Server
     - Execution time histogram for the policy engine partitioned by target ``obj_type``.
   * - ``authentik_policies_execution_time_bucket``
     - Worker
     - Policy binding execution latency partitioned by ``binding_target_type``.
   * - ``authentik_outposts_connected``
     - Server
     - Gauge showing active connected outposts partitioned by ``outpost`` identifier.
   * - ``authentik_tasks_workers``
     - Server, Worker
     - Number of active background workers, labeled with ``version`` and ``version_matched`` (signals version skew between server and worker).
   * - ``authentik_tasks_queued``
     - Server, Worker
     - Number of pending tasks in background queues, labeled by ``queue_name`` and ``actor_name``.
   * - ``authentik_tasks_total``
     - Worker
     - Counter of executed tasks partitioned by ``actor_name``.
   * - ``authentik_tasks_duration_milliseconds_bucket``
     - Worker
     - Histogram of task execution times partitioned by ``actor_name``.
   * - ``authentik_outpost_ldap_request_duration_seconds_count`` / ``_bucket``
     - Outpost
     - Latency and invocation counts of LDAP requests partitioned by operation ``type`` (bind, search, unbind).
   * - ``authentik_outpost_ldap_requests_rejected_total``
     - Outpost
     - Counter of rejected LDAP requests partitioned by ``reason``.
   * - ``authentik_outpost_connection``
     - Outpost
     - Binary gauge (1 = connected, 0 = disconnected) reflecting the WebSocket link from the outpost to the Authentik server.

.. important::
   **Metric aggregation guidelines**:
   
   * **Task queue de-duplication**: ``authentik_tasks_queued`` is emitted by both the server and worker instances. Always aggregate with ``max by (queue_name, actor_name)`` before summing across the model to avoid double-counting.
   * **Version skew detection**: Monitor ``authentik_tasks_workers{version_matched="false"}`` to identify mismatched worker versions following an upgrade.
   * **Outpost connectivity**: Alert on ``authentik_outpost_connection == 0``. If the outpost loses its WebSocket channel, it cannot synchronize directory configuration or execute non-cached authentication flows.
