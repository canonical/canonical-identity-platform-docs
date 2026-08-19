.. meta::
    :description: How to integrate Charmed Authentik with the Canonical Observability Stack (COS) for logs, metrics, dashboards, and traces.

.. _integrate-with-cos-authentik:

Integrate with the Canonical Observability Stack (COS)
======================================================

This guide describes how to integrate your Charmed Authentik deployment with the **Canonical Observability Stack (COS)** to forward logs to Loki, metrics to Prometheus, dashboards to Grafana, and traces to Tempo.

Prerequisites
-------------

* An active Charmed Authentik deployment (including ``authentik-server``, ``authentik-worker``, and ``authentik-ldap-outpost``).
* A deployed Canonical Observability Stack (COS Lite or individual charms: ``loki-k8s``, ``prometheus-k8s``, ``grafana-k8s``, and ``tempo-coordinator-k8s``).

Integration endpoints
---------------------

All three Authentik operator charms provide identical observability interfaces:

* **``logging``**: Forwards application and container logs to Loki via the ``loki_push_api`` interface.
* **``metrics-endpoint``**: Provides Prometheus scrape targets via the ``prometheus_scrape`` interface on port 9300.
* **``grafana-dashboard``**: Automatically provisions built-in Grafana dashboards for monitoring each charm.
* **``tracing``**: Forwards OpenTelemetry (OTLP) application spans to Tempo via the ``tempo-coordinator-k8s`` charm.

Step-by-step integration
------------------------

Step 1: Forward logs to Loki
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Integrate all three Authentik charms with Loki:

.. code-block:: bash

    juju integrate authentik-server:logging loki-k8s:logging
    juju integrate authentik-worker:logging loki-k8s:logging
    juju integrate authentik-ldap-outpost:logging loki-k8s:logging

Step 2: Forward telemetry metrics to Prometheus
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Connect the charms to Prometheus:

.. code-block:: bash

    juju integrate authentik-server:metrics-endpoint prometheus-k8s:metrics-endpoint
    juju integrate authentik-worker:metrics-endpoint prometheus-k8s:metrics-endpoint
    juju integrate authentik-ldap-outpost:metrics-endpoint prometheus-k8s:metrics-endpoint

.. note::
    Prometheus scrapes application metrics directly pod-to-pod on port 9300. The metrics port is deliberately not exposed on the Kubernetes ClusterIP service to avoid unauthenticated metrics exposure.

Step 3: Provision Grafana dashboards
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Forward built-in Grafana dashboards to visualize availability, request latency, background task queues, and LDAP operations:

.. code-block:: bash

    juju integrate authentik-server:grafana-dashboard grafana-k8s:grafana-dashboard
    juju integrate authentik-worker:grafana-dashboard grafana-k8s:grafana-dashboard
    juju integrate authentik-ldap-outpost:grafana-dashboard grafana-k8s:grafana-dashboard

Step 4: Stream application traces to Tempo
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To trace HTTP requests and internal workflows, integrate with the Tempo coordinator:

.. code-block:: bash

    juju integrate authentik-server:tracing tempo-coordinator-k8s:tracing
    juju integrate authentik-worker:tracing tempo-coordinator-k8s:tracing
    juju integrate authentik-ldap-outpost:tracing tempo-coordinator-k8s:tracing

Cross-model integrations
------------------------

If your Canonical Observability Stack is deployed in a separate Juju model (e.g. ``cos``), establish the integrations using cross-model offers:

.. code-block:: bash

    # In model 'cos' (if not already offered):
    juju offer loki-k8s:logging
    juju offer prometheus-k8s:metrics-endpoint
    juju offer grafana-k8s:grafana-dashboard
    juju offer tempo-coordinator-k8s:tracing

    # In model 'authentik':
    juju integrate authentik-server:logging admin/cos.loki-k8s
    juju integrate authentik-server:metrics-endpoint admin/cos.prometheus-k8s
    juju integrate authentik-server:grafana-dashboard admin/cos.grafana-k8s
    juju integrate authentik-server:tracing admin/cos.tempo-coordinator-k8s
