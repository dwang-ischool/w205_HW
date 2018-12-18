
# Dili Wang Annotations, Assignment 5

## Summary of work:
1. The docker-compose.yml file is in the redis-standalone folder here - https://github.com/mids-w205-martin-mims/assignment-05-dwang-ischool/tree/master/redis-standalone

2. The bonus exercise is inside of the notebook Simple_Redis_Test.ipynb, directly in this repo

3. The extras exercise is inside of the notebook Extras_Exercise.ipynb, directly inside of this repo


## Annotations of commands:

### Going to course content repo and pulling the latest version from github
14 cd ~/w205/course-content 
15 git pull --all

### Making the redis-standalone folder but inside of my assignment 5 repo:
16 mkdir ~/w205/assignment-05-dwang-ischool/redis-standalone 

### Going to the redis-standalone folder and copying the docker-compose.yml file from repo into folder:
17 cd ~/w205/assignment-05-dwang-ischool/redis-standalone 
18 cp ../../course-content/05-Storing-Data-II/example-0-docker-compose.yml docker-compose.yml

### Checking that my .yml file matches the content in the async slides:
19 cat docker-compose.yml

output:

```
---
version: '2'
services:
  redis:
    image: redis
    expose:
      - "6379"
    ports:
      - "6379:6379"
```

### Spinning up cluster
20 docker-compose up -d

### Check container is up:
21 docker-compose ps

output:
```
         Name                        Command               State           Ports          
-----------------------------------------------------------------------------------------
redisstandalone_redis_1   docker-entrypoint.sh redis ...   Up      0.0.0.0:6379->6379/tcp 
```

### Look at logs and verify that log ends in "Ready to accept connections":
22 docker-compose logs redis

### Running python to tryout redis inside:
23 ipython

```
In [1]: import redis 

In [2]: r = redis.Redis(host='localhost', port='6379')

In [3]: r.keys()
Out[3]: []

In [4]: exit
```

### Tear down stack 
24 docker-compose down

### Check Stack tear down
25 docker-compose ps

### After editing the docker-compose.yml file directly to add the midsw205/base:latest, check file:
26 cat docker-compose.yml

output:
```
---
version: '2'
services:
  redis:
    image: redis:latest
    expose:
      - "6379"
    #ports:
      #- "6379:6379"
      
mids:
    image: midsw205/base:latest
    stdin_open: true
    tty: true  
```

### Spin up cluster again:
27 docker-compose up -d

### Check both mids and redis containers are up:
28 docker-compose ps

ouput:
```
         Name                        Command               State    Ports   
---------------------------------------------------------------------------
redisstandalone_mids_1    /bin/bash                        Up      8888/tcp 
redisstandalone_redis_1   docker-entrypoint.sh redis ...   Up      6379/tcp 
```

### Look at logs and verify that log ends in "Ready to accept connections":
29 docker-compose logs redis

### connect to mids container
30 docker-compose exec mids bash

### running python, trying out redis, exiting python, exiting container

root@b9d8fcc2eb67:~# ipython

```
In [1]: import redis

In [2]: r = redis.Redis(host='redis', port = '6379')

In [3]: r.keys()
Out[3]: []

In [4]: exit
```
root@b9d8fcc2eb67:~# exit

### Tearing down stack again and verifying it is down:
32 docker-compose down
33 docker-compose ps

### Per next set of instructions adding and exposing a port to the docker-compose.yml file, then checking file:
34 cat docker-compose.yml

output:
```
---
version: '2'
services:
  redis:
    image: redis:latest
    expose:
      - "6379"
    #ports:
      #- "6379:6379"

  mids:
    image: midsw205/base:latest
    stdin_open: true
    tty: true
    expose:
      - "8888"
    ports:
      - "8888:8888"
```

### spinning up cluster again:
35 docker-compose up -d

### starting up a notebook:
36 docker-compose exec mids jupyter notebook --no-browser --port 8888 --ip 0.0.0.0 --allow-root

