.. _observability:

Observability in Identity Platform
==================================

The Canonical Identity Platform provides integration with the `Canonical Observability Stack (COS) <https://charmhub.io/topics/canonical-observability-stack>`_. This integration ensures that administrators can monitor the health, performance, and security of the identity infrastructure through standardized metrics, automated alerting, and pre-configured dashboards.

Key Observability Features
--------------------------

The observability suite across all Identity Platform charms focuses on three core pillars:

* **Health Monitoring (Prometheus):** Tracking unit availability (the ``up`` metric) to ensure the service is reachable.
* **Log Aggregation (Loki):** Centralizing logs to identify high-severity events (Error, Critical, Fatal) across the stack.
* **Visualization (Grafana):** Providing out-of-the-box dashboards to visualize traffic patterns, error rates, and system uptime.

Component Reference Guides
--------------------------

Each component of the Identity Platform provides its own set of specific metrics and alerts.
Explore the detailed technical references for each charm below:

.. toctree::
   :maxdepth: 1
   :glob:

   /reference/hydra-observability
   /reference/kratos-observability
   /reference/login-ui-observability
   /reference/openfga-observability

Common Alert Severities
-----------------------

Across the platform, alerts are categorized to help your operations team prioritize responses:

* **Fatal:** Total loss of service; requires immediate intervention.
* **Critical:** Service is active but at high risk of failure (e.g., only one unit remaining).
* **Error:** Specific application functions are failing, but the service remains partially available.
* **Warning:** Anomalies detected (e.g., low-frequency high-severity logs) that do not yet impact service.



Next Steps
----------

To begin monitoring your deployment, ensure your charms are integrated with the COS bundle:

.. code-block:: shell

    juju integrate <charm-name>:metrics-endpoint prometheus:metrics-endpoint
    juju integrate <charm-name>:grafana-dashboard grafana:grafana-dashboard
    juju integrate <charm-name>:log-proxy loki:logging
