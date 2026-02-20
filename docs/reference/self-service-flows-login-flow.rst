.. _self-service-flows-login-flow:

Login Flow
==========

The Login Flow begins when a user is trying to access an application (which we'll interchangeably refer to as an OAuth client).
The application initially executes its own logic for checking whether the user is already logged in (has a valid session cookie).
If no valid session is found, the application presents the user with an authentication page, where the Sign in with Canonical Identity button is present.

Clicking that button starts the (simplified) login flow described below:

.. image:: https://raw.githubusercontent.com/canonical/canonical-identity-platform-docs/main/Diagram_sources/login-flow.png
   :alt: Simplified Login Flow
   :align: center

.. dropdown:: See detailed diagram

    .. image:: https://raw.githubusercontent.com/canonical/canonical-identity-platform-docs/main/Diagram_sources/login-flow-detailed.png
       :alt: Detailed Login Flow
       :align: center

The flow begins with a call to Hydra’s public API: the OAuth client sends an authorization request, which is validated by Hydra. If the application client id and secret were found valid, Hydra will initially check if there are valid sessions.

If none are present, Hydra creates a CSRF challenge and redirects the user to Identity Platform Login UI. The Login UI handles the login challenge and initiates a self-service login flow in Charmed Kratos. The components exchange flow details and show the user a list of configured identity providers to complete the authentication challenge (e.g. Azure AD, Google, GitHub, ...).

When the user chooses an identity provider, Kratos creates a continuity parameter and redirects the browser to the external system with 422 status in order to complete the login.

If the authentication with the external identity provider was successful, Kratos issues a session cookie along with a login verifier parameter, which is sent over to Hydra.

Hydra accepts the login request if the parameters match. The consent is implicit, meaning that the OAuth server skips the consent screen as the client is trusted. It then transmits the session cookie and authorization code to the OAuth client.

Finally, the OAuth client exchanges the authorization code for an access token, an id token, and possibly a refresh token, depending on the client’s scope. The browser gets forwarded to the OAuth client’s redirect URL, which calls ``/userinfo`` endpoint to retrieve the current session’s user details, finally allowing the user to access the requested resource.
