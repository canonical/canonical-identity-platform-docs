.. meta::
    :description: How to configure Charmed Authentik to integrate upstream identity providers, Active Directory, and SAML or OIDC sources.

.. _integrate-upstream-providers-authentik:

Integrate upstream identity providers
=====================================

This guide describes how to configure Charmed Authentik to delegate authentication, federate user logins, or synchronize directory attributes with upstream Identity Providers—specifically corporate **Active Directory / LDAP** servers or external **SAML / OIDC** federated identity systems.

Prerequisites
-------------

This guide assumes you have an active Charmed Authentik deployment matching the topology established in the :doc:`Getting started tutorial </authentik/tutorial/getting-started>`. Specifically, you should have:

* An active ``authentik-server`` deployment integrated with its database and certificates.
* Access to the Authentik admin dashboard via your administrator credentials (``akadmin``).
* A reachable upstream identity provider (OIDC social provider or LDAP server) accessible from your Kubernetes cluster.

Upstream SAML and OIDC social login and federation
--------------------------------------------------

By configuring upstream federation, you can allow users to log in to your Authentik domain using external credentials (e.g. Google Workspace, Microsoft Entra ID, Okta, or Keycloak).

These settings are managed through the Authentik Admin Interface:

Step-by-step configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~

1. **Access the Admin Panel**: Log in to your Charmed Authentik instance using your administrator credentials (``akadmin``).
2. **Create a Social/Federated Source**:

   * In the sidebar, navigate to **Directory** → **Federation & Social Login**.
   * Select **Create** and select your target provider type (e.g. **OpenID Connect Source**, **SAML Source**, or social templates like **Google** or **Microsoft**).

3. **Configure Upstream Metadata**:

   * Enter a user-facing **Name** (e.g. ``Google``).
   * Enter your upstream Client ID and Client Secret.
   * For standard OIDC, supply the **Authorization URL**, **Token URL**, and **User-info URL** (or use the automatic Discovery/Issuer URL).

4. **Define Login Flows**:

   * Bind the upstream source to an enrollment flow (e.g., ``default-source-enrollment``) to automatically provision user profiles in Authentik's internal database upon successful social authentication.

5. **Add Source to Identification Stage (Required for Visibility)**:

   * To make the federated login button appear on your main portal login screen, navigate to **Flows and Stages** → **Stages**.
   * Locate and edit the **default-authentication-identification** stage.
   * In the **Sources** field, select your newly created social/federated source (e.g., ``Google``) and save the changes.

6. **Verify Login Option**:

   * Log out of your Authentik session.
   * The default login portal will now display your new federated login button (e.g., **Continue with Google**).

Upstream LDAP and Active Directory synchronization
--------------------------------------------------

To sync existing enterprise user accounts and organizational groups from an on-premises LDAP server or Active Directory domain, you can configure an upstream **LDAP Source**.

Authentik runs background synchronization tasks to pull and reconcile directory states via the ``authentik-worker`` charm.

Step-by-step configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~

1. **Access the Admin Panel**: Log in to the Authentik admin dashboard.
2. **Create the Upstream LDAP Source**:

   * Navigate to **Directory** → **Sources**.
   * Select **Create** and select **LDAP Source**.

3. **Configure Connection Parameters**:

   * **Name / Slug**: Enter a unique identifier (e.g. ``Active-Directory``).
   * **Server URI**: Enter the directory path (e.g. ``ldaps://ad.example.com:636``).
   * **Bind DN / Bind Password**: Supply the service account credentials used to execute read queries against your Active Directory server (e.g. ``cn=authentik-sync,cn=Users,dc=ad,dc=example,dc=com``).
   * **Base DN**: Specify the search scope under which users and groups reside (e.g. ``dc=ad,dc=example,dc=com``).

4. **Map Attributes and Synced Groups**:

   * Select **Sync Users** and **Sync Groups**.
   * (Optional) Configure property mapping templates to translate specific Active Directory fields (such as ``sAMAccountName`` or ``mail``) into Authentik properties.

5. **Trigger Synchronization**:

   * Once saved, select the newly created LDAP Source.
   * Under **Status**, select **Run Sync** to trigger an immediate reconciliation.
   * View progress logs in real-time. Background synchronization tasks are processed automatically by the ``authentik-worker`` charm.

Upstream documentation references
---------------------------------

For advanced federation and directory synchronization mapping rules, refer to the official upstream documentation:

* **Active Directory Sync Guide**: Detailed attribute mapping patterns and group-matching filters can be found in the `Upstream Active Directory Integration Guide <https://docs.goauthentik.io/users-sources/sources/directory-sync/active-directory/>`_.
* **Upstream Sources Overview**: Refer to the `Upstream Sources Reference <https://docs.goauthentik.io/users-sources/sources/>`_ for protocol listings and social sync configuration.
