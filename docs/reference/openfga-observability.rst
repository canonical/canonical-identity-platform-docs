.. _openfga-observability:

Observability setup in Charmed OpenFGA
======================================

The OpenFGA charmed operator provides integrations to the Canonical Observability Stack.

In this reference we will provide a technical description of the alert rules, and the dashboards the OpenFGA operator passes to the components of the COS. We will also showcase the Metrics we use to provide observability features.

.. note::
    See more: `Charmhub | OpenFGA > Integrations > metrics-endpoint, grafana-dashboard <https://charmhub.io/openfga-k8s/integrations>`_

Metrics
-------

up
~~

Description
^^^^^^^^^^^

When this counter equals one, that means the instance the metric was scraped from is available to the Prometheus deployment. If the counter equals zero, or the metric is not present, then the application is inaccessible to the Prometheus deployment.

Source
^^^^^^

Prometheus

Usage
^^^^^

This metric signals whether a unit is unavailable. By specifying the label ``juju_unit`` this metric tells you whether the unit is available to the Prometheus component or not.

Example: ``up{juju_unit="openfga-k8s/0"}``

Observed
^^^^^^^^

* **Alert Rules**: Rules in group OpenFGAUnavailable
* **Dashboard**: OpenFGA Unit Availability

level
~~~~~

Description
^^^^^^^^^^^

All OpenFGA logs have a level, to indicate the seriousness of an event. OpenFGA logs (and json formatted logs in general) can be filtered by log levels.

Source
^^^^^^

Loki

Usage
^^^^^

Use the LogQL’s abilities to enumerate log entries that match a regex template, and to parse json logs and populate fields such as level. Example: ``{juju_charm="openfga"} | json | level =~ "error"``

Observed
^^^^^^^^

* **Alert Rules**: Rules in group OpenFGAHighSeverityLog
* **Dashboard Visualisation**: OpenFGA High Severity Log Entries

Alerts
------

OpenFGAHighSeverityLog
~~~~~~~~~~~~~~~~~~~~~~

Description
^^^^^^^^^^^

Alerts in this alert group are fired based on how many log entries with the log level of error, critical, or fatal have been created in the last five minutes.

Data
^^^^

Loki

OpenFGAUnavailable-multiple
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Description
^^^^^^^^^^^

More than one unit in a multi-unit deployment is inactive.

Observed
^^^^^^^^

Multiple operator instances have encountered an issue from which they can not recover within a minute. Without other alerts suggesting otherwise, the alert also suggests that there is at least one healthy unit in the deployment.

Severity
^^^^^^^^

**Error**, because while the service is still available, there is now potential for a critical scenario.

OpenFGAUnavailable-all
~~~~~~~~~~~~~~~~~~~~~~

Description
^^^^^^^^^^^

No unit in the deployment is active.

Observed
^^^^^^^^

None of the units in the deployment are active. The service is inaccessible.

Severity
^^^^^^^^

**Fatal**. The service has failed, and can not recover without human intervention.

Dashboards
----------

OpenFGA
~~~~~~~

Description
^^^^^^^^^^^

* visualization for the number of log entries with levels error or above within 5 minutes spans.
* visualization displaying the availability of OpenFGA units.

Associated
^^^^^^^^^^

* Alert rules in alert group **OpenFGAHighSeverityLog**.
* Alert rules in alert group **OpenFGAUnavailable**.
