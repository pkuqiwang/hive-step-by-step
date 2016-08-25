# hive-step-by-step

## Setup environment 
Download Hortonworks HDP 2.4 [sandbox](https://hortonworks.com/downloads/#sandbox) and prepare the sandbox following this [instruction](http://hortonworks.com/hadoop-tutorial/learning-the-ropes-of-the-hortonworks-sandbox/)  

##Upload the dataset
Download the dataset from internet
```
wget http://stat-computing.org/dataexpo/2009/2007.csv.bz2 -O /tmp/flights_2007.csv.bz2

```
Then copy dataset to HDFS
```
hdfs dfs -rm -r -f /tmp/airflightsdelays
hdfs dfs -mkdir /tmp/airflightsdelays
hdfs dfs -put /tmp/flights_2007.csv.bz2 /tmp/flights_2008.csv.bz2 /tmp/airflightsdelays/
```
## Basic Hive operations
###create databases
```
create database demo;
show databases;
drop database demo;
show databases;
```
###Managed tables
```
--managed vs unmanaged table 
CREATE EXTERNAL TABLE users 
(id string, birth_date string, gender string) 
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' 
STORED AS TEXTFILE
LOCATION "/tmp/demo-data/user"
tblproperties ("skip.header.line.count"="1");

CREATE TABLE users_managed 
(id string, birth_date string, gender string) 
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' 
STORED AS TEXTFILE
tblproperties ("skip.header.line.count"="1");

SELECT id, gender, birth_date FROM users;
DESCRIBE FORMATTED users;
```
###Unmanaged table
```
CREATE TABLE users_managed 
(id string, birth_date string, gender string) 
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' 
STORED AS TEXTFILE
tblproperties ("skip.header.line.count"="1");

LOAD DATA INPATH '/tmp/demo-data/users.tsv' OVERWRITE INTO TABLE users_managed;

SELECT id, gender, birth_date FROM users_managed;
DESCRIBE FORMATTED users_managed;
```

###Load raw data to table
```
CREATE EXTERNAL TABLE users_raw 
(raw_str string) 
STORED AS TEXTFILE
LOCATION "/tmp/demo-data/user"
tblproperties ("skip.header.line.count"="1");

DESCRIBE FORMATTED users_raw;
SELECT raw_str FROM users_raw;
```
###Temporary table  
```
--temporary table
CREATE TEMPORARY TABLE users_temp 
(id string, birth_date string, gender string) 
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' 
STORED AS ORC;

INSERT INTO TABLE users_temp SELECT * FROM users_managed WHERE gender = "M";

DESCRIBE FORMATTED users_temp;
SELECT gender, id, birth_date FROM users_temp;
```

###JSON data inside Hive table
```
--JSON file format
ADD JAR /usr/hdp/current/hive-webhcat/share/hcatalog/hive-hcatalog-core.jar;

CREATE EXTERNAL TABLE json_full(
address struct<building:string, coord:array<float>, street:string, zipcode:string>, borough string, cuisine string, grades array<struct<`date`:struct<`date`:bigint>, grade:string, score:int>>, name string, restaurant_id string)
ROW FORMAT SERDE 'org.apache.hive.hcatalog.data.JsonSerDe'
LOCATION '/tmp/demo-data/json/';

SELECT name, address.building, address.street, grades[0].score FROM json_full;

CREATE EXTERNAL TABLE json_partial(
address struct<building:string, coord:array<float>, street:string, zipcode:string>, name string)
ROW FORMAT SERDE 'org.apache.hive.hcatalog.data.JsonSerDe'
LOCATION '/tmp/demo-data/json/';

SELECT name, address.building, address.street FROM json_partial;

DESCRIBE FORMATTED json_full;
DESCRIBE FORMATTED json_partial;
```
###View
```
--view
CREATE VIEW vw_male_users (id, birth_date) AS
SELECT id, birth_date FROM users_managed WHERE gender = "M";

SELECT id, birth_date FROM vw_male_users;

CREATE EXTERNAL TABLE products
(url string, category string) 
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' 
STORED AS TEXTFILE 
LOCATION "/tmp/demo-data/product"
tblproperties ("skip.header.line.count"="1");
```
###partition and bucket
```
-- partitioned tables
CREATE TABLE omniture_partition 
(ts string, ip string, url string, swid string, city string)
PARTITIONED BY (country string, state string)
CLUSTERED BY(swid) SORTED BY(ts) INTO 8 BUCKETS
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE;

INSERT INTO TABLE omniture_partition PARTITION(country, state)
SELECT ts, ip, url, swid, city, country, state
FROM omniture_managed;
```

###ETL process using Hive
```
--orc storage format compression method
CREATE TABLE users_orc_none 
(id string, birth_date string, gender string) 
STORED AS ORC
tblproperties ("orc.compress"="NONE");

CREATE TABLE users_orc_snappy
(id string, birth_date string, gender string) 
STORED AS ORC
tblproperties ("orc.compress"="SNAPPY");

CREATE TABLE users_orc_zlib
(id string, birth_date string, gender string) 
STORED AS ORC
tblproperties ("orc.compress"="ZLIB");

INSERT INTO TABLE users_orc_none SELECT * FROM users_managed;
INSERT INTO TABLE users_orc_snappy SELECT * FROM users_managed;
INSERT INTO TABLE users_orc_zlib SELECT * FROM users_managed;


--create simple ETL process
CREATE EXTERNAL TABLE omniturelogs
(
col_1 string, col_2 string, col_3 string, col_4 string, col_5 string, col_6 string, col_7 string, col_8 string, col_9 string, col_10 string, 
col_11 string, col_12 string, col_13 string, col_14 string, col_15 string, col_16 string, col_17 string, col_18 string, col_19 string, col_20 string, 
col_21 string, col_22 string, col_23 string, col_24 string, col_25 string, col_26 string, col_27 string, col_28 string, col_29 string, col_30 string, 
col_31 string, col_32 string, col_33 string, col_34 string, col_35 string, col_36 string, col_37 string, col_38 string, col_39 string, col_40 string, 
col_41 string, col_42 string, col_43 string, col_44 string, col_45 string, col_46 string, col_47 string, col_48 string, col_49 string, col_50 string, 
col_51 string, col_52 string, col_53 string
) 
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' 
STORED AS TEXTFILE 
LOCATION "/tmp/demo-data/omniture";

CREATE TABLE omniture_managed STORED AS ORC AS
SELECT col_2 as ts, col_8 as ip, col_13 as url, col_14 as swid, col_50 as city, col_53 as state, col_51 as country
FROM omniturelogs;
```
