# MyProject

## Create kind cluster

```bash
kind create cluster --name myproject --config kind/config.yaml
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
helm install myproject-redis bitnami/redis-cluster --namespace redis --values redis/values.yaml
```

```txt
NAME: myproject-redis
LAST DEPLOYED: Fri Jan  8 11:21:25 2021
NAMESPACE: redis
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
** Please be patient while the chart is being deployed **


To get your password run:
    export REDIS_PASSWORD=$(kubectl get secret --namespace redis myproject-redis-redis-cluster -o jsonpath="{.data.redis-password}" | base64 --decode)

You have deployed a Redis Cluster accessible only from within you Kubernetes Cluster.INFO: The Job to create the cluster will be created.To connect to your Redis cluster:

1. Run a Redis pod that you can use as a client:
kubectl run --namespace redis myproject-redis-redis-cluster-client --rm --tty -i --restart='Never' \
 --env REDIS_PASSWORD=$REDIS_PASSWORD \
--image docker.io/bitnami/redis-cluster:6.0.9-debian-10-r36 -- bash

2. Connect using the Redis CLI:

redis-cli -c -h myproject-redis-redis-cluster -a $REDIS_PASSWORD
```

### Production

```bash
helm install myproject-redis bitnami/redis-cluster --values values-production.yaml --namespace redis --values redis/values-prduction.yaml
```

## Postgres

https://github.com/bitnami/charts/tree/master/bitnami/postgresql-ha/#installing-the-chart

### Prerequisites

```bash
kubectl create namespace postgres
```

### Testing

```bash
helm install myproject-postgres bitnami/postgresql-ha --namespace postgres --values postgres/values.yaml
```

```txt
NAME: myproject-postgres
LAST DEPLOYED: Fri Jan  8 11:23:37 2021
NAMESPACE: postgres
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
** Please be patient while the chart is being deployed **

PostgreSQL can be accessed through Pgpool via port 5432 on the following DNS name from within your cluster:

    myproject-postgres-postgresql-ha-pgpool.postgres.svc.cluster.local

Pgpool acts as a load balancer for PostgreSQL and forward read/write connections to the primary node while read-only connections are forwarded to standby nodes.

To get the password for "postgres" run:

    export POSTGRES_PASSWORD=$(kubectl get secret --namespace postgres myproject-postgres-postgresql-ha-postgresql -o jsonpath="{.data.postgresql-password}" | base64 --decode)

To get the password for "repmgr" run:

    export REPMGR_PASSWORD=$(kubectl get secret --namespace postgres myproject-postgres-postgresql-ha-postgresql -o jsonpath="{.data.repmgr-password}" | base64 --decode)

To connect to your database run the following command:

    kubectl run myproject-postgres-postgresql-ha-client --rm --tty -i --restart='Never' --namespace postgres --image docker.io/bitnami/postgresql-repmgr:11.10.0-debian-10-r55 --env="PGPASSWORD=$POSTGRES_PASSWORD"  \
        --command -- psql -h myproject-postgres-postgresql-ha-pgpool -p 5432 -U postgres -d postgres

To connect to your database from outside the cluster execute the following commands:

    kubectl port-forward --namespace postgres svc/myproject-postgres-postgresql-ha-pgpool 5432:5432 &
    psql -h 127.0.0.1 -p 5432 -U postgres -d postgres
```

### Production

```bash
helm install myproject-postgres bitnami/postgresql-ha --namespace postgres --values postgres/values-production.yaml
```

## Crunchy (Postgres Operator)

https://github.com/CrunchyData/postgres-operator

### Prerequsites

```bash
kubectl create namespace pgo
kubectl apply -f crunchy/postgres-operator.yaml -n pgo
# When operator ready, launch the following command!
./crunchy/client-setup.sh


# Add env vars to be added to .bashrc or .zshrc
cat <<EOF >> ~/.zshrc
export PGOUSER="${HOME?}/.pgo/pgo/pgouser"
export PGO_CA_CERT="${HOME?}/.pgo/pgo/client.crt"
export PGO_CLIENT_CERT="${HOME?}/.pgo/pgo/client.crt"
export PGO_CLIENT_KEY="${HOME?}/.pgo/pgo/client.key"
export PGO_APISERVER_URL='https://127.0.0.1:8443'
export PGO_NAMESPACE=pgo
export PATH=${HOME?}/.pgo/pgo:$PATH
EOF
source ~/.zshrc
# You can know forward pgo operator service in another window
kubectl -n pgo port-forward svc/postgres-operator 8443:8443
# Get version
pgo version
```

