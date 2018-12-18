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

# Assignment 12

## Do the following:
- Spin up the cluster with the necessary containers.
- Run your web app.
- Generate data.
- Write events & check to make sure you were successful.
- Run queries with Presto - at least Select * from <your-table-name>

### Turn in your `/assignment-12-<user-name>/README.md` file. It should include:
1) A summary type explanation of the example. 
  * For example, for Week 6's activity, a summary would be: "We spun up a cluster with kafka, zookeeper, and the mids container. Then we published and consumed messages with kafka."
2) Your `docker-compose.yml`
3) Source code for the application(s) used.
4) Each important step in the process. For each step, include:
  * The command(s) 
  * The output (if there is any).  Be sure to include examples of generated events when available.
  * An explanation for what it achieves 
    * The explanation should be fairly detailed, e.g., instead of "publish to kafka" say what you're publishing, where it's coming from, going to etc.


# Assignment 12 Summary

1. For assignment 12, I copied the yaml file from the course content week 13 folder, which includes the zookeeper, kafka, hadoop, spark, presto and mids containers.  I also pulled the images for the containers and copied the python files:
```
cp ~/w205/course-content/13-Understanding-Data/docker-compose.yml .
docker-compose pull
cp ~/w205/course-content/13-Understanding-Data/*.py .
```

checking that the file matches the course content:
```
cat docker-compose.yml
```
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
    expose:
      - "9092"
      - "29092"
    extra_hosts:
      - "moby:127.0.0.1"

  cloudera:
    image: midsw205/hadoop:0.0.2
    hostname: cloudera
    expose:
      - "8020" # nn
      - "8888" # hue
      - "9083" # hive thrift
      - "10000" # hive jdbc
      - "50070" # nn http
    ports:
      - "8888:8888"
    extra_hosts:
      - "moby:127.0.0.1"

  spark:
    image: midsw205/spark-python:0.0.6
    stdin_open: true
    tty: true
    volumes:
      - ~/w205:/w205
    expose:
      - "8888"
    #ports:
    #  - "8888:8888"
    depends_on:
      - cloudera
    environment:
      HADOOP_NAMENODE: cloudera
      HIVE_THRIFTSERVER: cloudera:9083
    extra_hosts:
      - "moby:127.0.0.1"
    command: bash

  presto:
    image: midsw205/presto:0.0.1
    hostname: presto
    volumes:
      - ~/w205:/w205
    expose:
      - "8080"
    environment:
      HIVE_THRIFTSERVER: cloudera:9083
    extra_hosts:
      - "moby:127.0.0.1"
 
  mids:
    image: midsw205/base:0.1.9
    stdin_open: true
    tty: true
    volumes:
      - ~/w205:/w205
    expose:
      - "5000"
    ports:
      - "5000:5000"
    extra_hosts:
      - "moby:127.0.0.1"
```

2. I spin up the cluster with all container components. Then I check tht the containers are active:
```
docker-compose up -d
docker-compose ps
```
output:
```
             Name                           Command                           State                            Ports              
---------------------------------------------------------------------------------------------------------------------------------
assignment12dwangischool_cloud   /usr/bin/docker-entrypoint ...   Up                               10000/tcp, 50070/tcp,          
era_1                                                                                              8020/tcp,                      
                                                                                                   0.0.0.0:8888->8888/tcp,        
                                                                                                   9083/tcp                       
assignment12dwangischool_kafka   /etc/confluent/docker/run        Up                               29092/tcp, 9092/tcp            
_1                                                                                                                                
assignment12dwangischool_mids_   /bin/bash                        Up                               0.0.0.0:5000->5000/tcp,        
1                                                                                                  8888/tcp                       
assignment12dwangischool_prest   /usr/bin/docker-entrypoint ...   Up                               8080/tcp                       
o_1                                                                                                                               
assignment12dwangischool_spark   docker-entrypoint.sh bash        Up                               8888/tcp                       
_1                                                                                                                                
assignment12dwangischool_zooke   /etc/confluent/docker/run        Up                               2181/tcp, 2888/tcp, 32181/tcp, 
eper_1                                                                                             3888/tcp        
```

3. Next I run game_api.py using the Flask app.  Prior to running the file, I check the contents:
```
cat game_api.py
```
output:
```
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
We see that this file matches the one used in assignment 11 exactly.  There are 2 events total, the default response and purchase a sword.  Each time the an event is created through an API call mapped action, the event  is logged to Kafka with informative request.headers. 

I use the following command to run game_api.py using flask in a second window:
```
docker-compose exec mids env FLASK_APP=/w205/assignment-12-dwang-ischool/game_api.py flask run --host 0.0.0.0
```
initial ouptut:
```
 * Serving Flask app "game_api" 
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
```

4. In a third window, I first create an "events" topic:
```
docker-compose exec kafka kafka-topics --create --topic events --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181
```
I then use the following command to monitor kafka:
```
docker-compose exec mids kafkacat -C -b kafka:29092 -t events -o beginning
```

5. Next, I generate batch events using Apache bench.  

First I generate 10 default response events by user1.comcast.com using the following command to create 10 batched actions:
```
docker-compose exec mids \
  ab \
    -n 10 \
    -H "Host: user1.comcast.com" \
    http://localhost:5000/
```
I get the following output in the initial window:
```
This is ApacheBench, Version 2.3 <$Revision: 1706008 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/
 
Benchmarking localhost (be patient).....done
 
 
Server Software:        Werkzeug/0.14.1
Server Hostname:        localhost
Server Port:            5000
 
Document Path:          /
Document Length:        17 bytes
 
Concurrency Level:      1
Time taken for tests:   0.072 seconds
Complete requests:      10 
Failed requests:        0 
Total transferred:      1850 bytes 
HTML transferred:       300 bytes 
Requests per second:    209.45 [#/sec] (mean) 
Time per request:       4.775 [ms] (mean) 
Time per request:       4.775 [ms] (mean, across all concurrent re 
Transfer rate:          37.84 [Kbytes/sec] received 
 
Connection Times (ms) 
              min  mean[+/-sd] median   max 
Connect:        0    0   0.1      0       0 
Processing:     1    5   5.6      3      20 
Waiting:        0    2   1.3      1       3 
Total:          1    5   5.7      4      20 
 
Percentage of the requests served within a certain time (ms) 
  50%      4 
  66%      4 
  75%      4 
```
In my second window where flask is running, I see this output for 10 api calls matching the batched actions:
```
127.0.0.1 - - [17/Dec/2018 10:32:42] "GET / HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:32:42] "GET / HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:32:42] "GET / HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:32:42] "GET / HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:32:42] "GET / HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:32:42] "GET / HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:32:42] "GET / HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:32:42] "GET / HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:32:42] "GET / HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:32:42] "GET / HTTP/1.0" 200 -
```
In my third window where I am monitoring kafka in real time, I see the following events generated:
```
{"Host": "user1.comcast.com", "event_type": "default", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user1.comcast.com", "event_type": "default", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user1.comcast.com", "event_type": "default", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user1.comcast.com", "event_type": "default", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user1.comcast.com", "event_type": "default", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user1.comcast.com", "event_type": "default", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user1.comcast.com", "event_type": "default", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user1.comcast.com", "event_type": "default", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user1.comcast.com", "event_type": "default", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user1.comcast.com", "event_type": "default", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
```

