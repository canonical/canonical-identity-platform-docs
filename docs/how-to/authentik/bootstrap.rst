.. meta::
    :description: How to rotate administrative passwords and generate emergency recovery links in Charmed Authentik.

.. _bootstrap-authentik:

Bootstrap admin credentials and access
======================================

This guide walks you through rotating the default administrator password and generating emergency recovery links for administrative access in Charmed Authentik.

.. note::
    For initial deployment, first-time credential retrieval, and accessing the web interface, see the :doc:`Getting started with Charmed Authentik </tutorial/authentik/getting-started>` tutorial.

Rotate the administrator password
---------------------------------

After performing your initial login, you should rotate the automatically generated ``akadmin`` bootstrap password:

1. In the **Admin Interface** dashboard, select your user profile in the top-right corner.
2. Select **User Settings**.
3. Under the **Password** section, select **Change Password**.
4. Enter the current bootstrap password and specify a new secure password.
5. Select **Change Password** to commit the update.

.. warning::
    Once you change the password in the Authentik web interface, the password value previously returned by the ``get-bootstrap-admin-credentials`` Juju action will be stale.

Generate an emergency recovery link
-----------------------------------

If you lose administrative access or need to bypass standard authentication flows to repair configuration, you can generate a single-use emergency recovery link using the ``create-recovery-link`` Juju action.

Run the action on the ``authentik-server`` leader unit:

.. code-block:: bash

    juju run authentik-server/leader create-recovery-link username=akadmin duration=10

Action parameters:

* ``username`` (optional, string, default: ``akadmin``): The username of the account to recover.
* ``duration`` (optional, integer, default: ``10``): The validity period of the recovery link in minutes.

The action outputs a structured result containing:

* ``url``: The full recovery URL.
* ``path``: The relative URL path for the recovery flow.
* ``status``: Confirmation status of the link generation.

Paste the generated URL into your browser to log in directly and reset the account credentials.

Security considerations
-----------------------

* **Superuser API Token Exposure**: The ``get-bootstrap-admin-credentials`` action returns the ``bootstrap-token`` in plaintext. This token provides full administrative access via the Authentik REST API.
* **Recovery Link Scope**: The ``create-recovery-link`` action returns a URL that bypasses all standard authentication stages (including MFA).
* **Action Output Visibility**: Juju action results are visible in the Juju controller audit history to anyone with read access to the model.

Next steps
----------

* Learn :doc:`How to protect OIDC and OAuth applications <protect-oidc-applications>` to integrate downstream web services.
* Learn :doc:`How to protect LDAP applications <protect-ldap-applications>` to configure directory access.
* See :doc:`Common operational tasks <common-admin-tasks>` for scaling and performance tuning.
