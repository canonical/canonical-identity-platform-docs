.. _openfga-tutorial:

Charmed OpenFGA
===============

This tutorial shows how to set up a fully working OpenFGA server using our charm, MicroK8s and Juju. 

Set things up
-------------

Bootstrap a microk8s controller using juju ``3.6`` and create a new Juju model:

.. code-block:: shell

    $ juju add-model openfga
    Added 'openfga' model on microk8s/localhost with credential 'microk8s' for user 'admin'

.. note::

    See more: `Set up your test environment <https://documentation.ubuntu.com/juju/3.6/howto/manage-your-juju-deployment/set-up-your-juju-deployment-local-testing-and-development>`_

Deploy the charm
^^^^^^^^^^^^^^^^

Deploy the openfga charm:

.. code-block:: shell

    $ juju deploy openfga-k8s --channel edge
    Deploying "openfga-k8s" from local charm "openfga-k8s", revision 0 on ubuntu@22.04/stable

OpenFGA requires a way to persist data, in the case of our charm we enforce the usage of a ``postgreSQL`` database. We are going to use the `postgresql-k8s charm <https://charmhub.io/postgresql-k8s>`_

.. code-block:: shell

    $ juju deploy postgresql-k8s --channel 14/stable --trust
    Located charm "postgresql-k8s" in charm-hub, revision 233
    Deploying "postgresql-k8s" from charm-hub charm "postgresql-k8s", revision 233 in channel 14/stable on ubuntu@22.04/stable

Integrate with PostgreSQL
^^^^^^^^^^^^^^^^^^^^^^^^^

Once that is done (no need to wait for it to be ready) we can proceed in deploying ``openfga`` and integrate the 2 charms

.. code-block:: shell

    $ juju integrate postgresql-k8s:database openfga-k8s

after some time we should be able to inspect that all has been successfully deployed and connected

.. code-block:: shell

    $ juju status --relations

    Model    Controller          Cloud/Region        Version  SLA          Timestamp
    openfga  microk8s-localhost  microk8s/localhost  3.6.3    unsupported  15:59:57+02:00

    App             Version  Status  Scale  Charm           Channel  Rev  Address         Exposed  Message
    openfga-k8s              active      1  openfga-k8s                0  10.152.183.172  no
    postgresql-k8s  14.11    active      1  postgresql-k8s  14/stable  233  10.152.183.177  no       Primary

    Unit               Workload  Agent  Address       Ports  Message
    openfga-k8s/0* active    idle   10.1.245.154
    postgresql-k8s/0* active    idle   10.1.245.156         Primary

    Integration provider           Requirer                       Interface          Type     Message
    openfga-k8s:peer               openfga-k8s:peer               openfga-peer       peer
    postgresql-k8s:database        openfga-k8s:database           postgresql_client  regular
    postgresql-k8s:database-peers  postgresql-k8s:database-peers  postgresql_peers   peer
    postgresql-k8s:restart         postgresql-k8s:restart         rolling_op         peer
    postgresql-k8s:upgrade         postgresql-k8s:upgrade         upgrade            peer

Deployment checks
^^^^^^^^^^^^^^^^^

Once all is up we can verify that OpenFGA is up and running by creating a store using the APIs.

First, find the secret:

.. code-block:: shell

    $ juju secrets
    ID                    Owner           Rotation  Revision  Last updated
    co9vcjrmrojc77r2rd2g  openfga-k8s     never            1  20 minutes ago
    co9vd83mrojc77r2rd30  postgresql-k8s  never            1  19 minutes ago
    co9vg4bmrojc77r2rd3g  postgresql-k8s  never            1  13 minutes ago

    $ juju show-secret co9vcjrmrojc77r2rd2g --reveal
    co9vcjrmrojc77r2rd2g:
      revision: 1
      owner: openfga-k8s
      created: 2024-04-08T13:51:12Z
      updated: 2024-04-08T13:51:12Z
      content:
        token: tMkhBA0drx2nfqIubs9vR9KSeC3oIen5jYesTEL_gjM

Then we can try to create a store using the HTTP API (via ``httpie``):

.. code-block:: shell

    $ http POST :8080/stores name=openfga-demo Authorization:" Bearer tMkhBA0drx2nfqIubs9vR9KSeC3oIen5jYesTEL_gjM"
    HTTP/1.1 201 Created
    Content-Length: 143
    Content-Type: application/json
    Date: Mon, 08 Apr 2024 14:06:26 GMT
    Vary: Origin
    X-Http-Code: 201
    X-Request-Id: 1da6c68d-d3fe-4e01-b957-19da07cb5270

    {
        "created_at": "2024-04-08T14:06:26.848060Z",
        "id": "01HTZ0G7GZ4QEKHV82TV59H6ES",
        "name": "openfga-demo",
        "updated_at": "2024-04-08T14:06:26.848060Z"
    }

Grafana, Loki, and Prometheus
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This OpenFGA operator integrates with `Canonical Observability Stack <https://charmhub.io/topics/canonical-observability-stack>`_ (COS) bundle.
It comes with a Grafana dashboard as well as Loki and Prometheus alert rules for basic common scenarios.
To integrate with the COS bundle, after you `deploy it <https://charmhub.io/topics/canonical-observability-stack/tutorials/install-microk8s#heading--deploy-the-cos-lite-bundle>`_, you can run:

.. code-block:: shell

    $ juju integrate openfga:grafana-dashboard grafana:grafana-dashboard
    $ juju integrate openfga:metrics-endpoint prometheus:metrics-endpoint
    $ juju integrate loki:logging openfga:log-proxy

Scale
^^^^^

To scale the OpenFGA server we can exploit ``juju scale-application``:

.. code-block:: shell

    $ juju scale-application openfga-k8s 5
    openfga-k8s scaled to 5 units

In due time, we should be able to see that all the requested units have come up successfully:

.. code-block:: shell

    $ juju status --relations
    Model    Controller          Cloud/Region        Version  SLA          Timestamp
    openfga  microk8s-localhost  microk8s/localhost  3.1.7    unsupported  16:37:36+02:00

    App             Version  Status  Scale  Charm           Channel  Rev  Address         Exposed  Message
    openfga-k8s              active      5  openfga-k8s                0  10.152.183.172  no
    postgresql-k8s  14.11    active      1  postgresql-k8s  14/edge  233  10.152.183.177  no       Primary

    Unit               Workload  Agent  Address       Ports  Message
    openfga-k8s/0* active    idle   10.1.245.154
    openfga-k8s/1      active    idle   10.1.245.141
    openfga-k8s/2      active    idle   10.1.245.144
    openfga-k8s/3      active    idle   10.1.245.155
    openfga-k8s/4      active    idle   10.1.245.131
    postgresql-k8s/0* active    idle   10.1.245.156         Primary

    Integration provider           Requirer                       Interface          Type     Message
    openfga-k8s:peer               openfga-k8s:peer               openfga-peer       peer
    postgresql-k8s:database        openfga-k8s:database           postgresql_client  regular
    postgresql-k8s:database-peers  postgresql-k8s:database-peers  postgresql_peers   peer
    postgresql-k8s:restart         postgresql-k8s:restart         rolling_op         peer
    postgresql-k8s:upgrade         postgresql-k8s:upgrade         upgrade            peer

----

Tear things down
----------------

To tear things down, remove the entire ``openfga`` model in juju with

.. code-block:: shell

    juju destroy-model openfga
