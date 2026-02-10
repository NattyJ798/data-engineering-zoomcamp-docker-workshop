# Homework

## problem 0


### Part A: Creating External Table
```sql
-- create an external table using a gs bucket
CREATE OR REPLACE EXTERNAL TABLE `demo_dataset.external_yellow_tripdata`
OPTIONS (
  format = 'parquet',
  uris = ['gs://dtc-de-course-485716-554d345b4257-terra-bucket/yellow_tripdata_2024-*.parquet']
);
```

### Part B: creating materialized table

``` sql
create a non-partitioned table
CREATE OR REPLACE TABLE demo_dataset.yellow_tripdata_non_partitioned AS
SELECT * FROM demo_dataset.external_yellow_tripdata;
```

## Problem 1
```select COUNT(*) from demo_dataset.yellow_tripdata_non_partitioned; -- = 20332093 (C)```

## Problem 2

### materialized table
```sql
select COUNT(distinct PULocationID) from demo_dataset.yellow_tripdata_non_partitioned; -- = 155.12 MiB 
```

### external table
``` sql
select COUNT(distinct PULocationID) from `demo_dataset.external_yellow_tripdata`; -- = 0 B
```
**Answer: B**

## Problem 3 

### Single 

```sql
select PULocationID  from demo_dataset.yellow_tripdata_non_partitioned; -- = 155.12 MB 
```

### Both 
```sql
select PULocationID,DOLocationID  from demo_dataset.yellow_tripdata_non_partitioned; -- = 310.24 MB MiB
```

### Explanation
BigQuery is a columnar database, and it only scans the specific columns requested in the query. Querying two columns (PULocationID, DOLocationID) requires reading more data than querying one column (PULocationID), leading to a higher estimated number of bytes processed.


## Problem 4 
```sql
select count(*) from `demo_dataset.yellow_tripdata_non_partitioned`
where fare_amount = 0.00; --8333
``` 

**Answer**:D

## Problem 5
Partition by tpep_dropoff_datetime and Cluster on VendorID 

**Answer**:Partition by tpep_dropoff_datetime and Cluster on VendorID (A)

### Creation Query
```sql
CREATE OR REPLACE TABLE demo_dataset.yellow_tripdata_partitioned
PARTITION BY DATE(tpep_dropoff_datetime)
CLUSTER BY VendorID AS
SELECT * FROM demo_dataset.yellow_tripdata_non_partitioned;
```

### Problem 6

```sql
select count(distinct VendorID) 
from demo_dataset.yellow_tripdata_non_partitioned
where DATE(tpep_dropoff_datetime) between '2024-03-01' and '2024-03-15' -- 310.24 MB
```

```sql
select count(distinct VendorID) 
from demo_dataset.yellow_tripdata_partitioned
where DATE(tpep_dropoff_datetime) between '2024-03-01' and '2024-03-15' -- 26.84 MB
```

**Answer**:310.24 MB for non-partitioned table and 26.84 MB for the partitioned table (D)

### Problem 7
**Answer**:GCP Bucket (C)

## Problem 8
**Answer**:False. It's data dependent. Sometimes clustering will make sense. Other times, the juice will not be worth the squeeze.

### Problem 9
**Answer**: The estimated size is 0.0. This is because BigQuery stores the metadata within itself so it is a simple o(1) look up. No data processing required.


