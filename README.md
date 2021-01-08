# Skyview

## Create kind cluster

```bash
kind create cluster --name skyview --config config.yaml
```

## Add bitnami repo

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm search repo bitnami
```

## Redis

### Prerequistes

https://github.com/bitnami/charts/tree/master/bitnami/redis-cluster

```bash
kubectl create namespace redis
```

### Testing

```bash
helm install skyview-redis bitnami/redis-cluster --namespace redis --values Redis/values.yaml
```

```txt
NAME: skyview-redis
LAST DEPLOYED: Fri Jan  8 11:21:25 2021
NAMESPACE: redis
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
** Please be patient while the chart is being deployed **


To get your password run:
    export REDIS_PASSWORD=$(kubectl get secret --namespace redis skyview-redis-redis-cluster -o jsonpath="{.data.redis-password}" | base64 --decode)

You have deployed a Redis Cluster accessible only from within you Kubernetes Cluster.INFO: The Job to create the cluster will be created.To connect to your Redis cluster:

1. Run a Redis pod that you can use as a client:
kubectl run --namespace redis skyview-redis-redis-cluster-client --rm --tty -i --restart='Never' \
 --env REDIS_PASSWORD=$REDIS_PASSWORD \
--image docker.io/bitnami/redis-cluster:6.0.9-debian-10-r36 -- bash

2. Connect using the Redis CLI:

redis-cli -c -h skyview-redis-redis-cluster -a $REDIS_PASSWORD
```

### Production

```bash
helm install skyview-redis bitnami/redis-cluster --values values-production.yaml --namespace redis --values Redis/values-prduction.yaml
```

## Postgres

https://github.com/bitnami/charts/tree/master/bitnami/postgresql-ha/#installing-the-chart

### Prerequisites

```bash
kubectl create namespace postgres
```

### Testing

```bash
helm install skyview-postgres bitnami/postgresql-ha --namespace postgres --values Postgres/values.yaml
```

```txt
NAME: skyview-postgres
LAST DEPLOYED: Fri Jan  8 11:23:37 2021
NAMESPACE: postgres
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
** Please be patient while the chart is being deployed **

PostgreSQL can be accessed through Pgpool via port 5432 on the following DNS name from within your cluster:

    skyview-postgres-postgresql-ha-pgpool.postgres.svc.cluster.local

Pgpool acts as a load balancer for PostgreSQL and forward read/write connections to the primary node while read-only connections are forwarded to standby nodes.

To get the password for "postgres" run:

    export POSTGRES_PASSWORD=$(kubectl get secret --namespace postgres skyview-postgres-postgresql-ha-postgresql -o jsonpath="{.data.postgresql-password}" | base64 --decode)

To get the password for "repmgr" run:

    export REPMGR_PASSWORD=$(kubectl get secret --namespace postgres skyview-postgres-postgresql-ha-postgresql -o jsonpath="{.data.repmgr-password}" | base64 --decode)

To connect to your database run the following command:

    kubectl run skyview-postgres-postgresql-ha-client --rm --tty -i --restart='Never' --namespace postgres --image docker.io/bitnami/postgresql-repmgr:11.10.0-debian-10-r55 --env="PGPASSWORD=$POSTGRES_PASSWORD"  \
        --command -- psql -h skyview-postgres-postgresql-ha-pgpool -p 5432 -U postgres -d postgres

To connect to your database from outside the cluster execute the following commands:

    kubectl port-forward --namespace postgres svc/skyview-postgres-postgresql-ha-pgpool 5432:5432 &
    psql -h 127.0.0.1 -p 5432 -U postgres -d postgres
```

### Production

```bash
helm install skyview-postgres bitnami/postgresql-ha --namespace postgres --values Postgres/values-production.yaml
```

## Prometheus

https://github.com/bitnami/charts/tree/master/bitnami/kube-prometheus/#installing-the-chart

### Prerequisites

```bash
kubectl create namespace prometheus
```

### Testing

```bash
helm install skyview-postgres bitnami/kube-prometheus --namespace prometheus --values Prometheus/values.yaml
```

```txt
NAME: skyview-postgres
LAST DEPLOYED: Fri Jan  8 11:25:06 2021
NAMESPACE: prometheus
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
** Please be patient while the chart is being deployed **

