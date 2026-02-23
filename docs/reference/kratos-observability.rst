.. _kratos-observability:

Observability setup in Charmed Kratos
=====================================

The Kratos charmed Operator of the Identity Platform provides integrations to the Canonical Observability Stack.

In this reference we will provide a technical description of the alert rules, and the dashboards the Kratos operator passes to the components of the COS. We will also showcase the Metrics we use to provide observability features.

.. note::
    See more: `Charmhub | Kratos > Integrations > metrics-endpoint, grafana-dashboard <https://charmhub.io/kratos/integrations>`_


Metrics
-------

up
~~

Description
^^^^^^^^^^^

When this counter equals one, that means the instance the metric was scraped from is available to the Prometheus deployment. If the counter equals zero, or the metric is not present, then the application is inaccessible to the Prometheus deployment.

Source Component
^^^^^^^^^^^^^^^^

Prometheus

Intended use
^^^^^^^^^^^^

This metric signals whether a unit is unavailable. By specifying the label ``juju_unit`` this metric tells you whether the unit is available to the Prometheus component or not. Example: ``up{juju_unit="kratos/0"}``

Observed by
^^^^^^^^^^^

- **Alert Rules**: Rules in group KratosUnavailable
- **Dashboard**: Kratos Unit Availability

level
~~~~~

Description
^^^^^^^^^^^

All Kratos logs have a level, to indicate the seriousness of an event. Kratos logs (and json formatted logs in general) can be filtered by log levels.

Source Component
^^^^^^^^^^^^^^^^

Loki

Usage
^^^^^

Use the LogQL’s abilities to enumerate log entries that match a regex template, and to parse json logs and populate fields such as level. Example: ``{juju_charm="kratos"} | json | level =~ "error"``

Observed by
^^^^^^^^^^^

- **Alert Rules**: Rules in group KratosHighSeverityLog
- **Dashboard Visualisation**: Kratos High Severity Log Entries

Alert rules
-----------

KratosHighSeverityLog
~~~~~~~~~~~~~~~~~~~~~

Description
^^^^^^^^^^^

Alerts in this alert group are fired based on how many log entries with the log level of error, critical, or fatal have been created in the last five minutes.

Data Source
^^^^^^^^^^^

Loki

KratosUnavailable
~~~~~~~~~~~~~~~~~

Description
^^^^^^^^^^^

Alerts in this alert group observe the up metric of individual Kratos deployment units. In this context, a unit not being available by Prometheus is treated as being inactive. Alerts are fired when units are inactive for at least one minute.

Data Source
^^^^^^^^^^^

Prometheus

Dashboards
----------

Kratos High Severity Log Entries
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Description
^^^^^^^^^^^

Visualization for the number of log entries with levels error or above within 5 minutes spans.

Associated Alerts
^^^^^^^^^^^^^^^^^

Alert rules in alert group **KratosHighSeverityLog**.
