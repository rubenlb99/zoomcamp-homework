Question 1. Knowing docker tags

Run the command to get information on Docker

docker --help

Now run the command to get help on the "docker build" command:

docker build --help

Do the same for "docker run".

Which tag has the following text? - Automatically remove the container when it exits

--delete
--rc
--rmc
--rm   <-------->  Correct answer

------------------------------------------------------------------------------------------

Question 2. Understanding docker first run

Run docker with the python:3.9 image in an interactive mode and the entrypoint of bash. Now check the python modules that are installed ( use pip list ).

What is version of the package wheel ?

0.42.0  <-----------> In my case 0.44.0
1.0.0
23.0.1
58.1.0

-------------------------------------------------------------------------------------------

Prepare Postgres
Run Postgres and load data as shown in the videos We'll use the green taxi trips from September 2019:

wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-09.csv.gz

You will also need the dataset with zones:

wget https://s3.amazonaws.com/nyc-tlc/misc/taxi+_zone_lookup.csv

Download this data and put it into Postgres (with jupyter notebooks or with a pipeline)

----------------------------------------------------------------------------------------------------------------

Question 3. Count records
How many taxi trips were totally made on September 18th 2019?

Tip: started and finished on 2019-09-18.

Remember that lpep_pickup_datetime and lpep_dropoff_datetime columns are in the format timestamp (date and hour+min+sec) and not in date.

15767
15612 <-------------> Correct answer
15859
89009

Query:
        SELECT count(*) FROM green_trip_data where
            lpep_pickup_datetime::date = '2019-09-18' and
            lpep_dropoff_datetime::date = '2019-09-18'

-------------------------------------------------------------------------------------------

Question 4. Longest trip for each day
Which was the pick up day with the longest trip distance? Use the pick up time for your calculations.

Tip: For every trip on a single day, we only care about the trip with the longest distance.

2019-09-18
2019-09-16
2019-09-26 <--------------> Correct answer
2019-09-21

Query::
    SELECT lpep_pickup_datetime FROM green_trip_data 
    where trip_distance = (select max(trip_distance) from green_trip_data)

-------------------------------------------------------------------------------------------

Question 5. Three biggest pick up Boroughs

Consider lpep_pickup_datetime in '2019-09-18' and ignoring Borough has Unknown

Which were the 3 pick up Boroughs that had a sum of total_amount superior to 50000?

"Brooklyn" "Manhattan" "Queens" <-------------> Correct answer
"Bronx" "Brooklyn" "Manhattan"
"Bronx" "Manhattan" "Queens"
"Brooklyn" "Queens" "Staten Island"

Query:
    SELECT tzl."Borough" as TotalAmount FROM green_trip_data gtd
    JOIN taxi_zone_lookup tzl ON gtd."PULocationID" = tzl."LocationID"
    where gtd.lpep_pickup_datetime::date = '2019-09-18' and tzl."Borough" <> 'Unknown'
    group by tzl."Borough" 
    having sum(gtd.total_amount) > 50000
    order by sum(gtd.total_amount) desc

-------------------------------------------------------------------------------------------

Question 6. Largest tip

For the passengers picked up in September 2019 in the zone name Astoria which was the drop off zone that had the largest tip? We want the name of the zone, not the id.

Note: it's not a typo, it's tip , not trip

Central Park
Jamaica
JFK Airport
Long Island City/Queens Plaza

Didn`t get this result, my result was Soundview/Bruckner

Query: 
    SELECT tzl."Zone"
    FROM green_trip_data gtd
    join taxi_zone_lookup tzl on gtd."PULocationID" = tzl."LocationID"
    where gtd."DOLocationID" IN
        (SELECT gtd."DOLocationID"
        FROM green_trip_data gtd
        join taxi_zone_lookup tzl on gtd."PULocationID" = tzl."LocationID"
        where gtd.lpep_pickup_datetime::date >= '2019-09-01' and gtd.lpep_pickup_datetime < '2019-10-01' and tzl."Zone" = 'Astoria')
    order by gtd.tip_amount desc
    limit 1