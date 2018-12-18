# template-activity-03


# Query Project

- In the Query Project, you will get practice with SQL while learning about
  Google Cloud Platform (GCP) and BiqQuery. You'll answer business-driven
  questions using public datasets housed in GCP. To give you experience with
  different ways to use those datasets, you will use the web UI (BiqQuery) and
  the command-line tools, and work with them in jupyter notebooks.

- We will be using the Bay Area Bike Share Trips Data
  (https://cloud.google.com/bigquery/public-data/bay-bike-share). 

#### Problem Statement
- You're a data scientist at Ford GoBike (https://www.fordgobike.com/), the
  company running Bay Area Bikeshare. You are trying to increase ridership, and
  you want to offer deals through the mobile app to do so. What deals do you
  offer though? Currently, your company has three options: a flat price for a
  single one-way trip, a day pass that allows unlimited 30-minute rides for 24
  hours and an annual membership. 

- Through this project, you will answer these questions: 
  * What are the 5 most popular trips that you would call "commuter trips"?
  * What are your recommendations for offers (justify based on your findings)?


## Assignment 03 - Querying data from the BigQuery CLI - set up 

### What is Google Cloud SDK?
- Read: https://cloud.google.com/sdk/docs/overview

- If you want to go further, https://cloud.google.com/sdk/docs/concepts has
  lots of good stuff.

### Get Going

- Install Google Cloud SDK: https://cloud.google.com/sdk/docs/

- Try BQ from the command line:

  * General query structure

    ```
    bq query --use_legacy_sql=false '
        SELECT count(*)
        FROM
           `bigquery-public-data.san_francisco.bikeshare_trips`'
    ```

### Queries

1. Rerun last week's queries using bq command line tool (Paste your bq
   queries):

- What's the size of this dataset? (i.e., how many trips)
```sql
    bq query --use_legacy_sql=false '
        SELECT count(*)
        FROM `bigquery-public-data.san_francisco.bikeshare_trips`'
```
Result: 107501619
- What is the earliest start time and latest end time for a trip?
  - Query 1 - If we interpret minimum start time to mean the first trip ever in the data and maximum end time to mean the last trip ever in the dataset:
```sql
    bq query --use_legacy_sql=false '
      SELECT min(start_date) as min_start_time, max(end_date) as max_end_time
      FROM `bigquery-public-data.san_francisco.bikeshare_trips`'
```
Result: 

|   min_start_time    |    max_end_time     |
| --- | --- |
| 2013-08-29 09:08:00 | 2016-08-31 23:48:00 |

  - Query 2 - If we interpret the minimum start time to mean the time of the first trip, regardless of date, and the maximum end time of the last trip, regardless of date:

Min Start Time
```sql
    bq query --use_legacy_sql=false '
      SELECT
         EXTRACT(TIME FROM start_date) AS start_time
      FROM
        `bigquery-public-data.san_francisco.bikeshare_trips`
      ORDER BY
        start_time 
      LIMIT 1
```
Result:  00:00:00 

Max End Time
```sql
    bq query --use_legacy_sql=false '
      SELECT
         EXTRACT(TIME FROM end_date) AS end_time
      FROM
        `bigquery-public-data.san_francisco.bikeshare_trips`
      ORDER BY
        end_time DESC
      LIMIT 1
```

Result:  23:59:00

- How many bikes are there?
```sql
    bq query --use_legacy_sql=false '
      SELECT
        count(distinct bike_number)
      FROM `bigquery-public-data.san_francisco.bikeshare_trips`'
```
Result: 700

2. New Query (Paste your SQL query and answer the question in a sentence):

- How many trips are in the morning vs in the afternoon? 

For this question, I think that it depends on what is constituted as morning vs afternoon.  If we count morning trips as trips before 12:00 pm and afternoon trips as trips after 12:00 pm, regardless of time, then the queries are:

Morning:
```sql
    bq query --use_legacy_sql=false '
      SELECT count(*)
      FROM `bigquery-public-data.san_francisco.bikeshare_trips`
      WHERE TIME(start_date) <= "12:00:00"'
```
Result: 413092

Afternoon:
```sql
    bq query --use_legacy_sql=false '
      SELECT count(*)
      FROM `bigquery-public-data.san_francisco.bikeshare_trips`
      WHERE TIME(start_date) >= "12:00:00"'
```
Result: 571309

