# My Annotations, Assignment 8

## cloning the docker-compose.yml file from async 
25 cp ../course-content/07-Sourcing-Data/docker-compose.yml .

## checking that the docker-compose.yml file looks correct:
26 cat docker-compose.yml

output:
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
    extra_hosts:
      - "moby:127.0.0.1"
 
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
    extra_hosts:
      - "moby:127.0.0.1"
 
  spark:
    image: midsw205/spark-python:0.0.5
    stdin_open: true
    tty: true
    volumes:
      - ~/w205:/w205
    command: bash
    extra_hosts:
      - "moby:127.0.0.1"
 
  mids:
    image: midsw205/base:latest
    stdin_open: true
    tty: true
    volumes:
      - ~/w205:/w205
    extra_hosts:
      - "moby:127.0.0.1"
```

## getting the data
27 curl -L -o assessment-attempts-20180128-121051-nested.json https://goo.gl/ME6hjp

## spinning up cluster
28 docker-compose up -d

## checking that the cluster is coming up
29 docker-compose logs -f kafka

## checking the clusters are up
31 docker-compose ps

output:
```
                Name                            Command            State                    Ports                  
------------------------------------------------------------------------------------------------------------------
assignment07dwangischool_kafka_1       /etc/confluent/docker/run   Up      29092/tcp, 9092/tcp                     
assignment07dwangischool_mids_1        /bin/bash                   Up      8888/tcp                                
assignment07dwangischool_spark_1       docker-entrypoint.sh bash   Up                                              
assignment07dwangischool_zookeeper_1   /etc/confluent/docker/run   Up      2181/tcp, 2888/tcp, 32181/tcp, 3888/tcp 
```

## creating a topic called AssessmentData
32 docker-compose exec kafka kafka-topics --create --topic AssessmentData --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181

## check topic has been created
33 docker-compose exec kafka kafka-topics --describe --topic AssessmentData --zookeeper zookeeper:32181

output:
```
Topic:AssessmentData    PartitionCount:1        ReplicationFactor:1     Configs:
        Topic: AssessmentData   Partition: 0    Leader: 1       Replicas: 1     Isr: 1
