.. _glauth-utils-tutorial:

Charmed GLAuth Utility K8s
==========================

This tutorial aims to provide a general walkthrough to set up a fully working
GLAuth utility using ``glauth-utils`` charmed operator, MicroK8s, and Juju.

Set things up
-------------

Follow `this guide <https://documentation.ubuntu.com/juju/3.6/howto/manage-your-juju-deployment/set-up-your-juju-deployment-local-testing-and-development/>`_
to bootstrap a MicroK8s cloud running a Juju controller.

Create a Juju model:

.. code-block:: shell

    juju add-model dev

Deploy prerequisite charmed operators
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``glauth-utils`` charmed operator requires the following charmed operators
deployed in the MicroK8s cluster:

* `glauth-k8s-operator <https://charmhub.io/glauth-k8s>`_

.. code-block:: shell

    juju deploy glauth-k8s --channel edge --trust

Deploy ``glauth-utils`` charmed operator
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``glauth-utils`` charmed operator can be deployed as follows:

.. code-block:: shell

    juju deploy glauth-utils --channel edge --trust

Integrate with other charmed operators
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``glauth-utils`` charmed operator offers the ``glauth-auxiliary`` interface in
order to supplement the ``glauth-k8s`` charmed operator: 

.. code-block:: shell

    juju integrate glauth-utils:glauth-auxiliary glauth-k8s:glauth-auxiliary

Tear things down
----------------

Remove the ``dev`` Juju model:

.. code-block:: shell

    juju destroy-model dev