Answer: There are 413092 Trips in the morning and 571309 Trips in the afternoon. 

### Project Questions
Identify the main questions you'll need to answer to make recommendations (list
below, add as many questions as you need).

- Question 1: What are the 5 most popular routes and what are their average durations?

- Question 2: What is the average duration of trips taken by customers vs subscribers?

- Question 3: What are the 5 most popular routes taken by subscribers?

- Question 4: What are the 5 most popular routes taken by customers?

- Question 5: Of the 5 most popular routes, how many were taken by subscribers?

- Question 6: What are the top 10 of the most common times that trips take place? (for time grouped by hour)

- Question 7: Which 10 stations are the most popular?

- Question 8: Which stations are most popular during rush-hour time periods found in question 6?

- Question 9: What are the 10 most common routes during the most popular hours (rush-hour time periods in question 6) and how many subscribers versus customers travel these routes?

### Answers

Answer at least 4 of the questions you identified above You can use either
BigQuery or the bq command line tool.  Paste your questions, queries and
answers below.

- Question 1:  What are the 5 most popular trips and what are their average durations?
  - Answer: 
  
|start_station_name | end_station_name | trips | avg_duration_min|	
| --- | --- | --- | --- |
|Harry Bridges Plaza (Ferry Building)	| Embarcadero at Sansome |	9150	| 19.8	| 
|San Francisco Caltrain 2 (330 Townsend)	| Townsend at 7th |	8508	| 5.11	| 
|2nd at Townsend	| Harry Bridges Plaza (Ferry Building)	| 7620	| 9.7	| 
|Harry Bridges Plaza (Ferry Building)	| 2nd at Townsend	| 6888	| 10.63	| 
|Embarcadero at Sansome |	Steuart at Market	| 6874 |	8.52	| 
  - SQL query:
  ```sql
  SELECT 
    start_station_name, end_station_name,
    count(*) as trips,
    round(avg(duration_sec/60),2) as avg_duration_min
  FROM `bigquery-public-data.san_francisco.bikeshare_trips` 
  GROUP BY start_station_name, end_station_name
  ORDER by trips DESC
  LIMIT 5
  ```

- Question 2: What is the average duration of trips taken by customers vs subscribers?
  - Answer:
  The total number of trips taken by Subscribers is 846839 with an average duration time of 9.71 min, while the total number of trips taken by Customers is 136809 with average duration time of 61.98 min.  This is using using the subscriber_type field, where Subscriber = annual or 30-day member and Customer = 24-hour or 3-day member
  - SQL query:
  ```sql
  SELECT 
    subscriber_type,
    round(avg(duration_sec/60), 2) as avg_duration_min, 
    count(*) as trips
  FROM `bigquery-public-data.san_francisco.bikeshare_trips` 
  GROUP BY subscriber_type
  ```
  
- Question 3: What are the 5 most popular trips taken by Subscribers?
  - Answer:
  
|           start_station_name            |            end_station_name             | trips | avg_duration_min |
|---|---|---|---|
| San Francisco Caltrain 2 (330 Townsend) | Townsend at 7th                         |  8305 |             4.94 |
| 2nd at Townsend                         | Harry Bridges Plaza (Ferry Building)    |  6931 |             8.61 |
| Townsend at 7th                         | San Francisco Caltrain 2 (330 Townsend) |  6641 |             4.22 |
| Harry Bridges Plaza (Ferry Building)    | 2nd at Townsend                         |  6332 |             9.76 |
| Embarcadero at Sansome                  | Steuart at Market                       |  6200 |             6.66 |

  - SQL query:
  ```sql
  SELECT 
    start_station_name, end_station_name,
    count(*) as trips,
    round(avg(duration_sec/60),2) as avg_duration_min
  FROM `bigquery-public-data.san_francisco.bikeshare_trips` 
  WHERE subscriber_type = "Subscriber"
  GROUP BY start_station_name, end_station_name
  ORDER by trips DESC
  LIMIT 5
  ```
- Question 4: What are the 5 most popular trips taken by Customers:
  - Answer:

