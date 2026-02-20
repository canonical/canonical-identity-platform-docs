.. _link-social-accounts:

Link Accounts
=============

Account linking allows you to connect your social sign-in accounts (e.g., Google, GitHub)
to your Identity Platform account as an additional authentication method.
This enables seamless sign-in using multiple identity providers.

Prerequisites
-------------

Account linking is available only if the **built-in identity provider** is enabled in the Identity Platform.
For more details, refer to :doc:`this guide </how-to/use-local-identity-provider>`.

Linking a Social Sign-In Account
--------------------------------

1. Go to the **Connected accounts** page in your user settings.
2. Select **Connect** for the social provider you want to add.
3. Unless you're already signed in, you'll need to authenticate with the external identity provider to confirm the linking action.
4. After successful authentication, the social account will appear as connected.

.. image:: https://raw.githubusercontent.com/canonical/canonical-identity-platform-docs/main/Diagram_sources/manage_connected_accounts.png
   :alt: Manage connected accounts
   :align: center

Unlinking a Social Sign-In Account
----------------------------------

You can unlink a linked account directly from the **Connected accounts** page in a self-service manner.
You can also unlink accounts using a **Juju action** in Charmed Kratos.
For detailed steps, see the :doc:`How to manage users guide </how-to/manage-users>`.

Limitations
-----------

* You can only link accounts from available identity providers that you haven't previously registered with.
* Each social sign-in account can be linked to only one user (identity) in Kratos.