### using output token, successfully started a notebook pasting the following in my browser:
http://167.99.110.250:8888/?token=324dbcf608a2bf69a81640bb268e204e3301a9029080840e

### output of starting, then quitting the notebook:
```
[I 01:23:29.652 NotebookApp] Writing notebook server cookie secret to /w205/.local/share/jupyter/runtime/notebook_cookie_secret
[I 01:23:30.035 NotebookApp] JupyterLab beta preview extension loaded from /usr/local/lib/python3.5/dist-packages/jupyterlab
[I 01:23:30.035 NotebookApp] JupyterLab application directory is /usr/local/share/jupyter/lab
[I 01:23:30.043 NotebookApp] Serving notebooks from local directory: /w205
[I 01:23:30.043 NotebookApp] 0 active kernels
[I 01:23:30.043 NotebookApp] The Jupyter Notebook is running at:
[I 01:23:30.043 NotebookApp] http://0.0.0.0:8888/?token=324dbcf608a2bf69a81640bb268e204e3301a9029080840e
[I 01:23:30.043 NotebookApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
[C 01:23:30.044 NotebookApp] 
    
    Copy/paste this URL into your browser when you connect for the first time,
    to login with a token:
        http://0.0.0.0:8888/?token=324dbcf608a2bf69a81640bb268e204e3301a9029080840e
[I 01:24:20.841 NotebookApp] 302 GET /?token=324dbcf608a2bf69a81640bb268e204e3301a9029080840e (98.14.67.243) 1.32ms
^C[I 01:26:40.732 NotebookApp] interrupted
Serving notebooks from local directory: /w205
0 active kernels
The Jupyter Notebook is running at:
http://0.0.0.0:8888/?token=324dbcf608a2bf69a81640bb268e204e3301a9029080840e
Shutdown this notebook server (y/[n])? y
[C 01:26:45.055 NotebookApp] Shutdown confirmed
[I 01:26:45.056 NotebookApp] Shutting down 0 kernels
```

### dropping the cluster before the next step:
37 docker-compose down

### adding command to docker-compose.yml file for automating jupyter notebook startup, then checking file:
39 cat docker-compose.yml

output:
```
---
version: '2'
services:
  redis:
    image: redis:latest
    expose:
      - "6379"
    #ports:
      #- "6379:6379"
 
  mids:
    image: midsw205/base:latest
    stdin_open: true
    tty: true
    expose:
      - "8888"
    ports:
      - "8888:8888"
    command: jupyter notebook --no-browser --port 8888 --ip 0.0.0.0 --allow-root
```

### spinning up cluster:
40 docker-compose up -d

### get token:
42 docker-compose logs mids

output:
```
Attaching to redisstandalone_mids_1
mids_1   | [I 01:38:00.974 NotebookApp] Writing notebook server cookie secret to /w205/.local/share/jupyter/runtime/notebook_cookie_secret
mids_1   | [I 01:38:01.356 NotebookApp] JupyterLab beta preview extension loaded from /usr/local/lib/python3.5/dist-packages/jupyterlab
mids_1   | [I 01:38:01.356 NotebookApp] JupyterLab application directory is /usr/local/share/jupyter/lab
mids_1   | [I 01:38:01.368 NotebookApp] Serving notebooks from local directory: /w205
mids_1   | [I 01:38:01.369 NotebookApp] 0 active kernels
mids_1   | [I 01:38:01.370 NotebookApp] The Jupyter Notebook is running at:
mids_1   | [I 01:38:01.370 NotebookApp] http://0.0.0.0:8888/?token=d586926209a0ff01b912f0c7ebb55f9c9101a54d78098f56
mids_1   | [I 01:38:01.370 NotebookApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
mids_1   | [C 01:38:01.372 NotebookApp] 
mids_1   |     
mids_1   |     Copy/paste this URL into your browser when you connect for the first time,
mids_1   |     to login with a token:
mids_1   |         http://0.0.0.0:8888/?token=d586926209a0ff01b912f0c7ebb55f9c9101a54d78098f56
```

