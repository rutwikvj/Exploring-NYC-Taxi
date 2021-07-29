# Exploring the Yellow Taxi Trips from the New York Taxi Trips Dataset

* Meta Data - No. of Rows
```sql
SELECT count(*)
FROM `bigquery-public-data.new_york_taxi_trips.tlc_yellow_trips_2016`
```
ANS: 131165043

* Data check - check if all the pickup-entries of the data are from the year 2016
```sql
SELECT distinct(substr(string(pickup_datetime), 1,4))
FROM `bigquery-public-data.new_york_taxi_trips.tlc_yellow_trips_2016`
```
ANS: 2016

* Data Check - Similarly check if the dropoff dates are also from the year 2016 and 2017(considering that the trip was started on 31st Dec 2016 and ended on 1st Jan 2017)
```sql
SELECT distinct(substr(string(dropoff_datetime), 1,4))
FROM `bigquery-public-data.new_york_taxi_trips.tlc_yellow_trips_2016`
```
ANS: ![image](https://user-images.githubusercontent.com/87647766/127478765-76359d0b-e77b-4bb1-ac15-5dfe08b2a287.png)

We have to consider the years only 2016 and 2017. Also there are erroneous vendor, hence only vendors 1 and 2 must be considered. 

* Business Questions
  *  Which vendor covered a higher distance of trips: Creative Mobile Technologies(1) or VeriFone Inc(2) ?
  *  Which vendor has a higher collection of total amount: Creative Mobile Technologies(1) or VeriFone Inc(2) ?
  *  Check how the payment types vary for both the vendors: do customers associated with Vendor 1 prefer Card Payments over cash ?
  *  Average rate per kilometer for both the vendors(Amount includes fare, tax and tips)
  *  Average rate of the holiday season(trips between: 24th Dec-31st Dec) as compared to that of the normal days

### Which vendor covered a higher distance of trips: Creative Mobile Technologies(1) or VeriFone Inc(2) ?
```sql
SELECT sum(trip_distance) as total_distance,vendor_id
FROM `bigquery-public-data.new_york_taxi_trips.tlc_yellow_trips_2016`
where vendor_id IN ('1','2') and
substr(string(dropoff_datetime), 1,4) in ('2016','2017')
group by vendor_id
order by total_distance desc
```
ANS:  ![image](https://user-images.githubusercontent.com/87647766/127479924-a39f4853-28d1-403d-9ee4-762e4ae225dc.png)

We can see that vendor 1 i.e. Creative Mobile Technologies(1) covered a higher distance by almost double.

### Which vendor has a higher collection of total amount: Creative Mobile Technologies(1) or VeriFone Inc(2) ?
```sql
SELECT sum(total_amount) as total_amt,vendor_id
FROM `bigquery-public-data.new_york_taxi_trips.tlc_yellow_trips_2016`
where vendor_id IN ('1','2') and
substr(string(dropoff_datetime), 1,4) in ('2016','2017')
group by vendor_id
order by total_amt desc
```
ANS: ![image](https://user-images.githubusercontent.com/87647766/127480341-063a5589-5f13-4b50-a67f-74ea6202e75a.png)

Surprisingly vendor 2 has a higher amount collection compared to that of vendor 1.

###  Check how the payment types vary for both the vendors: do customers associated with Vendor 1 prefer Card Payments over cash ?
Note:  1= Credit card 2= Cash 3= No charge 4= Dispute 5= Unknown 6= Voided trip
First checking both the vendors together:
```sql
SELECT count(payment_type) as pmt_count,payment_type
FROM `bigquery-public-data.new_york_taxi_trips.tlc_yellow_trips_2016`
where vendor_id IN ('1','2') and
substr(string(dropoff_datetime), 1,4) in ('2016','2017')
group by payment_type
order by payment_type
```
ANS:  ![image](https://user-images.githubusercontent.com/87647766/127481257-eb67baa8-6035-45b8-8458-14a5dc17bf4b.png)

We can see that majority of the customers pay by credit card

For Vendor 1:
```sql
SELECT count(payment_type) as pmt_count,payment_type
FROM `bigquery-public-data.new_york_taxi_trips.tlc_yellow_trips_2016`
where vendor_id = '1' and
substr(string(dropoff_datetime), 1,4) in ('2016','2017')
group by payment_type
order by payment_type
```
ANS:  ![image](https://user-images.githubusercontent.com/87647766/127481534-a6bc8c4f-76f0-4c11-ac58-b648a12c0fc8.png)

For Vendor 2: 
```sql
SELECT count(payment_type) as pmt_count,payment_type
FROM `bigquery-public-data.new_york_taxi_trips.tlc_yellow_trips_2016`
where vendor_id = '2' and
substr(string(dropoff_datetime), 1,4) in ('2016','2017')
group by payment_type
order by payment_type
```
ANS: ![image](https://user-images.githubusercontent.com/87647766/127481907-e566fef6-7a92-4788-ada6-4bdbd0a9594a.png)

We can see that the payment types are almost indifferent to the two vendors. However Vendor 2 has no unknown payment types and a lot lesser disputed payment types than vendor 1.

### Average rate per kilometer for both the vendors(Amount includes fare, tax and tips)
```sql
with ta as
(SELECT sum(total_amount) as total_amt, vendor_id
FROM `bigquery-public-data.new_york_taxi_trips.tlc_yellow_trips_2016`
where vendor_id IN ('1','2') and
substr(string(dropoff_datetime), 1,4) in ('2016','2017')
group by vendor_id),
tm as
(SELECT sum(trip_distance) as total_dist, vendor_id
FROM `bigquery-public-data.new_york_taxi_trips.tlc_yellow_trips_2016`
where vendor_id IN ('1','2') and
substr(string(dropoff_datetime), 1,4) in ('2016','2017')
group by vendor_id
)
select ta.vendor_id, (total_amt/total_dist) as rate_per_mile
from ta left join tm on ta.vendor_id=tm.vendor_id
order by vendor_id
```
ANS:  ![image](https://user-images.githubusercontent.com/87647766/127484387-01c75486-51a8-4d82-b0ec-f622b74700d7.png)

We can see that vendor 1 is economical in terms of rate per mile than vendor 2 which has almost double rate per mile when companred to vendor 1. This might be one of the reasons why vendor 2 has a higher amount collection despite the lower distance covered.

### Average rate of the holiday season(trips between: 24th Dec-31st Dec) as compared to that of the normal days
```sql
with regular_season as(
with regular as(
SELECT sum(total_amount) as a, sum(trip_distance) as b
FROM `bigquery-public-data.new_york_taxi_trips.tlc_yellow_trips_2016`
where vendor_id IN ('1','2') and
substr(string(dropoff_datetime), 1,4) in ('2016','2017') and
substr(string(pickup_datetime), 6,2) not in ('12')
),
regular_dec as(
SELECT sum(total_amount) as c, sum(trip_distance) as d
FROM `bigquery-public-data.new_york_taxi_trips.tlc_yellow_trips_2016`
where vendor_id IN ('1','2') and
substr(string(dropoff_datetime), 1,4) in ('2016','2017') and
substr(string(pickup_datetime), 6,2) in ('12') and
substr(string(pickup_datetime), 9,2) not in ('24','25','26','27','28','29','30','31')
)
select (regular.a+regular_dec.c)/(regular.b+regular_dec.d) as rate_per_mile
from regular, regular_dec
),
holiday_season as(
SELECT sum(total_amount)/sum(trip_distance) as rate_per_mile
FROM `bigquery-public-data.new_york_taxi_trips.tlc_yellow_trips_2016`
where vendor_id IN ('1','2') and
substr(string(dropoff_datetime), 1,4) in ('2016','2017') and
substr(string(pickup_datetime), 6,2) in ('12') and
substr(string(pickup_datetime), 9,2) in ('24','25','26','27','28','29','30','31')
)
select rate_per_mile, "Holiday Season" from holiday_season
union all
select rate_per_mile, "Regular Season" from regular_season 
```
ANS:  ![image](https://user-images.githubusercontent.com/87647766/127501158-c6d554e0-22ac-462f-bbf6-128e6d4e7f43.png)

We can clearly see that the rate per mile during the holiday season is higher than the rate per mile regular season.


