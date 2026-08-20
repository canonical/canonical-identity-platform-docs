.. _integrate-with-openfga:

Integrate with OpenFGA
======================

If you are charming an application that supports using OpenFGA for authorization, you can integrate with the `OpenFGA charm <https://charmhub.io/openfga-k8s>`_ to automatically create a store that will persist even if the relation is removed.

Add an integration endpoint to ``metadata.yaml``
------------------------------------------------

The OpenFGA store information is communicated over relation data, where the OpenFGA charm acts as the provider and the application acts as the requirer. For consistency across the ecosystem, it is encouraged to name the relation ``openfga``.

Edit your charm’s ``metadata.yaml`` to add the following under the ``requires`` section:

.. code-block:: yaml

    requires:
        # any other providers your charm supports
        openfga:
            interface: openfga

Fetch the OpenFGA charm library
-------------------------------

The OpenFGA charm will create a store per OpenFGA relation, which is managed by the OpenFGA library.

.. code-block:: shell

    charmcraft fetch-lib charms.openfga_k8s.v1.openfga

The library offers an ``OpenFGARequires`` object, which provides sensible defaults and a simple API that you can use to connect with the OpenFGA server.

Use ``OpenFGARequires``
-----------------------

To initialise the library in your charm code:

.. code-block:: python

    class SomeCharm(CharmBase):
      def __init__(self, *args):
        self.openfga = OpenFGARequires(self, "test-openfga-store")
        self.framework.observe(
            self.openfga.on.openfga_store_created,
            self._on_openfga_store_created,
        )
        ...

        def _on_openfga_store_created(self, event: OpenFGAStoreCreateEvent):
            if not event.store_id:
                return

            info = self.openfga.get_store_info()
            if not info:
                return

            logger.info("store id {}".format(info.store_id))
            logger.info("token {}".format(info.token))
            logger.info("grpc_api_url {}".format(info.grpc_api_url))
            logger.info("http_api_url {}".format(info.http_api_url))
            ...

When this charm is integrated with OpenFGA using:

.. code-block:: shell

    juju integrate openfga:openfga some-charm:openfga

The OpenFGA charm will create a new store with the name ``test-openfga-store`` and provide the store ID, along with any information needed to connect to the server, in the relation databag.

If the relation is removed:

.. code-block:: shell

    juju remove-relation openfga:openfga some-charm:openfga

The store will not be removed, and the same store will be provided if the same charm is re-integrated with OpenFGA. To achieve this, the OpenFGA charm assumes that the store name requested by the application is unique.
