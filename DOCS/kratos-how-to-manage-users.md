The Identity Platform can be run in identity provider mode, meaning you can manage your users directly
rather than relying on third party identity providers, such as Google, Okta or Microsoft Entra ID.

Kratos is the main component of the Identity Platform responsible for identity and user management.

This guide explains common user management tasks that you can carry out in the built-in identity provider.
Users and user accounts are often referred to as identities, we'll use these terms interchangeably.

# Account management

## Create admin accounts

Admin accounts can be created by simply running a juju action in Charmed Kratos:

```
juju run kratos/0 create-admin-account email=<admin-email> password=<pwd> username=<username>
```

It is then advised to reset the newly created account's password by running the [reset-password](#reset-password) action.

## Create users

Regular users can be created in the Admin UI component. Note that this component is not deployed as part of the Identity Platform.
In order to add it to your deployment, run:
```
juju deploy identity-platform-admin-ui --channel edge --trust

juju integrate identity-platform-admin-ui:ingress traefik-k8s
juju integrate identity-platform-admin-ui:kratos-info kratos
juju integrate identity-platform-admin-ui:hydra-endpoint-info hydra
juju integrate identity-platform-admin-ui:oauth hydra
juju integrate identity-platform-admin-ui openfga-k8s
juju integrate identity-platform-admin-ui:receive-ca-cert self-signed-certificates
```

Once the application is active, you can use the Admin UI user management page to create new accounts and change their permissions.

## Get user details

You can fetch user details, such as identity traits (username, surname, phone number...) by running `get-identity` action in Kratos.

It can be done using the identity id:

```
juju run kratos/0 get-identity identity-id={identity_id}
```

Or email:

```
juju run kratos/0 get-identity email={email}
```

## Update users

Identities can be updated in Admin UI.

### Identity schemas

Account properties, such as surname or phone number, must match the identity traits defined in identity schemas.
If you want to include additional properties or mark any of them as mandatory, you can configure `identity_schemas`
in Charmed Kratos or Admin UI.

Note that email address is the account identifier by default.

## Reset password

Charmed Kratos offers a juju action to reset password of an identity by its email or id.
The password can be set to a specified value by passing `password-secret-id` as an action parameter.

To create a juju secret holding the password and grant it to kratos, run:

```shell
juju add-secret <secret-name> password=<new-password>
secret:cql684nmp25c75sflot0
juju grant-secret <secret-name> kratos
```

Then, run the action using identity id:

```shell
juju run kratos/0 reset-password identity-id={identity_id} password-secret-id=secret:cql684nmp25c75sflot0
```

Or email:

```shell
juju run kratos/0 reset-password email={email} password-secret-id=secret:cql684nmp25c75sflot0
```

If `password-secret-id` parameter is not provided, the action will return a self-service recovery code and link
to reset the password.

## Reset multi-factor authentication

Administrators can reset an identity's second authentication factor using either the identity id or email.
The type of credentials to be removed must be specified, supported values are `totp` and `lookup_secret` (commonly known as backup codes):

```
juju run kratos/0 reset-identity-mfa identity-id={identity_id} mfa-type={totp|lookup_secret}
```

Note that unless you disabled the `enforce_mfa` config option, the user will be asked
to add a time-based one-time password (TOTP) multi-factor authentication on the next login.

> See more: [Enforce multi-factor authentication](/t/15548)

## Invalidate sessions

The following action can be used to invalidate all identity sessions using either its id:

```
juju run kratos/0 invalidate-identity-sessions identity-id={identity_id}
```

Or email:

```
juju run kratos/0 invalidate-identity-sessions email={email}
```

## Delete users

You can delete existing users by their id:

```
juju run kratos/0 delete-identity identity-id={identity_id}
```

Or email:

```
juju run kratos/0 delete-identity email={email}
```

### list-oidc-accounts

This action can be used to list the OIDC accounts linked to an identity using either the identity id or email.

By id:

```shell
juju run kratos/0 list-oidc-accounts identity-id={identity_id}
```

By email:

```shell
juju run kratos/0 list-oidc-accounts email={email}
```

### unlink-oidc-account

This action can be used to unlink a user's external identity provider account from their identity
using either the identity id or email.
The credential id to be removed must be specified, you can find it with `list-oidc-accounts` action.

By id:

```shell
juju run kratos/0 unlink-oidc-account identity-id={identity_id} credential-id={oidc-identifier}
```

By email:

```shell
juju run kratos/0 unlink-oidc-account email={email} credential-id={oidc-identifier}
```

# Self-service flows
The Identity Platform implements flows that users can perform on their own instead of waiting for an administrative intervention.
See the [self-service flows reference](/t/15549) for more details.