## Testing

```bash
# Creating cluster
pgo create cluster -n pgo myproject-postgres
# Testing cluster
pgo test -n pgo myproject-postgres
# Getting information on users in the cluster
pgo show user -n pgo myproject-postgres
pgo show user -n pgo myproject-postgres --show-system-accounts

CLUSTER            USERNAME    PASSWORD                 EXPIRES STATUS ERROR
------------------ ----------- ------------------------ ------- ------ -----
myproject-postgres crunchyadm                           never   ok
myproject-postgres postgres    `Nu(K=9GFlV*n9QHv*]4oir@ never   ok
myproject-postgres primaryuser Kt*b-^g<AS=]14eHEiJ,NHRa never   ok
myproject-postgres testuser    j:>Yyd9.HoIOk6^@Y(-BYEOv never   ok

# Change user password
pgo update user -n pgo myproject-postgres --username=testuser --password="secure123"

# Getting postgres service
kubectl -n pgo get svc
# Result
NAME                                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
myproject-postgres                        ClusterIP   10.96.46.177    <none>        2022/TCP,5432/TCP            10m
myproject-postgres-backrest-shared-repo   ClusterIP   10.96.188.206   <none>        2022/TCP                     10m
postgres-operator                         ClusterIP   10.96.128.13    <none>        8443/TCP,4171/TCP,4150/TCP   30m

# Port forwarding postgres service
kubectl -n pgo port-forward svc/myproject-postgres 5432:5432

# Add PgAdmin4
pgo create pgadmin -n pgo myproject-postgres
kubectl -n pgo port-forward svc/hippo-pgadmin 5050:5050
kubectl -n pgo port-forward svc/myproject-postgres-pgadmin 5050:5050
# You can now goto pgadmin ui on http://localhost:5050 using login testuser and password secure123
```

## Prometheus

https://github.com/bitnami/charts/tree/master/bitnami/kube-prometheus/#installing-the-chart

### Prerequisites

```bash
kubectl create namespace prometheus
```

### Testing

```bash
helm install myproject-postgres bitnami/kube-prometheus --namespace prometheus --values prometheus/values.yaml
```

```txt
NAME: myproject-postgres
LAST DEPLOYED: Fri Jan  8 11:25:06 2021
NAMESPACE: prometheus
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
** Please be patient while the chart is being deployed **

Watch the Prometheus Operator Deployment status using the command:

    kubectl get deploy -w --namespace prometheus -l app.kubernetes.io/name=kube-prometheus-operator,app.kubernetes.io/instance=myproject-postgres

Watch the Prometheus StatefulSet status using the command:

    kubectl get sts -w --namespace prometheus -l app.kubernetes.io/name=kube-prometheus-prometheus,app.kubernetes.io/instance=myproject-postgres

Prometheus can be accessed via port "9090" on the following DNS name from within your cluster:

    myproject-postgres-kube-prom-prometheus.prometheus.svc.cluster.local

To access Prometheus from outside the cluster execute the following commands:

    echo "Prometheus URL: http://127.0.0.1:9090/"
    kubectl port-forward --namespace prometheus svc/myproject-postgres-kube-prom-prometheus 9090:9090

Watch the Alertmanager StatefulSet status using the command:

    kubectl get sts -w --namespace prometheus -l app.kubernetes.io/name=kube-prometheus-alertmanager,app.kubernetes.io/instance=myproject-postgres

Alertmanager can be accessed via port "9093" on the following DNS name from within your cluster:

    myproject-postgres-kube-prom-alertmanager.prometheus.svc.cluster.local

To access Alertmanager from outside the cluster execute the following commands:

    echo "Alertmanager URL: http://127.0.0.1:9093/"
    kubectl port-forward --namespace prometheus svc/myproject-postgres-kube-prom-alertmanager 9093:9093

```

### Production

```bash
helm install myproject-postgres bitnami/kube-prometheus --namespace prometheus --values prometheus/values-production.yaml
```

## Strimzi

https://strimzi.io/quickstarts/

### Prerequisistes

```bash
kubectl create namespace kafka
```

### Installation

```bash
kubectl apply -n kafka -f strimzi/strimzi-cluster-operator.yaml
kubectl apply -n kafka -f strimzi/kafka-persistent.yaml:
kubectl apply -n kafka -f strimzi/topics.yaml
```

## Kafka

### Prerequisites

```bash
kubectl create namespace kafka
```

### Testing

```bash
helm install myproject-kafka bitnami/kafka --namespace kafka --values kafka/values.yaml
```

```txt
AME: myproject-kafka
LAST DEPLOYED: Fri Jan  8 12:17:23 2021
NAMESPACE: kafka
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
** Please be patient while the chart is being deployed **

Kafka can be accessed by consumers via port 9092 on the following DNS name from within your cluster:

    myproject-kafka.kafka.svc.cluster.local

Each Kafka broker can be accessed by producers via port 9092 on the following DNS name(s) from within your cluster:

    myproject-kafka-0.myproject-kafka-headless.kafka.svc.cluster.local:9092

To create a pod that you can use as a Kafka client run the following commands:

    kubectl run myproject-kafka-client --restart='Never' --image docker.io/bitnami/kafka:2.7.0-debian-10-r1 --namespace kafka --command -- sleep infinity
    kubectl exec --tty -i myproject-kafka-client --namespace kafka -- bash

    PRODUCER:
        kafka-console-producer.sh \

            --broker-list myproject-kafka-0.myproject-kafka-headless.kafka.svc.cluster.local:9092 \
            --topic test

    CONSUMER:
        kafka-console-consumer.sh \

            --bootstrap-server myproject-kafka.kafka.svc.cluster.local:9092 \
            --topic test \
            --from-beginning
```

### Production

```bash
helm install myproject-kafka bitnami/kafka --namespace kafka --values kafka/values-production.yaml
```

Output:

```txt
NAME: myproject-kafka
LAST DEPLOYED: Fri Jan  8 12:37:29 2021
NAMESPACE: kafka
STATUS: deployed
REVISION: 2
TEST SUITE: None
NOTES:
** Please be patient while the chart is being deployed **

Kafka can be accessed by consumers via port 9092 on the following DNS name from within your cluster:

    myproject-kafka.kafka.svc.cluster.local

Each Kafka broker can be accessed by producers via port 9092 on the following DNS name(s) from within your cluster:

    myproject-kafka-0.myproject-kafka-headless.kafka.svc.cluster.local:9092
    myproject-kafka-1.myproject-kafka-headless.kafka.svc.cluster.local:9092
    myproject-kafka-2.myproject-kafka-headless.kafka.svc.cluster.local:9092

You need to configure your Kafka client to access using SASL authentication. To do so, you need to create the 'kafka_jaas.conf' and 'client.properties' configuration files by executing these commands:

    - kafka_jaas.conf:

cat > kafka_jaas.conf <<EOF
KafkaClient {
org.apache.kafka.common.security.scram.ScramLoginModule required
username="user"
password="$(kubectl get secret myproject-kafka-jaas -n kafka -o jsonpath='{.data.client-passwords}' | base64 --decode | cut -d , -f 1)";
};
EOF

    - client.properties:

cat > client.properties <<EOF
security.protocol=SASL_PLAINTEXT
sasl.mechanism=SCRAM-SHA-256
EOF

To create a pod that you can use as a Kafka client run the following commands:

    kubectl run myproject-kafka-client --restart='Never' --image docker.io/bitnami/kafka:2.7.0-debian-10-r1 --namespace kafka --command -- sleep infinity
    kubectl cp --namespace kafka /path/to/client.properties myproject-kafka-client:/tmp/client.properties
    kubectl cp --namespace kafka /path/to/kafka_jaas.conf myproject-kafka-client:/tmp/kafka_jaas.conf
    kubectl exec --tty -i myproject-kafka-client --namespace kafka -- bash
    export KAFKA_OPTS="-Djava.security.auth.login.config=/tmp/kafka_jaas.conf"

    PRODUCER:
        kafka-console-producer.sh \
            --producer.config /tmp/client.properties \
            --broker-list myproject-kafka-0.myproject-kafka-headless.kafka.svc.cluster.local:9092,myproject-kafka-1.myproject-kafka-headless.kafka.svc.cluster.local:9092,myproject-kafka-2.myproject-kafka-headless.kafka.svc.cluster.local:9092 \
            --topic test

    CONSUMER:
        kafka-console-consumer.sh \
            --consumer.config /tmp/client.properties \
            --bootstrap-server myproject-kafka.kafka.svc.cluster.local:9092 \
            --topic test \
            --from-beginning
```

```bash
cat > kafka/kafka_jaas.conf <<EOF
KafkaClient {
org.apache.kafka.common.security.scram.ScramLoginModule required
username="user"
password="$(kubectl get secret myproject-kafka-jaas -n kafka -o jsonpath='{.data.client-passwords}' | base64 --decode | cut -d , -f 1)";
};
EOF

    - client.properties:

cat > kafka/client.properties <<EOF
security.protocol=SASL_PLAINTEXT
sasl.mechanism=SCRAM-SHA-256
EOF


kubectl run myproject-kafka-client --restart='Never' --image docker.io/bitnami/kafka:2.7.0-debian-10-r1 --namespace kafka --command -- sleep infinity
kubectl cp --namespace kafka kafka/client.properties myproject-kafka-client:/tmp/client.properties
kubectl cp --namespace kafka kafka/kafka_jaas.conf myproject-kafka-client:/tmp/kafka_jaas.conf
kubectl exec --tty -i myproject-kafka-client --namespace kafka -- bash
export KAFKA_OPTS="-Djava.security.auth.login.config=/tmp/kafka_jaas.conf"
# CREATE TOPIC
kafka-topics.sh --create --topic test --zookeeper myproject-kafka-zookeeper.kafka.svc.cluster.local:2181 --partitions 3 --replication-factor 3
# PRODUCER
kafka-console-producer.sh \
    --producer.config /tmp/client.properties \
    --broker-list myproject-kafka-0.myproject-kafka-headless.kafka.svc.cluster.local:9092,myproject-kafka-1.myproject-kafka-headless.kafka.svc.cluster.local:9092,myproject-kafka-2.myproject-kafka-headless.kafka.svc.cluster.local:9092 \
    --topic test

# CONSUMER:
kafka-console-consumer.sh \
    --consumer.config /tmp/client.properties \
    --bootstrap-server myproject-kafka.kafka.svc.cluster.local:9092 \
    --topic test \
    --from-beginning
```

## Cert Manager

https://cert-manager.io/docs/installation/kubernetes/

### Installation

```bash
kubectl create namespace cert-manager
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --version v1.1.0
# Result
NAME: cert-manager
LAST DEPLOYED: Mon Jan 11 09:28:13 2021
NAMESPACE: cert-manager
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
cert-manager has been deployed successfully!

In order to begin issuing certificates, you will need to set up a ClusterIssuer
or Issuer resource (for example, by creating a 'letsencrypt-staging' issuer).

More information on the different types of issuers and how to configure them
can be found in our documentation:

https://cert-manager.io/docs/configuration/

For information on how to configure cert-manager to automatically provision
Certificates for Ingress resources, take a look at the `ingress-shim`
documentation:

https://cert-manager.io/docs/usage/ingress/
```

## Nginx Ingress

### Installation

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm search repo ingress-nginx/ingress-nginx
kubectl create namespace nginx-ingress
helm install  myproject-nginx-ingress -n nginx-ingress ingress-nginx/ingress-nginx --version 3.19.0

# Result
NAME: myproject-nginx-ingress
LAST DEPLOYED: Mon Jan 11 11:28:59 2021
NAMESPACE: nginx-ingress
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
The ingress-nginx controller has been installed.
It may take a few minutes for the LoadBalancer IP to be available.
You can watch the status by running 'kubectl --namespace nginx-ingress get services -o wide -w myproject-nginx-ingress-ingress-nginx-controller'

An example Ingress that makes use of the controller:

  apiVersion: networking.k8s.io/v1beta1
  kind: Ingress
  metadata:
    annotations:
      kubernetes.io/ingress.class: nginx
    name: example
    namespace: foo
  spec:
    rules:
      - host: www.example.com
        http:
          paths:
            - backend:
                serviceName: exampleService
                servicePort: 80
              path: /
    # This section is only required if TLS is to be enabled for the Ingress
    tls:
        - hosts:
            - www.example.com
          secretName: example-tls

If TLS is enabled for the Ingress, a Secret containing the certificate and key must also be provided:

  apiVersion: v1
  kind: Secret
  metadata:
    name: example-tls
    namespace: foo
  data:
    tls.crt: <base64 encoded cert>
    tls.key: <base64 encoded key>
  type: kubernetes.io/tls

```
