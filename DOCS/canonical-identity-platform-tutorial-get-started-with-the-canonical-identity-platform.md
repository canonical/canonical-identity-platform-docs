<a id="get-started-with-the-canonical-identity-platform"></a>
# Get started with the Canonical Identity Platform

Whatever your deployment -- whether it's on Kubernetes, VMs, or across clouds -- you likely need a reliable way to manage authentication, authorization, and often single sign-on (SSO).

Traditionally, setting this up involves assembling and configuring multiple components, securing and scaling them,
and ensuring seamless integration -- a process that demands deep, domain-specific expertise.
And if it’s not done right, the consequences can be serious: from misconfigurations that break user access to security vulnerabilities that put your entire system at risk.

Enter the Canonical Identity Platform: a composable identity broker and identity provider built on trusted open source projects
from [Ory](https://www.ory.sh/open-source/), [PostgreSQL](https://www.postgresql.org/), and [Traefik Labs](https://traefik.io/).
With expert operations and configurations encoded in reusable operators called 'charms', and a powerful model-driven deployment engine in Juju,
the Identity Platform eliminates complexity and lets you focus on your users, not your infrastructure.

## What you'll need

This tutorial assumes you have
- A Juju controller (v3.1+) bootstrapped on a MicroK8s cluster with `metallb` addon or Canonical Kubernetes (v1.25.0+) that is ready to use.
See, e.g., [Juju | Set up your deployment - local testing and development](https://documentation.ubuntu.com/juju/3.6/howto/manage-your-deployment/#set-up-your-deployment-local-testing-and-development).
- The Terraform CLI. See [Hashicorp | Install Terraform](https://developer.hashicorp.com/terraform/install) (v1.5.0+).

## What you'll do
In this tutorial you will deploy the Identity Platform and use it to provide SSO to an OIDC compatible charm.

## Deploy the Identity Platform

To set up the Canonical Identity Platform, you’ll need to deploy, configure, and integrate its component charms.
We’ve taken advantage of the Terraform Provider for Juju to create a Terraform plan that automates this process for you.

Start by cloning the GitHub repository that hosts these Terraform modules:

```shell
git clone https://github.com/canonical/iam-bundle-integration.git && cd iam-bundle-integration
git checkout v0.6.0
```

Then navigate to the tutorial example and deploy the stack:
```shell
terraform -chdir=examples/tutorial init
terraform -chdir=examples/tutorial apply -auto-approve
```

<details>
<summary>Expand to view the Terraform plan</summary>

```shell
Terraform used the selected providers to generate the following execution plan. Resource actions
are indicated with the following symbols:
  + create
 <= read (data resources)

Terraform will perform the following actions:

  # juju_integration.traefik_certs will be created
  + resource "juju_integration" "traefik_certs" {
      + id    = (known after apply)
      + model = "core"

      + application {
          + endpoint = "certificates"
          + name     = "self-signed-certificates"
        }
      + application {
          + endpoint = "certificates"
          + name     = "traefik-public"
        }
    }

  # juju_model.core will be created
  + resource "juju_model" "core" {
      + credential = (known after apply)
      + id         = (known after apply)
      + name       = "core"
      + type       = (known after apply)
      + uuid       = (known after apply)
    }

  # juju_model.iam will be created
  + resource "juju_model" "iam" {
      + credential = (known after apply)
      + id         = (known after apply)
      + name       = "iam"
      + type       = (known after apply)
      + uuid       = (known after apply)
    }

  # juju_offer.ingress_offer will be created
  + resource "juju_offer" "ingress_offer" {
      + application_name = "traefik-public"
      + endpoints        = [
          + "ingress",
        ]
      + id               = (known after apply)
      + model            = "core"
      + name             = "ingress"
      + url              = (known after apply)
    }

  # juju_offer.postgresql_offer will be created
  + resource "juju_offer" "postgresql_offer" {
      + application_name = "postgresql-k8s"
      + endpoints        = [
          + "database",
        ]
      + id               = (known after apply)
      + model            = "core"
      + name             = "postgresql"
      + url              = (known after apply)
    }

  # juju_offer.send_ca_certificate_offer will be created
  + resource "juju_offer" "send_ca_certificate_offer" {
      + application_name = "self-signed-certificates"
      + endpoints        = [
          + "send-ca-cert",
        ]
      + id               = (known after apply)
      + model            = "core"
      + name             = "send-ca-cert"
      + url              = (known after apply)
    }

  # juju_offer.traefik_route_offer will be created
  + resource "juju_offer" "traefik_route_offer" {
      + application_name = "traefik-public"
      + endpoints        = [
          + "traefik-route",
        ]
      + id               = (known after apply)
      + model            = "core"
      + name             = "traefik-route"
      + url              = (known after apply)
    }

  # module.certificates.juju_application.self-signed-certificates will be created
  + resource "juju_application" "self-signed-certificates" {
      + config      = {}
      + constraints = "arch=amd64"
      + id          = (known after apply)
      + machines    = (known after apply)
      + model       = "core"
      + model_type  = (known after apply)
      + name        = "self-signed-certificates"
      + placement   = (known after apply)
      + principal   = (known after apply)
      + storage     = (known after apply)
      + trust       = false
      + units       = 1

      + charm {
          + base     = "ubuntu@22.04"
          + channel  = "latest/stable"
          + name     = "self-signed-certificates"
          + revision = (known after apply)
          + series   = (known after apply)
        }
    }

  # module.iam.data.juju_offer.ca_certificate will be read during apply
  # (config refers to values not yet known)
 <= data "juju_offer" "ca_certificate" {
      + application_name = (known after apply)
      + endpoints        = (known after apply)
      + id               = (known after apply)
      + model            = (known after apply)
      + name             = (known after apply)
      + url              = (known after apply)
    }

  # module.iam.data.juju_offer.database will be read during apply
  # (config refers to values not yet known)
 <= data "juju_offer" "database" {
      + application_name = (known after apply)
      + endpoints        = (known after apply)
      + id               = (known after apply)
      + model            = (known after apply)
      + name             = (known after apply)
      + url              = (known after apply)
    }

  # module.iam.data.juju_offer.ingress will be read during apply
  # (config refers to values not yet known)
 <= data "juju_offer" "ingress" {
      + application_name = (known after apply)
      + endpoints        = (known after apply)
      + id               = (known after apply)
      + model            = (known after apply)
      + name             = (known after apply)
      + url              = (known after apply)
    }

  # module.iam.juju_application.hydra will be created
  + resource "juju_application" "hydra" {
      + config      = {}
      + constraints = (known after apply)
      + id          = (known after apply)
      + machines    = (known after apply)
      + model       = "iam"
      + model_type  = (known after apply)
      + name        = "hydra"
      + placement   = (known after apply)
      + principal   = (known after apply)
      + storage     = (known after apply)
      + trust       = true
      + units       = 1

      + charm {
          + base     = "ubuntu@22.04"
          + channel  = "latest/stable"
          + name     = "hydra"
          + revision = (known after apply)
          + series   = (known after apply)
        }
    }

  # module.iam.juju_application.kratos will be created
  + resource "juju_application" "kratos" {
      + config      = {}
      + constraints = (known after apply)
      + id          = (known after apply)
      + machines    = (known after apply)
      + model       = "iam"
      + model_type  = (known after apply)
      + name        = "kratos"
      + placement   = (known after apply)
      + principal   = (known after apply)
      + storage     = (known after apply)
      + trust       = true
      + units       = 1

      + charm {
          + base     = "ubuntu@22.04"
          + channel  = "latest/stable"
          + name     = "kratos"
          + revision = (known after apply)
          + series   = (known after apply)
        }
    }

  # module.iam.juju_application.login_ui will be created
  + resource "juju_application" "login_ui" {
      + config      = {}
      + constraints = (known after apply)
      + id          = (known after apply)
      + machines    = (known after apply)
      + model       = "iam"
      + model_type  = (known after apply)
      + name        = "login-ui"
      + placement   = (known after apply)
      + principal   = (known after apply)
      + storage     = (known after apply)
      + trust       = true
      + units       = 1

      + charm {
          + base     = "ubuntu@22.04"
          + channel  = "latest/stable"
          + name     = "identity-platform-login-ui-operator"
          + revision = (known after apply)
          + series   = (known after apply)
        }
    }

  # module.iam.juju_integration.hydra_database will be created
  + resource "juju_integration" "hydra_database" {
      + id    = (known after apply)
      + model = "iam"

      + application {
          + endpoint = "pg-database"
          + name     = "hydra"
        }
      + application {
          + endpoint  = (known after apply)
          + offer_url = (known after apply)
        }
    }

  # module.iam.juju_integration.hydra_login_ui_ui_info will be created
  + resource "juju_integration" "hydra_login_ui_ui_info" {
      + id    = (known after apply)
      + model = "iam"

      + application {
          + endpoint = "ui-endpoint-info"
          + name     = "hydra"
        }
      + application {
          + endpoint = "ui-endpoint-info"
          + name     = "login-ui"
        }
    }

  # module.iam.juju_integration.hydra_public_ingress will be created
  + resource "juju_integration" "hydra_public_ingress" {
      + id    = (known after apply)
      + model = "iam"

      + application {
          + endpoint = "public-ingress"
          + name     = "hydra"
        }
      + application {
          + endpoint  = (known after apply)
          + offer_url = (known after apply)
        }
    }

  # module.iam.juju_integration.kratos_database will be created
  + resource "juju_integration" "kratos_database" {
      + id    = (known after apply)
      + model = "iam"

      + application {
          + endpoint = "pg-database"
          + name     = "kratos"
        }
      + application {
          + endpoint  = (known after apply)
          + offer_url = (known after apply)
        }
    }

  # module.iam.juju_integration.kratos_hydra_info will be created
  + resource "juju_integration" "kratos_hydra_info" {
      + id    = (known after apply)
      + model = "iam"

      + application {
          + endpoint = "hydra-endpoint-info"
          + name     = "hydra"
        }
      + application {
          + endpoint = "hydra-endpoint-info"
          + name     = "kratos"
        }
    }

  # module.iam.juju_integration.kratos_login_ui_info will be created
  + resource "juju_integration" "kratos_login_ui_info" {
      + id    = (known after apply)
      + model = "iam"

      + application {
          + endpoint = "kratos-info"
          + name     = "kratos"
        }
      + application {
          + endpoint = "kratos-info"
          + name     = "login-ui"
        }
    }

  # module.iam.juju_integration.kratos_login_ui_ui_info will be created
  + resource "juju_integration" "kratos_login_ui_ui_info" {
      + id    = (known after apply)
      + model = "iam"

      + application {
          + endpoint = "ui-endpoint-info"
          + name     = "kratos"
        }
      + application {
          + endpoint = "ui-endpoint-info"
          + name     = "login-ui"
        }
    }

  # module.iam.juju_integration.kratos_public_ingress will be created
  + resource "juju_integration" "kratos_public_ingress" {
      + id    = (known after apply)
      + model = "iam"

      + application {
          + endpoint = "public-ingress"
          + name     = "kratos"
        }
      + application {
          + endpoint  = (known after apply)
          + offer_url = (known after apply)
        }
    }

  # module.iam.juju_integration.login_ui_hydra_info will be created
  + resource "juju_integration" "login_ui_hydra_info" {
      + id    = (known after apply)
      + model = "iam"

      + application {
          + endpoint = "hydra-endpoint-info"
          + name     = "hydra"
        }
      + application {
          + endpoint = "hydra-endpoint-info"
          + name     = "login-ui"
        }
    }

  # module.iam.juju_integration.login_ui_public_ingress will be created
  + resource "juju_integration" "login_ui_public_ingress" {
      + id    = (known after apply)
      + model = "iam"

      + application {
          + endpoint = "ingress"
          + name     = "login-ui"
        }
      + application {
          + endpoint  = (known after apply)
          + offer_url = (known after apply)
        }
    }

  # module.iam.juju_offer.kratos_info_offer will be created
  + resource "juju_offer" "kratos_info_offer" {
      + application_name = "kratos"
      + endpoints        = [
          + "kratos-info",
        ]
      + id               = (known after apply)
      + model            = "iam"
      + name             = "kratos-info-offer"
      + url              = (known after apply)
    }

  # module.iam.juju_offer.oauth_offer will be created
  + resource "juju_offer" "oauth_offer" {
      + application_name = "hydra"
      + endpoints        = [
          + "oauth",
        ]
      + id               = (known after apply)
      + model            = "iam"
      + name             = "oauth-offer"
      + url              = (known after apply)
    }

  # module.postgresql.juju_application.k8s_postgresql will be created
  + resource "juju_application" "k8s_postgresql" {
      + config             = {}
      + constraints        = "arch=amd64"
      + id                 = (known after apply)
      + machines           = (known after apply)
      + model              = "core"
      + model_type         = (known after apply)
      + name               = "postgresql-k8s"
      + placement          = (known after apply)
      + principal          = (known after apply)
      + resources          = {}
      + storage            = (known after apply)
      + storage_directives = {
          + "pgdata" = "10G"
        }
      + trust              = true
      + units              = 1

      + charm {
          + base     = "ubuntu@22.04"
          + channel  = "14/stable"
          + name     = "postgresql-k8s"
          + revision = (known after apply)
          + series   = (known after apply)
        }
    }

  # module.traefik.juju_application.traefik will be created
  + resource "juju_application" "traefik" {
      + config             = {}
      + constraints        = "arch=amd64"
      + id                 = (known after apply)
      + machines           = (known after apply)
      + model              = "core"
      + model_type         = (known after apply)
      + name               = "traefik-public"
      + placement          = (known after apply)
      + principal          = (known after apply)
      + storage            = (known after apply)
      + storage_directives = {}
      + trust              = true
      + units              = 1

      + charm {
          + base     = (known after apply)
          + channel  = "latest/stable"
          + name     = "traefik-k8s"
          + revision = (known after apply)
          + series   = (known after apply)
        }
    }

Plan: 25 to add, 0 to change, 0 to destroy.
```

</details>

Terraform will create two Juju models (workspaces). These models are implicitly associated with your existing controller
and MicroK8s cloud and hold your deployed, configured, and integrated Identify Platform charmed applications
and their charmed dependencies.

This process will take several minutes, depending on your hardware and network speed.

The `iam` model contains all the crucial identity applications:
- [Charmed Ory Hydra](https://charmhub.io/hydra): the OAuth/OIDC server
- [Charmed Ory Kratos](https://charmhub.io/kratos): the user management and authentication component
- [Login UI operator](https://github.com/canonical/identity-platform-login-ui-operator): a middleware which routes calls between the different services and serves the login/error pages

And the `core` model contains all of their shared dependencies:
- [Charmed Postgresql](https://charmhub.io/postgresql): the SQL database of choice
- [Charmed Traefik](https://charmhub.io/traefik-k8s): which will be used for ingress
- [Self Signed Certificates](https://charmhub.io/self-signed-certificates): for managing the TLS certificates that our ingress will use.

You can switch between those models using `juju switch <iam|core>` and track the progress of your deployment by running:

```shell
juju status --watch 1s
```

This command displays the status of the installation and information about the model, like IP addresses, ports, versions etc.

Your deployment is ready when `juju status` displays the following output:

```
Model  Controller     Cloud/Region        Version  SLA          Timestamp
iam    my-controller  microk8s/localhost  3.6.7    unsupported  15:54:43+02:00

SAAS        Status   Store  URL
ingress     active   local  admin/core.ingress
postgresql  active   local  admin/core.postgresql

App       Version  Status   Scale  Charm                                Channel        Rev  Address         Exposed  Message
hydra     v2.3.0   active       1  hydra                                latest/stable  362  10.152.183.80   no       
kratos    v1.3.1   active       1  kratos                               latest/stable  527  10.152.183.36   no       
login-ui  0.20.0   active       1  identity-platform-login-ui-operator  latest/stable  166  10.152.183.192  no       

Unit         Workload  Agent  Address      Ports  Message
hydra/0*     active    idle   10.1.57.88          
kratos/0*    active    idle   10.1.57.102         
login-ui/0*  active    idle   10.1.57.92          

Offer              Application  Charm   Rev  Connected  Endpoint     Interface    Role
kratos-info-offer  kratos       kratos  527  0/0        kratos-info  kratos_info  provider
oauth-offer        hydra        hydra   362  0/0        oauth        oauth        provider
```

## Set up the identity provider

So far, you’ve deployed the Canonical Identity Platform core components, but you still need a way to log in.

You have two choices: create accounts right away using the built-in identity provider,
or connect an external provider (e.g. Google, Entra ID, GitHub...).
You can choose to use just one of these options -- either local or external -- or combine them to let users sign in through both.

### Use the built-in identity provider

The Identity Platform comes with a built-in identity provider,
allowing you to manage user accounts internally without relying on a third-party service.
You can also let your users authenticate both with external providers and the local one.

The local identity provider feature is enabled by default in Charmed Kratos.

At this stage, you can create your personal admin account:

```shell
juju run kratos/0 create-admin-account email=<your-email> username=<username>
```

After the account is created, use the provided password reset link to set your password and complete the setup.

Congratulations, you're all set to use the built-in identity provider to sign in.

### Connect the Canonical Identity Platform to an external identity provider

> This option assumes that you have a GitHub account.

To use an external identity provider alongside the local one,
you can leverage [Kratos External IdP Integrator](https://charmhub.io/kratos-external-idp-integrator).

In this step we are going to:
- create a GitHub confidential client and configure it with Kratos redirect URL
- extend our Identity Platform deployment by adding the integrator charm
- connect the Identity Platform with GitHub.

First, we need to know our deployment’s `redirect_uri`. Get the Kratos base URL provided by the ingress integration:

```shell
juju show-unit kratos/0 --endpoint public-ingress
```

It will look similar to `https://10.64.140.43/iam-kratos`.

The `redirect_uri` should be: `<kratos-base-url>/self-service/methods/oidc/callback/github`.
We will use this callback URL to configure your client in GitHub.

Next, go to GitHub developer settings and [register a new OAuth application](https://github.com/settings/applications/new).
Provide the Kratos redirect URL as the authorization callback URL. Take note of your newly generated client ID and secret.

> See more: [GitHub | Create an OAuth App](https://docs.github.com/en/apps/oauth-apps/building-oauth-apps/creating-an-oauth-app)

We can now configure the Identity Platform with the GitHub client credentials.
Start by creating a `vars.tfvars` file in the `examples/tutorial` directory
where you specify the GitHub configuration and enable the integrator charm:

```
enable_kratos_external_idp_integrator = true

idp_provider_config = {
  client_id   = "<client-id>"
  provider    = "github"
  provider_id = "github"
}

idp_provider_credentials = {
  client_secret = "<client-secret>"
}
```

Finally, re-apply the module to deploy Kratos External IdP Integrator:

```shell
terraform -chdir=examples/tutorial apply -var-file="./vars.tfvars"
```

> See more: [Charmhub | Kratos External IdP Integrator | Integrate with external identity providers](https://charmhub.io/kratos-external-idp-integrator/docs/how-to/integrate-external-identity-provider)

This integrator charm enables Charmed Kratos to work with an external identity provider.

After a while, the `kratos-external-idp-integrator` status will change to active.
This means that GitHub has been added as a Kratos sign-in provider.

Congratulations, your deployment now uses GitHub as your external identity provider.

## Use the Identity Platform to provide SSO

Now that our identity broker is ready and the Platform can also serve as an identity provider, we can provide SSO to an application.
For this purpose we are going to use Grafana. To deploy [Charmed Grafana](https://charmhub.io/grafana-k8s), run:

```
juju switch iam
juju deploy grafana-k8s
```

This command deploys Charmed Grafana in the same model as the Identity Platform.

The newly deployed Grafana requires ingress in order to properly integrate with the Identity Platform.
We can leverage the existing Traefik instance for this purpose.
You can integrate Charmed Grafana with Traefik by running:

```shell
juju integrate grafana-k8s:ingress admin/core.traefik-route
```

Now, you can connect Grafana with the Identity Platform. To do this we need to integrate Grafana with the OIDC server, Hydra:

```shell
juju integrate grafana-k8s:oauth hydra
```

This command registers Grafana as an OIDC client in Hydra and configures Grafana to use the Canonical Identity Platform as an authentication provider.

Finally, integrate Grafana with the `self-signed-certificates` charmed application:

```shell
juju integrate grafana-k8s:receive-ca-cert admin/core.send-ca-cert
```

Once all applications are ready, log in to Grafana through GitHub or with your admin user created in the previous step.

## Log in to Grafana through the Identity Platform

To access the Grafana dashboard:
1. Get the traefik-public IP:

    ```shell
    juju show-unit grafana-k8s/0 | yq e '.["grafana-k8s/0"].relation-info[] | select(.endpoint=="ingress") | .application-data.external_host' -
    ```

2. Use the traefik-public IP to access the Grafana dashboard at `http://<traefik_public_ip>/iam-grafana-k8s`.

Upon navigating to the URL you will be presented with the following login screen:

![Grafana sign-in page]( https://raw.githubusercontent.com/canonical/canonical-identity-platform-docs/main/Diagram_sources/deploy_iam_bundle_1.png "Grafana Login UI")

Click on **Sign in with external identity provider** and, after you trust the self-signed TLS cert, you will be redirected to the login page where you can select to authenticate with GitHub or your local user:

![Identity Platform sign-in page](https://raw.githubusercontent.com/canonical/canonical-identity-platform-docs/main/Diagram_sources/identity_platform_sign_in_page.png "IAM Login UI")


After a successful login, you will be redirected back to Grafana.
Congratulations, your GitHub users and locally created accounts can now access your Grafana dashboards!

> See more: [Charmhub | Identity Plaform](https://charmhub.io/identity-platform)

## Tear things down

To clean up the resources created during this tutorial, run:

```shell
terraform -chdir=examples/tutorial destroy
```

This will remove all charmed applications and destroy the Juju models that were provisioned as part of the deployment.
