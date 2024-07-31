This tutorial shows how to set up a fully working kratos server using our charm, MicroK8s and Juju 

# Set things up

Bootstrap a [microk8s controller](https://juju.is/docs/juju/set-up--tear-down-your-test-environment#heading--set-up-automatically) using juju `3.4` and create a new Juju model:

```shell
$ juju add-model kratos
Added 'kratos' model on microk8s/localhost with credential 'microk8s' for user 'admin'
```

> See more: [Set up your test environment automatically](https://juju.is/docs/juju/set-up--tear-down-your-test-environment#heading--set-up-automatically)



# Watch the kratos charm transform the way to deploy, configure, integrate, and manage kratos on any Kubernetes cloud

kratos requires a way to persist data, in the case of our charm we enforce the usage of a `postgreSQL` database

As mentioned, we need a persistent way to store `kratos` data, we are going to be using the [`postgresql-k8s` charm](https://charmhub.io/postgresql-k8s) 


```shell
$ juju deploy postgresql-k8s --channel 14/stable --trust postgresql
Deployed "postgresql" from charm-hub charm "postgresql-k8s", revision 193 in channel 14/stable on ubuntu@22.04/stable
```

Once that is done (no need to wait for it to be ready) we can proceed in deploying `kratos` and integrate the 2 charms 


```shell
$ juju deploy kratos --channel 0.2/stable
Deployed "kratos" from charm-hub charm "kratos", revision 399 in channel 0.2/stable on ubuntu@22.04/stable
```

## Integrate with PostgreSQL


```shell 
$ juju integrate postgresql:database kratos
```

after some time we should be able to inspect that all has been successfully deployed and connected


```shell
$ juju status --relations                           

Model   Controller          Cloud/Region        Version  SLA          Timestamp
kratos  microk8s-localhost  microk8s/localhost  3.4.2    unsupported  09:45:28+01:00

App          Version  Status  Scale  Charm           Channel        Rev  Address         Exposed  Message
kratos       v1.1.0   active      1  kratos          0.2/stable     399  10.152.183.42   no       
postgresql   14.10    active      1  postgresql-k8s  14/stable      193  10.152.183.100  no       Primary

Unit            Workload  Agent  Address       Ports  Message
kratos/0*       active    idle   10.1.245.173         
postgresql/0*   active    idle   10.1.245.146         Primary

Integration provider       Requirer                   Interface          Type     Message
kratos:kratos-peers        kratos:kratos-peers        kratos-peers       peer     
postgresql:database        kratos:pg-database         postgresql_client  regular  
postgresql:database-peers  postgresql:database-peers  postgresql_peers   peer     
postgresql:restart         postgresql:restart         rolling_op         peer     
postgresql:upgrade         postgresql:upgrade         upgrade            peer     
```

## Grafana, Loki, and Prometheus

The kratos operator integrates with [Canonical Observability Stack](https://charmhub.io/topics/canonical-observability-stack) (COS) bundle.
It comes with a Grafana dashboard as well as Loki and Prometheus alert rules for basic common scenarios.
To integrate with the COS bundle, after you [deploy it](https://charmhub.io/topics/canonical-observability-stack/tutorials/install-microk8s#heading--deploy-the-cos-lite-bundle), you can run:


```shell
$ juju integrate kratos:grafana-dashboard grafana:grafana-dashboard
$ juju integrate kratos:metrics-endpoint prometheus:metrics-endpoint
$ juju integrate loki:logging kratos:log-proxy
```


# Tear things down

To tear things down, remove the entire `kratos` model in juju with 

```shell
juju destroy-model kratos
```
