
1. Add kafka instance
```
version: "3.9"

services:

zookeeper:

restart: always

image: docker.io/bitnami/zookeeper:3.8

ports:

- "2181:2181"

environment:

- ALLOW_ANONYMOUS_LOGIN=yes

- ZOO_AUTOPURGE_INTERVAL=1

networks:

- container

logging:

driver: none

  

kafka:

restart: always

image: &kafka-image docker.io/bitnami/kafka:3.3

ports:

- "9093:9093"

- "9092:9092"

environment:

- KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper:2181

- KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CLIENT:PLAINTEXT,EXTERNAL:PLAINTEXT

- KAFKA_CFG_LISTENERS=CLIENT://:9092,EXTERNAL://:9093

- KAFKA_CFG_ADVERTISED_LISTENERS=CLIENT://kafka:9092,EXTERNAL://localhost:9093

- KAFKA_CFG_INTER_BROKER_LISTENER_NAME=CLIENT

- ALLOW_PLAINTEXT_LISTENER=yes

- KAFKA_ZOOKEEPER_TLS_VERIFY_HOSTNAME=false

- KAFKA_CFG_AUTO_CREATE_TOPICS_ENABLE=true

depends_on:

zookeeper:

condition: service_started

networks:

- container

logging:

driver: none
```

2. Add init container
```

# create kafka raw topic

kafka-init:

image: *kafka-image

networks:

- container

command: [ "/bin/bash", "-c", "/create_topic.sh"]

environment:

- KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper:2181

- TEST_TOPIC_NAME=test

depends_on:

kafka:

condition: service_started

volumes:

- type: bind

source: ./create_topic.sh

target: /create_topic.sh

init: true
```