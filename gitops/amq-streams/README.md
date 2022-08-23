# Installing a Kafka instance to use with Service Registry


### Installing the Red Hat Integration - AMQ Streams Operator
You can install the operator manually in Openshift console using the Operator Hub or you can automate its creation with the provided yamls in this folder.

### Creating an Kafka Instance

Once the operator is installed you can create an instance of Kafka, as in the example below:


```yaml
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  name: kafka-registry
  namespace: service-registry-amq-streams
spec:
  entityOperator:
    topicOperator: {}
    userOperator: {}
  kafka:
    config:
      default.replication.factor: 3
      inter.broker.protocol.version: '3.1'
      min.insync.replicas: 2
      offsets.topic.replication.factor: 3
      transaction.state.log.min.isr: 2
      transaction.state.log.replication.factor: 3
    listeners:
      - name: plain
        port: 9092
        tls: false
        type: internal
      - name: tls
        port: 9093
        tls: true
        type: internal
    replicas: 3
    storage:
      size: 1Gi
      type: persistent-claim
    version: 3.1.0
  zookeeper:
    replicas: 3
    storage:
      size: 1Gi
      type: persistent-claim
```

It's recommended to use **persistent-claim** for your storage definitions so your Kafka instance will not be ephemeral