|          start_station_name          |           end_station_name           | trips | avg_duration_min |
|---|---|---|---|
| Harry Bridges Plaza (Ferry Building) | Embarcadero at Sansome               |  3667 |            38.06 |
| Embarcadero at Sansome               | Embarcadero at Sansome               |  2545 |            78.74 |
| Harry Bridges Plaza (Ferry Building) | Harry Bridges Plaza (Ferry Building) |  2004 |           113.37 |
| Embarcadero at Sansome               | Harry Bridges Plaza (Ferry Building) |  1638 |            27.99 |
| Embarcadero at Vallejo               | Embarcadero at Sansome               |  1345 |            44.59 |

  - SQL query:
  ```sql
  SELECT 
    start_station_name, end_station_name,
    count(*) as trips,
    round(avg(duration_sec/60),2) as avg_duration_min
  FROM `bigquery-public-data.san_francisco.bikeshare_trips` 
  WHERE subscriber_type = "Customer"
  GROUP BY start_station_name, end_station_name
  ORDER by trips DESC
  LIMIT 5 
  ```
- Question 5: Of the 5 most popular routes, how many were taken by subscribers?
  - Answer:
  
|start_station_name |	end_station_name |	total_trips	| subscriber_trips |	customer_trips|
|---|---|---|---|---|
|Harry Bridges Plaza (Ferry Building) | Embarcadero at Sansome | 9150 | 5483 | 3667 |	
|San Francisco Caltrain 2 (330 Townsend) | Townsend at 7th | 8508 | 8305 | 203 | 
|2nd at Townsend | Harry Bridges Plaza (Ferry Building) | 7620 | 6931 | 689 | 
|Harry Bridges Plaza (Ferry Building) | 2nd at Townsend | 6888 | 6332 | 556| 
|Embarcadero at Sansome | Steuart at Market | 6874 | 6200 | 674 |

  - SQL Query:
  ```sql
  SELECT start_station_name, end_station_name, 
    sum(trips) as total_trips, 
    sum(case when subscriber_type = "Subscriber" then trips else 0 end) as subscriber_trips, 
    sum(case when subscriber_type = "Customer" then trips else 0 end) as customer_trips
   FROM `constant-setup-216321.query_results.group_by_stations_and_subscribertype`
   GROUP BY start_station_name, end_station_name
   ORDER BY total_trips desc
   LIMIT 5
    
  #NOTE: constant-setup-216321.query_results.group_by_stations_and_subscribertype is my own Saved View
  ##It is created from the following query
  SELECT 
    start_station_name, end_station_name, subscriber_type,
    count(*) as trips,
    round(avg(duration_sec/60),2) as avg_duration_min
  FROM `bigquery-public-data.san_francisco.bikeshare_trips` 
  GROUP BY start_station_name, end_station_name, subscriber_type
  ORDER by trips DESC
  ```
- Question 6: What are the top 10 of the most common times that trips take place? (for time grouped by hour)?
  - Answer:

| start_hour | trips  |
|---|---|
|          8 | 132464 |
|         17 | 126302 |
|          9 |  96118 |
|         16 |  88755 |
|         18 |  84569 |
|          7 |  67531 |
|         15 |  47626 |
|         12 |  46950 |
|         13 |  43714 |
|         10 |  42782 |

  - SQL Query:
  ```sql 
  SELECT
    EXTRACT(HOUR FROM start_date) as start_hour, 
    count(*) as trips
  FROM `bigquery-public-data.san_francisco.bikeshare_trips`
  GROUP BY start_hour
  ORDER BY trips DESC
  LIMIT 10
  ```
- Question 7: Which 10 stations are the most popular?
  - Answer:
  
| trips |              start_station_name               |
|---|---|
| 72683 | San Francisco Caltrain (Townsend at 4th)      |
| 56100 | San Francisco Caltrain 2 (330 Townsend)       |
| 49062 | Harry Bridges Plaza (Ferry Building)          |
| 41137 | Embarcadero at Sansome                        |
| 39936 | 2nd at Townsend                               |
| 39200 | Temporary Transbay Terminal (Howard at Beale) |
| 38531 | Steuart at Market                             |
| 35142 | Market at Sansome                             |
| 34894 | Townsend at 7th                               |
| 30209 | Market at 10th                                |

  - SQL Query:
  ```sql
  SELECT
    start_station_name,
    count(*) as trips
  FROM `bigquery-public-data.san_francisco.bikeshare_trips`
  GROUP BY start_station_name
  ORDER BY trips DESC
  ```
- Question 8:  Which 10 stations are most popular during 10 most popular hours?
  - Answer:  Based on question #6, the top 10 most popular times are 7:00am-10:00am, 12:00pm-1:00pm, and 3:00pm-6:00pm. I ran a separate query for each of these time periods, and here are the results -
  - 7:00am-10:00am:

|              start_station_name               | trips |
|---|---|
| San Francisco Caltrain (Townsend at 4th)      | 42573 |
| San Francisco Caltrain 2 (330 Townsend)       | 33927 |
| Harry Bridges Plaza (Ferry Building)          | 22828 |
| Temporary Transbay Terminal (Howard at Beale) | 20121 |
| Steuart at Market                             | 16661 |
| 2nd at Townsend                               | 12571 |
| Grant Avenue at Columbus Avenue               | 11827 |
| Market at Sansome                             |  9835 |
| Embarcadero at Bryant                         |  9682 |
| Market at 10th                                |  9322 |

  - 12:00pm-1:00pm:
    
|          start_station_name          | trips |
|---|---|
| Embarcadero at Sansome               |  4706 |
| Market at Sansome                    |  3943 |
| Market at 4th                        |  3479 |
| Steuart at Market                    |  3224 |
| 2nd at Townsend                      |  3212 |
| Powell Street BART                   |  3170 |
| Townsend at 7th                      |  2868 |
| 2nd at South Park                    |  2723 |
| Embarcadero at Bryant                |  2613 |

  - 3:00pm-6:00pm:

|              start_station_name               | trips |
|---|---|
| Embarcadero at Sansome                        | 17976 |
| 2nd at Townsend                               | 16775 |
| Townsend at 7th                               | 16728 |
| San Francisco Caltrain (Townsend at 4th)      | 15412 |
| Market at Sansome                             | 13650 |
| Steuart at Market                             | 12800 |
| 2nd at South Park                             | 12275 |
| San Francisco Caltrain 2 (330 Townsend)       | 11689 |
| Temporary Transbay Terminal (Howard at Beale) | 11608 |
| Embarcadero at Folsom                         | 11530 |

  - SQL Queries: 
  ```sql
  #7:00am-10:00am
  SELECT
    start_station_name,
    count(*) as trips
  FROM `bigquery-public-data.san_francisco.bikeshare_trips`
  WHERE TIME(start_date) >= "07:00:00" AND TIME(start_date) <= "11:00:00"
  GROUP BY start_station_name
  ORDER BY trips DESC
  LIMIT 10
  
  #12:00pm-1:00pm
  SELECT
    start_station_name,
    count(*) as trips
  FROM `bigquery-public-data.san_francisco.bikeshare_trips`
  WHERE TIME(start_date) >= "12:00:00" AND TIME(start_date) <= "14:00:00"
  GROUP BY start_station_name
  ORDER BY trips DESC
  LIMIT 10

  #3:00pm-6:00pm
  SELECT
    start_station_name,
    count(*) as trips
  FROM `bigquery-public-data.san_francisco.bikeshare_trips`
  WHERE TIME(start_date) >= "15:00:00" AND TIME(start_date) <= "19:00:00"
  GROUP BY start_station_name
  ORDER BY trips DESC
  LIMIT 10

  ```
  

- Question 9: What are the 5 most common routes during the most popular hours (rush-hour time periods in question 6) and how many subscribers versus customers travel these routes?
  - Answer: Like Question 8, I ran 3 separate queries for the 3 most popular time periods.  Here are the results:
  - 7:00am-10:00am:
  
| start_station_name    |   end_station_name   | total_trips | subscriber_trips | customer_trips |
|---|----|---|---|---|
| Harry Bridges Plaza (Ferry Building)     | 2nd at Townsend |        4702 |             4605 |             97 |
| San Francisco Caltrain 2 (330 Townsend)  | Townsend at 7th  |        4181 |             4112 |             69 |
| Steuart at Market                        | 2nd at Townsend   |        4116 |             4049 |             67 |
| Harry Bridges Plaza (Ferry Building)     | Embarcadero at Sansome  |        3720 |             3127 |            593 |
| San Francisco Caltrain (Townsend at 4th) | Temporary Transbay Terminal (Howard at Beale) | 3635 | 3583 |             52 |
  
  - 12:00pm-1:00pm:
  