Next I generate 10 purchase sword events in a batch by user1.comcast.com:
```
docker-compose exec mids \
  ab \
    -n 10 \
    -H "Host: user1.comcast.com" \
    http://localhost:5000/purchase_a_sword
```
I see the following output in my primary terminal window:
```
This is ApacheBench, Version 2.3 <$Revision: 1706008 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/
 
Benchmarking localhost (be patient).....done
 
 
Server Software:        Werkzeug/0.14.1
Server Hostname:        localhost
Server Port:            5000
 
Document Path:          /purchase_a_sword
Document Length:        17 bytes
 
Concurrency Level:      1
Time taken for tests:   0.072 seconds
Complete requests:      10
Failed requests:        0
Total transferred:      1720 bytes
HTML transferred:       170 bytes
Requests per second:    139.34 [#/sec] (mean)
Time per request:       7.177 [ms] (mean)
Time per request:       7.177 [ms] (mean, across all concurrent requests)
Transfer rate:          23.40 [Kbytes/sec] received
 
Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.2      0       1
Processing:     3    7   4.4      7      18
Waiting:        2    5   2.3      6      10
Total:          3    7   4.6      7      19
 
Percentage of the requests served within a certain time (ms)
  50%      7
  66%      7
  75%      7
  80%     11
  90%     19
  95%     19
  98%     19
  99%     19
 100%     19 (longest request)
```
In my second terminal windwo where I am monitoring flask, I see the following:
```
127.0.0.1 - - [17/Dec/2018 10:32:42] "GET / HTTP/1.0" 200 - 
127.0.0.1 - - [17/Dec/2018 10:32:42] "GET / HTTP/1.0" 200 - 
127.0.0.1 - - [17/Dec/2018 10:32:42] "GET / HTTP/1.0" 200 - 
127.0.0.1 - - [17/Dec/2018 10:32:42] "GET / HTTP/1.0" 200 - 
127.0.0.1 - - [17/Dec/2018 10:32:42] "GET / HTTP/1.0" 200 - 
127.0.0.1 - - [17/Dec/2018 10:32:42] "GET / HTTP/1.0" 200 - 
127.0.0.1 - - [17/Dec/2018 10:32:42] "GET / HTTP/1.0" 200 - 
127.0.0.1 - - [17/Dec/2018 10:32:42] "GET / HTTP/1.0" 200 - 
127.0.0.1 - - [17/Dec/2018 10:32:42] "GET / HTTP/1.0" 200 - 
127.0.0.1 - - [17/Dec/2018 10:32:42] "GET / HTTP/1.0" 200 - 
127.0.0.1 - - [17/Dec/2018 10:40:21] "GET /purchase_a_sword HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:40:21] "GET /purchase_a_sword HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:40:21] "GET /purchase_a_sword HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:40:21] "GET /purchase_a_sword HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:40:21] "GET /purchase_a_sword HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:40:21] "GET /purchase_a_sword HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:40:21] "GET /purchase_a_sword HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:40:21] "GET /purchase_a_sword HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:40:21] "GET /purchase_a_sword HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:40:21] "GET /purchase_a_sword HTTP/1.0" 200 -
```
Which shows the 20 API calls that matches the 20 actions I have generated thus far. 

In my third terminal window where I am monitoring kafka, I see the following:
```
{"Host": "user1.comcast.com", "event_type": "default", "Accept": "*/*", "User-Age 
nt": "ApacheBench/2.3"} 
{"Host": "user1.comcast.com", "event_type": "default", "Accept": "*/*", "User-Age 
nt": "ApacheBench/2.3"} 
{"Host": "user1.comcast.com", "event_type": "default", "Accept": "*/*", "User-Age 
nt": "ApacheBench/2.3"} 
{"Host": "user1.comcast.com", "event_type": "default", "Accept": "*/*", "User-Age 
nt": "ApacheBench/2.3"} 
{"Host": "user1.comcast.com", "event_type": "default", "Accept": "*/*", "User-Age 
nt": "ApacheBench/2.3"} 
{"Host": "user1.comcast.com", "event_type": "default", "Accept": "*/*", "User-Age 
nt": "ApacheBench/2.3"} 
{"Host": "user1.comcast.com", "event_type": "default", "Accept": "*/*", "User-Age 
nt": "ApacheBench/2.3"} 
{"Host": "user1.comcast.com", "event_type": "default", "Accept": "*/*", "User-Age 
nt": "ApacheBench/2.3"} 
{"Host": "user1.comcast.com", "event_type": "default", "Accept": "*/*", "User-Age 
nt": "ApacheBench/2.3"} 
{"Host": "user1.comcast.com", "event_type": "default", "Accept": "*/*", "User-Age 
nt": "ApacheBench/2.3"} 
{"Host": "user1.comcast.com", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user1.comcast.com", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user1.comcast.com", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user1.comcast.com", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user1.comcast.com", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user1.comcast.com", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user1.comcast.com", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user1.comcast.com", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user1.comcast.com", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user1.comcast.com", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
```
Which are the 20 events I have generated thus far by user1.comcast.com, who performed 10 default actions and 10 purchase sword actions. 

Next I generate 20 more events, 10 default and 10 purchase sword actions, by user2.att.com, using the following 2 commands:
```
docker-compose exec mids \
  ab \
    -n 10 \
    -H "Host: user2.att.com" \
    http://localhost:5000/
    
docker-compose exec mids \
  ab \
    -n 10 \
    -H "Host: user2.att.com" \
    http://localhost:5000/purchase_a_sword
```

