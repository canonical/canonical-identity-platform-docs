# How to Link Accounts in Identity Platform

Account linking allows you to connect your social sign-in accounts (e.g., Google, GitHub)
to your Identity Platform account as an additional authentication method.
This enables seamless sign-in using multiple identity providers.

## Prerequisites

Account linking is available only if the **built-in identity provider** is enabled in Identity Platform.  
For more details, refer to [this guide](/t/15548).

## Linking a Social Sign-In Account

1. Go to the **Connected accounts** page in your user settings.  
2. Select **Connect** for the social provider you want to add.  
3. Unless you're already signed in, you'll need to authenticate with the external identity provider to confirm the linking action.
4. After successful authentication, the social account will appear as connected.

![Alt]( https://raw.githubusercontent.com/canonical/canonical-identity-platform-docs/main/Diagram_sources/manage_connected_accounts.png "Manage connected accounts")

## Unlinking a Social Sign-In Account

You can unlink a linked account directly from the **Connected accounts** page in a self-service manner.
You can also unlink accounts using a **Juju action** in Charmed Kratos.
For detailed steps, see the [How to manage users guide](/t/15547).

## Limitations

- You can only link accounts from available identity providers that you haven't previously registered with.  
- Each social sign-in account can be linked to only one user (identity) in Kratos.  
