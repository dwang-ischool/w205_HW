# My Annotations, Assignment 6

## going into my w205 directory
2  cd w205

## checking what is in my directory
3  ls

## generating my assignment 6 repo in my w205 directory
4  git clone https://github.com/mids-w205-martin-mims/assignment-06-dwang-ischool

## checking that the assignment 6 repo is there
5  ls

## going into the folder for my assignment 6 repo, where I will conduct all remaining commands:
6  cd assignment-06-dwang-ischool

## copying the .yml file from the kafka folder, which was created during the week 6 live session
7  cp ~/w205/kafka/docker-compose.yml .

## checking that the file was copied 
8  ls

## checking the contents of the file
9  cat docker-compose.yml

### output from previous command:
```
---
version: '2'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_CLIENT_PORT: 32181
      ZOOKEEPER_TICK_TIME: 2000
    expose:
      - "2181"
      - "2888"
      - "32181"
      - "3888"

  kafka:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:32181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:29092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
    volumes:
      - ~/w205:/w205
    expose:
      - "9092"
      - "29092"
      
  mids:
    image: midsw205/base:latest
    stdin_open: true
    tty: true
    volumes:
      - ~/w205:/w205
```

I can see that the file contains the commands to spin up the zookeeper, kafka and mids clusters, as instructed in the week 6 async.  It is important tha the lines for the mids container are included so I can use kafkacat

## download the data
10  curl -L -o assessment-attempts-20180128-121051-nested.json https://goo.gl/ME6hjp

## spinning up the kafka, zookeeper and mids clusters
11  docker-compose up -d

## watching the kafka cluster come up
12  docker-compose logs -f kafka

## checking that all three clusters are up
13  docker-compose ps

### output:
```
                Name                            Command            State                    Ports                  
------------------------------------------------------------------------------------------------------------------
assignment06dwangischool_kafka_1       /etc/confluent/docker/run   Up      29092/tcp, 9092/tcp                     
assignment06dwangischool_mids_1        /bin/bash                   Up      8888/tcp                                
assignment06dwangischool_zookeeper_1   /etc/confluent/docker/run   Up      2181/tcp, 2888/tcp, 32181/tcp, 3888/tcp 
```

## creating a topic, "foo"
15  docker-compose exec kafka kafka-topics --create --topic foo --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181

## checking that the topic "foo" was indeed created
16  docker-compose exec kafka kafka-topics --describe --topic foo --zookeeper zookeeper:32181

## checking the data
18  docker-compose exec mids bash -c "cat /w205/assignment-06-dwang-ischool/assessment-attempts-20180128-121051-nested.json"

## checking out the data by using jq to pretty print
19  docker-compose exec mids bash -c "cat /w205/assignment-06-dwang-ischool/assessment-attempts-20180128-121051-nested.json | jq '.'"

## trying another jq command to print entities as single lines, so each entity can be a separate message
20  docker-compose exec mids bash -c "cat /w205/assignment-06-dwang-ischool/assessment-attempts-20180128-121051-nested.json | jq '.[]' -c"

## publish 100 test messages to the foo topic with kafka console producer
21  docker-compose exec mids bash -c "cat /w205/assignment-06-dwang-ischool/assessment-attempts-20180128-121051-nested.json | jq '.[]' -c | kafkacat -P -b kafka:29092 -t foo && echo 'Produced 100 messages.'"

## consume 42 messages with kafka console consumer
22  docker-compose exec kafka kafka-console-consumer --bootstrap-server kafka:29092 --topic foo --from-beginning --max-messages 42

## see some messages
23  docker-compose exec mids bash -c "kafkacat -C -b kafka:29092 -t foo -o beginning -e"

## see the count of messages
24  docker-compose exec mids bash -c "kafkacat -C -b kafka:29092 -t foo -o beginning -e" | wc -l

## composing down my clusters
26  docker-compose down