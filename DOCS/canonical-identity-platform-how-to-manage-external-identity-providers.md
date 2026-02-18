The Identity Platform comes with a built-in identity and user management system, but can also act as an identity broker. This means it is able to rely on external identity providers to authenticate users and manage user attributes. This document demonstrates how to integrate with external providers.

[note]

This guide explains how to integrate the Identity Platform with Google, Microsoft Entra ID and GitHub.
See the full list of identity providers you can integrate with [here](https://www.ory.sh/docs/kratos/social-signin/generic).

[/note]

## Add an external identity provider

To integrate the Identity Platform with an external Identity Provider, you need to register a client for Kratos in the Identity Provider and then provide the client credentials to Kratos.

### Register the client

In this section we are going to register an OAuth2 client that Kratos can use to authenticate users. If you have already registered a client and configured its redirect URI, you can just skip to the next section.

Each external Identity Provider has a different flow for registering a client. A list with instructions for some of the most common Providers can be seen below, if you can’t find your Provider below refer to their documentation.

You will need to provide the client’s `redirect_uri` to the provider, which is the URL to which the user will be redirected to after they log in. You don’t need to provide it on registration time though.

You can either calculate the URI yourself or get it after you provide the client credentials to Kratos.

The `redirect_uri` will be:

```https://<kratos-public-url>/self-service/methods/oidc/callback/<provider-id>```

To get the `kratos-public-url`, run:

```juju show-unit kratos/leader --endpoint public-route```

The `provider-id` can be anything you want. Every provider that is registered with Kratos needs to have a different `provider-id`. If you don’t provide a `provider-id`, one will be auto-generated.

After registering the provider you need to have the following information: `client_id`, `client_secret`.

#### Microsoft Entra ID

You will need to create a confidential client in Azure AD and retrieve the client_id of the client.

> See more: [Microsoft | Azure > Register a new application](https://learn.microsoft.com/en-us/azure/healthcare-apis/register-application#register-a-new-application)

Once the client is registered, create a secret.

> See more: [Microsoft | Azure Register a new application > Certificates & Secrets](https://learn.microsoft.com/en-us/azure/healthcare-apis/register-application#certificates--secrets)


You then need to retrieve the tenant_id.

> See more: [Microsoft | Azure > How to find a tenant](https://learn.microsoft.com/en-us/azure/active-directory/fundamentals/how-to-find-tenant)

#### Google

To create a confidential client in Google follow the instructions found in the [Google documentation](https://developers.google.com/identity/protocols/oauth2#1.-obtain-oauth-2.0-credentials-from-the-dynamic_data.setvar.console_name-.).

#### GitHub

To create a confidential client in GitHub, go to developer settings and [register a new GitHub application](https://github.com/settings/applications/new),
providing Kratos redirect url as the authorization callback url.

Next, generate a client secret and make sure to copy it along with the client id
as it will be required in the next step.

### Provide the Client Credentials to Kratos

Now that we have registered a client we need to provide the client credentials to Kratos. For this we are going to use an integrator charm, the purpose of this charm is to provide configuration to Kratos.

First we need to deploy the charm and integrate it with kratos by running:

```
juju deploy kratos-external-idp-integrator
juju integrate kratos kratos-external-idp-integrator
```

[note]

Each external identity provider you want to integrate with the Identity Platform
will require a separate instance of the integrator charm.

[/note]

Then we need to configure the integrator charm. Depending on the provider that we use, a different set of configurations is needed. A list of instructions for some of the most common providers can be seen below. Please refer to the [integrator charm](https://charmhub.io/kratos-external-idp-integrator) and the [Kratos documentation](https://www.ory.sh/docs/kratos/social-signin/overview) for further details.

Once you have configured the provider you will be able to choose to login in with that provider in the platform’s login page.

#### Microsoft Entra ID

If your provider is Entra ID the following configuration is needed:

```
juju config kratos-external-idp-integrator \
  provider=microsoft \
  client_id=<client_id> \
  client_secret=<client_secret> \
  microsoft_tenant_id=<tenant_id>
```

#### Google

If your provider is Google the following configuration is needed:

```
juju config kratos-external-idp-integrator \
  provider=google \
  client_id=<client_id> \
  client_secret=<client_secret>
```

#### GitHub

If your provider is GitHub the following configuration is needed:

```
juju config kratos-external-idp-integrator \
  provider=github \
  client_id=<client-id> \
  client_secret=<client-secret> \
  provider_id=github \
  scope=user:email
```

### Choose Provider ID

You can now also choose a `provider-id`, if you wish to, by running:

```
juju config kratos-external-idp-integrator provider_id=<provider-id>
```

### Get the redirect_uri

You can run `juju status` to inspect the status of the charm. Once the charm becomes active, you can get the `redirect_uri` of the client by running:

```
juju run kratos-external-idp-integrator/0 get-redirect-uri
```

## Remove External Provider

To remove an external provider from Kratos all you need to is remove the relation to the integrator by running:

```
juju remove-integration kratos kratos-external-idp-integrator
```