Initial outputs in my primary terminal window:
default:
```
This is ApacheBench, Version 2.3 <$Revision: 1706008 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking localhost (be patient).....done


Server Software:        Werkzeug/0.14.1
Server Hostname:        localhost
Server Port:            5000

Document Path:          /
Document Length:        30 bytes

Concurrency Level:      1
Time taken for tests:   0.052 seconds
Complete requests:      10
Failed requests:        0
Total transferred:      1850 bytes
HTML transferred:       300 bytes
Requests per second:    191.58 [#/sec] (mean)
Time per request:       5.220 [ms] (mean)
Time per request:       5.220 [ms] (mean, across all concurrent requests)
Transfer rate:          34.61 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.1      0       0
Processing:     2    5   3.4      4      13
Waiting:        0    3   2.7      2       8
Total:          2    5   3.4      4      13
 
Percentage of the requests served within a certain time (ms)
  50%      4
  66%      5
  75%      7
  80%      9
  90%     13
  95%     13
  98%     13
  99%     13
 100%     13 (longest request)
```
puchase sword:
```
This is ApacheBench, Version 2.3 <$Revision: 1706008 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking localhost (be patient).....done


Server Software:        Werkzeug/0.14.1
Server Hostname:        localhost
Server Port:            5000

Document Path:          /purchase_a_sword
Document Length:        17 bytes

Concurrency Level:      1
Time taken for tests:   0.083 seconds
Complete requests:      10
Failed requests:        0
Total transferred:      1720 bytes
HTML transferred:       170 bytes
Requests per second:    120.67 [#/sec] (mean)
Time per request:       8.287 [ms] (mean)
Time per request:       8.287 [ms] (mean, across all concurrent requests)
Transfer rate:          20.27 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.3      0       1
Processing:     3    8   3.8      8      17
Waiting:        0    6   5.1      5      17
Total:          3    8   3.8      8      17

Percentage of the requests served within a certain time (ms)
  50%      8
  66%      9
  75%     10
  80%     10
  90%     17
  95%     17
  98%     17
  99%     17
 100%     17 (longest request)
```

In my 2nd terminal window where flask is running, I see the following 40 api calls that match the 20 actions performed by the 2 users:
```
 * Serving Flask app "game_api" 
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit) 
127.0.0.1 - - [17/Dec/2018 10:32:42] "GET / HTTP/1.0" 200 - 
127.0.0.1 - - [17/Dec/2018 10:32:42] "GET / HTTP/1.0" 200 - 
127.0.0.1 - - [17/Dec/2018 10:32:42] "GET / HTTP/1.0" 200 - 
127.0.0.1 - - [17/Dec/2018 10:32:42] "GET / HTTP/1.0" 200 - 
127.0.0.1 - - [17/Dec/2018 10:32:42] "GET / HTTP/1.0" 200 - 
127.0.0.1 - - [17/Dec/2018 10:32:42] "GET / HTTP/1.0" 200 - 
127.0.0.1 - - [17/Dec/2018 10:32:42] "GET / HTTP/1.0" 200 - 
127.0.0.1 - - [17/Dec/2018 10:32:42] "GET / HTTP/1.0" 200 - 
127.0.0.1 - - [17/Dec/2018 10:32:42] "GET / HTTP/1.0" 200 - 
127.0.0.1 - - [17/Dec/2018 10:32:42] "GET / HTTP/1.0" 200 - 
127.0.0.1 - - [17/Dec/2018 10:40:21] "GET /purchase_a_sword HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:40:21] "GET /purchase_a_sword HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:40:21] "GET /purchase_a_sword HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:40:21] "GET /purchase_a_sword HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:40:21] "GET /purchase_a_sword HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:40:21] "GET /purchase_a_sword HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:40:21] "GET /purchase_a_sword HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:40:21] "GET /purchase_a_sword HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:40:21] "GET /purchase_a_sword HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:40:21] "GET /purchase_a_sword HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:48:09] "GET / HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:48:09] "GET / HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:48:09] "GET / HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:48:09] "GET / HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:48:09] "GET / HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:48:09] "GET / HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:48:09] "GET / HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:48:09] "GET / HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:48:09] "GET / HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:48:09] "GET / HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:48:50] "GET /purchase_a_sword HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:48:50] "GET /purchase_a_sword HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:48:50] "GET /purchase_a_sword HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:48:50] "GET /purchase_a_sword HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:48:50] "GET /purchase_a_sword HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:48:50] "GET /purchase_a_sword HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:48:50] "GET /purchase_a_sword HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:48:50] "GET /purchase_a_sword HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:48:50] "GET /purchase_a_sword HTTP/1.0" 200 -
127.0.0.1 - - [17/Dec/2018 10:48:50] "GET /purchase_a_sword HTTP/1.0" 200 -
```

In my 3rd terminal window where kafka is running, I see the following 40 events that match the 20 actions performed by the 2 users:
```
"Host": "user1.comcast.com", "event_type": "default", "Accept": "*/*", "User-Age 
nt": "ApacheBench/2.3"} 
{"Host": "user1.comcast.com", "event_type": "default", "Accept": "*/*", "User-Age 
nt": "ApacheBench/2.3"} 
{"Host": "user1.comcast.com", "event_type": "default", "Accept": "*/*", "User-Age 
nt": "ApacheBench/2.3"} 
{"Host": "user1.comcast.com", "event_type": "default", "Accept": "*/*", "User-Age 
nt": "ApacheBench/2.3"} 
{"Host": "user1.comcast.com", "event_type": "default", "Accept": "*/*", "User-Age 
nt": "ApacheBench/2.3"} 
{"Host": "user1.comcast.com", "event_type": "default", "Accept": "*/*", "User-Age 
nt": "ApacheBench/2.3"} 
{"Host": "user1.comcast.com", "event_type": "default", "Accept": "*/*", "User-Age 
nt": "ApacheBench/2.3"} 
{"Host": "user1.comcast.com", "event_type": "default", "Accept": "*/*", "User-Age 
nt": "ApacheBench/2.3"} 
{"Host": "user1.comcast.com", "event_type": "default", "Accept": "*/*", "User-Age 
nt": "ApacheBench/2.3"} 
{"Host": "user1.comcast.com", "event_type": "default", "Accept": "*/*", "User-Age 
nt": "ApacheBench/2.3"} 
{"Host": "user1.comcast.com", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user1.comcast.com", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user1.comcast.com", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user1.comcast.com", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user1.comcast.com", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user1.comcast.com", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user1.comcast.com", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user1.comcast.com", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user1.comcast.com", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user1.comcast.com", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user2.att.com", "event_type": "default", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user2.att.com", "event_type": "default", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user2.att.com", "event_type": "default", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user2.att.com", "event_type": "default", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user2.att.com", "event_type": "default", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user2.att.com", "event_type": "default", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user2.att.com", "event_type": "default", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user2.att.com", "event_type": "default", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user2.att.com", "event_type": "default", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user2.att.com", "event_type": "default", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user2.att.com", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user2.att.com", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user2.att.com", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user2.att.com", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user2.att.com", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user2.att.com", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user2.att.com", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user2.att.com", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user2.att.com", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
{"Host": "user2.att.com", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
```
In summary, I generated 40 events total, 20 from user1.comcast.com and 20 from user2.att.com.  Within these 40 events, there are 20 default events as 10 were generated from user1.comcast.com and 10 from user2.att.com.  There are 20 purchase sword events, 10 were generated from user1.comcast.com and 10 from user2.att.com.

