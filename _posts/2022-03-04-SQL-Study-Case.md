---
title: SQL Study Case
tags: [Bigquery, SQL]
style: 
color: 
description: SQL Study Case Solution.
---

<div class="m-3" id="problem">
    <br />
    <h2>🧠 Study Case:</h2>
</div>

<div align="center">
  <img src="../assets/img/sqltask1.png">
</div>

<div align="center">
  <img src="../assets/img/sqltask2.png">
</div>

<div class="m-3" id="data">
    <br />
    <h2>🎯 Solution:</h2>
</div>

Access Bigquery San Francisco GoBike dataset [here.](https://console.cloud.google.com/marketplace/product/san-francisco-public-data/sf-bike-share?project=earnest-smithy-343515)

in this solution i compute Berkeley region performance, here is the SQL code that i wrote to solve this study case: 

#### Solution a:

```sql
WITH berkeley_station as (
    SELECT a.name as region_name,b.station_id, b.name as station_name, b.region_id
    FROM `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions` as a
    INNER JOIN `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info` as b
    ON a.region_id = b.region_id
    WHERE b.region_id = (
        SELECT a.region_id
        FROM `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions` as a
        WHERE a.name = 'Berkeley'
    )
),
berkeley_trips as (
    SELECT
    ROW_NUMBER() OVER (ORDER BY t.start_date ASC) AS no_trips,
        Berkeley_station.region_id, Berkeley_station.region_name,
        CAST(t.duration_sec as FLOAT64) as duration_sec, t.start_date, t.start_station_name,
        t.start_station_id, t.end_date, t.end_station_name, t.end_station_id
    FROM `bigquery-public-data.san_francisco_bikeshare.bikeshare_trips` as t
    INNER JOIN Berkeley_station
    ON t.start_station_name = Berkeley_station.station_name
    WHERE EXTRACT(YEAR FROM t.start_date) = 2018
    ORDER BY duration_sec ASC
)

SELECT *
FROM berkeley_trips
WHERE no_trips = 3500;
```

#### Solution b:

```sql
WITH berkeley_station as (
    SELECT a.name as region_name,b.station_id, b.name as station_name, b.region_id
    FROM `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions` as a
    INNER JOIN `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info` as b
    ON a.region_id = b.region_id
    WHERE b.region_id = (
        SELECT a.region_id
        FROM `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions` as a
        WHERE a.name = 'Berkeley'
    )
),
berkeley_trips as (
    SELECT
    ROW_NUMBER() OVER (ORDER BY t.start_date ASC) AS no_trips,
        Berkeley_station.region_id, Berkeley_station.region_name,
        CAST(t.duration_sec as FLOAT64) as duration_sec, t.start_date, t.start_station_name,
        t.start_station_id, t.end_date, t.end_station_name, t.end_station_id
    FROM `bigquery-public-data.san_francisco_bikeshare.bikeshare_trips` as t
    INNER JOIN Berkeley_station
    ON t.start_station_name = Berkeley_station.station_name
    WHERE EXTRACT(YEAR FROM t.start_date) = 2018
    ORDER BY duration_sec ASC
)


SELECT APPROX_QUANTILES(DISTINCT duration_sec, 100)[OFFSET(90)] AS longest_trip_duration_sec
FROM berkeley_trips
```

#### Solution c:

```sql
WITH berkeley_station as (
    SELECT a.name as region_name,b.station_id, b.name as station_name, b.region_id
    FROM `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions` as a
    INNER JOIN `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info` as b
    ON a.region_id = b.region_id
    WHERE b.region_id = (
        SELECT a.region_id
        FROM `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions` as a
        WHERE a.name = 'Berkeley'
    )
),
berkeley_trips as (
    SELECT
    ROW_NUMBER() OVER (ORDER BY t.start_date ASC) AS no_trips,
        Berkeley_station.region_id, Berkeley_station.region_name,
        CAST(t.duration_sec as FLOAT64) as duration_sec, t.start_date, t.start_station_name,
        t.start_station_id, t.end_date, t.end_station_name, t.end_station_id
    FROM `bigquery-public-data.san_francisco_bikeshare.bikeshare_trips` as t
    INNER JOIN Berkeley_station
    ON t.start_station_name = Berkeley_station.station_name
    WHERE EXTRACT(YEAR FROM t.start_date) = 2018
    ORDER BY duration_sec ASC
),
time_between as (
        SELECT
        start_date,
        TIMESTAMP_DIFF(start_date, prev_start_date, second) as in_between_rental_time_sec
        FROM(
            SELECT start_date,
            LAG(start_date) OVER(ORDER BY start_date ASC) prev_start_date
            FROM berkeley_trips
        )
)

SELECT AVG(in_between_rental_time_sec)
FROM time_between
```

#### Solution d:

```sql
WITH allregion_station as (
    SELECT a.name as region_name,b.station_id, b.name as station_name, b.region_id
    FROM `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions` as a
    INNER JOIN `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info` as b
    ON a.region_id = b.region_id
),
allregion_trips as (
    SELECT
    ROW_NUMBER() OVER (ORDER BY t.start_date ASC) AS no_trips,
        allregion_station.region_id, allregion_station.region_name,
        CAST(t.duration_sec as FLOAT64) as duration_sec, t.start_date, t.start_station_name,
        t.start_station_id, t.end_date, t.end_station_name, t.end_station_id
    FROM `bigquery-public-data.san_francisco_bikeshare.bikeshare_trips` as t
    INNER JOIN allregion_station
    ON t.start_station_name = allregion_station.station_name
    WHERE EXTRACT(YEAR FROM t.start_date) = 2018
    ORDER BY duration_sec ASC
)

SELECT
    region_id, region_name,
    COUNT(*) as total_trips, 
    APPROX_QUANTILES(DISTINCT duration_sec, 100)[OFFSET(90)] AS longest_trip_duration_sec
FROM allregion_trips
GROUP BY 1,2
ORDER BY total_trips DESC
```

<div class="m-3" id="data">
    <br />
    <h2>🎯 The answers:</h2>
</div>

- Berkeley region reach its 3500th rental on 2018-01-26
- The longest trip duration is 4976 second
- The average in-between rental time by duration between trips is 391.81
- Berkeley region rank is 3
