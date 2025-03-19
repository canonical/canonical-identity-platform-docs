The Canonical Identity Platform is a composable identity broker and identity provider based on open source products from [Ory](https://www.ory.sh/open-source/), [PostgreSQL](https://www.postgresql.org/) and [Traefik Labs](https://traefik.io/).

> See more: [Charmhub | Identity Plaform](https://charmhub.io/identity-platform)

The core components of the Canonical Identity Platform are grouped in a Juju Bundle. In this tutorial you will deploy the Identity Platform juju Bundle and use it to provide SSO to an OIDC compatible charm.

# Requirements

This tutorial assumes you have
- A Juju controller (v3.1+) bootstrapped on a MicroK8s cluster that is ready to use. See, e.g., [Charm SDK | Tutorial > Set up Juju](https://juju.is/docs/juju/get-started-with-juju#heading--prepare-your-cloud).

# Deploy the Identity Platform bundle

For the Identity Platform bundle deployment to go smoothly, make sure the `metallb` MicroK8s addon is enabled.

```
microk8s enable metallb:10.64.140.43-10.64.140.49
```

Create a dedicated model for the Identity Platform bundle. To do that, run:

```
juju add-model iam
```

To deploy the Identity Platform bundle, run:

```
juju deploy identity-platform --trust --channel 0.3/edge
```

The Juju controller will now fetch the Identity Platform bundle from Charmhub and begin deploying it on the MicroK8s cloud.
This process will take several minutes, depending on your hardware and network speed. It will install the following charmed applications:
- [Charmed Postgresql](https://charmhub.io/postgresql) - the SQL database of choice
- [Charmed Ory Hydra](https://charmhub.io/hydra) - the solution Oauth/OIDC server
- [Charmed Ory Kratos](https://charmhub.io/kratos) - the user management and authentication component
- [Charmed Traefik](https://charmhub.io/traefik-k8s) - which will be used for ingress
- [Login UI operator](https://github.com/canonical/identity-platform-login-ui-operator) - a middleware which routes calls between the different services and serves the login/error pages

The bundle will also deploy the following helper charms:
- [Kratos External IdP Integrator](https://charmhub.io/kratos-external-idp-integrator) - an integrator charm that will be used later in this tutorial to configure an external identity provider
- [Self Signed Certificates](https://charmhub.io/self-signed-certificates), for managing the TLS certificates that our ingress will use

You can track the progress by running:

```
juju status --watch 1s
```

This command displays the status of the installation and information about the model, like IP addresses, ports, versions etc.

When ```juju status``` displays the following output it means that the bundle is ready:

```
Model       Controller     Cloud/Region        Version  SLA          Timestamp
iam         my-controller  microk8s/localhost  3.4.5    unsupported  12:02:03Z

App                                  Version  Status   Scale  Charm                                Channel        Rev  Address         Exposed  Message
hydra                                v2.3.0   active       1  hydra                                               304  10.152.183.187  no
identity-platform-login-ui-operator  0.17.0   active       1  identity-platform-login-ui-operator  latest/edge    117  10.152.183.171  no
kratos                               v1.1.0   active       1  kratos                               latest/edge    470  10.152.183.175  no
kratos-external-idp-integrator                waiting      1  kratos-external-idp-integrator       latest/edge    245  10.152.183.200  no       Provider is ready
postgresql-k8s                       14.11    active       1  postgresql-k8s                       14/stable      281  10.152.183.180  no
self-signed-certificates                      active       1  self-signed-certificates             latest/stable  155  10.152.183.94   no
traefik-admin                        v2.11.0  active       1  traefik-k8s                          latest/stable  194  10.152.183.201  no       Serving at 10.64.140.44
traefik-public                       v2.11.0  active       1  traefik-k8s                          latest/stable  194  10.152.183.181  no       Serving at 10.64.140.43

Unit                                    Workload  Agent  Address      Ports  Message
hydra/0*                                active    idle   10.1.24.150
identity-platform-login-ui-operator/0*  active    idle   10.1.24.148
kratos-external-idp-integrator/0        active    idle   10.1.24.171         Invalid configuration: Missing required configuration 'issuer_url' for provider 'generic'
kratos/0*                               active    idle   10.1.24.161
postgresql-k8s/0*                       active    idle   10.1.24.184         Primary
self-signed-certificates/0*             active    idle   10.1.24.137
traefik-admin/0*                        active    idle   10.1.24.177         Serving at 10.64.140.44
traefik-public/0*                       active    idle   10.1.24.141         Serving at 10.64.140.43
```

The `kratos-external-idp-integrator` is in a blocked state. In the next step we will configure it with an identity provider so our Identity Platform becomes fully operational.

# Connect the Canonical Identity Platform with an external Identity Provider

> This part of the tutorial assumes that:
> - you are in possession of a Microsoft Entra ID tenant and a user with sufficient permissions to provision a client application
> - you have a client registered in Entra ID; you need to know the `client_id`, `client_secret` and `tenant_id`.

> See more: [Microsoft | Register a new application](https://learn.microsoft.com/en-us/azure/healthcare-apis/register-application#register-a-new-application), [Microsoft | Register a new application > Certificates & Secrets](https://learn.microsoft.com/en-us/azure/healthcare-apis/register-application#certificates--secrets) and [Microsoft | How to find your Microsoft Entra tenant ID](https://learn.microsoft.com/en-us/entra/fundamentals/how-to-find-tenant)


[note]

If you don't have a Microsoft Entra ID tenant, you can try out the Identity Platform in identity provider mode.
To do that, proceed to the next tutorial section.

[/note]

In this tutorial we are going to connect the Identity Platform with Microsoft Entra ID.

First, we need to inform Entra ID of our deployment’s `redirect_uri`. Get the Kratos base URL:

```
juju run traefik-public/0 show-proxied-endpoints | yq '.proxied-endpoints' | jq '.kratos.url'
```

The `redirect_uri` should be: `<kratos-base-url>/self-service/methods/oidc/callback/microsoft`. Use this to configure your client in Entra ID.

> See more: [Microsoft | Redirect URI](https://learn.microsoft.com/en-us/entra/identity-platform/reply-url)

Now use the Entra ID client credentials to configure the Kratos external IDP integrator charm:

```
juju config kratos-external-idp-integrator \
  provider=microsoft \
  provider_id=microsoft \
  client_id=<client_id> \
  client_secret=<client_secret> \
  microsoft_tenant_id=<tenant_id>
```

> See more: [Charmhub | Kratos External Idp Integrator](https://charmhub.io/kratos-external-idp-integrator)

After a while, the `kratos-external-idp-integrator` status will change to active. This means that Entra ID has been added as a Kratos sign-in provider.
Congratulations, your deployment now uses Microsoft Entra ID as your external identity provider.

# Use the built-in Identity Provider

The Identity Platform comes with a built-in identity provider that you can use to manage users internally
instead of delegating it to a third-party identity provider.
You can also let your users authenticate both with external providers and the local one.

The local identity provider feature is enabled by default in Identity Platform version `0.3`.

For the purpose of this tutorial, we will create an admin user:

```
juju run kratos/0 create-admin-account email=test@example.com password=test username=admin
```

Congratulations, you can now use the built-in identity provider to sign in.

# Use Identity Platform to provide SSO

Now that our Identity Broker is ready, we can use it to provide SSO to an application. For this purpose we are going to use Grafana. To deploy [Charmed Grafana](https://charmhub.io/grafana-k8s), run:

```
juju deploy grafana-k8s
```

This command deploys Charmed Grafana in the same model as the Identity Platform bundle.

The newly deployed Grafana requires ingress in order to properly integrate with the Identity Platform. We can use the existing Traefik instance, that was deployed together with the bundle, to get that.
You can integrate Charmed Grafana with Traefik by running:

```
juju integrate grafana-k8s:ingress traefik-public
```

Now, you can connect Grafana with the bundle. To do this we need to integrate Grafana with the bundle’s OIDC server, Hydra:

```
juju integrate grafana-k8s:oauth hydra
```

This command registers Grafana as an OIDC client in Hydra and configures Grafana to use the Canonical Identity Platform as an authentication provider.

Finally, integrate Grafana with self-signed-certificates operator:
```
juju integrate grafana-k8s:receive-ca-cert self-signed-certificates:send-ca-cert
```
Once all applications are ready, log in to Grafana through Entra ID or with your admin user created in the previous step.

# Validate the SSO integration

To access the Grafana dashboard:
1. Get the traefik-public IP by running the following command:

```
juju status --format json | yq '.applications.traefik-public.address'
```
2. Use the traefik-public IP to access the Grafana dashboard at `http://<traefik_public_ip>/iam-grafana-k8s`

Upon navigating to the URL you will be presented with the following login screen:

![Alt]( https://raw.githubusercontent.com/canonical/canonical-identity-platform-docs/main/Diagram_sources/deploy_iam_bundle_1.png "Grafana Login UI")

Click on *Sign in with external identity provider* and, after you trust the self-signed TLS cert, you will be redirected to the login page where you can select to authenticate with Microsoft Entra ID or your local user:

![Alt]( https://raw.githubusercontent.com/canonical/canonical-identity-platform-docs/main/Diagram_sources/identity_platform_sign_in_page.png "IAM Login UI")

After a successful login, you will be redirected back to Grafana.
Congratulations, your Microsoft Entra ID users and locally created accounts can now access your Grafana dashboards!