6. Next I run the filtered_writes.py file using spark. First I check out the contents of the filtered_writes.py file:
```
cat filtered_writes.py
```
output:
```
#!/usr/bin/env python
"""Extract events from kafka and write them to hdfs
"""
import json
from pyspark.sql import SparkSession, Row
from pyspark.sql.functions import udf
 
 
@udf('boolean')
def is_purchase(event_as_json):
    event = json.loads(event_as_json)
    if event['event_type'] == 'purchase_sword':
        return True
    return False
 
 
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
 
    purchase_events = raw_events \
        .select(raw_events.value.cast('string').alias('raw'),
                raw_events.timestamp.cast('string')) \
        .filter(is_purchase('raw'))
 
    extracted_purchase_events = purchase_events \
        .rdd \
        .map(lambda r: Row(timestamp=r.timestamp, **json.loads(r.raw))) \
        .toDF()
    extracted_purchase_events.printSchema()
    extracted_purchase_events.show()
 
    extracted_purchase_events \
        .write \
        .mode('overwrite') \
        .parquet('/tmp/purchases')
 
 
if __name__ == "__main__":
    main()
```

We see from here that pyspark commands will be used to create a purchases parquet file in hdsfs. The main thing to note is that the purchases file will only contain purchase events, or specifically events where event_type = "purchase_sword".  Once this filter is a applied the events will be transformed into a nice table, extracted_purchase_events, that shows only the 20 purchase events, and this table will be written to the parquet file. 

In my fourth terminal window, I use spark to run the filtered_writes.py file:
```
docker-compose exec spark spark-submit /w205/assignment-12-dwang-ischool/filtered_writes.py
```
Output shows that the run was successful. As expected, I see the following well formatted table in my output which shows the 20 purchase events that were successfully filtered:
```
+------+-----------------+---------------+--------------+--------------------+
|Accept|             Host|     User-Agent|    event_type|           timestamp|
+------+-----------------+---------------+--------------+--------------------+
|   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-12-17 10:40:...|
|   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-12-17 10:40:...|
|   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-12-17 10:40:...|
|   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-12-17 10:40:...|
|   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-12-17 10:40:...|
|   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-12-17 10:40:...|
|   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-12-17 10:40:...|
|   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-12-17 10:40:...|
|   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-12-17 10:40:...|
|   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-12-17 10:40:...|
|   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-12-17 10:48:...|
|   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-12-17 10:48:...|
|   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-12-17 10:48:...|
|   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-12-17 10:48:...|
|   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-12-17 10:48:...|
|   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-12-17 10:48:...|
|   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-12-17 10:48:...|
|   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-12-17 10:48:...|
|   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-12-17 10:48:...|
|   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-12-17 10:48:...|
+------+-----------------+---------------+--------------+--------------------+
```
I also check the status of hadoop to see that the purchases file was successfully created:
```
docker-compose exec cloudera hadoop fs -ls /tmp/
```
output:
```
Found 4 items
drwxrwxrwt   - mapred mapred              0 2016-04-06 02:26 /tmp/hadoop-yarn
drwx-wx-wx   - hive   supergroup          0 2018-12-17 10:04 /tmp/hive
drwxrwxrwt   - mapred hadoop              0 2016-04-06 02:28 /tmp/logs
drwxr-xr-x   - root   supergroup          0 2018-12-17 11:04 /tmp/purchases
```

7. To prepare for querying using presto, I will next run a modified version of filtered_writes.py, called write_hive_table.py.  This will register a temporary table of purchases in the hive metastore so I can query later.  The contents of the write_hive_table.py are:
```
cat write_hive_table.py
```
output:
```
#!/usr/bin/env python
"""Extract events from kafka and write them to hdfs
"""
import json
from pyspark.sql import SparkSession, Row
from pyspark.sql.functions import udf


@udf('boolean')
def is_purchase(event_as_json):
    event = json.loads(event_as_json)
    if event['event_type'] == 'purchase_sword':
        return True
    return False


def main():
    """main
    """
    spark = SparkSession \
        .builder \
        .appName("ExtractEventsJob") \
        .enableHiveSupport() \
        .getOrCreate()

    raw_events = spark \
        .read \
        .format("kafka") \
        .option("kafka.bootstrap.servers", "kafka:29092") \
        .option("subscribe", "events") \
        .option("startingOffsets", "earliest") \
        .option("endingOffsets", "latest") \
        .load()
 
    purchase_events = raw_events \
        .select(raw_events.value.cast('string').alias('raw'),
                raw_events.timestamp.cast('string')) \
        .filter(is_purchase('raw'))

    extracted_purchase_events = purchase_events \
        .rdd \
        .map(lambda r: Row(timestamp=r.timestamp, **json.loads(r.raw))) \
        .toDF()
    extracted_purchase_events.printSchema()
    extracted_purchase_events.show()
 
    extracted_purchase_events.registerTempTable("extracted_purchase_events")

    spark.sql("""
        create external table purchases
        stored as parquet
        location '/tmp/purchases'
        as
        select * from extracted_purchase_events
    """)


if __name__ == "__main__":
    main()
```

Next I write to the hive metastore using the following command:
```
docker-compose exec spark spark-submit /w205/assignment-12-dwang-ischool/write_hive_table.py
```
Since write_hive_table.py performs almost the same commands as filtered_writes.py except for the part at the end where a temporary table in the hive metastore is created, we see the following formatted table in the output from the above command.  As expected, we see that the table captures only the 20 purchase events out of th 40 total events, 10 from each of the 2 users:
```
+------+-----------------+---------------+--------------+--------------------+
|Accept|             Host|     User-Agent|    event_type|           timestamp|
+------+-----------------+---------------+--------------+--------------------+
|   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-12-17 10:40:...|
|   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-12-17 10:40:...|
|   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-12-17 10:40:...|
|   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-12-17 10:40:...|
|   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-12-17 10:40:...|
|   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-12-17 10:40:...|
|   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-12-17 10:40:...|
|   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-12-17 10:40:...|
|   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-12-17 10:40:...|
|   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-12-17 10:40:...|
|   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-12-17 10:48:...|
|   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-12-17 10:48:...|
|   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-12-17 10:48:...|
|   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-12-17 10:48:...|
|   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-12-17 10:48:...|
|   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-12-17 10:48:...|
|   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-12-17 10:48:...|
|   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-12-17 10:48:...|
|   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-12-17 10:48:...|
|   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-12-17 10:48:...|
+------+-----------------+---------------+--------------+--------------------+
```

In a 5th terminal window, I run presto using the following command:
```
docker-compose exec presto presto --server presto:8080 --catalog hive --schema default
```

