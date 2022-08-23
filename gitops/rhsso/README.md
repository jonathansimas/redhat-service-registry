# Installing a Red Hat SSO instance to use with Service Registry

### Installing the operator

You can install the operator manually in Openshift console using the Operator Hub or you can automate its creation with the provided yamls in this folder.

### Creating an Keycloak instance

Once the operator is installed you can create an Keycloak instance, as in the example below:

```
apiVersion: keycloak.org/v1alpha1
kind: Keycloak
metadata:
  labels:
    app: sso
  name: service-registry-keycloak
  namespace: service-registry-rhsso
spec:
  externalAccess:
    enabled: true
  podDisruptionBudget:
    enabled: True
  instances: 1
```

### Configuring Keycloak: Create a KeycloakRealm

You can use the Red Hat SSO Operator to add a new Realm to Keycloak, or ` oc apply -f keycloak-realm.yaml` with the provided file in this repo:

```
apiVersion: keycloak.org/v1alpha1
kind: KeycloakRealm
metadata:
  name: registry-keycloakrealm
spec:
  instanceSelector:
    matchLabels:
      app: sso
  realm:
    displayName: Registry
    enabled: true
    id: registry
    realm: registry
    sslRequired: none
    roles:
      realm:
        - name: sr-admin
        - name: sr-developer
        - name: sr-readonly
    clients:
      - clientId: registry-client-ui
        implicitFlowEnabled: true
        redirectUris:
          - '*'
        standardFlowEnabled: true
        webOrigins:
          - '*'
        publicClient: true
      - clientId: registry-client-api
        implicitFlowEnabled: true
        redirectUris:
          - '*'
        standardFlowEnabled: true
        webOrigins:
          - '*'
        publicClient: true
    users:
      - credentials:
          - temporary: false
            type: password
            value: changeme
        enabled: true
        realmRoles:
          - sr-admin
        username: registry-admin
      - credentials:
          - temporary: false
            type: password
            value: changeme
        enabled: true
        realmRoles:
          - sr-developer
        username: registry-developer
      - credentials:
          - temporary: false
            type: password
            value: changeme
        enabled: true
        realmRoles:
          - sr-readonly
        username: registry-user
```

### Exposing RHSSO to the Service Registry

If you're using a self signed TLS certificate in your cluster, you will probably have an error when trying to connect you APIcurio to RHSSO bacause APICurio doesn't accepts invalid ssl certificates. 
As a workaround in **non-production environments** you can expose an http endpoint for RHSSO:

Create a service targeting keycloak's port 8080:

```
apiVersion: v1
kind: Service
metadata:
  name: keycloak-http
  labels:
    app: keycloak
spec:
  ports:
    - name: keycloak-http
      protocol: TCP
      port: 8080
      targetPort: 8080
  selector:
    app: keycloak
    component: keycloak
  type: ClusterIP
  sessionAffinity: None
```

And then expose this service with an http route:

```
kind: Route
apiVersion: route.openshift.io/v1
metadata:
  name: keycloak-http
  labels:
    app: keycloak
spec:
  to:
    kind: Service
    name: keycloak-http
    weight: 100
  port:
    targetPort: keycloak-http
  wildcardPolicy: None
```

### Troubleshooting

#### 1 - Error in Logout

*Parameter 'redirect_uri' no longer supported. Please use 'post_logout_redirect_uri' with 'id_token_hint' for this endpoint. Alternatively you can enable backwards compatibility option 'legacy-logout-redirect-uri' of oidc login protocol in the server configuration.*

If you're getting this error in Keycloak logs, you have RHSSO version 7.6 or higher and the message is clear, so you need to edit your standalone.xml to include the snipit below:

```
<spi name="login-protocol">
  <provider name="openid-connect" enabled="true">
    <properties>
      <property name="legacy-logout-redirect-uri" value="true"/>
    </properties>
  </provider>
</spi>
```

Alternatively, you can use jboss-cli to apply the following configuration:

```
/subsystem=keycloak-server/spi=login-protocol:add
/subsystem=keycloak-server/spi=login-protocol/provider=openid-connect:add(enabled=true)
/subsystem=keycloak-server/spi=login-protocol/provider=openid-connect:write-attribute(name=properties.legacy-logout-redirect-uri,value=true)

```