.. meta::
    :description: How to manage users, groups, and directory memberships in Charmed Authentik.

.. _manage-users-and-groups-authentik:

Manage users and groups
=======================

This guide walks you through identity directory operations inside Charmed Authentik, explaining core identity concepts and administrative workflows for user creation, group creation, and membership assignment.

Core identity concepts
----------------------

Charmed Authentik serves as your unified identity source of truth. Key identity concepts include:

* **Users**: Identity entities representing real-world users, system operators, or programmatic consumers. See the `Upstream User Overview <https://docs.goauthentik.io/users-sources/user/>`_.
* **Groups**: Logical collections of users used to streamline policy application, role-based access control (RBAC), and user attribute mapping. See the `Upstream Group Overview <https://docs.goauthentik.io/users-sources/groups/>`_.
* **Service Accounts**: Programmatic user accounts designed to authenticate background processes and integration services without interactive login prompts.

Basic identity management workflows
-----------------------------------

Directory structures and memberships are managed from the Authentik Admin Interface under the **Directory** section.

Create a new user
~~~~~~~~~~~~~~~~~

1. Log in to the Authentik dashboard and navigate to the **Admin Interface**.
2. In the left-hand navigation sidebar, expand **Directory** and select **Users**.
3. Select **Create** at the top of the pane.
4. Fill in the user profile:

   * **Username**: The unique login identifier (e.g., ``jdoe``).
   * **Name**: The display name (e.g., ``John Doe``).
   * **Email**: The user's corporate email address.

5. Select **Create** to register the account.
6. **Set Password**: By default, new users do not have a password set. To assign one:

   * Select the newly created user in the list.
   * Go to the **Actions** menu and select **Set Password**.
   * Enter a password or select **Generate** and save.

Create a new group
~~~~~~~~~~~~~~~~~~

1. Expand **Directory** in the sidebar and select **Groups**.
2. Select **Create** at the top of the pane.
3. Fill in the group configuration:

   * **Name**: Specify a descriptive name (e.g., ``Platform Engineers``).
   * **Parent Group**: (Optional) Assign to a parent group to establish hierarchical access.

4. Select **Create** to save.

Assign users to a group
~~~~~~~~~~~~~~~~~~~~~~~

You can manage group membership either from the Group view or the User view:

From the Group view (recommended for bulk assignments)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

1. Under **Directory** → **Groups**, select the group you wish to populate.
2. Navigate to the **Members** tab.
3. Select **Add existing user**.
4. Select the user(s) you wish to add from the modal list and select **Add**.

From the User view
^^^^^^^^^^^^^^^^^^

1. Under **Directory** → **Users**, select the specific user.
2. Navigate to the **Groups** tab.
3. Select **Add to group**.
4. Select the target group and select **Add**.

Next steps
----------

* Learn :doc:`How to protect OIDC and OAuth applications <protect-oidc-applications>` to enable SSO integrations.
* Learn :doc:`How to protect LDAP applications <protect-ldap-applications>` to integrate directory consumers.
* Refer to the `Upstream Authentik Identity Docs <https://docs.goauthentik.io/users-sources/user/>`_ for details on advanced attributes, custom user paths, and policy-driven provisioning.
