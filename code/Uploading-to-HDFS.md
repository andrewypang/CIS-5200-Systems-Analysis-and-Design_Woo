# Uploading Data to HDFS and Creating Hive Tables

## 1. Create staging folder on the server

```bash
ssh apang5@132.226.148.236
mkdir -p termProject_staging
exit
```

---

## 2. Upload CSV files to the server staging folder

```bash
scp crimeData.csv apang5@132.226.148.236:~/termProject_staging
scp arrestData.csv apang5@132.226.148.236:~/termProject_staging
scp stopData.csv apang5@132.226.148.236:~/termProject_staging
```

---

## 3. SSH back into the server

```bash
ssh apang5@132.226.148.236
```

---

## 4. Create HDFS folders

```bash
hdfs dfs -mkdir -p termProject

hdfs dfs -mkdir -p termProject/crime
hdfs dfs -mkdir -p termProject/arrest
hdfs dfs -mkdir -p termProject/stop
```

---

## 5. Upload CSV files from staging folder to HDFS

```bash
hdfs dfs -put ~/termProject_staging/crimeData.csv termProject/crime/
hdfs dfs -put ~/termProject_staging/arrestData.csv termProject/arrest/
hdfs dfs -put ~/termProject_staging/stopData.csv termProject/stop/
```

---

## 6. Open Beeline

```bash
beeline
```

---

## 7. Use database

```sql
USE apang5;
```

---

## 8. Create Crime Data table

```sql
DROP TABLE IF EXISTS crime_data;

CREATE EXTERNAL TABLE crime_data (
    dr_no BIGINT,
    date_rptd STRING,
    date_occ STRING,
    time_occ INT,
    area INT,
    area_name STRING,
    reporting_district INT,
    part_1_2 INT,
    crm_cd INT,
    crm_cd_desc STRING,
    mocodes STRING,
    vict_age INT,
    vict_sex STRING,
    vict_descent STRING,
    premis_cd DOUBLE,
    premis_desc STRING,
    weapon_used_cd DOUBLE,
    weapon_desc STRING,
    status STRING,
    status_desc STRING,
    crm_cd_1 DOUBLE,
    crm_cd_2 DOUBLE,
    crm_cd_3 DOUBLE,
    crm_cd_4 DOUBLE,
    location STRING,
    cross_street STRING,
    lat DOUBLE,
    lon DOUBLE
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.OpenCSVSerde'
WITH SERDEPROPERTIES (
    "separatorChar" = ",",
    "quoteChar" = "\"",
    "escapeChar" = "\\"
)
STORED AS TEXTFILE
LOCATION '/user/apang5/termProject/crime/'
TBLPROPERTIES ("skip.header.line.count"="1");
```

---

## 9. Create Arrest Data table

```sql
DROP TABLE IF EXISTS arrest_data;

CREATE EXTERNAL TABLE arrest_data (
    report_id BIGINT,
    report_type STRING,
    arrest_date STRING,
    arrest_time DOUBLE,
    area_id INT,
    area_name STRING,
    reporting_district INT,
    age INT,
    sex_code STRING,
    descent_code STRING,
    charge_group_code DOUBLE,
    charge_group_description STRING,
    arrest_type_code STRING,
    charge STRING,
    charge_description STRING,
    disposition_description STRING,
    address STRING,
    cross_street STRING,
    lat DOUBLE,
    lon DOUBLE,
    location STRING,
    booking_date STRING,
    booking_time DOUBLE,
    booking_location STRING,
    booking_location_code DOUBLE
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.OpenCSVSerde'
WITH SERDEPROPERTIES (
    "separatorChar" = ",",
    "quoteChar" = "\"",
    "escapeChar" = "\\"
)
STORED AS TEXTFILE
LOCATION '/user/apang5/termProject/arrest/'
TBLPROPERTIES ("skip.header.line.count"="1");
```

---

## 10. Create Stop Data table

```sql
DROP TABLE IF EXISTS stop_data;

CREATE EXTERNAL TABLE stop_data (
    stop_number BIGINT,
    form_reference_number BIGINT,
    sex_code STRING,
    descent_code STRING,
    descent_description STRING,
    stop_date STRING,
    stop_time STRING,
    officer_1_serial_number DOUBLE,
    officer_1_division_number STRING,
    division_description_1 STRING,
    officer_2_serial_number DOUBLE,
    officer_2_division_number STRING,
    division_description_2 STRING,
    reporting_district STRING,
    stop_type STRING,
    post_stop_activity_indicator STRING
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.OpenCSVSerde'
WITH SERDEPROPERTIES (
    "separatorChar" = ",",
    "quoteChar" = "\"",
    "escapeChar" = "\\"
)
STORED AS TEXTFILE
LOCATION '/user/apang5/termProject/stop/'
TBLPROPERTIES ("skip.header.line.count"="1");
```

---

## 11. Verify tables

```sql
SHOW TABLES;

SELECT * FROM crime_data LIMIT 5;
SELECT * FROM arrest_data LIMIT 5;
SELECT * FROM stop_data LIMIT 5;
```
