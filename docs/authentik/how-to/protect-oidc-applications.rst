.. meta::
    :description: How to integrate downstream web applications with Charmed Authentik for Single Sign-On (SSO) using OIDC or OAuth2.

.. _protect-oidc-applications-authentik:

Protect OIDC and OAuth applications
===================================

This guide describes how to integrate downstream web applications (specifically **Charmed Grafana**) with Charmed Authentik to enable Single Sign-On (SSO) via OpenID Connect (OIDC) or OAuth 2.0.

Prerequisites
-------------

This guide assumes you have an active Charmed Authentik deployment matching the topology established in the :doc:`Getting started tutorial </authentik/tutorial/getting-started>`. Specifically, you should have:

* An active ``authentik-server`` deployment integrated with its database and certificates.
* An active Traefik Ingress controller (``traefik-public``) deployed (typically in a shared administrative model, e.g., ``core``).
* Administrative access (``akadmin``) to the Authentik dashboard.

Technical mechanism: The ``oauth`` relation
-------------------------------------------

In Juju, application integration is automated via relation interfaces. The ``authentik-server`` charm provides the ``oauth`` relation interface.

When a consuming application charm (such as ``grafana-k8s``) integrates with Authentik:

1. **Dynamic Client Creation**: The ``authentik-server`` charm calls the Authentik API to dynamically create a new **OIDC Provider** and **Application** entity.
2. **Secure Credentials Generation**: The charm generates a unique ``client_id`` and ``client_secret``.
3. **Data Exchange**: The server charm publishes the credentials, endpoints (issuer URL, authorization endpoint, token endpoint), and scopes into the relation databag.
4. **Auto-Configuration**: The consuming application charm reads these variables and configures its authentication engine automatically.

Step-by-step integration
------------------------

Step 1: Deploy and expose Charmed Grafana
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Deploy Grafana within your Juju model:

.. code-block:: bash

    juju deploy grafana-k8s --channel 2/stable

To expose Grafana's dashboard externally through your Traefik Ingress, establish an ingress integration using Traefik's ``traefik-route`` endpoint:

.. code-block:: bash

    # If Traefik is in the same model:
    juju integrate grafana-k8s:ingress traefik-public:traefik-route

    # If Traefik is deployed in a different model (e.g., 'core'):
    juju integrate grafana-k8s:ingress core.traefik-public:traefik-route

Step 2: Establish the OAuth relation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To connect Grafana with the Authentik Server OIDC provider, establish the integration over the ``oauth`` endpoint:

Via Juju CLI
^^^^^^^^^^^^

.. code-block:: bash

    juju integrate grafana-k8s:oauth authentik-server:oauth

Via Terraform
^^^^^^^^^^^^^

Add the following integration block to your Terraform configuration:

.. code-block:: hcl

    resource "juju_integration" "grafana_auth" {
      model = "authentik"
      application {
        name     = "grafana-k8s"
        endpoint = "oauth"
      }
      application {
        name     = "authentik-server"
        endpoint = "oauth"
      }
    }

Step 3: Configure TLS certificate trust (Required for self-signed CAs)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Because Grafana must execute backend HTTP requests to Authentik to exchange tokens, it must trust the certificate authority (CA) that issued the certificate terminating TLS at Traefik.

If you are using self-signed certificates, integrate Grafana with your CA provider to transfer the trust chain:

.. code-block:: bash

    # If the CA provider is in the same model as Grafana:
    juju integrate grafana-k8s:receive-ca-cert self-signed-certificates:send-ca-cert

    # If the CA provider is in another model (e.g. 'core', as in the tutorial topology),
    # offer it once and consume it cross-model:
    juju offer core.self-signed-certificates:send-ca-cert
    juju integrate -m authentik grafana-k8s:receive-ca-cert admin/core.self-signed-certificates

Step 4: Verify SSO authentication
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. **Retrieve Grafana's Ingress URL**:
   Find the external URL of your Grafana dashboard:

   .. code-block:: bash

       juju status grafana-k8s

2. Navigate to your Grafana URL. The URL path is model-dependent and follows the template ``https://<traefik-ip>/<model-name>-<app-name>`` (e.g., ``https://<traefik-ip>/authentik-grafana-k8s`` if deployed in the ``authentik`` model).
3. The login page will display a federated authentication button (typically labeled **Sign in with external identity provider**).
4. Select the button to be redirected to your Authentik login portal.
5. Authenticate using your credentials (e.g., ``akadmin`` or a created user account).
6. Upon successful authentication, you will be redirected back to Grafana with an active session.

Step 5: Disconnecting and revocation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To revoke access or remove the application from the identity perimeter:

.. code-block:: bash

    juju remove-relation grafana-k8s:oauth authentik-server:oauth

The operator automatically deletes the OIDC Provider and Application entities from the Authentik database, invalidating the credentials.

Upstream documentation references
---------------------------------

While the core OIDC setup and credential exchange are automated by Juju, advanced configurations are managed through the Authentik Admin Interface:

* **Customizing Scopes & Claims**: Add custom user attributes to OIDC tokens. See the `Upstream Authentik OIDC Claims Guide <https://docs.goauthentik.io/add-secure-apps/providers/oauth2/>`_.
* **Custom Authentication Flows**: Require custom authorization policies or MFA verification before a user can access a specific application. See the `Upstream Authentik Policy Overview <https://docs.goauthentik.io/security/policy/>`_.
