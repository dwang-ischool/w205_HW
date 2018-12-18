# Project 3 Setup

- You're a data scientist at a game development company.  
- Your latest mobile game has two events you're interested in tracking: 
- `buy a sword` & `join guild`...
- Each has metadata

## Project 3 Task
- Your task: instrument your API server to catch and analyze these two
event types.
- This task will be spread out over the last four assignments (9-12).

---

# Assignment 09

## Follow the steps we did in class 
- for both the simple flask app and the more complex one.

### Turn in your `/assignment-09-<user-name>/README.md` file. It should include:
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
1. First, I cloned the assignment 9 repo into my droplet and navigated into the repo's folder:
```
git clone https://github.com/mids-w205-martin-mims/assignment-09-dwang-ischool.git
cd assignment-09-dwang-ischool/
```

2. While insidet he assignment 9 repo, I copied the docker-compose.yml file and the 2.py files from the week 9 course-content repo using the following commands: 
```
cp ~/w205/course-content/09-Ingesting-Data/docker-compose.yml .
cp ~/w205/course-content/09-Ingesting-Data/*.py .
```

3. Next I check the docker-compose.yml file to make sure that it contains the zookeeper, kafka, and mids containers. 
Using the command:
```
cat docker-compose.yml
```

I get the following output:
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
    expose:
      - "9092"
      - "29092"

  mids:
    image: midsw205/base:0.1.8
    stdin_open: true
    tty: true
    expose:
      - "5000"
    ports:
      - "5000:5000"
    volumes:
      - ~/w205:/w205
```

Which matches the example from class

4. I then spin up the cluster with the kafka zookeeper and mids components.  I also check that the containers are active:
```
docker-compose up -d
docker-compose ps
```

I get the following output, which shows that the containers did successfully spin up:
```
                Name                            Command            State                    Ports                  
------------------------------------------------------------------------------------------------------------------
assignment09dwangischool_kafka_1       /etc/confluent/docker/run   Up      29092/tcp, 9092/tcp                     
assignment09dwangischool_mids_1        /bin/bash                   Up      0.0.0.0:5000->5000/tcp, 8888/tcp        
assignment09dwangischool_zookeeper_1   /etc/confluent/docker/run   Up      2181/tcp, 2888/tcp, 32181/tcp, 3888/tcp 
```

5. Using the following command, I create a topic called "events".  The "events" topic will be where messages are published to. 
```
docker-compose exec kafka kafka-topics --create --topic events --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181
```
output:
```
Created topic "events".
```

6. Before using Flask to run the basic_game_api.py file, I check the contents of the file using the following command:
```
cat basic_game_api.py
```

I see the following output:
```
#!/usr/bin/env python
from flask import Flask
app = Flask(__name__)

@app.route("/")
def default_response():
    return "This is the default response!"

@app.route("/purchase_a_sword")
def purchase_sword():
    # business logic to purchase sword
    return "Sword Purchased!"
```

From this file, I see that we will be using the Flask method from the python flask libray to map our actions to single API calls.  The first method, using "@app.route("/")" maps the default_response action while the second method using "@app.route("/purchase_a_sword")" will map the sword purchansing action. 

Next, I run the basic_game_api.py file using flask:
```
docker-compose exec mids env FLASK_APP=/w205/flask-with-kafka/basic_game_api.py flask run
```

The initial output shows:
```
 * Serving Flask app "basic_game_api"
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```

7. In order to see API calls, I must perform the necessary actions.  First I open a **separate** terminal window, where I once again navigate to my assignment 9 repo where the clusters have been spun up and flask is running:
```
cd w205
cd assignment-09-dwang-ischool/
```

8. Next in my **separate** terminal window I perform a single action, in this case the default_response action, using the following curl command:
```
docker-compose exec mids curl http://localhost:5000/
```

In the **separate** terminal window, I see the following output:
```
This is the default response!
```
Which is the output that is dictated by the return statement in the default_response() method of the basic_game_api.py file.

In the **original** terminal window, where the flask app is running, I see the following output:
```
127.0.0.1 - - [19/Nov/2018 14:47:57] "GET / HTTP/1.1" 200 
```
Which shows that an API call has been made.

9. Next in my **separate** terminal window I perform another single action, in this case the purchase_sword() action, using the following curl command

```
docker-compose exec mids curl http://localhost:5000/purchase_a_sword
```

In the **separate** terminal window, I see the following output:
```
Sword Purchased!
```
Which is the output that is dictated byt he return statement in the purchase_sword() method in the basic_game_api.py file.

In the **original** terminal window, where the flask app is running, I see the following output:
```
127.0.0.1 - - [19/Nov/2018 14:53:34] "GET /purchase_a_sword HTTP/1.1" 200 -
```

Which again shows that an API call has been made.  This time, the call is distinguished by "purchase_a_sword" which comes after the "/" after GET.


10. Lastly, I perform 7 actions in the following order:
default_response
purchase_sword
purchase_sword
purchase_sword
default_response
default_response
purchase_sword

In my **separate** terminal window, I use the following commands:
```
docker-compose exec mids curl http://localhost:5000/
docker-compose exec mids curl http://localhost:5000/purchase_a_sword
docker-compose exec mids curl http://localhost:5000/purchase_a_sword
docker-compose exec mids curl http://localhost:5000/purchase_a_sword
docker-compose exec mids curl http://localhost:5000/
docker-compose exec mids curl http://localhost:5000/
docker-compose exec mids curl http://localhost:5000/purchase_a_sword
```

Which results in the following outputs:
```
This is the default response!
Sword Purchased!
Sword Purchased!
Sword Purchased!
This is the default response!
This is the default response!
Sword Purchased!
```

In the **original** terminal window, where the flask app is running, I see the following output:
```
127.0.0.1 - - [19/Nov/2018 14:56:28] "GET / HTTP/1.1" 200 -
127.0.0.1 - - [19/Nov/2018 14:56:31] "GET /purchase_a_sword HTTP/1.1" 200 -
127.0.0.1 - - [19/Nov/2018 14:56:32] "GET /purchase_a_sword HTTP/1.1" 200 -
127.0.0.1 - - [19/Nov/2018 14:56:35] "GET /purchase_a_sword HTTP/1.1" 200 -
127.0.0.1 - - [19/Nov/2018 14:56:37] "GET / HTTP/1.1" 200 -
127.0.0.1 - - [19/Nov/2018 14:56:39] "GET / HTTP/1.1" 200 -
127.0.0.1 - - [19/Nov/2018 14:56:45] "GET /purchase_a_sword HTTP/1.1" 200 -
```
Which represents all 7 API calls.  Watching the **original** terminal window also allowed me to see the API calls in real time, one for each of my commands. 

11. To conclude the exercise with basic_game_api.py, I exit the flask app using the following command:
```
^C
```

12. To prepare for the generation of events using Kafka, I first check the contents of the game_api.py file:

Using the following command:
```
cat game_api.py
```

I get the following output:
```
#!/usr/bin/env python
from kafka import KafkaProducer
from flask import Flask
app = Flask(__name__)
event_logger = KafkaProducer(bootstrap_servers='kafka:29092')
events_topic = 'events'

