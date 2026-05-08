# Kafka playground

In this example, we're creating Kafka cluster using Docker Compose with KRaftt mode.

Prerequisites:

- Docker and Docker compose installed

This setup will include:

- 3 Kafka brokers
- Kafka UI for cluster management
- Docker network for inter-broker communication
- Persistent volume storage

## Step 1. Project setup

Create a directory structure for Kafka cluster.

```shell
git@github.com:BudjakovDmitry/kafka-playground.git
cd kafka-playground
mkdir -p kafka{1,2,3}/data
```

## Step 2. Understand the Configuration

### Network configuration

```yml
networks:
    kafka-net:
        driver: bridge
```

This create an isolated network for our Kafka cluster, ensuring secure communication between containers

### Broker configuration

Each broker has several important settings:

**Process Roles**

```
KAFKA_PROCESS_ROLES: 'broker,controller'
```

This enables KRaft mode with each broker also acting as a controller.

**Listeners**

```
KAFKA_LESTENERS: 'PLAINTEXT://kafka1:9092,CONTROLLER://kafka1:9093'
KAFKA_ADVERTISED_LISTENERS: 'PLAINTEXT://kafka:9092'
```

These configure how other clients and brokers connect to the cluster.


**Replication settings**

```
KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 3
KAFKA_DEFAULT_REPLICATION_FACTORU: 3
KAFKA_MIN_INSYNC_REPLICAS: 2
```

`KAFKA_MIN_INSYNC_REPLICAS`: specifies the minimum number of replicas that must acknowledge a write operation.

`KAFKA_DEFAULT_REPLICATION_FACTOR`: this setting determines how many copies of each partition are created across the cluster.

`KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR`: controls replication for the special \_\_consumer\_offset topic. This topic tracks consumer group positions (which messages have been read). Value 3 means consumer offsets are stored in 3 different broker.

## Step 3. Launching the Cluster

```shell
docker-compose up -d
```

Verify all containers are running:

```shell
docker-compose ps
```

Kafka GUI now available at `http://localhost:8080`

## Step 4. Testing the Cluster

```shell
docker exec -it kafka1 kafka-topics \
    --create \
    --topic my-topic \
    --bootstrap-server kafka1:9092 \
    --replication-factor 3 \
    --partitions 3
```

List all topics:

```
docker exec -it kafka1 kafka-topics \
    --list \
    --bootstrap-server kafka1:9092
```

Send message to the topic

```shell
docker exec -it kafka1 kafka-console-producer \
--topic my-topic \
--bootstrap-server kafka1:9092
```

Read messages from topic

```shell
docker exec -it kafka1 kafka-console-consumer \
    --from-beginning \
    --topic my-topic \
    --bootstrap-server kafka1:9092
```
