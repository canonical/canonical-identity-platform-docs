.. _canonical-identity-platform-security-explained:

Security in Canonical Identity Platform
=======================================

Security is a paramount concern in any identity management system, especially in an enterprise-grade platform like Canonical's Identity Platform. This document provides an overview of security within the Canonical Identity Platform, discussing common security risks, built-in protections, cryptographic approaches, and best practices. This guide is designed to help orient your thinking about security and provide guides to help you secure your identity infrastructure.

Understanding Common Security Risks
-----------------------------------

The first step in securing any platform is understanding the risks it faces. When dealing with identity systems, the primary risks include:

* **Unauthorised Access:** Attackers may attempt to gain unauthorised access to the system by exploiting vulnerabilities or obtaining credentials through phishing or brute force attacks.
* **Data Breaches:** Compromised credentials or poorly protected data can lead to breaches of sensitive user information.
* **Insider Threats:** Trusted insiders with access to sensitive data may intentionally or unintentionally misuse their access.
* :woke-ignore:`Man-in-the-Middle` **(MITM) Attacks:** Attackers may intercept communications between users and the platform, stealing data or credentials.
* **Misconfigurations:** Improper configurations in access control, authentication mechanisms, or storage can expose the system to vulnerabilities.

To mitigate these risks, it is essential to employ both proactive and reactive security measures.

Built-in Security Features
--------------------------

Canonical Identity Platform comes with several built-in security features to protect your identity infrastructure.

Open source
~~~~~~~~~~~

Identity Platform is a composable solution based on open source products from `Ory <https://www.ory.sh/open-source/>`_, `PostgreSQL <https://www.postgresql.org/>`_, `OpenFGA <https://openfga.dev/>`_ and `Traefik Labs <https://traefik.io/>`_. It makes it possible to review the source code and to use the power of community in identifying and fixing vulnerabilities.

Authentication Mechanisms
~~~~~~~~~~~~~~~~~~~~~~~~~

The platform supports multiple secure authentication methods, including:

* **Multi-Factor Authentication (MFA):** Adds an additional layer of security by requiring users to verify their identity through multiple methods.
* **OAuth 2.0 and OpenID Connect:** These are industry-standard protocols for secure authorization and authentication. They allow secure token-based authentication, which reduces the risk of credential compromise.

Access Control
~~~~~~~~~~~~~~

Granular `Relationship-based access control <https://en.wikipedia.org/wiki/Relationship-based_access_control>`_ (ReBAC) allows administrators to assign specific permissions based on relationships between subjects and resources, ensuring that users only have the access necessary for their responsibilities (allowing for the application of the least privilege principle).

Identity management
~~~~~~~~~~~~~~~~~~~

Canonical Identity Platform is a composable identity broker and identity provider. Identity management is possible via `integration with external identity providers <https://charmhub.io/topics/canonical-identity-platform/how-to/integrate-external-identity-provider>`_ or by using `Local Identity Provider in Identity Platform <https://charmhub.io/topics/canonical-identity-platform/how-to/use-local-identity-provider>`_.

Observability
~~~~~~~~~~~~~

Canonical Identity Platform has built-in integration with the `Canonical Observability Stack <https://charmhub.io/topics/canonical-observability-stack>`_ to provide users with monitoring, logging and alerting capabilities.

Best Practices for Securing Canonical Identity Platform
-------------------------------------------------------

While the platform includes several security features out of the box, following security best practices ensures additional protection against emerging threats. Here are some key recommendations:

Regularly Update and Patch
~~~~~~~~~~~~~~~~~~~~~~~~~~

Always ensure the platform and its dependencies are up-to-date to mitigate known vulnerabilities. Canonical releases regular security patches and updates.

Enforce Strong Password Policies
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Enforcing strong password policies, including minimum password length, is crucial to reduce the risk of brute force attacks.

Limit Privileged Access
~~~~~~~~~~~~~~~~~~~~~~~

Use the principle of least privilege by granting the minimal level of access required for users to perform their tasks.

Enable Encryption in Transit and at Rest
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Protect data in transit using protocols such as HTTPS/TLS and ensure sensitive data is encrypted when stored.

Implement Monitoring, Logging and Alerting
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Regularly review system logs to detect anomalies and implement alerts for critical events such as failed login attempts, privilege escalations, or configuration changes.

Integrate with `Canonical Observability Stack <https://charmhub.io/topics/canonical-identity-platform/how-to/integrate-cos>`__ to start using monitoring, logging and alerting based on open source `components <https://charmhub.io/topics/canonical-observability-stack/editions/lite>`_ such as Grafana, Prometheus and Alertmanager.

Cryptographic tech in Canonical Identity Platform
-------------------------------------------------

The Identity Platform is a composable solution consisting of individual charms integrated together with the power of Juju relations.
Explore the detailed security technical references for each charm below:

.. toctree::
   :maxdepth: 1
   :glob:

   /explanation/hydra-security
   /explanation/kratos-security
   /explanation/openfga-security
   /explanation/identity-platform-login-ui-security
   /explanation/kratos-external-idp-integrator-security

Reporting Security Issues
-------------------------

Please `submit an issue <https://github.com/canonical/iam-bundle/issues>`_ for the Identity Platform or for an `individual component <https://charmhub.io/identity-platform>`_ on its Github page.
