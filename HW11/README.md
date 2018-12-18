# Project 3 Setup

- You're a data scientist at a game development company.  
- Your latest mobile game has two events you're interested in tracking: 
- `buy a sword` & `join guild`...
- Each has metadata

## Project 3 Task
- Your task: instrument your API server to catch and analyze event types.
- This task will be spread out over the last four assignments (9-12).

## Project 3 Task Options 

- All: Game shopping cart data used for homework 
- Advanced option 1: Generate and filter more types of items.
- Advanced option 2: Enhance the API to accept parameters for purchases (sword/item type) and filter
- Advanced option 3: Shopping cart data & track state (e.g., user's inventory) and filter


---

# Assignment 11

## Follow the steps we did in class 


### Turn in your `/assignment-10-<user-name>/README.md` file. It should include:
1) A summary type explanation of the example. 
  * For example, for Week 6's activity, a summary would be: "We spun up a cluster with kafka, zookeeper, and the mids container. Then we published and consumed messages with kafka."
2) Your `docker-compose.yml`
3) Source code for the application(s) used.
4) Each important step in the process. For each step, include:
  * The command(s) 
  * The output (if there is any).  Be sure to include examples of generated events when available.
  * An explanation for what it achieves 
    * The explanation should be fairly detailed, e.g., instead of "publish to kafka" say what you're publishing, where it's coming from, going to etc.


# Class Activity Summary

1. First I cloned the assignment 11 repo into my droplet and navigated to the repo's folder:
```
git clone https://github.com/mids-w205-martin-mims/assignment-11-dwang-ischool
cd assignment-11-dwang-ischool/
```

2. While inside the assignment 10 repo, I copied the docker-compose.yml file and python files from the course content folder for unit 11:
```
~/w205/course-content/11-Storing-Data-III/docker-compose.yml .
~/w205/course-content/11-Storing-Data-III/*.py .
```

3. Next I check the docker-compose.yml file to make sure that it contains the zookeeper, kafka, spark, hadoop, and mids containers:
```
cat docker-compose.yml
```

I see the following output:
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
    expose:
      - "9092"
      - "29092"
    extra_hosts:
      - "moby:127.0.0.1"

  cloudera:
    image: midsw205/cdh-minimal:latest
    expose:
      - "8020" # nn
      - "50070" # nn http
      - "8888" # hue
    #ports:
    #- "8888:8888"
    extra_hosts:
      - "moby:127.0.0.1"

  spark:
    image: midsw205/spark-python:0.0.5
    stdin_open: true
    tty: true
    expose:
      - "8888"
    ports:
      - "8888:8888"
    volumes:
      - "~/w205:/w205"
    command: bash
    depends_on:
      - cloudera
    environment:
      HADOOP_NAMENODE: cloudera
    extra_hosts:
      - "moby:127.0.0.1"

  mids:
    image: midsw205/base:latest
    stdin_open: true
    tty: true
    expose:
      - "5000"
    ports:
      - "5000:5000"
    volumes:
      - "~/w205:/w205"
    extra_hosts:
      - "moby:127.0.0.1"
```

4. I spin up the cluster with zookeper, kafka, hadoop, spark and mids components. Then I check tht the containers are active:
```
docker-compose up -d
docker-compose ps
```
I get the following output, which shows that the containers did successfully spin up:
```
            Name                         Command                         State                          Ports
-------------------------------------------------------------------------------------------------------------------------
assignment11dwangischool_clo   cdh_startup_script.sh          Up                             11000/tcp, 11443/tcp,
udera_1                                                                                      19888/tcp, 50070/tcp,
                                                                                             8020/tcp, 8088/tcp,
                                                                                             8888/tcp, 9090/tcp
