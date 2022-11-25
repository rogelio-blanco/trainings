Kafka Distributed Streaming Platform
====================================

_Publish and Subscribe / Process / Store_


## Start Kafka
* Kafka uses ZooKeeper as a distributed backend.

### Start Zookeeper
```
bin/zookeeper-server-start.sh config/zookeeper.properties
```

### Start Kafka
```
bin/kafka-server-start.sh config/server.properties
```

## Topics

### Create Topic
```
bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic test
```

### List Topics
```
bin/kafka-topics.sh --list --bootstrap-server localhost:9092
```

## Messages
### Send Message
```
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
```


## Consumers
### Start Consumer
```
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
```





## Operate Kafka using Docker

### Start ZooKeeper using Docker
```
docker pull wurstmeister/zookeeper:latest
docker run -d \
  -p 2181:2181 \
  --name zookeeper \
  wurstmeister/zookeeper:latest
```

### Start Kafka using Docker
```
docker pull wurstmeister/kafka:latest

KAFKA_BROKER_ID="001"
KAFKA_CREATE_TOPICS="test0:1:3,test1:1:1:compact"
## test0 will have 1 partition and 3 replicas
## test1 will have 1 partition, 1 replica and a cleanup.policy set to compact.


docker run -d -p 9094:9094 \
  -e KAFKA_BROKER_ID="${KAFKA_BROKER_ID}" \
  -e KAFKA_CREATE_TOPICS="${KAFKA_CREATE_TOPICS}" \
  -e HOSTNAME_COMMAND="docker info | grep ^Name: | cut -d' ' -f 2" \
  -e KAFKA_ZOOKEEPER_CONNECT="zookeeper:2181" \
  -e KAFKA_LISTENER_SECURITY_PROTOCOL_MAP="INSIDE:PLAINTEXT,OUTSIDE:PLAINTEXT" \
  -e KAFKA_ADVERTISED_LISTENERS="INSIDE://:9092,OUTSIDE://_{HOSTNAME_COMMAND}:9094" \
  -e KAFKA_LISTENERS="INSIDE://:9092,OUTSIDE://:9094" \
  -e KAFKA_INTER_BROKER_LISTENER_NAME="INSIDE" \
  -v /var/run/docker.sock:/var/run/docker.sock \
  --link zookeeper:zookeeper \
  --name kafka \
  wurstmeister/kafka:latest


```

### List running containers
```
docker ps --format 'table {{.ID}}\t{{.Names}}\t{{.Image}}\t{{.Status}}'
```


### View stdout logs
```
docker logs kafka
docker logs zookeeper
```


### Run Kafka Commands inside the container
```
## List Brokers
docker exec -ti kafka /usr/bin/broker-list.sh

## List Topics
docker exec -ti kafka /opt/kafka/bin/kafka-topics.sh --list --zookeeper zookeeper:2181

## Create a Topic
docker exec -ti kafka /opt/kafka/bin/kafka-topics.sh --create --zookeeper zookeeper:2181 --replication-factor 1 --partitions 1 --topic test2


## List Topics
docker exec -ti kafka /opt/kafka/bin/kafka-topics.sh --list --zookeeper zookeeper:2181

```