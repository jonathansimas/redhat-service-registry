# Installing Service Registry with AMQ Streams and RHSSO in Openshift

Service Registry is based on the Apicurio Registry open source community project. For details, see https://github.com/apicurio/apicurio-registry.

In this document we demonstrate how to install Service Registry and integrate it with AMQ Streams and Red Hat SSO. Please see amq-streams.md and rhsso.md for instructions of how to install them.

## 1. Install Red Hat Integration - Service Registry Operator

In Openshift >> Operator Hub select and install *Red Hat Integration - Service Registry Operator* in the desired namespace.

## 2. Create Apicurio Registry instance

In the Service Registry Operator click to create a new instance of APIcurio. You must define the kafka service url and some keycloak info, as in the example below:


```
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





