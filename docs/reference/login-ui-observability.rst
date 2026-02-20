.. _login-ui-observability:

Observability setup in Charmed Identity Platform Login UI
=========================================================

The Identity Platform Login UI Operator of the Identity Platform provides integrations to the Canonical Observability Stack.

In this reference we will provide a technical description of the alert rules, and the dashboards the Identity Platform Login UI operator passes to the components of the COS. We will also showcase the Metrics we use to provide observability features.

.. note::
    See more: `Charmhub | Identity Platform Login UI Operator > Integrations > metrics-endpoint,  grafana-dashboard <https://charmhub.io/identity-platform-login-ui-operator/integrations>`_


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

This metric signals whether a unit is unavailable. By specifying the label ``juju_unit`` this metric tells you whether the unit is available to the Prometheus component or not. Example: ``up{juju_unit="identity-platform-login-ui-operator/0"}``

Observed by
^^^^^^^^^^^

- Alert Rules: Rules in group LoginUIUnavailable
- Dashboard: Login UI Unit Availability

severity
~~~~~~~~

Description
^^^^^^^^^^^

All Identity Platform Login UI logs have a severity level to indicate the seriousness of an event. Identity Platform Login UI logs (and json formatted logs in general) can be filtered by these levels.

Source Component
^^^^^^^^^^^^^^^^

Loki

Usage
^^^^^

Use the LogQL’s abilities to enumerate log entries that match a regex template, and to parse json logs and populate fields such as level. Example: ``{juju_charm="identity-platform-login-ui-operator"} | json | severity =~ "error"``

Observed by
^^^^^^^^^^^

- Alert Rules: Rules in group LoginUiHighSeverityLog
- Dashboard Visualisation: Login UI High Severity Log Entries

Alert rules
-----------

LoginUIHighSeverityLog
~~~~~~~~~~~~~~~~~~~~~~

Description
^^^^^^^^^^^

Alerts in this alert group are fired based on how many log entries with the log level of error, critical, or fatal have been created in the last five minutes.

Data Source
^^^^^^^^^^^^

Loki

LoginUIUnavailable
~~~~~~~~~~~~~~~~~~

Description
^^^^^^^^^^^

Alerts in this alert group observe the up metric of individual Identity Platform Login UI deployment units. In this context, a unit not being available by Prometheus is treated as being inactive. Alerts are fired when units are inactive for at least one minute.

Data Source
^^^^^^^^^^^

Prometheus

Dashboards
----------

Login UI High Severity Log Entries
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Description
^^^^^^^^^^^

Visualization for the number of log entries with levels error or above produced by the Identity Platform Login UI Operator deployment within 5 minutes spans.

Associated Alerts
^^^^^^^^^^^^^^^^^

Alert rules in alert group **LoginUi HighSeverityLog**.
