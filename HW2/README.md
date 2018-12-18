# Query Project
- In the Query Project, you will get practice with SQL while learning about Google Cloud Platform (GCP) and BiqQuery. You'll answer business-driven questions using public datasets housed in GCP. To give you experience with different ways to use those datasets, you will use the web UI (BiqQuery) and the command-line tools, and work with them in jupyter notebooks.
- We will be using the Bay Area Bike Share Trips Data (https://cloud.google.com/bigquery/public-data/bay-bike-share). 

#### Problem Statement
- You're a data scientist at Ford GoBike (https://www.fordgobike.com/), the company running Bay Area Bikeshare. You are trying to increase ridership, and you want to offer deals through the mobile app to do so. What deals do you offer though? Currently, your company has three options: a flat price for a single one-way trip, a day pass that allows unlimited 30-minute rides for 24 hours and an annual membership. 

- Through this project, you will answer these questions: 
  * What are the 5 most popular trips that you would call "commuter trips"?
  * What are your recommendations for offers (justify based on your findings)?


## Assignment 02: Querying Data with BigQuery

### What is Google Cloud?
- Read: https://cloud.google.com/docs/overview/

### Get Going

- Go to https://cloud.google.com/bigquery/
- Click on "Try it Free"
- It asks for credit card, but you get $300 free and it does not autorenew after the $300 credit is used, so go ahead (OR CHANGE THIS IF SOME SORT OF OTHER ACCESS INFO)
- Now you will see the console screen. This is where you can manage everything for GCP
- Go to the menus on the left and scroll down to BigQuery
- Now go to https://cloud.google.com/bigquery/public-data/bay-bike-share 
- Scroll down to "Go to Bay Area Bike Share Trips Dataset" (This will open a BQ working page.)


### Some initial queries
Paste your SQL query and answer the question in a sentence.

- What's the size of this dataset? (i.e., how many trips)
  - Answer: 983648
  - SQL Query: 
    ```sql
    SELECT count(*) FROM [bigquery-public-data:san_francisco.bikeshare_trips]
    ```

- What is the earliest start time and latest end time for a trip?
  - Answer 1 - This is if we interpret minimum start time to mean the first trip ever in the data and maximum end time to mean the last trip ever in the data: 
    - Earliest Start Time = 9:08 AM (2013-08-29 09:08:00 UTC)
    - Latest End Time = 11:48 PM (2016-08-31 23:48:00 UTC)
  - SQL Query: 
    ```sql
    SELECT min(start_date), max(end_date) FROM [bigquery-public-data:san_francisco.bikeshare_trips] 
    ```   
  - Answer 2 - This is if we interpret the minimum start time to mean the time of the first trip, regardless of date, and the maximum end time of the last trip, regardless of date:
    - Earliest Start Time = 12:00 AM (00:00:00)
    - Latest End Time = 11:59 PM (23:59:00)
  - SQL Query:
    ```sql
    #Start
    SELECT
      EXTRACT(TIME FROM start_date) AS start_time
    FROM
      `bigquery-public-data.san_francisco.bikeshare_trips`
    ORDER BY
      start_time 
    LIMIT 1
    
    #End
    SELECT
      EXTRACT(TIME FROM end_date) AS end_time
    FROM
      `bigquery-public-data.san_francisco.bikeshare_trips`
    ORDER BY
      end_time DESC 
    LIMIT 1
    ```
- How many bikes are there?
  - Answer: 700
  - SQL Query: 
    ```sql  
    SELECT count(distinct bike_number) FROM [bigquery-public-data:san_francisco.bikeshare_trips]
    ```  
(since bike number represents a unique bike, selecting all unique bike numbers and counting them would give the total number of bikes)

### Questions of your own
- Make up 3 questions and answer them using the Bay Area Bike Share Trips Data.
- Use the SQL tutorial (https://www.w3schools.com/sql/default.asp) to help you with mechanics.

- **Question 1: What is the average duration, in minutes, of each of the 10 most popular trips?**
  * Answer:
  
|start_station_name | end_station_name | trips | avg_duration_min|	
| --- | --- | --- | --- |
|Harry Bridges Plaza (Ferry Building)	| Embarcadero at Sansome |	9150	| 19.8	| 
|San Francisco Caltrain 2 (330 Townsend)	| Townsend at 7th |	8508	| 5.11	| 
|2nd at Townsend	| Harry Bridges Plaza (Ferry Building)	| 7620	| 9.7	| 
|Harry Bridges Plaza (Ferry Building)	| 2nd at Townsend	| 6888	| 10.63	| 
|Embarcadero at Sansome |	Steuart at Market	| 6874 |	8.52	| 
|Townsend at 7th	| San Francisco Caltrain 2 (330 Townsend) |	6836	| 4.65 |	 
|Embarcadero at Folsom |	San Francisco Caltrain (Townsend at 4th)	| 6351	| 11.63	| 
|San Francisco Caltrain (Townsend at 4th) |	Harry Bridges Plaza (Ferry Building) |	6215	| 13.28	| 
|Steuart at Market	| 2nd at Townsend	| 6039	| 9.59	|
|Steuart at Market	| San Francisco Caltrain (Townsend at 4th) |	5959	| 12.17 | 

  * SQL query:
```sql
SELECT 
 start_station_name, end_station_name,
 count(*) as trips,
 round(avg(duration_sec/60),2) as avg_duration_min
FROM [bigquery-public-data:san_francisco.bikeshare_trips] 
GROUP BY start_station_name, end_station_name
ORDER by trips DESC
LIMIT 10
```

- **Question 2: What are the 10 most common trip durations in minutes, for durations time grouped by 10 min? (In other words, how many trips are there between 0-10 min, 10-20 min, 20-30 min, etc, and which 10 min duration bin size is most popular?)**
  * Answer:
  
| duration_min_bin_start	| duration_min_bin_end	|trips |	
| --- | --- | --- |
| 0.0	| 10.0	| 594628 |
| 10.0	| 20.0 | 303297 |	
| 20.0	| 30.0	| 38355	|
| 30.0	| 40.0	| 10232	|
| 40.0	| 50.0	| 5057	|
| 50.0	| 60.0	| 3983	|
| 60.0	| 70.0	| 2852 |	
| 70.0	| 80.0	| 2306	|
| 80.0	| 90.0	| 2070	|
| 90.0	| 100.0 | 1723 |	

  * SQL query:
```sql
SELECT 
 floor(duration_sec/600)*10 as duration_min_bin_start, 
 (floor(duration_sec/600)+1)*10 as duration_min_bin_end,
 count(*) as trips
FROM [bigquery-public-data:san_francisco.bikeshare_trips] 
GROUP BY duration_min_bin_start, duration_min_bin_end
ORDER by trips DESC
LIMIT 10
```

- **Question 3: What is the average duration and number of trips by customer type?**

  * Answer:
  
|subscriber_type | avg_duration_min	| trips	|
| --- | --- | --- |
|Customer	| 61.98	| 136809	| 
|Subscriber	| 9.71 | 846839 |
 
  * SQL query:
```sql
SELECT 
 subscriber_type,
 round(avg(duration_sec/60), 2) as avg_duration_min, 
 count(*) as trips
FROM [bigquery-public-data:san_francisco.bikeshare_trips] 
GROUP BY subscriber_type
```