At the "presto:default>" prompt, I enter the following:
```
presto:default> show tables;
```
and I get the following output which shows me that there is one table called purchases:
```
   Table   
-----------
 purchases 
(1 row)

Query 20181217_122021_00002_b6xpp, FINISHED, 1 node
Splits: 2 total, 0 done (0.00%)
0:00 [0 rows, 0B] [0 rows/s, 0B/s]
```
Next I enter a command that describes the table:
```
presto:default> describe purchases;
```
and I get the following output:
```
   Column   |  Type   | Comment 
------------+---------+---------
 accept     | varchar |         
 host       | varchar |         
 user-agent | varchar |         
 event_type | varchar |         
 timestamp  | varchar |         
(5 rows)

Query 20181217_122206_00003_b6xpp, FINISHED, 1 node
Splits: 2 total, 0 done (0.00%)
0:01 [0 rows, 0B] [0 rows/s, 0B/s]
```

We see that the table consists of 5 columns - accept, host, user-agent, event_type, timestamp.  The first 4 columns were extracted from our events in kafka.  They are the 4 dictionary keys that define every event when they were intially generated.  When running spark above, when a fifth column was added for the timestamp.  We see that these columns in the output match the column headers in the purchases table shown above. 

We can then query from the purchases table using the following:
```
presto:default> select * from purchases;
```
this outputs:
```
 accept |       host        |   user-agent    |   event_type   |        timestamp        
--------+-------------------+-----------------+----------------+-------------------------
 */*    | user1.comcast.com | ApacheBench/2.3 | purchase_sword | 2018-12-17 10:40:21.202 
 */*    | user1.comcast.com | ApacheBench/2.3 | purchase_sword | 2018-12-17 10:40:21.215 
 */*    | user1.comcast.com | ApacheBench/2.3 | purchase_sword | 2018-12-17 10:40:21.226 
 */*    | user1.comcast.com | ApacheBench/2.3 | purchase_sword | 2018-12-17 10:40:21.23  
 */*    | user1.comcast.com | ApacheBench/2.3 | purchase_sword | 2018-12-17 10:40:21.237 
 */*    | user1.comcast.com | ApacheBench/2.3 | purchase_sword | 2018-12-17 10:40:21.242 
 */*    | user1.comcast.com | ApacheBench/2.3 | purchase_sword | 2018-12-17 10:40:21.248 
 */*    | user1.comcast.com | ApacheBench/2.3 | purchase_sword | 2018-12-17 10:40:21.251 
 */*    | user1.comcast.com | ApacheBench/2.3 | purchase_sword | 2018-12-17 10:40:21.258 
 */*    | user1.comcast.com | ApacheBench/2.3 | purchase_sword | 2018-12-17 10:40:21.266 
 */*    | user2.att.com     | ApacheBench/2.3 | purchase_sword | 2018-12-17 10:48:50.609 
 */*    | user2.att.com     | ApacheBench/2.3 | purchase_sword | 2018-12-17 10:48:50.618 
 */*    | user2.att.com     | ApacheBench/2.3 | purchase_sword | 2018-12-17 10:48:50.627 
 */*    | user2.att.com     | ApacheBench/2.3 | purchase_sword | 2018-12-17 10:48:50.637 
 */*    | user2.att.com     | ApacheBench/2.3 | purchase_sword | 2018-12-17 10:48:50.654 
 */*    | user2.att.com     | ApacheBench/2.3 | purchase_sword | 2018-12-17 10:48:50.659 
 */*    | user2.att.com     | ApacheBench/2.3 | purchase_sword | 2018-12-17 10:48:50.666 
 */*    | user2.att.com     | ApacheBench/2.3 | purchase_sword | 2018-12-17 10:48:50.673 
 */*    | user2.att.com     | ApacheBench/2.3 | purchase_sword | 2018-12-17 10:48:50.679 
 */*    | user2.att.com     | ApacheBench/2.3 | purchase_sword | 2018-12-17 10:48:50.685 
(20 rows)

Query 20181217_122643_00004_b6xpp, FINISHED, 1 node
Splits: 2 total, 0 done (0.00%)
0:03 [0 rows, 0B] [0 rows/s, 0B/s]
```
This matches the output table that we saw in the spark job previously, which is the table created for the 20 purchase events generated.  

8. To prepare our spark for streaming, we need to ensure that the filtered purchase events need to have a well defined schema.  To this, I will use the filter_swords_batch.py with the following contents:
```
cat filter_swords_batch.py
```
output:
```
#!/usr/bin/env python
"""Extract events from kafka and write them to hdfs
"""
import json
from pyspark.sql import SparkSession
from pyspark.sql.functions import udf, from_json
from pyspark.sql.types import StructType, StructField, StringType


def purchase_sword_event_schema():
    """
    root
    |-- Accept: string (nullable = true)
    |-- Host: string (nullable = true)
    |-- User-Agent: string (nullable = true)
    |-- event_type: string (nullable = true)
    |-- timestamp: string (nullable = true)
    """
    return StructType([
        StructField("Accept", StringType(), True),
        StructField("Host", StringType(), True),
        StructField("User-Agent", StringType(), True),
        StructField("event_type", StringType(), True),
    ])


@udf('boolean')
def is_sword_purchase(event_as_json):
    """udf for filtering events
    """
    event = json.loads(event_as_json)
    if event['event_type'] == 'purchase_sword':
        return True
    return False


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

    sword_purchases = raw_events \
        .filter(is_sword_purchase(raw_events.value.cast('string'))) \
        .select(raw_events.value.cast('string').alias('raw_event'),
                raw_events.timestamp.cast('string'),
                from_json(raw_events.value.cast('string'),
                          purchase_sword_event_schema()).alias('json')) \
        .select('raw_event', 'timestamp', 'json.*')

    sword_purchases.printSchema()
    sword_purchases.show(100)


if __name__ == "__main__":
    main()
```
We see here that unlike the previous spark file, the purchase_sword_event_schema() defines the exact schema that all filtered purchase events should have.  

I then run it using:
```
docker-compose exec spark spark-submit /w205/assignment-12-dwang-ischool/filter_swords_batch.py
```