|start_station_name          | end_station_name           | total_trips | subscriber_trips | customer_trips |
|---|----|---|---|---|
| Harry Bridges Plaza (Ferry Building) | Embarcadero at Sansome |        1232 |              343 |            889 |
| Embarcadero at Sansome  | Embarcadero at Sansome |         631 |               64 |            567 |
| Embarcadero at Sansome | Harry Bridges Plaza (Ferry Building) |         623 |              299 |            324 |
| 2nd at Townsend | Harry Bridges Plaza (Ferry Building) |         561 |              380 |            181 |
| Harry Bridges Plaza (Ferry Building) | Harry Bridges Plaza (Ferry Building) |  529 |    89 |            440 |
  
  - 3:00pm-6:00pm:
  
|start_station_name | end_station_name             | total_trips | subscriber_trips | customer_trips |
|---|----|---|---|---|
| 2nd at Townsend | Harry Bridges Plaza (Ferry Building)     |        5084 |             4837 |            247 |
| Embarcadero at Folsom  | San Francisco Caltrain (Townsend at 4th) |        4820 |             4712 |            108 |
| Embarcadero at Sansome | Steuart at Market |        4380 |             4041 |            339 |
| Temporary Transbay Terminal (Howard at Beale) | San Francisco Caltrain (Townsend at 4th) |4034 | 3970 |64 |
| Steuart at Market | San Francisco Caltrain (Townsend at 4th) | 3997 |  3878 |            119 |

  - SQL Query: 
    - 7:00am-10:00am:
    ```sql
    SELECT start_station_name, end_station_name, 
      sum(trips) as total_trips, 
      sum(case when subscriber_type = "Subscriber" then trips else 0 end) as subscriber_trips, 
      sum(case when subscriber_type = "Customer" then trips else 0 end) as customer_trips
    FROM `constant-setup-216321.query_results.morning_rush_popular_routes`
    GROUP BY start_station_name, end_station_name
    ORDER BY total_trips desc
    LIMIT 5
    
    #Where constant-setup-216321.query_results.morning_rush_popular_routes is my Saved View using the following query:
    SELECT 
      start_station_name, end_station_name, subscriber_type,
      count(*) as trips,
      round(avg(duration_sec/60),2) as avg_duration_min
    FROM `bigquery-public-data.san_francisco.bikeshare_trips` 
    WHERE TIME(start_date) >= "07:00:00" AND TIME(start_date) <= "11:00:00"
    GROUP BY start_station_name, end_station_name, subscriber_type
    ORDER by trips DESC
    ```
    - 12:00pm-1:00pm:
    ```sql
    SELECT start_station_name, end_station_name, 
      sum(trips) as total_trips, 
      sum(case when subscriber_type = "Subscriber" then trips else 0 end) as subscriber_trips, 
      sum(case when subscriber_type = "Customer" then trips else 0 end) as customer_trips
    FROM `constant-setup-216321.query_results.lunch_rush_popular_routes`
    GROUP BY start_station_name, end_station_name
    ORDER BY total_trips desc
    LIMIT 5
    
    #Where constant-setup-216321.query_results.lunch_rush_popular_routes is my Saved View using the following query:
    SELECT 
      start_station_name, end_station_name, subscriber_type,
      count(*) as trips,
      round(avg(duration_sec/60),2) as avg_duration_min
    FROM `bigquery-public-data.san_francisco.bikeshare_trips` 
    WHERE TIME(start_date) >= "12:00:00" AND TIME(start_date) <= "14:00:00"
    GROUP BY start_station_name, end_station_name, subscriber_type
    ORDER by trips DESC
    ```
    - 3:00pm-6:00pm:
    ```sql
    SELECT start_station_name, end_station_name, 
      sum(trips) as total_trips, 
      sum(case when subscriber_type = "Subscriber" then trips else 0 end) as subscriber_trips, 
      sum(case when subscriber_type = "Customer" then trips else 0 end) as customer_trips
    FROM `constant-setup-216321.query_results.evening_rush_popular_routes`
    GROUP BY start_station_name, end_station_name
    ORDER BY total_trips desc
    LIMIT 5
    
    #Where constant-setup-216321.query_results.evening_rush_popular_routes is my Saved View using the following query:
    SELECT 
      start_station_name, end_station_name, subscriber_type,
      count(*) as trips,
      round(avg(duration_sec/60),2) as avg_duration_min
    FROM `bigquery-public-data.san_francisco.bikeshare_trips` 
    WHERE TIME(start_date) >= "15:00:00" AND TIME(start_date) <= "19:00:00"
    GROUP BY start_station_name, end_station_name, subscriber_type
    ORDER by trips DESC
    ```