@app.route("/")
def default_response():
    event_logger.send(events_topic, 'default'.encode())
    return "This is the default response!"

@app.route("/purchase_a_sword")
def purchase_sword():
    # business logic to purchase sword
    # log event to kafka
    event_logger.send(events_topic, 'purchased_sword'.encode())
    return "Sword Purchased!"
```

I can see that the game_api.py file differ from the basic_game_api.py8 file in several different ways.  The most important is that the game_api.py file uses the KafkaProducer from python kafka library.  We use the KafkaProducer to create an event_logger function which then logs the event of each action mapping to an API call as an encoded string, which then gets published to our topic "events", which was created step 5. 

We can see from this file that the default_response() action creates an event denoted by text "default" while the purchase_sword() action creates an event denoted by the text "purchased_sword".  The API call mapping per action remains identical, so we once we run this file using flask, we should be able to see the same api calls.  

13. First I run game_api.py file using flask using the following command:
```
docker-compose exec mids env FLASK_APP=/w205/flask-with-kafka/game_api.py flask run
```

I see this output to signify that the flask app is indeed running:
```
 * Serving Flask app "game_api"
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```

14. As with step 10, I will perform 7 actions, 3 of which are the default_response() action and 4 are purchasing a sword.  The actions will be performed in the following order:
default_response
purchase_sword
purchase_sword
purchase_sword
default_response
default_response
purchase_sword

In a **separate** terminal window, I perform the actions using the following commands:
```
docker-compose exec mids curl http://localhost:5000/
docker-compose exec mids curl http://localhost:5000/purchase_a_sword
docker-compose exec mids curl http://localhost:5000/purchase_a_sword
docker-compose exec mids curl http://localhost:5000/purchase_a_sword
docker-compose exec mids curl http://localhost:5000/
docker-compose exec mids curl http://localhost:5000/
docker-compose exec mids curl http://localhost:5000/purchase_a_sword
```

which results in the following outputs in my **separate** terminal window:
```
This is the default response!
Sword Purchased!
Sword Purchased!
Sword Purchased!
This is the default response!
This is the default response!
Sword Purchased!
```

In my **original** terminal window where the flask app is running, I see the following output:
```
127.0.0.1 - - [19/Nov/2018 15:13:53] "GET / HTTP/1.1" 200 -
127.0.0.1 - - [19/Nov/2018 15:14:12] "GET /purchase_a_sword HTTP/1.1" 200 -
127.0.0.1 - - [19/Nov/2018 15:14:16] "GET /purchase_a_sword HTTP/1.1" 200 -
127.0.0.1 - - [19/Nov/2018 15:14:18] "GET /purchase_a_sword HTTP/1.1" 200 -
127.0.0.1 - - [19/Nov/2018 15:14:21] "GET / HTTP/1.1" 200 -
127.0.0.1 - - [19/Nov/2018 15:14:23] "GET / HTTP/1.1" 200 -
127.0.0.1 - - [19/Nov/2018 15:14:25] "GET /purchase_a_sword HTTP/1.1" 200 -
```
Which are the 7 API calls that correspond to my 7 actions in the order in which they were made.  This part is not different from the activity in 10. 

However, the difference in this particular example is that the "events" topic should now show 7 events.  I use the following command to then consume the events from the "events" topic using Kafkacat.  Please note that this command is run in the **separate** terminal window:
```
docker-compose exec mids bash -c "kafkacat -C -b kafka:29092 -t events -o beginning -e"
```

I then get the following output:
```
default
purchased_sword
purchased_sword
purchased_sword
default
default
purchased_sword
% Reached end of topic events [0] at offset 7: exiting
```

As discussed in my explanation for 12, the events that are published to the "events" topic are in the form of encoded strings.  The "default" encodes the event of the default_response() action while the "purchase_sword" string encodes the event of the purchase_sword() action.  The events are published in the exact order in which the actions were performed, and also match the order of the mapped API calls. 

14.  I exit the flask app:
```
^C
```

Then I take down the cluster:
```
docker-compose down
```

History files are saved using the following commands:

**original** terminal window: 
```
history --> original_window.txt
```

**separate** terminal window:
```
history --> separate_window.txt
```