assignment11dwangischool_kaf   /etc/confluent/docker/run      Up                             29092/tcp, 9092/tcp
ka_1
assignment11dwangischool_mid   /bin/bash                      Up                             0.0.0.0:5000->5000/tcp,
s_1                                                                                          8888/tcp
assignment11dwangischool_spa   docker-entrypoint.sh bash      Up                             0.0.0.0:8888->8888/tcp
rk_1
assignment11dwangischool_zoo   /etc/confluent/docker/run      Up                             2181/tcp, 2888/tcp,
keeper_1                                                                                     32181/tcp, 3888/tcp
```

5. I use the following command to check the hadoop logs:
```
docker-compose logs -f cloudera
```
I get the following output and so have confirmation that hdfs is running properly:
```
Attaching to assignment11dwangischool_cloudera_1
cloudera_1   | Start HDFS
cloudera_1   | starting datanode, logging to /var/log/hadoop-hdfs/hadoop-hdfs-datanode-bfea404d947f.out
cloudera_1   |  * Started Hadoop datanode (hadoop-hdfs-datanode):
cloudera_1   | starting namenode, logging to /var/log/hadoop-hdfs/hadoop-hdfs-namenode-bfea404d947f.out
cloudera_1   |  * Started Hadoop namenode:
cloudera_1   | starting secondarynamenode, logging to /var/log/hadoop-hdfs/hadoop-hdfs-secondarynamenode-bfea404d947f.out
cloudera_1   |  * Started Hadoop secondarynamenode:
cloudera_1   | Start Components
cloudera_1   | Press Ctrl+P and Ctrl+Q to background this process.
cloudera_1   | Use exec command to open a new bash instance for this instance (Eg. "docker exec -i -tCONTAINER_ID bash"). Container ID can be obtained using "docker ps" command.
cloudera_1   | Start Terminal
cloudera_1   | Press Ctrl+C to stop instance.
```

6. I check out the current state of hadoop using the following command:
```
docker-compose exec cloudera hadoop fs -ls /tmp/
```
and I see the following output:
```
Found 2 items
drwxrwxrwt   - mapred mapred              0 2018-02-06 18:27 /tmp/hadoop-yarn
drwx-wx-wx   - root   supergroup          0 2018-12-08 17:51 /tmp/hive
```

7. Next I create my "events" topic:
```
docker-compose exec kafka kafka-topics --create --topic events --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181
```

8. Before I run the game_api.py file in flask, I check the contents of the file:
```
cat game_api.py
```
and see the following contents:
```python
#!/usr/bin/env python
import json
from kafka import KafkaProducer
from flask import Flask, request

app = Flask(__name__)
producer = KafkaProducer(bootstrap_servers='kafka:29092')


def log_to_kafka(topic, event):
    event.update(request.headers)
    producer.send(topic, json.dumps(event).encode())


@app.route("/")
def default_response():
    default_event = {'event_type': 'default'}
    log_to_kafka('events', default_event)
    return "This is the default response!\n"


@app.route("/purchase_a_sword")
def purchase_a_sword():
    purchase_sword_event = {'event_type': 'purchase_sword'}
    log_to_kafka('events', purchase_sword_event)
    return "Sword Purchased!\n"