Due to sword_purchases.show() in the filter_swords_batch.py file, I see this table in my output:
```
+--------------------+--------------------+------+-----------------+---------------+--------------+
|           raw_event|           timestamp|Accept|             Host|     User-Agent|    event_type|
+--------------------+--------------------+------+-----------------+---------------+--------------+
|{"Host": "user1.c...|2018-12-18 03:08:...|   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|
|{"Host": "user1.c...|2018-12-18 03:08:...|   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|
|{"Host": "user1.c...|2018-12-18 03:08:...|   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|
|{"Host": "user1.c...|2018-12-18 03:08:...|   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|
|{"Host": "user1.c...|2018-12-18 03:08:...|   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|
|{"Host": "user1.c...|2018-12-18 03:08:...|   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|
|{"Host": "user1.c...|2018-12-18 03:08:...|   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|
|{"Host": "user1.c...|2018-12-18 03:08:...|   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|
|{"Host": "user1.c...|2018-12-18 03:08:...|   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|
|{"Host": "user1.c...|2018-12-18 03:08:...|   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|
|{"Host": "user2.a...|2018-12-18 03:08:...|   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|
|{"Host": "user2.a...|2018-12-18 03:08:...|   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|
|{"Host": "user2.a...|2018-12-18 03:08:...|   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|
|{"Host": "user2.a...|2018-12-18 03:08:...|   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|
|{"Host": "user2.a...|2018-12-18 03:08:...|   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|
|{"Host": "user2.a...|2018-12-18 03:08:...|   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|
|{"Host": "user2.a...|2018-12-18 03:08:...|   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|
|{"Host": "user2.a...|2018-12-18 03:08:...|   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|
|{"Host": "user2.a...|2018-12-18 03:08:...|   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|
|{"Host": "user2.a...|2018-12-18 03:08:...|   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|
+--------------------+--------------------+------+-----------------+---------------+--------------+
```
This table has the same schema we saw in #6, but it has an extra column for raw_event.  

9. To turn on the events stream, I will use filter_swords_stream.py, which has the following contents:
```
cat filter_swords_stream.py
```
output:
```
#!/usr/bin/env python
"""Extract events from kafka and write them to hdfs
"""
import json
from pyspark.sql import SparkSession
from pyspark.sql.functions import udf, from_json
from pyspark.sql.types import StructType, StructField, StringType


def purchase_sword_event_schema():
    """
    root
    |-- Accept: string (nullable = true)
    |-- Host: string (nullable = true)
    |-- User-Agent: string (nullable = true)
    |-- event_type: string (nullable = true)
    |-- timestamp: string (nullable = true)
    """
    return StructType([
        StructField("Accept", StringType(), True),
        StructField("Host", StringType(), True),
        StructField("User-Agent", StringType(), True),
        StructField("event_type", StringType(), True),
    ])


@udf('boolean')
def is_sword_purchase(event_as_json):
    """udf for filtering events
    """
    event = json.loads(event_as_json)
    if event['event_type'] == 'purchase_sword':
        return True
    return False


def main():
    """main
    """
    spark = SparkSession \
        .builder \
        .appName("ExtractEventsJob") \
        .getOrCreate()

    raw_events = spark \
        .readStream \
        .format("kafka") \
        .option("kafka.bootstrap.servers", "kafka:29092") \
        .option("subscribe", "events") \
        .load()

    sword_purchases = raw_events \
        .filter(is_sword_purchase(raw_events.value.cast('string'))) \
        .select(raw_events.value.cast('string').alias('raw_event'),
                raw_events.timestamp.cast('string'),
                from_json(raw_events.value.cast('string'),
                          purchase_sword_event_schema()).alias('json')) \
        .select('raw_event', 'timestamp', 'json.*')

    query = sword_purchases \
        .writeStream \
        .format("console") \
        .start()

    query.awaitTermination()


if __name__ == "__main__":
    main()
```

Like filter_swords_batch.py in #8, filter_swords_stream.py has a pre-defined schema for the filtered purchase events.  However, there are several differences.  For one, instead of using a read spark command for the raw_events, it is using the .readStream spark commmand.  sword_purchases now represents writing to the stream in the console rather than our well formatted table.  The offset for this is actually at the end of all events that have already been generated, so when we run this for the very first time, we will seen an empty table for sword_purchases, since no new events have been added to the stream. 

To start streaming, I run the following command:
```
docker-compose exec spark spark-submit /w205/assignment-12-dwang-ischool/filter_swords_stream.py
```

And as expected, since the offset starts at the end of all the events previously generated, the current state of sword_purchase table contains no events.  We see an empty table in the output:
```
+---------+---------+------+----+----------+----------+
|raw_event|timestamp|Accept|Host|User-Agent|event_type|
+---------+---------+------+----+----------+----------+
+---------+---------+------+----+----------+----------+
```

Next, I will feed the stream 40 more events.  Using the same commands as before, I have user1 and user2 each create 20 events, 10 default action and 10 purchase sword action:
```
docker-compose exec mids ab -n 10 -H "Host: user1.comcast.com" http://localhost:5000/
docker-compose exec mids ab -n 10 -H "Host: user1.comcast.com" http://localhost:5000/purchase_a_sword
docker-compose exec mids ab -n 10 -H "Host: user2.att.com" http://localhost:5000/
docker-compose exec mids ab -n 10 -H "Host: user2.att.com" http://localhost:5000/purchase_a_sword
```

For each of the 4 commands, I see the following 4 versions of the sword_purchases table outputted in the terminal window where spark stream is running:

The first output corresponds to the user1 creating 10 default response events (first command). We expect no events to be filtered to the sword purchases table, since no sword puchase events occurred
```
+---------+---------+------+----+----------+----------+
|raw_event|timestamp|Accept|Host|User-Agent|event_type|
+---------+---------+------+----+----------+----------+
+---------+---------+------+----+----------+----------+
```

The second output corresponds to user1 creating 10 sword_purchase events (2nd command):
```
+--------------------+--------------------+------+-----------------+---------------+--------------+
|           raw_event|           timestamp|Accept|             Host|     User-Agent|    event_type|
+--------------------+--------------------+------+-----------------+---------------+--------------+
|{"Host": "user1.c...|2018-12-18 03:47:...|   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|
|{"Host": "user1.c...|2018-12-18 03:47:...|   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|
|{"Host": "user1.c...|2018-12-18 03:47:...|   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|
|{"Host": "user1.c...|2018-12-18 03:47:...|   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|
|{"Host": "user1.c...|2018-12-18 03:47:...|   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|
|{"Host": "user1.c...|2018-12-18 03:47:...|   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|
|{"Host": "user1.c...|2018-12-18 03:47:...|   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|
|{"Host": "user1.c...|2018-12-18 03:47:...|   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|
+--------------------+--------------------+------+-----------------+---------------+--------------+
```

When I enter the 3rd command to have user2 create 10 default response events, we again see the empty table in the output:
```
+---------+---------+------+----+----------+----------+
|raw_event|timestamp|Accept|Host|User-Agent|event_type|
+---------+---------+------+----+----------+----------+
+---------+---------+------+----+----------+----------+
```

