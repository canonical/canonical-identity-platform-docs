The Charmed Ory Kratos component of the Identity Platform is an identity and user management system 
that allows you to manage users locally rather than relying on third party identity providers.

Alongside tasks that system administrators can perform, it also allows users to carry out certain actions on their own.
We will refer to those operations as self-service flows. This document will list them and explain their mechanisms.

## Account recovery

Account recovery flow allows registered users to regain access to their account if they forgot their password.
It can be initiated by clicking the "Reset password" link on the login screen.

![Alt]( https://raw.githubusercontent.com/canonical/canonical-identity-platform-docs/main/Diagram_sources/identity_platform_sign_in_page.png "IAM Login UI")

If an SMTP server was configured, the user will receive an email containing a recovery link and code that need to be used
in order to recover access to the account.

## Password reset

Once logged in, users can change their passwords.

![Alt]( https://raw.githubusercontent.com/canonical/canonical-identity-platform-docs/main/Diagram_sources/idp_reset_password.png "Reset password")

## Profile and account management

Users can also add or reset other authentication methods:
- (un)link a time-based one-time password (TOTP) authenticator app
- generate backup codes as a fallback 2fa method
- add a passwordless login method
- link social sign in accounts.