```
I see that this file resembles the game_api_with_extended_json_events.py file from unit 10 homework, except there are only 2 types of evens instead of 3. As before, the two actions that map to API calls are the default action and the purchase_a_sword action.  The most important thing is that events generated will be very informative as they contain information from event.update(request.headers). 

Now in my main window, I will run game_api.py file using flask app:
```
docker-compose exec mids env FLASK_APP=/w205/assignment-11-dwang-ischool/game_api.py flask run --host 0.0.0.0
```
In a separate terminal window, I generate some events using the following commands:
```
docker-compose exec mids curl http://localhost:5000/
docker-compose exec mids curl http://localhost:5000/
docker-compose exec mids curl http://localhost:5000/purchase_a_sword
docker-compose exec mids curl http://localhost:5000/purchase_a_sword
docker-compose exec mids curl http://localhost:5000/purchase_a_sword
docker-compose exec mids curl http://localhost:5000/
docker-compose exec mids curl http://localhost:5000/purchase_a_sword
docker-compose exec mids curl http://localhost:5000/
docker-compose exec mids curl http://localhost:5000/purchase_a_sword
```
In the original window where flask is running, I see the following output in the flask app:
```
 * Serving Flask app "game_api"
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
127.0.0.1 - - [08/Dec/2018 18:45:14] "GET / HTTP/1.1" 200 -
127.0.0.1 - - [08/Dec/2018 18:45:19] "GET / HTTP/1.1" 200 -
127.0.0.1 - - [08/Dec/2018 18:45:27] "GET /purchase_a_sword HTTP/1.1" 200 -
127.0.0.1 - - [08/Dec/2018 18:45:29] "GET /purchase_a_sword HTTP/1.1" 200 -
127.0.0.1 - - [08/Dec/2018 18:45:32] "GET /purchase_a_sword HTTP/1.1" 200 -
127.0.0.1 - - [08/Dec/2018 18:46:19] "GET / HTTP/1.1" 200 -
127.0.0.1 - - [08/Dec/2018 18:46:24] "GET /purchase_a_sword HTTP/1.1" 200 -
127.0.0.1 - - [08/Dec/2018 18:46:30] "GET / HTTP/1.1" 200 -
127.0.0.1 - - [08/Dec/2018 18:46:32] "GET /purchase_a_sword HTTP/1.1" 200 -
```
Each line of the flask app output corresponds simultaneously to each of the 9 actions I performed.  As expected they appear in the following order - default, default, purchase a sword, purchase a sword, default, purchase a sword, default, purchase a sword. 

In the separate terminal window, I use the following command to read the events from Kafka:
```
docker-compose exec mids kafkacat -C -b kafka:29092 -t events -o beginning -e
```
And I see the following output which for the 9 events which I generated.  Again, they match the exact order of my actions - default, default, purchase a sword, purchase a sword, default, purchase a sword, default, purchase a sword
```
{"Host": "localhost:5000", "event_type": "default", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
{"Host": "localhost:5000", "event_type": "default", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
{"Host": "localhost:5000", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
{"Host": "localhost:5000", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
{"Host": "localhost:5000", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
{"Host": "localhost:5000", "event_type": "default", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
{"Host": "localhost:5000", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
{"Host": "localhost:5000", "event_type": "default", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
{"Host": "localhost:5000", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
% Reached end of topic events [0] at offset 9: exiting
```

9. Unlike homework 10, I will not be running pyspark interactive window this time.  Instead, we will use spark to write our data as parquet files to HDFS using pyspark code in various python files.  The first one is the extract_events.py file. 

First i check the contents of the file:
```
cat extract_events.py
```
Which has the following output:
```python
#!/usr/bin/env python
"""Extract events from kafka and write them to hdfs
"""

import json
from pyspark.sql import SparkSession


def main():
    """main
    """
    spark = SparkSession \
        .builder \
        .appName("ExtractEventsJob") \
        .getOrCreate()

    raw_events = spark \
        .read \
        .format("kafka") \
        .option("kafka.bootstrap.servers", "kafka:29092") \
        .option("subscribe", "events") \
        .option("startingOffsets", "earliest") \
        .option("endingOffsets", "latest") \
        .load()

    events = raw_events.select(raw_events.value.cast('string'))
    extracted_events = events.rdd.map(lambda x: json.loads(x.value)).toDF()
    extracted_events.show()

    extracted_events \
        .write \
        .parquet("/tmp/extracted_events")


if __name__ == "__main__":
    main()
```
We see that the contents of this file has commands similar to what we ran as line by line in the pyspark interactive windwow last time. In addition to the file that was provided in course content folder, I added a line "extracted_events.show()" so that I can see the extracted_events dataframe that will be written to hdfs in the output logs.  The difference from last homework is that  the commands at once from the extract_events.py file using the following spark-submit command in terminal window:
```
docker-compose exec spark spark-submit /w205/assignment-11-dwang-ischool/extract_events.py
```
From the logs that outputted, I do not see any errors, so I can assume that my file was written to hadoop correctly. The most important output is the following dataframe that I can see from the extracted_events.show() in the extract_events.py file:
```
+------+--------------+-----------+--------------+
|Accept|          Host| User-Agent|    event_type|
+------+--------------+-----------+--------------+
|   */*|localhost:5000|curl/7.47.0|       default|
|   */*|localhost:5000|curl/7.47.0|       default|
|   */*|localhost:5000|curl/7.47.0|purchase_sword|
|   */*|localhost:5000|curl/7.47.0|purchase_sword|
|   */*|localhost:5000|curl/7.47.0|purchase_sword|
|   */*|localhost:5000|curl/7.47.0|       default|
|   */*|localhost:5000|curl/7.47.0|purchase_sword|
|   */*|localhost:5000|curl/7.47.0|       default|
|   */*|localhost:5000|curl/7.47.0|purchase_sword|
+------+--------------+-----------+--------------+
```
As we can see from this output that the 9 events which I generated was extracted and written into a dataframe where the rows correspond to each event.  Every event from our output in 8 is a dictionary of elements, where the dictionary keys are now the column headers and the values consists of the entries in each column.  

I then check hadoop using:
```
docker-compose exec cloudera hadoop fs -ls /tmp/
```
And I see the following output:
```
Found 3 items
drwxr-xr-x   - root   supergroup          0 2018-12-08 19:08 /tmp/extracted_events
drwxrwxrwt   - mapred mapred              0 2018-02-06 18:27 /tmp/hadoop-yarn
drwx-wx-wx   - root   supergroup          0 2018-12-08 17:51 /tmp/hive
```
We can see that the extracted_events file was created successfully.  We can examine it even further using the following command:
```
docker-compose exec cloudera hadoop fs -ls /tmp/extracted_events/
```
Which has the following output:
```
Found 2 items
-rw-r--r--   1 root supergroup          0 2018-12-08 19:08 /tmp/extracted_events/_SUCCESS
-rw-r--r--   1 root supergroup       1189 2018-12-08 19:08 /tmp/extracted_events/part-00000-2568c9a6-1b2d-4309-abf4-cee013b3f55f-c000.snappy.parquet
```

10.  In a second example, I will now run the transform_events.py using spark-submit command. Before doing so, I take a look at the contents of my transform_events.py file:
```python
#!/usr/bin/env python
"""Extract events from kafka and write them to hdfs
"""
import json
from pyspark.sql import SparkSession, Row
from pyspark.sql.functions import udf


@udf('string')
def munge_event(event_as_json):
    event = json.loads(event_as_json)
    event['Host'] = "moe"
    event['Cache-Control'] = "no-cache"
    return json.dumps(event)


def main():
    """main
    """
    spark = SparkSession \
        .builder \
        .appName("ExtractEventsJob") \
        .getOrCreate()

    raw_events = spark \
        .read \
        .format("kafka") \
        .option("kafka.bootstrap.servers", "kafka:29092") \
        .option("subscribe", "events") \
        .option("startingOffsets", "earliest") \
        .option("endingOffsets", "latest") \
        .load()

    munged_events = raw_events \
        .select(raw_events.value.cast('string').alias('raw'),
                raw_events.timestamp.cast('string')) \
        .withColumn('munged', munge_event('raw'))
    munged_events.show()

    extracted_events = munged_events \
        .rdd \
        .map(lambda r: Row(timestamp=r.timestamp, **json.loads(r.munged))) \
        .toDF()
    extracted_events.show()

    extracted_events \
        .write \
        .mode("overwrite") \
        .parquet("/tmp/extracted_events")


if __name__ == "__main__":
    main()
```

Unlike the extract_events.py file, this spark file contains several more commands.  Like before, we first extract the raw_events from the events topic.  However, we are now creating a munged_events intermediate dataframe consisting of 3 columns - raw events, timestamp and munged.  The munged column in the munged_events dataframe will be created by passing the raw_events through the munge_event function, where the json event is turned into a dictionary, the local host is set to "moe", and then the dictionary is transformed back to a string. The extracted_events dataframe, which is the one written to the parquet file, is then created using the munged_events dataframe.  

We will see 2 dataframe outputs in the logs - munged_events and extracted_events. 

First I run the spark-submit command:
```
docker-compose exec spark spark-submit /w205/assignment-11-dwang-ischool/transform_events.py
```
I see no error statements in the output, so I assume that everything ran succssfully.  When I scroll through the output logs, I see the following intermediate table for munged_events.  We can see the munged column and that the localhost has been changed to "moe". 
```
+--------------------+--------------------+--------------------+
|                 raw|           timestamp|              munged|
+--------------------+--------------------+--------------------+
|{"Host": "localho...|2018-12-08 18:45:...|{"Host": "moe", "...|
|{"Host": "localho...|2018-12-08 18:45:...|{"Host": "moe", "...|
|{"Host": "localho...|2018-12-08 18:45:...|{"Host": "moe", "...|
|{"Host": "localho...|2018-12-08 18:45:...|{"Host": "moe", "...|
|{"Host": "localho...|2018-12-08 18:45:...|{"Host": "moe", "...|
|{"Host": "localho...|2018-12-08 18:46:...|{"Host": "moe", "...|
|{"Host": "localho...|2018-12-08 18:46:...|{"Host": "moe", "...|
|{"Host": "localho...|2018-12-08 18:46:...|{"Host": "moe", "...|
|{"Host": "localho...|2018-12-08 18:46:...|{"Host": "moe", "...|
+--------------------+--------------------+--------------------+
```
Later on in the output, I see the second dataframe that corresponds to extracted_events and is the one that is written to hdfs:
```
+------+-------------+----+-----------+--------------+--------------------+
|Accept|Cache-Control|Host| User-Agent|    event_type|           timestamp|
+------+-------------+----+-----------+--------------+--------------------+
|   */*|     no-cache| moe|curl/7.47.0|       default|2018-12-08 18:45:...|
|   */*|     no-cache| moe|curl/7.47.0|       default|2018-12-08 18:45:...|
|   */*|     no-cache| moe|curl/7.47.0|purchase_sword|2018-12-08 18:45:...|
|   */*|     no-cache| moe|curl/7.47.0|purchase_sword|2018-12-08 18:45:...|
|   */*|     no-cache| moe|curl/7.47.0|purchase_sword|2018-12-08 18:45:...|
|   */*|     no-cache| moe|curl/7.47.0|       default|2018-12-08 18:46:...|
|   */*|     no-cache| moe|curl/7.47.0|purchase_sword|2018-12-08 18:46:...|
|   */*|     no-cache| moe|curl/7.47.0|       default|2018-12-08 18:46:...|
|   */*|     no-cache| moe|curl/7.47.0|purchase_sword|2018-12-08 18:46:...|
+------+-------------+----+-----------+--------------+--------------------+
```
As with the example in #9, we see that our events have once again successfully been written into a dataframe where the dictionary values have been parsed into columns and the dictionary keys are the headers.  The major difference in this version, which we did not have in the previous version is that we now have a column for the timestamp during which the event was created.  In addition, we can see again that the Host, which used to be localhost, is now "moe"

11. Lastly, I run the 3rd example using the file separate_events.py to handle different event types that have different schema. In the case of events with a different numbers of elements, we will not be able to use the transform_events method because we cannot build dataframes where rows have different number of columns.  

First I check the contents of the separate_events.py file:
```python
#!/usr/bin/env python
"""Extract events from kafka and write them to hdfs
"""
import json
from pyspark.sql import SparkSession, Row
from pyspark.sql.functions import udf


@udf('string')
def munge_event(event_as_json):
    event = json.loads(event_as_json)
    event['Host'] = "moe"
    event['Cache-Control'] = "no-cache"
    return json.dumps(event)


def main():
    """main
    """
    spark = SparkSession \
        .builder \
        .appName("ExtractEventsJob") \
        .getOrCreate()

    raw_events = spark \
        .read \
        .format("kafka") \
        .option("kafka.bootstrap.servers", "kafka:29092") \
        .option("subscribe", "events") \
        .option("startingOffsets", "earliest") \
        .option("endingOffsets", "latest") \
        .load()

    munged_events = raw_events \
        .select(raw_events.value.cast('string').alias('raw'),
                raw_events.timestamp.cast('string')) \
        .withColumn('munged', munge_event('raw'))

    extracted_events = munged_events \
        .rdd \
        .map(lambda r: Row(timestamp=r.timestamp, **json.loads(r.munged))) \
        .toDF()

    sword_purchases = extracted_events \
        .filter(extracted_events.event_type == 'purchase_sword')
    sword_purchases.show()
    sword_purchases \
        .write \
        .mode("overwrite") \
        .parquet("/tmp/sword_purchases")

    default_hits = extracted_events \
        .filter(extracted_events.event_type == 'default')
    default_hits.show()
    default_hits \
        .write \
        .mode("overwrite") \
        .parquet("/tmp/default_hits")


if __name__ == "__main__":
    main()
```

The main difference in this file from the exercise in #10 is that we are filtering out events by the same event type into 2 separate datafromes - sword_purchases and default_hits.  We will also be writing these as spearate files into hdfs. 

This time, when we run this, we will not see the munged_events or extracted_events because the .show() commands are missing.  Instead, we expect to see in the logs the sword_purchases dataframe and the default_hits dataframe from their respective .show() commands in the above file.  

I will run the separate_events.py file using spark-submit:
```
docker-compose exec spark spark-submit /w205/assignment-11-dwang-ischool/separate_events.py
```

I see no error messages, which tells me that it ran successfully.  As expected from the code in the separate_events.py file, I can see 2 separate dataframes in my logs.  The first is one has filtered all of the purchase sword events from my original set of generated events.  In #8, I generated a total of 9 events, where 5 of them are purchase_sword events.  We see that they have successfully been extracted and transfromed into the sword_purchases dataframe. 

```
+------+-------------+----+-----------+--------------+--------------------+
|Accept|Cache-Control|Host| User-Agent|    event_type|           timestamp|
+------+-------------+----+-----------+--------------+--------------------+
|   */*|     no-cache| moe|curl/7.47.0|purchase_sword|2018-12-08 18:45:...|
|   */*|     no-cache| moe|curl/7.47.0|purchase_sword|2018-12-08 18:45:...|
|   */*|     no-cache| moe|curl/7.47.0|purchase_sword|2018-12-08 18:45:...|
|   */*|     no-cache| moe|curl/7.47.0|purchase_sword|2018-12-08 18:46:...|
|   */*|     no-cache| moe|curl/7.47.0|purchase_sword|2018-12-08 18:46:...|
+------+-------------+----+-----------+--------------+--------------------+
```

Next in the logs, I see a second dataframe for my default action events generated in #8.  Of the 9 events I generated, 4 were default actions and I can see from this table in the output logs that my default action events were successfully extracted and transformed into the default_hits dataframe:
```
+------+-------------+----+-----------+----------+--------------------+
|Accept|Cache-Control|Host| User-Agent|event_type|           timestamp|
+------+-------------+----+-----------+----------+--------------------+
|   */*|     no-cache| moe|curl/7.47.0|   default|2018-12-08 18:45:...|
|   */*|     no-cache| moe|curl/7.47.0|   default|2018-12-08 18:45:...|
|   */*|     no-cache| moe|curl/7.47.0|   default|2018-12-08 18:46:...|
|   */*|     no-cache| moe|curl/7.47.0|   default|2018-12-08 18:46:...|
+------+-------------+----+-----------+----------+--------------------+
```

Lastly, I run the following command to check the status of hadoop:
```
docker-compose exec cloudera hadoop fs -ls /tmp/
```
and see the following output:
```
drwxr-xr-x   - root   supergroup          0 2018-12-08 20:44 /tmp/default_hits
drwxr-xr-x   - root   supergroup          0 2018-12-08 20:16 /tmp/extracted_events
drwxrwxrwt   - mapred mapred              0 2018-02-06 18:27 /tmp/hadoop-yarn
drwx-wx-wx   - root   supergroup          0 2018-12-08 17:51 /tmp/hive
drwxr-xr-x   - root   supergroup          0 2018-12-08 20:44 /tmp/sword_purchases
```
which shows that default_hits and sword_purchase dataframes were successfully written into parquet files in hdfs per their respective commands in the separate_events.py file. 


12. Having successfully completed the exercise, I exit flask app and take down my contains:
```
docker-compose down
```

And save my commands from the two terminal windows:

original terminal window where I started the cluster and ran flask app:
```
history > original_window.txt
```
separate terminal window where I read from kafka, ran spark-submit commands, and checked hadoop:
```
history > separate_window.txt
```






```