When I enter the 4th command to have user 2 create 10 sword purchase events, I see this table:
```
+--------------------+--------------------+------+-------------+---------------+--------------+
|           raw_event|           timestamp|Accept|         Host|     User-Agent|    event_type|
+--------------------+--------------------+------+-------------+---------------+--------------+
|{"Host": "user2.a...|2018-12-18 03:49:...|   */*|user2.att.com|ApacheBench/2.3|purchase_sword|
|{"Host": "user2.a...|2018-12-18 03:49:...|   */*|user2.att.com|ApacheBench/2.3|purchase_sword|
|{"Host": "user2.a...|2018-12-18 03:49:...|   */*|user2.att.com|ApacheBench/2.3|purchase_sword|
|{"Host": "user2.a...|2018-12-18 03:49:...|   */*|user2.att.com|ApacheBench/2.3|purchase_sword|
|{"Host": "user2.a...|2018-12-18 03:49:...|   */*|user2.att.com|ApacheBench/2.3|purchase_sword|
|{"Host": "user2.a...|2018-12-18 03:49:...|   */*|user2.att.com|ApacheBench/2.3|purchase_sword|
|{"Host": "user2.a...|2018-12-18 03:49:...|   */*|user2.att.com|ApacheBench/2.3|purchase_sword|
|{"Host": "user2.a...|2018-12-18 03:49:...|   */*|user2.att.com|ApacheBench/2.3|purchase_sword|
|{"Host": "user2.a...|2018-12-18 03:49:...|   */*|user2.att.com|ApacheBench/2.3|purchase_sword|
+--------------------+--------------------+------+-------------+---------------+--------------+
```

10. Lastly, I write from a stream using the write_swords_stream.py file.  It contains the following contents:
```
cat write_swords_stream.py
```
output:
```
#!/usr/bin/env python
"""Extract events from kafka and write them to hdfs
"""
import json
from pyspark.sql import SparkSession
from pyspark.sql.functions import udf, from_json
from pyspark.sql.types import StructType, StructField, StringType


def purchase_sword_event_schema():
    """
    root
    |-- Accept: string (nullable = true)
    |-- Host: string (nullable = true)
    |-- User-Agent: string (nullable = true)
    |-- event_type: string (nullable = true)
    |-- timestamp: string (nullable = true)
    """
    return StructType([
        StructField("Accept", StringType(), True),
        StructField("Host", StringType(), True),
        StructField("User-Agent", StringType(), True),
        StructField("event_type", StringType(), True),
    ])


@udf('boolean')
def is_sword_purchase(event_as_json):
    """udf for filtering events
    """
    event = json.loads(event_as_json)
    if event['event_type'] == 'purchase_sword':
        return True
    return False


def main():
    """main
    """
    spark = SparkSession \
        .builder \
        .appName("ExtractEventsJob") \
        .getOrCreate()

    raw_events = spark \
        .readStream \
        .format("kafka") \
        .option("kafka.bootstrap.servers", "kafka:29092") \
        .option("subscribe", "events") \
        .load()

    sword_purchases = raw_events \
        .filter(is_sword_purchase(raw_events.value.cast('string'))) \
        .select(raw_events.value.cast('string').alias('raw_event'),
                raw_events.timestamp.cast('string'),
                from_json(raw_events.value.cast('string'),
                          purchase_sword_event_schema()).alias('json')) \
        .select('raw_event', 'timestamp', 'json.*')

    sink = sword_purchases \
        .writeStream \
        .format("parquet") \
        .option("checkpointLocation", "/tmp/checkpoints_for_sword_purchases") \
        .option("path", "/tmp/sword_purchases") \
        .trigger(processingTime="10 seconds") \
        .start()

    sink.awaitTermination()


if __name__ == "__main__":
    main()
```
Here, we can see that the different real time versions of the sword_purchases table from the stream wil be written as parquet files in hdfs

