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

# Assignment 10

## Follow the steps we did in class 


### Turn in your `/assignment-10-<user-name>/README.md` file. It should include:
1) A summary type explanation of the example. 
  * For example, for Week 6's activity, a summary would be: "We spun up a cluster with kafka, zookeeper, and the mids container. Then we published and consumed messages with kafka."
2) Your `docker-compose.yml`
3) Source code for the flask application(s) used.
4) Each important step in the process. For each step, include:
  * The command(s) 
  * The output (if there is any).  Be sure to include examples of generated events when available.
  * An explanation for what it achieves 
    * The explanation should be fairly detailed, e.g., instead of "publish to kafka" say what you're publishing, where it's coming from, going to etc.



# Class Activity Summary

1. First I cloned the assignment 10 repo into my droplet and navigated into the repo's folder:
```
git clone https://github.com/mids-w205-martin-mims/assignment-10-dwang-ischool
cd assignment-10-dwang-ischool/
```

2. While inside the assignment 10 repo, I copied the docker-compose.yml file and the python files:
```
 ~/w205/course-content/10-Transforming-Streaming-Data/docker-compose.yml .
 ~/w205/course-content/10-Transforming-Streaming-Data/*.py .
```

3. Next I check the docker-compose.yml file to make sure that it contains the zookeeper, kafka, spark, and mids containers:
```
cat docker-compose.yml
```

4. I spin up the cluster with zookeper, kafka, spark and mids components. Then I check tht the containers are active:
```
docker-compose up -d
docker-compose ps
```
I get the following output, which shows that the containers did successfully spin up:
```
                Name                            Command            State                    Ports                  
------------------------------------------------------------------------------------------------------------------
assignment10dwangischool_kafka_1       /etc/confluent/docker/run   Up      29092/tcp, 9092/tcp                     
assignment10dwangischool_mids_1        /bin/bash                   Up      0.0.0.0:5000->5000/tcp, 8888/tcp        
assignment10dwangischool_spark_1       docker-entrypoint.sh bash   Up      0.0.0.0:8888->8888/tcp                  
assignment10dwangischool_zookeeper_1   /etc/confluent/docker/run   Up      2181/tcp, 2888/tcp, 32181/tcp, 3888/tcp 
```

5. Using the following command, I create a topic called "events".  I will be publishing messages to this topic later:
```
docker-compose exec kafka kafka-topics --create --topic events --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181
```

This output shows that "events" topic created successfully:
```
Created topic "events".
```

6. Before I run the game_api_with_json_events.py file with Flask, I check its contents:
```
cat game_api_with_json_events.py
```

output:
```python
#!/usr/bin/env python
import json
from kafka import KafkaProducer
from flask import Flask

app = Flask(__name__)
producer = KafkaProducer(bootstrap_servers='kafka:29092')


def log_to_kafka(topic, event):
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

@app.route("/purchase_a_frog")
def purchase_a_frog():
    purchase_frog_event = {'event_type': 'purchase_frog'}
    log_to_kafka('events', purchase_frog_event)
    return "Frog Purchased!\n"
```

The game_api_with_json_events.py file resembles the game_api.py file from homework 9, but there are a couple of difference.  Like in homework 9, Flask is used to map our actions to single API calls.  This time, are 3 available actions - default_response(), purchase_a_sword(), and purchase_a_frog().  Like in homework 9, we are also using KafkaProducer which publishes events that result from the API calls to "events" topic.  In game_api.py file in homework 9, our events were encoded as text data type.  However, here in this activity, our events will be more informative, and will be encoded as python dictionaries (or json messages) that display the event type.

Next, I will run the game_api_with_json_events.py file using Flask:
```
docker-compose exec mids env FLASK_APP=/w205/assignment-10-dwang-ischool/game_api_with_json_events.py flask run --host 0.0.0.0
```

The initial output shows:
```
.py flask run --host 0.0.0.0
 * Serving Flask app "game_api_with_json_events"
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
```

7. In order to see the API calls in Flask, I will be performing the necessary actions in a **seperate** terminal window.  In my separate window, I navigate to my homework 10 repo where the cluster and Flask are running:
```
cd w205/assignment-10-dwang-ischool/
```

In the same **separate** terminal window, I perform 2 actions, the default_response action and the purchase a sword action, using the following curl commands:

```
docker-compose exec mids curl http://localhost:5000/
docker-compose exec mids curl http://localhost:5000/purchase_a_sword
```

In the same **separate** terminal window, I receive the following text outputs that signify my actions were performed.  The outputs appear after each command respectively:
```
This is the default response!
Sword Purchased!
```

In my **original** window, where Flask is running, I see the 2 API calls that correspond to the 2 actions I performed:

```
 * Serving Flask app "game_api_with_json_events"
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
127.0.0.1 - - [29/Nov/2018 20:46:52] "GET / HTTP/1.1" 200 -
127.0.0.1 - - [29/Nov/2018 20:46:57] "GET /purchase_a_sword HTTP/1.1" 200 -
```

For fun, I decide to perform 5 more actions, in the following order:
- purchase a sword
- puchase a frog
- puchase a frog
- default
- puchase a frog

I do these 5 actions in my **separate** window using the following curl commands:
```
docker-compose exec mids curl http://localhost:5000/purchase_a_sword
docker-compose exec mids curl http://localhost:5000/purchase_a_frog
docker-compose exec mids curl http://localhost:5000/purchase_a_frog
docker-compose exec mids curl http://localhost:5000/
docker-compose exec mids curl http://localhost:5000/purchase_a_frog
```
Which have the following outputs:
```
Sword Purchased!
Frog Purchased!
Frog Purchased!
This is the default response!
Frog Purchased!
```

In my **original** terminal window, I see the following output in the running Flask app:
```
127.0.0.1 - - [29/Nov/2018 20:52:28] "GET /purchase_a_sword HTTP/1.1" 200 -
127.0.0.1 - - [29/Nov/2018 20:52:46] "GET /purchase_a_frog HTTP/1.1" 200 -
127.0.0.1 - - [29/Nov/2018 20:53:11] "GET /purchase_a_frog HTTP/1.1" 200 -
127.0.0.1 - - [29/Nov/2018 20:53:14] "GET / HTTP/1.1" 200 -
127.0.0.1 - - [29/Nov/2018 20:53:21] "GET /purchase_a_frog HTTP/1.1" 200 -
```
Which corresponds to the 5 actions I just performed.


In addition, I perform 2 more actions, purchase_a_sword and purchase_a_frog by typing the following directly into my brower:
```
http://167.99.110.250:5000/purchase_a_sword
http://167.99.110.250:5000/purchase_a_frog
```
Which have these output messages, respectively, in my browser:
```
Sword Purchased!
Frog Purchased!
```

simultaneously, I see the following outputs in my Flask app:
```
98.14.67.243 - - [29/Nov/2018 20:56:42] "GET /purchase_a_sword HTTP/1.1" 200 -
98.14.67.243 - - [29/Nov/2018 20:56:57] "GET /purchase_a_frog HTTP/1.1" 200 -
```

In total, I performed 9 actions. 

8. To read the events from the "events" topic, I use the following command in my **separate** terminal window:
```
docker-compose exec mids kafkacat -C -b kafka:29092 -t events -o beginning -e
```

The command then has the following output:
```
{"event_type": "default"}
{"event_type": "purchase_sword"}
{"event_type": "purchase_sword"}
{"event_type": "purchase_frog"}
{"event_type": "purchase_frog"}
{"event_type": "default"}
{"event_type": "purchase_frog"}
{"event_type": "purchase_sword"}
{"event_type": "purchase_frog"}
% Reached end of topic events [0] at offset 10: exiting
```

Which matches the 9 actions that I performed in step 7. As we can see here, each event is a JSON events which shows the event type of the event.

I exit the Flask app in preparation for the next part of the activity:
```
^C
```

9. Before running game_api_with_extended_json_events.py with Flask, I check the contents of the file:
```
cat game_api_with_extended_json_events.py
```

The command shows this as output:
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

@app.route("/purchase_a_frog")
def purchase_a_frog():
    purchase_frog_event = {'event_type': 'purchase_frog'}
    log_to_kafka('events', purchase_frog_event)
    return "Frog Purchased!\n"