Watch the Prometheus Operator Deployment status using the command:

    kubectl get deploy -w --namespace prometheus -l app.kubernetes.io/name=kube-prometheus-operator,app.kubernetes.io/instance=skyview-postgres

Watch the Prometheus StatefulSet status using the command:

    kubectl get sts -w --namespace prometheus -l app.kubernetes.io/name=kube-prometheus-prometheus,app.kubernetes.io/instance=skyview-postgres

Prometheus can be accessed via port "9090" on the following DNS name from within your cluster:

    skyview-postgres-kube-prom-prometheus.prometheus.svc.cluster.local

To access Prometheus from outside the cluster execute the following commands:

    echo "Prometheus URL: http://127.0.0.1:9090/"
    kubectl port-forward --namespace prometheus svc/skyview-postgres-kube-prom-prometheus 9090:9090

Watch the Alertmanager StatefulSet status using the command:

    kubectl get sts -w --namespace prometheus -l app.kubernetes.io/name=kube-prometheus-alertmanager,app.kubernetes.io/instance=skyview-postgres

Alertmanager can be accessed via port "9093" on the following DNS name from within your cluster:

    skyview-postgres-kube-prom-alertmanager.prometheus.svc.cluster.local

To access Alertmanager from outside the cluster execute the following commands:

    echo "Alertmanager URL: http://127.0.0.1:9093/"
    kubectl port-forward --namespace prometheus svc/skyview-postgres-kube-prom-alertmanager 9093:9093

```

### Production

```bash
helm install skyview-postgres bitnami/kube-prometheus --namespace prometheus --values Prometheus/values-production.yaml
```

## Strimzi [NOT WORKING YET!! ]

**_ISSUE:_** Error: strimzi.operator.cluster.model.NoImageException: No image for version 2.7.0 in {2.5.1=strimzi/kafka:0.20.1-kafka-2.5.1, 2.6.0=strimzi/kafka:0.20.1-kafka-2.6.0, 2.5.0=strimzi/kafka:0.20.1-kafka-2.5.0}

https://strimzi.io/quickstarts/

### Prerequisistes

```bash
kubectl create namespace kafka
```

### Installation

```bash
kubectl apply -n kafka -f Strimzi/strimzi-cluster-operator.yaml
kubectl apply -n kafka -f Strimzi/kafka-persistent.yaml
```

## Kafka

### Prerequisites

```bash
kubectl create namespace kafka
```

### Testing

```bash
helm install skyview-kafka bitnami/kafka --namespace kafka --values Kafka/values.yaml
```

```txt
AME: skyview-kafka
LAST DEPLOYED: Fri Jan  8 12:17:23 2021
NAMESPACE: kafka
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
** Please be patient while the chart is being deployed **

Kafka can be accessed by consumers via port 9092 on the following DNS name from within your cluster:

    skyview-kafka.kafka.svc.cluster.local

Each Kafka broker can be accessed by producers via port 9092 on the following DNS name(s) from within your cluster:

    skyview-kafka-0.skyview-kafka-headless.kafka.svc.cluster.local:9092

To create a pod that you can use as a Kafka client run the following commands:

    kubectl run skyview-kafka-client --restart='Never' --image docker.io/bitnami/kafka:2.7.0-debian-10-r1 --namespace kafka --command -- sleep infinity
    kubectl exec --tty -i skyview-kafka-client --namespace kafka -- bash

    PRODUCER:
        kafka-console-producer.sh \

            --broker-list skyview-kafka-0.skyview-kafka-headless.kafka.svc.cluster.local:9092 \
            --topic test

    CONSUMER:
        kafka-console-consumer.sh \

            --bootstrap-server skyview-kafka.kafka.svc.cluster.local:9092 \
            --topic test \
            --from-beginning
```

### Production

```bash
helm install skyview-kafka bitnami/kafka --namespace kafka --values Kafka/values-production.yaml
```

Output:

```txt
NAME: skyview-kafka
LAST DEPLOYED: Fri Jan  8 12:37:29 2021
NAMESPACE: kafka
STATUS: deployed
REVISION: 2
TEST SUITE: None
NOTES:
** Please be patient while the chart is being deployed **

Kafka can be accessed by consumers via port 9092 on the following DNS name from within your cluster:

    skyview-kafka.kafka.svc.cluster.local