### successfully opened notebook by pasting the following in browser:
http://167.99.110.250:8888/?token=d586926209a0ff01b912f0c7ebb55f9c9101a54d78098f56



## Bonus:

Following the previous step, in my brower, I opened a new Python 3 notebook.  I tried out some simple commands with the following outputs:

```
import redis
r = redis.Redis(host='redis', port='6379')
r.keys()

Out[1]: []

r.set('Dili', 'Wang')
value = r.get('Dili')
print(value)

b'Wang'

r.keys()
Out[5]: [b'Dili']

r.set('Hello', 'World')
print(r.get('Hello'))

b'World'

r.keys()
Out[8]: [b'Hello', b'Dili']
```

The notebook of my test is saved to the same repo as this file, and is called "Simple_Redis_Test.ipynb"

### cluster dropped after the bonus exercise:
43 docker-compose down



## Extras:

### download data:
44 curl -L -o trips.csv https://goo.gl/QvHLKe

### add volumns to the docker-compose.yml, checking the file:
45 cat docker-compose.yml

output:
```
---
version: '2'
services:
  redis:
    image: redis:latest
    expose:
      - "6379"
    #ports:
      #- "6379:6379"

  mids:
    image: midsw205/base:latest
    stdin_open: true
    tty: true
    volumes:
        - ~/w205/assignment-05-dwang-ischool/redis-standalone:/w205
    expose:
      - "8888"
    ports:
      - "8888:8888"
    command: jupyter notebook --no-browser --port 8888 --ip 0.0.0.0 --allow-root
```

### spin up cluster
46 docker-compose up -d

### get token:
47 docker-compose logs mids

output:
```
Attaching to redisstandalone_mids_1
mids_1   | [I 02:04:15.242 NotebookApp] Writing notebook server cookie secret to /w205/.local/share/jupyter/runtime/notebook_cookie_secret
mids_1   | [I 02:04:15.692 NotebookApp] JupyterLab beta preview extension loaded from /usr/local/lib/python3.5/dist-packages/jupyterlab
mids_1   | [I 02:04:15.692 NotebookApp] JupyterLab application directory is /usr/local/share/jupyter/lab
mids_1   | [I 02:04:15.703 NotebookApp] Serving notebooks from local directory: /w205
mids_1   | [I 02:04:15.703 NotebookApp] 0 active kernels
mids_1   | [I 02:04:15.703 NotebookApp] The Jupyter Notebook is running at:
mids_1   | [I 02:04:15.703 NotebookApp] http://0.0.0.0:8888/?token=3b0c9188dff58f8c02466f543e35ba85d240d471661979c7
mids_1   | [I 02:04:15.704 NotebookApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
mids_1   | [C 02:04:15.704 NotebookApp] 
mids_1   |     
mids_1   |     Copy/paste this URL into your browser when you connect for the first time,
mids_1   |     to login with a token:
mids_1   |         http://0.0.0.0:8888/?token=3b0c9188dff58f8c02466f543e35ba85d240d471661979c7
```

### successfully opened notebook by pasting the following in browser:
http://167.99.110.250:8888/?token=3b0c9188dff58f8c02466f543e35ba85d240d471661979c7

saw that the docker-compose.yml file and trips.csv were inside the jupyter hub

### ran the following commands in a python 3 notebook:
```
import redis
import pandas as pd

trips=pd.read_csv('trips.csv')

date_sorted_trips = trips.sort_values(by='end_date')

date_sorted_trips.head()

for trip in date_sorted_trips.itertuples():
  print(trip.end_date, '', trip.bike_number, '', trip.end_station_name)
  
current_bike_locations = redis.Redis(host='redis', port='6379')
current_bike_locations.keys()

for trip in date_sorted_trips.itertuples():
  current_bike_locations.set(trip.bike_number, trip.end_station_name)
  
current_bike_locations.get('92')
```

The location of bike 92 is "San Jose Civic Center"!

The notebook of this exercise is called "Extras_Exercise.ipynb" and in the same repo as this file

### drop cluster
48  docker-compose down