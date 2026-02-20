.. _perform-database-migration:

Perform database migration
==========================

This document provides some migration strategies that can be adopted in large-scale, production environments. 

Charmed `Kratos <https://github.com/canonical/kratos-operator>`_ and `Hydra <https://github.com/canonical/hydra-operator>`_ in the Identity Platform `Juju Bundle <https://github.com/canonical/iam-bundle>`_ use Charmed `PostgreSQL <https://charmhub.io/postgresql-k8s>`_ as the data backend. Since upstream Ory Kratos/Hydra introduce SQL migrations between releases it is important to perform the right steps to avoid service interruptions.

Some facts and reminders
------------------------

* Charm releases may contain the new version of the corresponding Ory open source product. This means each Charm release will probably require you to do migration. You should refer to the Charm releases (e.g. `Kratos releases <https://github.com/canonical/kratos-operator/releases>`_) and the Ory product release changelogs (e.g. `Ory Kratos CHANGELOG.md <https://github.com/ory/kratos/blob/master/CHANGELOG.md>`_) to check whether a migration is needed.
* The upstream Ory products provide `CLI <https://www.ory.sh/docs/kratos/cli/kratos-migrate-sql>`_ to assist with database migrations, however they do not provide further guidelines for migrating database in production environments or large-scale distributed systems.
* There is no silver bullet when coming to database migrations. If your case does not fit in any strategies described in this document please feel free to reach out on `Charmhub <https://discourse.charmhub.io/>`_ or our public `Matrix channel <https://matrix.to/#/!3xNKp3PFtjTOKbwJykMRzruCbSEMkXb8ZRrDKk1vknI?via=ubuntu.com>`_.

Database migration strategies
-----------------------------

.. note::
    This guide has been developed using Kratos as an example. The migration strategy for the Hydra Charm generally follows the same process.

Recommended Strategy
~~~~~~~~~~~~~~~~~~~~

This migration strategy falls into concepts of redundancy (e.g. blue/green deployments) and traffic switchover. 

1. **Prepare** and deploy a new Kratos Charm of the SAME VERSION as the one you are currently using and a PostgreSQL Charm. Integrate the two Charms.

.. code-block:: shell

    juju deploy kratos <new-kratos-app> --channel <channel> --revision <original-rev>
    juju deploy postgresql <new-postgresql-app> --channel <channel>
    juju integrate <new-kratos-app> <new-postgresql-app>

2. Use database migration systems or database replication mechanisms to sync source database to target database. Wait for the target database is almost/fully synchronized with the source database.
3. Stop writing traffic to the source database. Wait for all remaining data to drain to the target database. The source and target databases are now fully synchronized.
4. Upgrade the new Kratos Charm.

.. code-block:: shell

    juju refresh <new-kratos-app> --channel <channel> --revision <new-rev>

5. Trigger the migration action. **Note: depending on the data size, you may want to use a large timeout threshold**.

.. code-block:: shell

    juju run <new-kratos-app>/<leader> run-migration timeout=<timeout-in-seconds>

6. Once migration is completed, switch over the traffic to the new Kratos Charm.

The following diagram further illustrates the process:

.. image:: https://raw.githubusercontent.com/canonical/canonical-identity-platform-docs/main/Diagram_sources/migration.png
   :alt: Kratos Database Migration
   :align: center

.. attention::
    * Migration strategies can vary significantly in different use cases due to SLA/SLOs, migration downtime tolerances, overall deployment architectures, traffic volumes and patterns, etc. You may want to develop and maintain a migration strategy tailored for your use cases.
    * You can select the most convenient migration systems/tools to use.
    * Replication strategies between two PostgreSQL Charms will be provided in the relevant Charmhub topic pages.

Basic Strategy (non-critical environments)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. warning::
    **CAUTION:** this method updates the database schemas in-place. Please consider it for a non-production environment and apply it with discretion.

Upgrade the Charm by running the following command:

.. code-block:: shell

    juju refresh <kratos-app> --channel <channel> --revision <revision>

With the Charm upgraded, you can trigger the migration action by running:

.. code-block:: shell

    juju run <kratos-app>/<leader> run-migration timeout=<timeout in seconds>

You can check the status of the action by running:

.. code-block:: shell

    juju show-task <task-id>

Kratos identity schema upgrade
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You may want to initiate an update for an `identity schema <https://www.ory.sh/docs/kratos/manage-identities/identity-schema>`_ used in the Kratos Charm. In this case, please refer to the `best practices <https://www.ory.sh/docs/kratos/manage-identities/best-practices#updating-identity-schemas>`_ to plan the migration.

Migration best Practices
------------------------

Albeit not specific to the Identity platform, it is important to consider the following points when performing database migrations:

* Inspect and understand the system traffic patterns. In general, web application live traffic shows a tidal pattern. Plan ahead and perform the migration plans during the traffic low peak time.
* Prepare a fallback/rollback strategy when the migration plan fails. 
* Prepare a testing/staging environment which resembles the production environment to simulate the migration and fallback/rollback strategies before moving forward to production.
* Perform database backups at the critical points of migration plan, e.g. before migration starts, after draining the source database, etc. The PostgreSQL Charm also supports backup and restore operations.
* Perform database backups using the replicas instead of primary. If possible, add a new replica specifically responsible for backup jobs.
* Perform database completeness and consistency validations after the migration.
* Depending on your operational / resiliency requirements you could follow the `database per service <https://microservices.io/patterns/data/database-per-service.html>`_ pattern by amending the `Identity Platform bundle <https://github.com/canonical/iam-bundle>`_ configuration.