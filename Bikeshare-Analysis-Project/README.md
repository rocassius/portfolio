# Project 1: Query Project

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


---

## Part 1 - Querying Data with BigQuery

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
    * SQL Query:
```
SELECT COUNT(DISTINCT(trip_id))
  FROM `my-test-project-251101.bike_trip_data.bike_trips`
```
Answer: `983648`

- What is the earliest start time and latest end time for a trip?
Earliest start time query:
```
SELECT MIN(EXTRACT(TIME from start_date))
  FROM `my-test-project-251101.bike_trip_data.bike_trips` 
```
Earliest start time `00:00:00`

Latest end time Query
```
SELECT MAX(EXTRACT(TIME from end_date))
  FROM `my-test-project-251101.bike_trip_data.bike_trips` 
```
Latest end time: `23:59:00`

- How many bikes are there?
SQL Query:
```
WITH total_bike_tbl AS 
  (SELECT MAX(total_bikes) AS total_bikes_max
    FROM `my-test-project-251101.bike_trip_data.total_bikes` 
    GROUP BY station_id) 
  SELECT SUM(total_bikes_max) FROM total_bike_tbl
```
Answer: `1392`




### Questions of your own
- Make up 3 questions and answer them using the Bay Area Bike Share Trips Data.
- Use the SQL tutorial (https://www.w3schools.com/sql/default.asp) to help you with mechanics.

- Question 1: What starting station is associated with the longest bike trips on average?
  * Answer: `University and Emerson | 6455.08`
  * SQL query:
  ```
  WITH avg_duration_by_start_station AS 
  (SELECT AVG(duration_sec) AS avg_duration_sec, start_station_name
    FROM `my-test-project-251101.bike_trip_data.bike_trips` 
    GROUP BY start_station_name) 
  SELECT start_station_name, avg_duration_sec
    FROM avg_duration_by_start_station
    ORDER BY avg_duration_sec DESC
    LIMIT 1
   ```

- Question 2: What percentage of rides are associated with each current subscriber type in the most recent year?
  * Answer:
  ```
  Customer | 0.110
  Subscriber | 0.890
  ```
  
  * SQL query:
  ```
  SELECT subscriber_type, COUNT(*) / 
  (SELECT COUNT(DISTINCT(trip_id)) 
      FROM `my-test-project-251101.bike_trip_data.bike_trips`
      WHERE EXTRACT(YEAR FROM start_date) = 2016)
  FROM `my-test-project-251101.bike_trip_data.bike_trips`
    WHERE EXTRACT(YEAR FROM start_date) = 2016
    GROUP BY subscriber_type

  ```

- Question 3: What percentage of rides, make a full circle, i.e., have the same start and end stations.
  * Answer:`0.032579`
  * SQL query:
  ```
  SELECT COUNT(*) / 
  (SELECT COUNT(*) 
    FROM `my-test-project-251101.bike_trip_data.bike_trips`)
   FROM `my-test-project-251101.bike_trip_data.bike_trips`
   WHERE start_station_id = end_station_id 
   ```



---

## Part 2 - Querying data from the BigQuery CLI - set up 

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
        SELECT COUNT(DISTINCT(trip_id))
            FROM `my-test-project-251101.bike_trip_data.bike_trips`'
    ```

### Queries

1. Rerun last week's queries using bq command line tool (Paste your bq
   queries):

- What's the size of this dataset? (i.e., how many trips)

    ```
    bq query --use_legacy_sql=false '
        SELECT count(*)
        FROM `bigquery-public-data.san_francisco.bikeshare_trips`'
    ```

- What is the earliest start time and latest end time for a trip?
    
    Earliest Start Time Query:
    ```
    bq query --use_legacy_sql=false '
        SELECT MIN(EXTRACT(TIME from start_date))
            FROM `my-test-project-251101.bike_trip_data.bike_trips` '
    ```
    
    Latest End Time Query
    ```
    bq query --use_legacy_sql=false '
        SELECT MAX(EXTRACT(TIME from end_date))
            FROM `my-test-project-251101.bike_trip_data.bike_trips`'
    ```

- How many bikes are there?

    ```
    bq query --use_legacy_sql=false '
        WITH total_bike_tbl AS 
          (SELECT MAX(total_bikes) AS total_bikes_max
              FROM `my-test-project-251101.bike_trip_data.total_bikes` 
        GROUP BY station_id) 
        SELECT SUM(total_bikes_max) 
            FROM total_bike_tbl'
    ```
    
2. New Query (Paste your SQL query and answer the question in a sentence):

- How many trips are in the morning vs in the afternoon?


QUERY:

    ```
    bq query --use_legacy_sql=false '
        WITH mid_hour_tbl AS (
          SELECT EXTRACT(HOUR 
            FROM TIMESTAMP_ADD(start_date, INTERVAL TIMESTAMP_DIFF(end_date, start_date, MINUTE) MINUTE)) AS middle_hour
            FROM `my-test-project-251101.bike_trip_data.bike_trips`
          )
          SELECT count(middle_hour) AS trip_count,
              CASE 
                WHEN middle_hour < 12 THEN "morning"
                WHEN middle_hour < 18 THEN "afternoon"
                ELSE "evening"
                END AS time_of_day
            FROM mid_hour_tbl
            GROUP BY time_of_day'
    ```
    
ANSWER:
```
+------------+-------------+
| trip_count | time_of_day |
+------------+-------------+
|     203727 | evening     |
|     379777 | afternoon   |
|     400144 | morning     |
+------------+-------------+
```


### Project Questions
Identify the main questions you'll need to answer to make recommendations (list
below, add as many questions as you need).

- Question 1: What percentage of bike trips happen within commuting hours?

- Question 2: What are the average and stand deviation of trip duration for subscribers versus customers?

- Question 3: What percentage of trips happen on weekends? 

- Question 4: What are the average and stand deviation of trip duration on weekends? 


### Answers

Answer at least 4 of the questions you identified above You can use either
BigQuery or the bq command line tool.  Paste your questions, queries and
answers below.

- Question 1: What percentage of bike trips happen within commuting hours?
  * Answer: `0.3437`
  * SQL query:
  ```
  WITH hour_tbl AS (
  SELECT EXTRACT(HOUR FROM start_date) as start_hour, EXTRACT(HOUR FROM end_date) AS end_hour
  FROM `my-test-project-251101.bike_trip_data.bike_trips`
  )
  SELECT COUNT(*) / 
        (SELECT COUNT(*) FROM `my-test-project-251101.bike_trip_data.bike_trips`)
    FROM hour_tbl 
    WHERE (start_hour > 7 AND end_hour < 10) OR
          (start_hour > 17 AND end_hour < 20)
  ```        

- Question 2: What are the average and stand deviation of trip duration for subscribers versus customers?
  * Answer:
  ```
    +-----------------+-------------------+-------------------+
    | subscriber_type | avg_duration_min  |  sd_duration_min  |
    +-----------------+-------------------+-------------------+
    | Customer        | 61.97975267221685 | 812.8085318495606 |
    | Subscriber      | 9.712737328662122 | 48.24265348752558 |
    +-----------------+-------------------+-------------------+
  ```
  
  * SQL query:
  ```
  SELECT subscriber_type, AVG(duration_sec/60) AS avg_duration_min, STDDEV(duration_sec/60) AS sd_duration_min
      FROM `my-test-project-251101.bike_trip_data.bike_trips`
      GROUP BY subscriber_type
  ```

- Question 3: What percentage of trips happen on weekends? 
  * Answer: `0.224`
  * SQL query:
  ```
  SELECT COUNT(*)/
          (SELECT COUNT(*) FROM `my-test-project-251101.bike_trip_data.bike_trips`)
      FROM `my-test-project-251101.bike_trip_data.bike_trips`
      WHERE EXTRACT(DAYOFWEEK FROM start_date) = 6 OR 
            EXTRACT(DAYOFWEEK FROM start_date) = 7
  ```


- Question 4: What are the average and standard deviation of trip duration on weekends? 
  * Answer:
  ```
    +--------------------+-------------------+
    |  avg_duration_min  |  sd_duration_min  |
    +--------------------+-------------------+
    | 23.575579704222736 | 627.9756335569056 |
    +--------------------+-------------------+
  ```
  
  * SQL query:
  ```
  SELECT AVG(duration_sec/60) AS avg_duration_min, STDDEV(duration_sec/60) AS sd_duration_min
      FROM `my-test-project-251101.bike_trip_data.bike_trips`
      WHERE EXTRACT(DAYOFWEEK FROM start_date) = 6 OR 
            EXTRACT(DAYOFWEEK FROM start_date) = 7
  ```


---

## Part 3 - Employ notebooks to synthesize query project results

### Get Going

Use JupyterHub on your midsw205 cloud instance to create a new python3 notebook. 


#### Run queries in the notebook 

```
! bq query --use_legacy_sql=FALSE '<your-query-here>'
```

- NOTE: 
- Queries that return over 16K rows will not run this way, 
- Run groupbys etc in the bq web interface and save that as a table in BQ. 
- Query those tables the same way as in `example.ipynb`


#### Report
- Short description of findings and recommendations 
- Add data visualizations to support recommendations 

### Resource: see example .ipynb file 

[Example Notebook](example.ipynb)
