# Installing Service Registry with AMQ Streams and RHSSO in Openshift

> *This is a resumed extract from the Service Registry documentation, so you can get it up and running faster. If you are having any issues, please read the full documentation at [access.redhat.com](https://access.redhat.com/documentation/en-us/red_hat_integration/2022.q3/html/installing_and_deploying_service_registry_on_openshift/index)*

Service Registry is based on the Apicurio Registry open source community project. For details, see https://github.com/apicurio/apicurio-registry.

In this document we demonstrate how to install Service Registry and integrate it with AMQ Streams and Red Hat SSO. Please see [AMQ Streams](https://github.com/redhat-banco-do-brasil/service-registry/tree/main/gitops/amq-streams) and [RHSSO](https://github.com/redhat-banco-do-brasil/service-registry/tree/main/gitops/rhsso) for instructions of how to install them for use with Service Registry.

## 1. Install Red Hat Integration - Service Registry Operator

In Openshift >> Operator Hub select and install *Red Hat Integration - Service Registry Operator* in the desired namespace.

## 2. Create Apicurio Registry instance

In the Service Registry Operator click to create a new instance of APIcurio. You must define the kafka service url and some keycloak info, as in the example below:


```yaml
kind: ApicurioRegistry
apiVersion: registry.apicur.io/v1
metadata:
  name: example-apicurioregistry-kafkasql
  namespace: iib-service-registry
spec:
  configuration:
    kafkasql:
      bootstrapServers: 'kafka-service.svc:9092'
    persistence: kafkasql
    security:
      keycloak:
        apiClientId: registry-client-api
        realm: registry
        uiClientId: registry-client-ui
        url: >-
          http://keycloak-registry.apps.my-cluster.com/auth
```