```

## checking assessment-attempts-20180128-121051-nested.json file using ugly print:
34 docker-compose exec mids bash -c "cat /w205/assignment-07-dwang-ischool/assessment-attempts-20180128-121051-nested.json"

## checking assessment-attempts-20180128-121051-nested.json file using pretty print:
35 docker-compose exec mids bash -c "cat /w205/assignment-07-dwang-ischool/assessment-attempts-20180128-121051-nested.json | jq '.'"

## test printing assessment-attempts-20180128-121051-nested.json as individual messages:
36 docker-compose exec mids bash -c "cat /w205/assignment-07-dwang-ischool/assessment-attempts-20180128-121051-nested.json | jq '.[]' -c"

## publish messages using kafkacat:
37 docker-compose exec mids  bash -c "cat /w205/assignment-07-dwang-ischool/assessment-attempts-20180128-121051-nested.json\
 | jq '.[]' -c\
 | kafkacat -P -b kafka:29092 -t AssessmentData && echo 'Published Messages.'"
 
## run spark using the spark container:
39 docker-compose exec spark pyspark

## Following are commands run in pyspark:

### read the messages from kafka:
messages = spark \
  .read \
  .format("kafka") \
  .option("kafka.bootstrap.servers", "kafka:29092") \
  .option("subscribe","AssessmentData")\
  .option("startingOffsets", "earliest") \
  .option("endingOffsets", "latest") \
  .load() 

### view the schema:
messages.printSchema()

output:
```
root
 |-- key: binary (nullable = true)
 |-- value: binary (nullable = true)
 |-- topic: string (nullable = true)
 |-- partition: integer (nullable = true)
 |-- offset: long (nullable = true)
 |-- timestamp: timestamp (nullable = true)
 |-- timestampType: integer (nullable = true)
 ```
 
### cast messages as string:
messages_as_strings=messages.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")

### taking a look at the string messages:
messages_as_strings.show()

output:
```
| key|               value|
+----+--------------------+
|null|{"keen_timestamp"...|
|null|{"keen_timestamp"...|
|null|{"keen_timestamp"...|
|null|{"keen_timestamp"...|
|null|{"keen_timestamp"...|
|null|{"keen_timestamp"...|
|null|{"keen_timestamp"...|
|null|{"keen_timestamp"...|
|null|{"keen_timestamp"...|
|null|{"keen_timestamp"...|
|null|{"keen_timestamp"...|
|null|{"keen_timestamp"...|
|null|{"keen_timestamp"...|
|null|{"keen_timestamp"...|
|null|{"keen_timestamp"...|
|null|{"keen_timestamp"...|
|null|{"keen_timestamp"...|
|null|{"keen_timestamp"...|
|null|{"keen_timestamp"...|
|null|{"keen_timestamp"...|
+----+--------------------+
only showing top 20 rows
```

### print the schema of the string messages:
messages_as_strings.printSchema()

output:
```
root
 |-- key: string (nullable = true)
 |-- value: string (nullable = true)
```

### count the number of messages:
messages_as_strings.count()

ouput:
```
3280
```

### taking and viewing one message from the value column from messages_as_string output:
messages_as_strings.select('value').take(1)

output:
```
[Row(value='{"keen_timestamp":"1516717442.735266","max_attempts":"1.0","started_at":"2018-01-23T14:23:19.082Z","base_exam_id":"37f0a30a-7464-11e6-aa92-a8667f27e5dc","user_exam_id":"6d4089e4-bde5-4a22-b65f-18bce9ab79c8","sequences":{"questions":[{"user_incomplete":true,"user_correct":false,"options":[{"checked":true,"at":"2018-01-23T14:23:24.670Z","id":"49c574b4-5c82-4ffd-9bd1-c3358faf850d","submitted":1,"correct":true},{"checked":true,"at":"2018-01-23T14:23:25.914Z","id":"f2528210-35c3-4320-acf3-9056567ea19f","submitted":1,"correct":true},{"checked":false,"correct":true,"id":"d1bf026f-554f-4543-bdd2-54dcf105b826"}],"user_submitted":true,"id":"7a2ed6d3-f492-49b3-b8aa-d080a8aad986","user_result":"missed_some"},{"user_incomplete":false,"user_correct":false,"options":[{"checked":true,"at":"2018-01-23T14:23:30.116Z","id":"a35d0e80-8c49-415d-b8cb-c21a02627e2b","submitted":1},{"checked":false,"correct":true,"id":"bccd6e2e-2cef-4c72-8bfa-317db0ac48bb"},{"checked":true,"at":"2018-01-23T14:23:41.791Z","id":"7e0b639a-2ef8-4604-b7eb-5018bd81a91b","submitted":1,"correct":true}],"user_submitted":true,"id":"bbed4358-999d-4462-9596-bad5173a6ecb","user_result":"incorrect"},{"user_incomplete":false,"user_correct":true,"options":[{"checked":false,"at":"2018-01-23T14:23:52.510Z","id":"a9333679-de9d-41ff-bb3d-b239d6b95732"},{"checked":false,"id":"85795acc-b4b1-4510-bd6e-41648a3553c9"},{"checked":true,"at":"2018-01-23T14:23:54.223Z","id":"c185ecdb-48fb-4edb-ae4e-0204ac7a0909","submitted":1,"correct":true},{"checked":true,"at":"2018-01-23T14:23:53.862Z","id":"77a66c83-d001-45cd-9a5a-6bba8eb7389e","submitted":1,"correct":true}],"user_submitted":true,"id":"e6ad8644-96b1-4617-b37b-a263dded202c","user_result":"correct"},{"user_incomplete":false,"user_correct":true,"options":[{"checked":false,"id":"59b9fc4b-f239-4850-b1f9-912d1fd3ca13"},{"checked":false,"id":"2c29e8e8-d4a8-406e-9cdf-de28ec5890fe"},{"checked":false,"id":"62feee6e-9b76-4123-bd9e-c0b35126b1f1"},{"checked":true,"at":"2018-01-23T14:24:00.807Z","id":"7f13df9c-fcbe-4424-914f-2206f106765c","submitted":1,"correct":true}],"user_submitted":true,"id":"95194331-ac43-454e-83de-ea8913067055","user_result":"correct"}],"attempt":1,"id":"5b28a462-7a3b-42e0-b508-09f3906d1703","counts":{"incomplete":1,"submitted":4,"incorrect":1,"all_correct":false,"correct":2,"total":4,"unanswered":0}},"keen_created_at":"1516717442.735266","certification":"false","keen_id":"5a6745820eb8ab00016be1f1","exam_name":"Normal Forms and All That Jazz Master Class"}')]
```

### storing the first message:
import json
first_message=json.loads(messages_as_strings.select('value').take(1)[0].value)

### print statements that unravels the first message for some data:
print(first_message['sequences']['questions'][0])

ouput:
```
{'user_incomplete': True, 'user_correct': False, 'options': [{'checked': True, 'at': '2018-01-23T14:23:24.670Z', 'id': '49c574b4-5c82-4ffd-9bd1-c3358faf850d', 'submitted': 1, 'correct': True}, {'checked': True, 'at': '2018-01-23T14:23:25.914Z', 'id': 'f2528210-35c3-4320-acf3-9056567ea19f', 'submitted': 1, 'correct': True}, {'checked': False, 'correct': True, 'id': 'd1bf026f-554f-4543-bdd2-54dcf105b826'}], 'user_submitted': True, 'id': '7a2ed6d3-f492-49b3-b8aa-d080a8aad986', 'user_result': 'missed_some'}
```

print(first_message['sequences']['questions'][0]['options'])

output:
```
[{'checked': True, 'at': '2018-01-23T14:23:24.670Z', 'id': '49c574b4-5c82-4ffd-9bd1-c3358faf850d', 'submitted': 1, 'correct': True}, {'checked': True, 'at': '2018-01-23T14:23:25.914Z', 'id': 'f2528210-35c3-4320-acf3-9056567ea19f', 'submitted': 1, 'correct': True}, {'checked': False, 'correct': True, 'id': 'd1bf026f-554f-4543-bdd2-54dcf105b826'}]
```

print(first_message['sequences']['questions'][0]['options'][0]['id'])

output:
```
49c574b4-5c82-4ffd-9bd1-c3358faf850d
```

### exiting pyspark:
exit()

## recording the pyspark history:
40 docker-compose exec spark cat /root/.python_history

output:
```
messages = spark \
  .read \
  .format("kafka") \
  .option("kafka.bootstrap.servers", "kafka:29092") \
  .option("subscribe","AssessmentData")\
  .option("startingOffsets", "earliest") \
  .option("endingOffsets", "latest") \
  .load() 
messages.printSchema()
messages_as_strings=messages.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")
messages_as_strings.show()
messages_as_strings.printSchema()
messages_as_strings.count()
messages_as_strings.select('value').take(1)
first_message=json.loads(messages_as_strings.select('value').take(1)[0].value)
import json
first_message=json.loads(messages_as_strings.select('value').take(1)[0].value)
print(first_message['sequences']['questions'][0])
print(first_message['sequences']['questions'][0]['options'])
print(first_message['sequences']['questions'][0]['options'][0]['id'])
exit()
```

## also created pyspark history file in repo:
41 docker-compose exec spark cat /root/.python_history > pyspark_history.txt

### take down cluster:
42 docker-compose down