I run the write_swords_stream.py file using:
```
docker-compose exec spark spark-submit /w205/assignment-12-dwang-ischool/write_swords_stream.py
```
I feed the stream using a loop:
```

while true; do
  docker-compose exec mids \
    ab -n 10 -H "Host: user1.comcast.com" \
      http://localhost:5000/purchase_a_sword
  sleep 10
done
```
Which creates 10 purchase sword events by user 1 ever 10 seconds.  After a few minutes, I check hadoop using:
```
docker-compose exec cloudera hadoop fs -ls /tmp/sword_purchases
```
and I am able to see all the parquet files created in the time interval:
```
Found 44 items
drwxr-xr-x   - root supergroup          0 2018-12-18 04:02 /tmp/sword_purchases/_spark_metadata
-rw-r--r--   1 root supergroup       2322 2018-12-18 03:59 /tmp/sword_purchases/part-00000-00a198af-a014-46b5-8d36-0b242cbebab9-c000.snappy.parquet
-rw-r--r--   1 root supergroup       2323 2018-12-18 03:59 /tmp/sword_purchases/part-00000-08f8fb02-3253-4d72-9841-41a01a91f771-c000.snappy.parquet
-rw-r--r--   1 root supergroup       2323 2018-12-18 03:58 /tmp/sword_purchases/part-00000-0d9a4d9b-0bfc-498b-a32e-b7cda93713f4-c000.snappy.parquet
-rw-r--r--   1 root supergroup       2322 2018-12-18 04:01 /tmp/sword_purchases/part-00000-11501e18-02dc-48ed-b750-9a80fe20daeb-c000.snappy.parquet
-rw-r--r--   1 root supergroup       2320 2018-12-18 04:02 /tmp/sword_purchases/part-00000-11ef728f-150f-42f1-9cf9-984be1cd20e4-c000.snappy.parquet
-rw-r--r--   1 root supergroup       2325 2018-12-18 04:02 /tmp/sword_purchases/part-00000-166d2895-17ed-4578-85bf-248ce2627d38-c000.snappy.parquet
-rw-r--r--   1 root supergroup       2320 2018-12-18 03:58 /tmp/sword_purchases/part-00000-1e2a7542-3ffd-47ea-a40a-5bae21227774-c000.snappy.parquet
-rw-r--r--   1 root supergroup       2324 2018-12-18 03:59 /tmp/sword_purchases/part-00000-22d3ac14-fb82-482b-a41a-69361be2855c-c000.snappy.parquet
-rw-r--r--   1 root supergroup       2325 2018-12-18 03:55 /tmp/sword_purchases/part-00000-24e30fee-efb2-4a7e-a62d-f378ae43386b-c000.snappy.parquet
-rw-r--r--   1 root supergroup       2320 2018-12-18 03:56 /tmp/sword_purchases/part-00000-257ba0e6-403c-46ba-9ff7-45d7635aa2d4-c000.snappy.parquet
-rw-r--r--   1 root supergroup       2326 2018-12-18 03:56 /tmp/sword_purchases/part-00000-2765dd19-8fd3-4ff4-9a0b-710b2b8a8f09-c000.snappy.parquet
-rw-r--r--   1 root supergroup       2320 2018-12-18 03:57 /tmp/sword_purchases/part-00000-301f49ba-585c-49eb-a5d1-3e9baaceea9d-c000.snappy.parquet
-rw-r--r--   1 root supergroup       2322 2018-12-18 04:00 /tmp/sword_purchases/part-00000-32669234-4640-488b-a5d2-ccf810a4497c-c000.snappy.parquet
-rw-r--r--   1 root supergroup       2322 2018-12-18 04:01 /tmp/sword_purchases/part-00000-39593228-e5a2-401f-9dad-e75bf770cb47-c000.snappy.parquet
-rw-r--r--   1 root supergroup       2319 2018-12-18 03:59 /tmp/sword_purchases/part-00000-42e6b637-36d0-47c6-bada-fec9af438f56-c000.snappy.parquet
-rw-r--r--   1 root supergroup       2318 2018-12-18 04:01 /tmp/sword_purchases/part-00000-52e8d775-bd7b-4d4c-a687-357e89adef2e-c000.snappy.parquet
-rw-r--r--   1 root supergroup       2321 2018-12-18 03:56 /tmp/sword_purchases/part-00000-62caadf6-74ad-479c-9cdf-d0ac57ae347d-c000.snappy.parquet
-rw-r--r--   1 root supergroup       2322 2018-12-18 03:59 /tmp/sword_purchases/part-00000-66a73c6b-59c8-4033-910c-525e296c08c7-c000.snappy.parquet
-rw-r--r--   1 root supergroup       2325 2018-12-18 03:56 /tmp/sword_purchases/part-00000-679fb827-cec7-433b-a7e1-9c1ca2b350ab-c000.snappy.parquet
-rw-r--r--   1 root supergroup       2325 2018-12-18 04:02 /tmp/sword_purchases/part-00000-6cf84cca-acc6-470f-b7cc-cc39028ec038-c000.snappy.parquet
-rw-r--r--   1 root supergroup       2325 2018-12-18 03:55 /tmp/sword_purchases/part-00000-6e40ca52-927a-4638-9380-3ae3ab7869aa-c000.snappy.parquet
-rw-r--r--   1 root supergroup        688 2018-12-18 03:54 /tmp/sword_purchases/part-00000-73e2b287-9d7f-487b-875b-b3eb3d8bddda-c000.snappy.parquet
-rw-r--r--   1 root supergroup       2324 2018-12-18 03:57 /tmp/sword_purchases/part-00000-81b57863-afdc-492d-a330-c3f2116a6947-c000.snappy.parquet
-rw-r--r--   1 root supergroup       2330 2018-12-18 04:02 /tmp/sword_purchases/part-00000-8c6ec2ff-c1c2-485f-8bcc-86bd1f881fae-c000.snappy.parquet
-rw-r--r--   1 root supergroup       2321 2018-12-18 04:00 /tmp/sword_purchases/part-00000-9071f5c8-cbc7-48be-b6fc-fb6608789898-c000.snappy.parquet
-rw-r--r--   1 root supergroup       2318 2018-12-18 03:59 /tmp/sword_purchases/part-00000-9459bdf7-4b69-43d6-9dc2-6523db1073bf-c000.snappy.parquet
-rw-r--r--   1 root supergroup       2321 2018-12-18 03:58 /tmp/sword_purchases/part-00000-9c828923-1b21-498d-9fbc-ae5b449a913c-c000.snappy.parquet
-rw-r--r--   1 root supergroup       2324 2018-12-18 03:58 /tmp/sword_purchases/part-00000-9e4df0af-c300-45a5-be18-ebfa310140e4-c000.snappy.parquet
-rw-r--r--   1 root supergroup       2320 2018-12-18 04:00 /tmp/sword_purchases/part-00000-a2389b92-ee52-40b7-a56f-4218c0b7dcf4-c000.snappy.parquet
-rw-r--r--   1 root supergroup       2322 2018-12-18 03:56 /tmp/sword_purchases/part-00000-a57e23f4-5aa9-4ff3-83ff-50d5494bd74c-c000.snappy.parquet
-rw-r--r--   1 root supergroup       2323 2018-12-18 04:00 /tmp/sword_purchases/part-00000-a7d35103-d089-42e0-8e6b-f86a1b9b98b6-c000.snappy.parquet
-rw-r--r--   1 root supergroup       2321 2018-12-18 03:55 /tmp/sword_purchases/part-00000-b65937d7-efb5-4fc2-945f-855a06b365a9-c000.snappy.parquet
-rw-r--r--   1 root supergroup       2319 2018-12-18 03:57 /tmp/sword_purchases/part-00000-b674bc75-05d0-4843-9152-d6181152fcd5-c000.snappy.parquet
-rw-r--r--   1 root supergroup       2318 2018-12-18 04:01 /tmp/sword_purchases/part-00000-bba9d8a5-ef7b-4363-afdb-db7a2deba35e-c000.snappy.parquet
-rw-r--r--   1 root supergroup       2322 2018-12-18 03:56 /tmp/sword_purchases/part-00000-cd1c4c97-7ade-4dc5-a1e2-2bfffeeda581-c000.snappy.parquet
-rw-r--r--   1 root supergroup       2321 2018-12-18 04:00 /tmp/sword_purchases/part-00000-d6afcc73-b12d-454c-bab3-fb1f0d67e961-c000.snappy.parquet
-rw-r--r--   1 root supergroup       2322 2018-12-18 04:02 /tmp/sword_purchases/part-00000-dce27683-f990-421d-a50a-685a7c378164-c000.snappy.parquet
-rw-r--r--   1 root supergroup       2321 2018-12-18 03:57 /tmp/sword_purchases/part-00000-e078fb7c-aa25-48af-9ffc-f2aeb95c64ff-c000.snappy.parquet
-rw-r--r--   1 root supergroup       2324 2018-12-18 03:57 /tmp/sword_purchases/part-00000-e49897be-0164-4bff-a62c-82d97ddecf5c-c000.snappy.parquet
-rw-r--r--   1 root supergroup       2320 2018-12-18 03:57 /tmp/sword_purchases/part-00000-e9f51fb7-abb2-4feb-8d35-5dc8ad525b75-c000.snappy.parquet
-rw-r--r--   1 root supergroup       2323 2018-12-18 04:00 /tmp/sword_purchases/part-00000-ec21628b-dfb6-4887-9a75-fe20e2120df2-c000.snappy.parquet
-rw-r--r--   1 root supergroup       2323 2018-12-18 04:01 /tmp/sword_purchases/part-00000-ec40453f-a095-4e8c-944b-2f953fabbb79-c000.snappy.parquet
-rw-r--r--   1 root supergroup       2317 2018-12-18 03:58 /tmp/sword_purchases/part-00000-f9c00287-a943-4159-b057-08e10046f6a2-c000.snappy.parquet
```
And the list continues to grow the longer I wait. 

11.  Once I am done, I end the continuous streaming apache bench loop.  I also exist both flask, kafka, presto, and the spark stream.
I then take down the cluster using:
```
docker-compose down
```

And I save my commands history:
```
history > history.txt
```