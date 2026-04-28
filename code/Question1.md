# Arrest Data Analysis

**Create Arrest Data Table to Pull Data Necessary for Answering Questions:**
**What’s the Number of Arrests per year? Are there more and more arrests nowadays?**
**What’s the perpetrator like?**
**- What's the age range of the arrest?**
**- What’s the Race/Ethnicity of the arrest?**
**- What are they getting arrested for?**


```
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
LOCATION '/user/username/termProject/arrest/'
TBLPROPERTIES ("skip.header.line.count"="1");
```
# Double Check if Table is not null
```
SELECT * FROM arrest_data LIMIT 10;
```

# 1) Arrests Per Year & More Arrests Nowadays?

**Question:** What’s the number of arrests per year? Are there more and more arrests nowadays?

```
SELECT
  a.arrest_year,
  a.num_arrests,
  a.num_arrests - b.num_arrests AS change_from_previous_year
FROM
(
  SELECT
    year(from_unixtime(unix_timestamp(arrest_date, 'MM/dd/yyyy'))) AS arrest_year,
    COUNT(*) AS num_arrests
  FROM arrest_data
  WHERE arrest_date IS NOT NULL
  GROUP BY year(from_unixtime(unix_timestamp(arrest_date, 'MM/dd/yyyy')))
) a
LEFT JOIN
(
  SELECT
    year(from_unixtime(unix_timestamp(arrest_date, 'MM/dd/yyyy'))) + 1 AS arrest_year,
    COUNT(*) AS num_arrests
  FROM arrest_data
  WHERE arrest_date IS NOT NULL
  GROUP BY year(from_unixtime(unix_timestamp(arrest_date, 'MM/dd/yyyy')))
) b
ON a.arrest_year = b.arrest_year
ORDER BY a.arrest_year;
```

# Export Data from 1:
```
DROP TABLE IF EXISTS arrests_by_sex_export;

CREATE TABLE arrests_by_sex_export
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE
LOCATION "/user/rrami165/tmp/arrests_by_sex_export"
AS
SELECT
    sex_code,
    COUNT(*) AS num_arrests
FROM arrest_data
GROUP BY sex_code
ORDER BY num_arrests DESC;

hdfs dfs -get /user/username/tmp/arrests_by_sex_export ~/arrests_by_sex_export
mv ~/arrests_by_sex_export/000000_0 ~/arrests_by_sex.tsv
scp your_username@host1:~/arrests_by_sex.tsv .
```

# 2): What’s the perpetrator like? This is usually answered with age, sex, and descent/race. What are they getting arrested for?
```
SELECT
    sex_code,
    COUNT(*) AS num_arrests
FROM arrest_data
GROUP BY sex_code
ORDER BY num_arrests DESC;
```
```
SELECT
    CASE
        WHEN age < 18 THEN 'Under 18'
        WHEN age BETWEEN 18 AND 24 THEN '18-24'
        WHEN age BETWEEN 25 AND 34 THEN '25-34'
        WHEN age BETWEEN 35 AND 44 THEN '35-44'
        WHEN age BETWEEN 45 AND 54 THEN '45-54'
        WHEN age BETWEEN 55 AND 64 THEN '55-64'
        ELSE '65+'
    END AS age_range,
    COUNT(*) AS num_arrests
FROM arrest_data
WHERE age IS NOT NULL
GROUP BY
    CASE
        WHEN age < 18 THEN 'Under 18'
        WHEN age BETWEEN 18 AND 24 THEN '18-24'
        WHEN age BETWEEN 25 AND 34 THEN '25-34'
        WHEN age BETWEEN 35 AND 44 THEN '35-44'
        WHEN age BETWEEN 45 AND 54 THEN '45-54'
        WHEN age BETWEEN 55 AND 64 THEN '55-64'
        ELSE '65+'
    END
ORDER BY age_range;
```
```
SELECT
    descent_code,
    COUNT(*) AS num_arrests
FROM arrest_data
GROUP BY descent_code
ORDER BY num_arrests DESC;
```
```
SELECT
  charge_group_description,
  COUNT(*) AS num_arrests
FROM arrest_data
WHERE charge_group_description IS NOT NULL
GROUP BY charge_group_description
ORDER BY num_arrests DESC
LIMIT 20;
```
# Export 2:
```
DROP TABLE IF EXISTS arrests_by_sex_export;

CREATE TABLE arrests_by_sex_export
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE
LOCATION "/user/rrami165/tmp/arrests_by_sex_export"
AS
SELECT
    sex_code,
    COUNT(*) AS num_arrests
FROM arrest_data
GROUP BY sex_code
ORDER BY num_arrests DESC;
```
```
DROP TABLE IF EXISTS arrests_by_age_range_export;

CREATE TABLE arrests_by_age_range_export
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE
LOCATION "/user/rrami165/tmp/arrests_by_age_range_export"
AS
SELECT
    CASE
        WHEN age < 18 THEN 'Under 18'
        WHEN age BETWEEN 18 AND 24 THEN '18-24'
        WHEN age BETWEEN 25 AND 34 THEN '25-34'
        WHEN age BETWEEN 35 AND 44 THEN '35-44'
        WHEN age BETWEEN 45 AND 54 THEN '45-54'
        WHEN age BETWEEN 55 AND 64 THEN '55-64'
        ELSE '65+'
    END AS age_range,
    COUNT(*) AS num_arrests
FROM arrest_data
WHERE age IS NOT NULL
GROUP BY
    CASE
        WHEN age < 18 THEN 'Under 18'
        WHEN age BETWEEN 18 AND 24 THEN '18-24'
        WHEN age BETWEEN 25 AND 34 THEN '25-34'
        WHEN age BETWEEN 35 AND 44 THEN '35-44'
        WHEN age BETWEEN 45 AND 54 THEN '45-54'
        WHEN age BETWEEN 55 AND 64 THEN '55-64'
        ELSE '65+'
    END
ORDER BY age_range;
```
```
DROP TABLE IF EXISTS arrests_by_descent_export;

CREATE TABLE arrests_by_descent_export
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE
LOCATION "/user/rrami165/tmp/arrests_by_descent_export"
AS
SELECT
    descent_code,
    COUNT(*) AS num_arrests
FROM arrest_data
GROUP BY descent_code
ORDER BY num_arrests DESC;
```
```
DROP TABLE IF EXISTS arrests_by_charge_description_export;

CREATE TABLE arrests_by_charge_description_export
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE
LOCATION "/user/rrami165/tmp/arrests_by_charge_description_export"
AS
SELECT
    charge_description,
    COUNT(*) AS num_arrests
FROM arrest_data
WHERE charge_description IS NOT NULL
GROUP BY charge_description
ORDER BY num_arrests DESC;
```

