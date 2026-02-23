.. _integrate-with-oidc-compatible-charms:

Integrate your OIDC-compatible charm
============================================

The Identity Platform provides seamless integration with your OIDC compatible charms using the power of juju relations. We are going to assume that:

1. You have deployed the :doc:`Identity Platform </tutorial/canonical-identity-platform>`.
2. You have deployed an OIDC compatible charmed application.

To connect an OIDC compatible charmed application with the Identity Platform, integrate it with ``hydra``:

.. code-block:: shell

    juju integrate hydra <OIDC compatible charmed application>

Use ``juju status`` to inspect the progress of the integration. After the applications have settled down, you should be able to log in to your application using the Identity Platform.

.. note::
    See more: `Charmhub | Hydra > Integrations > oauth <https://charmhub.io/hydra/integrations>`_