Each Kafka broker can be accessed by producers via port 9092 on the following DNS name(s) from within your cluster:

    skyview-kafka-0.skyview-kafka-headless.kafka.svc.cluster.local:9092
    skyview-kafka-1.skyview-kafka-headless.kafka.svc.cluster.local:9092
    skyview-kafka-2.skyview-kafka-headless.kafka.svc.cluster.local:9092

You need to configure your Kafka client to access using SASL authentication. To do so, you need to create the 'kafka_jaas.conf' and 'client.properties' configuration files by executing these commands:

    - kafka_jaas.conf:

cat > kafka_jaas.conf <<EOF
KafkaClient {
org.apache.kafka.common.security.scram.ScramLoginModule required
username="user"
password="$(kubectl get secret skyview-kafka-jaas -n kafka -o jsonpath='{.data.client-passwords}' | base64 --decode | cut -d , -f 1)";
};
EOF

    - client.properties:

cat > client.properties <<EOF
security.protocol=SASL_PLAINTEXT
sasl.mechanism=SCRAM-SHA-256
EOF

To create a pod that you can use as a Kafka client run the following commands:

    kubectl run skyview-kafka-client --restart='Never' --image docker.io/bitnami/kafka:2.7.0-debian-10-r1 --namespace kafka --command -- sleep infinity
    kubectl cp --namespace kafka /path/to/client.properties skyview-kafka-client:/tmp/client.properties
    kubectl cp --namespace kafka /path/to/kafka_jaas.conf skyview-kafka-client:/tmp/kafka_jaas.conf
    kubectl exec --tty -i skyview-kafka-client --namespace kafka -- bash
    export KAFKA_OPTS="-Djava.security.auth.login.config=/tmp/kafka_jaas.conf"

    PRODUCER:
        kafka-console-producer.sh \
            --producer.config /tmp/client.properties \
            --broker-list skyview-kafka-0.skyview-kafka-headless.kafka.svc.cluster.local:9092,skyview-kafka-1.skyview-kafka-headless.kafka.svc.cluster.local:9092,skyview-kafka-2.skyview-kafka-headless.kafka.svc.cluster.local:9092 \
            --topic test

    CONSUMER:
        kafka-console-consumer.sh \
            --consumer.config /tmp/client.properties \
            --bootstrap-server skyview-kafka.kafka.svc.cluster.local:9092 \
            --topic test \
            --from-beginning
```

```bash
cat > Kafka/kafka_jaas.conf <<EOF
KafkaClient {
org.apache.kafka.common.security.scram.ScramLoginModule required
username="user"
password="$(kubectl get secret skyview-kafka-jaas -n kafka -o jsonpath='{.data.client-passwords}' | base64 --decode | cut -d , -f 1)";
};
EOF

    - client.properties:

cat > Kafka/client.properties <<EOF
security.protocol=SASL_PLAINTEXT
sasl.mechanism=SCRAM-SHA-256
EOF


kubectl run skyview-kafka-client --restart='Never' --image docker.io/bitnami/kafka:2.7.0-debian-10-r1 --namespace kafka --command -- sleep infinity
kubectl cp --namespace kafka Kafka/client.properties skyview-kafka-client:/tmp/client.properties
kubectl cp --namespace kafka Kafka/kafka_jaas.conf skyview-kafka-client:/tmp/kafka_jaas.conf
kubectl exec --tty -i skyview-kafka-client --namespace kafka -- bash
export KAFKA_OPTS="-Djava.security.auth.login.config=/tmp/kafka_jaas.conf"
# CREATE TOPIC
kafka-topics.sh --create --topic test --zookeeper skyview-kafka-zookeeper.kafka.svc.cluster.local:2181 --partitions 3 --replication-factor 3
# PRODUCER
kafka-console-producer.sh \
    --producer.config /tmp/client.properties \
    --broker-list skyview-kafka-0.skyview-kafka-headless.kafka.svc.cluster.local:9092,skyview-kafka-1.skyview-kafka-headless.kafka.svc.cluster.local:9092,skyview-kafka-2.skyview-kafka-headless.kafka.svc.cluster.local:9092 \
    --topic test

# CONSUMER:
kafka-console-consumer.sh \
    --consumer.config /tmp/client.properties \
    --bootstrap-server skyview-kafka.kafka.svc.cluster.local:9092 \
    --topic test \
    --from-beginning
```
