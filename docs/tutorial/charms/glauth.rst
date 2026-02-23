.. _glauth-tutorial:

Charmed GLAuth K8s
==================

This tutorial aims to provide a general walkthrough to set up a fully working
GLAuth server using ``glauth-k8s`` charmed operator, MicroK8s, and Juju. 

Set things up
-------------

Follow `this guide <https://documentation.ubuntu.com/juju/3.6/howto/manage-your-juju-deployment/set-up-your-juju-deployment-local-testing-and-development/>`_
to bootstrap a MicroK8s cloud running a Juju controller.

Create a Juju model:

.. code-block:: shell

    juju add-model dev

Deploy prerequisite charmed operators
-------------------------------------

The ``glauth-k8s`` charmed operator requires the following charmed operators
deployed in the MicroK8s cluster:

* `postgresql-k8s-operator <https://charmhub.io/postgresql-k8s>`_
* `self-signed-certificates-operator <https://charmhub.io/self-signed-certificates>`_

.. code-block:: shell

    juju deploy postgresql-k8s --channel 14/stable --trust

    juju deploy self-signed-certificates

Deploy ``glauth-k8s`` charmed operator
--------------------------------------

The ``glauth-k8s`` charmed operator can be deployed as follows:

.. code-block:: shell

    juju deploy glauth-k8s --channel edge --trust

Integrate with other charmed operators
--------------------------------------

The ``glauth-k8s`` charmed operator needs to integrate with the ``postgresql-k8s``
and ``self-signed-certificates`` charmed operators to reach ``active`` state:

.. code-block:: shell

    juju integrate glauth-k8s postgresql-k8s

    juju integrate glauth-k8s self-signed-certificates

The ``glauth-k8s`` charmed operator offers the ``ldap`` integration with any LDAP
client charmed operator following the ``ldap`` interface protocol. Assuming we
have deployed such a client charmed operator, we can proceed to integrate it
with the ``glauth-k8s``:

.. code-block:: shell

    juju integrate <client-charm>:ldap glauth-k8s:ldap

The GLAuth supports the ``StartTLS`` operation, and the ``glauth-k8s`` charmed
operator enables it by default. In order to allow the client to trust the
self-signed certificates, we need to integrate the client charmed operator with
the ``glauth-k8s`` charmed operator for the ``certificate_transfer`` interface
protocol:

.. code-block:: shell

    juju integrate <client-charm>:send-ca-cert glauth-k8s:send-ca-cert

Furthermore,
the `glauth-utils charmed operator <https://charmhub.io/glauth-utils>`_ allows
us to apply data changes by using
the `LDIF <https://datatracker.ietf.org/doc/html/rfc2849>`_. To integrate with
the ``glauth-utils``:

.. code-block:: shell

    juju deploy glauth-utils --channel edge --trust

    juju integrate glauth-k8s glauth-utils

A sample of supported LDIF content records can be
found `here <https://github.com/canonical/glauth-utils/blob/main/SAMPLES.md>`_.

We now should be able to reach to the following deployment status:

.. code-block:: shell

    $ juju status --relations

    Model  Controller          Cloud/Region        Version  SLA          Timestamp
    dev    microk8s-localhost  microk8s/localhost  3.2.0    unsupported  16:12:33Z

    App                       Version  Status  Scale  Charm                     Channel    Rev  Address         Exposed  Message
    client                             active      1  client                                 0  10.152.183.78   no
    glauth-k8s                         active      1  glauth-k8s                edge        13  10.152.183.231  no
    glauth-utils                       active      1  glauth-utils              edge         4  10.152.183.26   no
    postgresql-k8s            14.10    active      1  postgresql-k8s            14/stable  193  10.152.183.163  no       Primary
    self-signed-certificates           active      1  self-signed-certificates  stable      72  10.152.183.21   no

    Unit                         Workload  Agent  Address      Ports  Message
    client/0* active    idle   10.1.48.115
    glauth-k8s/0* active    idle   10.1.48.96
    glauth-utils/0* active    idle   10.1.48.107
    postgresql-k8s/0* active    idle   10.1.48.77          Primary
    self-signed-certificates/0* active    idle   10.1.48.86

    Integration provider                   Requirer                       Interface             Type     Message
    glauth-k8s:glauth-auxiliary            glauth-utils:glauth-auxiliary  glauth_auxiliary      regular
    glauth-k8s:glauth-peers                glauth-k8s:glauth-peers        glauth_peers          peer
    glauth-k8s:ldap                        client:ldap                    ldap                  regular
    glauth-k8s:send-ca-cert                client:send-ca-cert            certificate_transfer  regular
    postgresql-k8s:database                glauth-k8s:pg-database         postgresql_client     regular
    postgresql-k8s:database-peers          postgresql-k8s:database-peers  postgresql_peers      peer
    postgresql-k8s:restart                 postgresql-k8s:restart         rolling_op            peer
    postgresql-k8s:upgrade                 postgresql-k8s:upgrade         upgrade               peer
    self-signed-certificates:certificates  glauth-k8s:certificates        tls-certificates      regular

Tear things down
----------------

Remove the ``dev`` Juju model:

.. code-block:: shell

    juju destroy-model dev