```
This file looks almost identical to game_api_with_json_events.py. We are still publishing json messages to the "events" topic for the same 3 actions that are mapped to API calls.  However, the messages will contain even more information, because the log_to_kafka includes an extra step to update the event message with a request.headers. 

Next, I run the game_api_with_json_events.py file using the Flask app again:
```
docker-compose exec mids env FLASK_APP=/w205/assignment-10-dwang-ischool/game_api_with_extended_json_events.py flask run --host 0.0.0.0
```
Initial output looks like this:
```
 * Serving Flask app "game_api_with_extended_json_events"
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
```

10. Next I will generate 9 events total, 6 through terminal and 3 through my browser.  

First in my **separate** window, I perform these 5 actions to generate events using curl commands:
```
docker-compose exec mids curl http://localhost:5000/purchase_a_sword
docker-compose exec mids curl http://localhost:5000/purchase_a_frog
docker-compose exec mids curl http://localhost:5000/purchase_a_frog
docker-compose exec mids curl http://localhost:5000/
docker-compose exec mids curl http://localhost:5000/
docker-compose exec mids curl http://localhost:5000/purchase_a_frog
```
And get the following corresponding outputs after each command:
```
Sword Purchased!
Frog Purchased!
Frog Purchased!
This is the default response!
This is the default response!
Frog Purchased!
```

Simultanesouly, in my **original** terminal window, where Flask app is running, I see the following API calls mapped from my 6 terminal window actions:
```
 * Serving Flask app "game_api_with_extended_json_events"
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
127.0.0.1 - - [29/Nov/2018 21:15:16] "GET /purchase_a_sword HTTP/1.1" 200 -
127.0.0.1 - - [29/Nov/2018 21:15:23] "GET /purchase_a_frog HTTP/1.1" 200 -
127.0.0.1 - - [29/Nov/2018 21:15:26] "GET /purchase_a_frog HTTP/1.1" 200 -
127.0.0.1 - - [29/Nov/2018 21:15:32] "GET / HTTP/1.1" 200 -
127.0.0.1 - - [29/Nov/2018 21:15:33] "GET / HTTP/1.1" 200 -
127.0.0.1 - - [29/Nov/2018 21:15:43] "GET /purchase_a_frog HTTP/1.1" 200 -
```

Next, for fun, I generate 3 events in the browser by typing the following directly into my browser:
```
http://167.99.110.250:5000/purchase_a_sword
http://167.99.110.250:5000/purchase_a_frog
http://167.99.110.250:5000/
```
For each address I typed into my browser, I received the following messages in my broswer window, respectively:
```
Sword Purchased!
Frog Purchased!
This is the default response!
```

Simultanesouly, in my **original** terminal window, where Flask app is running, I see the following API calls mapped from my 3 browser window actions:
```
98.14.67.243 - - [29/Nov/2018 21:21:04] "GET /purchase_a_sword HTTP/1.1" 200 -
98.14.67.243 - - [29/Nov/2018 21:21:15] "GET /purchase_a_frog HTTP/1.1" 200 -
98.14.67.243 - - [29/Nov/2018 21:21:21] "GET / HTTP/1.1" 200 -
```

11. In my **separate** terminal window, I run the following command to read the events from Kafka:
```
docker-compose exec mids kafkacat -C -b kafka:29092 -t events -o beginning -e 
```
I see the following output as a result:
```
{"event_type": "default"} 
{"event_type": "purchase_sword"} 
{"event_type": "purchase_sword"} 
{"event_type": "purchase_frog"} 
{"event_type": "purchase_frog"} 
{"event_type": "default"} 
{"event_type": "purchase_frog"} 
{"event_type": "purchase_sword"} 
{"event_type": "purchase_frog"} 
{"Host": "localhost:5000", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "curl/7.47.0"} 
{"Host": "localhost:5000", "event_type": "purchase_frog", "Accept": "*/*", "User-Agent": "curl/7.47.0"} 
{"Host": "localhost:5000", "event_type": "purchase_frog", "Accept": "*/*", "User-Agent": "curl/7.47.0"} 
{"Host": "localhost:5000", "event_type": "default", "Accept": "*/*", "User-Agent": "curl/7.47.0"} 
{"Host": "localhost:5000", "event_type": "default", "Accept": "*/*", "User-Agent": "curl/7.47.0"} 
{"Host": "localhost:5000", "event_type": "purchase_frog", "Accept": "*/*", "User-Agent": "curl/7.47.0"} 
{"Accept-Language": "en-US,en;q=0.9", "event_type": "purchase_sword", "Host": "167.99.110.250:5000", "Accept": "text/html,application/x 
html+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8", "Upgrade-Insecure-Requests": "1", "Connection": "keep-alive", "Cookie" 
: "user-id=science|Thu%2C%2018%20Oct%202018%2016%3A31%3A08%20GMT|i764hQ8rHjoH9cKq89Mw7%2BAGlf9zsO5FHy13ihq7Ydo%3D; csrf-token=c43770d1- 
69d3-4688-8190-1f2186a421a1; _xsrf=2|0c8edcc1|f3cbccafa7c7c4dcce9035d7f0dc2197|1543518700", "User-Agent": "Mozilla/5.0 (Macintosh; Inte 
l Mac OS X 10_13_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.110 Safari/537.36", "Accept-Encoding": "gzip, deflate"} 
{"Accept-Language": "en-US,en;q=0.9", "event_type": "purchase_frog", "Host": "167.99.110.250:5000", "Accept": "text/html,application/xh 
tml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8", "Upgrade-Insecure-Requests": "1", "Connection": "keep-alive", "Cookie": 
 "user-id=science|Thu%2C%2018%20Oct%202018%2016%3A31%3A08%20GMT|i764hQ8rHjoH9cKq89Mw7%2BAGlf9zsO5FHy13ihq7Ydo%3D; csrf-token=c43770d1-6 
9d3-4688-8190-1f2186a421a1; _xsrf=2|0c8edcc1|f3cbccafa7c7c4dcce9035d7f0dc2197|1543518700", "User-Agent": "Mozilla/5.0 (Macintosh; Intel 
 Mac OS X 10_13_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.110 Safari/537.36", "Accept-Encoding": "gzip, deflate"} 
{"Accept-Language": "en-US,en;q=0.9", "event_type": "default", "Host": "167.99.110.250:5000", "Accept": "text/html,application/xhtml+xm 
l,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8", "Upgrade-Insecure-Requests": "1", "Connection": "keep-alive", "Cookie": "user 
-id=science|Thu%2C%2018%20Oct%202018%2016%3A31%3A08%20GMT|i764hQ8rHjoH9cKq89Mw7%2BAGlf9zsO5FHy13ihq7Ydo%3D; csrf-token=c43770d1-69d3-46 
88-8190-1f2186a421a1; _xsrf=2|0c8edcc1|f3cbccafa7c7c4dcce9035d7f0dc2197|1543518700", "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac O 
S X 10_13_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.110 Safari/537.36", "Accept-Encoding": "gzip, deflate"} 
```

As expected, the first 9 events are simple json format for the first part of the activity in step 7.  The next 6 events of the format "{"Host": "localhost:5000", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "curl/7.47.0"}" correspond to the 6 curl command I just ran in step 10.  They are more informative versions that include the headers, as well as the event types. 

The last 3 events of format "{"Accept-Language": "en-US,en;q=0.9", "event_type": "purchase_sword", "Host": "167.99.110.250:5000", "Accept": "text/html,application/x 
html+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8", "Upgrade-Insecure-Requests": "1", "Connection": "keep-alive", "Cookie" 
: "user-id=science|Thu%2C%2018%20Oct%202018%2016%3A31%3A08%20GMT|i764hQ8rHjoH9cKq89Mw7%2BAGlf9zsO5FHy13ihq7Ydo%3D; csrf-token=c43770d1- 
69d3-4688-8190-1f2186a421a1; _xsrf=2|0c8edcc1|f3cbccafa7c7c4dcce9035d7f0dc2197|1543518700", "User-Agent": "Mozilla/5.0 (Macintosh; Inte 
l Mac OS X 10_13_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.110 Safari/537.36", "Accept-Encoding": "gzip, deflate"}"

are extremely informative.  They are run from my browser.  

12. Next, I run the spark shell in my **separate** terminal window:
```
docker-compose exec spark pyspark
```

I use the following to read my events from kafka into raw_events and I examine raw_events:
```
raw_events = spark \
  .read \
  .format("kafka") \
  .option("kafka.bootstrap.servers", "kafka:29092") \
  .option("subscribe","events") \
  .option("startingOffsets", "earliest") \
  .option("endingOffsets", "latest") \
  .load() 
raw_events.show()
```

I see this output:
```
+----+--------------------+------+---------+------+--------------------+-------------+
| key|               value| topic|partition|offset|           timestamp|timestampType|
+----+--------------------+------+---------+------+--------------------+-------------+
|null|[7B 22 65 76 65 6...|events|        0|     0|2018-11-29 20:46:...|            0|
|null|[7B 22 65 76 65 6...|events|        0|     1|2018-11-29 20:46:...|            0|
|null|[7B 22 65 76 65 6...|events|        0|     2|2018-11-29 20:46:...|            0|
|null|[7B 22 65 76 65 6...|events|        0|     3|2018-11-29 20:52:...|            0|
|null|[7B 22 65 76 65 6...|events|        0|     4|2018-11-29 20:52:...|            0|
|null|[7B 22 65 76 65 6...|events|        0|     5|2018-11-29 20:53:...|            0|
|null|[7B 22 65 76 65 6...|events|        0|     6|2018-11-29 20:53:...|            0|
|null|[7B 22 65 76 65 6...|events|        0|     7|2018-11-29 20:53:...|            0|
|null|[7B 22 65 76 65 6...|events|        0|     8|2018-11-29 20:56:...|            0|
|null|[7B 22 65 76 65 6...|events|        0|     9|2018-11-29 20:56:...|            0|
|null|[7B 22 65 76 65 6...|events|        0|    10|2018-11-29 21:03:...|            0|
|null|[7B 22 65 76 65 6...|events|        0|    11|2018-11-29 21:03:...|            0|
|null|[7B 22 48 6F 73 7...|events|        0|    12|2018-11-29 21:15:...|            0|
|null|[7B 22 48 6F 73 7...|events|        0|    13|2018-11-29 21:15:...|            0|
|null|[7B 22 48 6F 73 7...|events|        0|    14|2018-11-29 21:15:...|            0|
|null|[7B 22 48 6F 73 7...|events|        0|    15|2018-11-29 21:15:...|            0|
|null|[7B 22 48 6F 73 7...|events|        0|    16|2018-11-29 21:15:...|            0|
|null|[7B 22 48 6F 73 7...|events|        0|    17|2018-11-29 21:15:...|            0|
|null|[7B 22 41 63 63 6...|events|        0|    18|2018-11-29 21:21:...|            0|
|null|[7B 22 41 63 63 6...|events|        0|    19|2018-11-29 21:21:...|            0|
+----+--------------------+------+---------+------+--------------------+-------------+
```


Next I cast the events as strings and then I examine the events:
```
raw_events.cache()
events = raw_events.select(raw_events.value.cast('string'))
events.show()
```
output:
```
+--------------------+ 
|               value| 
+--------------------+ 
|{"event_type": "d...| 
|{"event_type": "d...| 
|{"event_type": "p...| 
|{"event_type": "p...| 
|{"event_type": "p...| 
|{"event_type": "p...| 
|{"event_type": "d...| 
|{"event_type": "p...| 
|{"event_type": "p...| 
|{"event_type": "p...| 
|{"event_type": "p...| 
|{"event_type": "d...| 
|{"Host": "localho...| 
|{"Host": "localho...| 
|{"Host": "localho...| 
|{"Host": "localho...| 
|{"Host": "localho...| 
|{"Host": "localho...| 
|{"Accept-Language...| 
|{"Accept-Language...| 
+--------------------+ 
only showing top 20 rows 
```

Lastly, I create a nice table for the events, called extracted_events, and then I examine extracted_events:
```
from pyspark.sql import Row
import json
extracted_events = events.rdd.map(lambda x: json.loads(x.value)).toDF()
extracted_events.show()
```
ouput:
```
+--------------+
|    event_type|
+--------------+
|       default|
|       default|
|purchase_sword|
|purchase_sword|
| purchase_frog|
| purchase_frog|
|       default|
| purchase_frog|
|purchase_sword|
| purchase_frog|
|purchase_sword|
|       default|
|purchase_sword|
| purchase_frog|
| purchase_frog|
|       default|
|       default|
| purchase_frog|
|purchase_sword|
| purchase_frog|
+--------------+
only showing top 20 rows
```

Once I am done examining my events in spark, I exit pyspark:
```
exit()
```

Lastly spin down my cluster:
```
docker-compose down
```

I also store my original terminal window activity:
```
history > originalwindow.txt
```
and my secondary terminal window activity:
```
history > secondarywindow.txt
```

