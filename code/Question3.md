# Stop Data Analysis

## A) Stops per Year

**Question:** Are LAPD stops decreasing year-to-year?

### 0. Remove old files

```bash
hdfs dfs -rm -r /user/apang5/tmp/stops_per_year_export
```

### 1. Beeline SQL

```sql
USE apang5;

DROP TABLE IF EXISTS stops_per_year_export;

CREATE TABLE stops_per_year_export
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE
LOCATION "/user/apang5/tmp/stops_per_year_export"
AS
SELECT
    SUBSTR(stop_date, 7, 4) AS year,
    COUNT(*) AS stop_count
FROM stop_data
WHERE stop_date IS NOT NULL
  AND TRIM(stop_date) <> ''
GROUP BY SUBSTR(stop_date, 7, 4)
ORDER BY year;
```

### 2. Check the HDFS output

```bash
hdfs dfs -ls /user/apang5/tmp/stops_per_year_export
```

### 3. Go to the export folder

```bash
cd ~/termProject_exportedData
```

### 4. Copy the part file(s) from HDFS

```bash
hdfs dfs -get /user/apang5/tmp/stops_per_year_export/00000*_0
```

### 5. Merge output

```bash
cat 00000*_0 > stops_per_year_export.txt
```

### 6. Add the header row

```bash
echo -e "year\tstop_count" > stops_per_year_export_with_header.txt

cat stops_per_year_export.txt >> stops_per_year_export_with_header.txt
```

### 7. Copy it back to the local machine

```bash
scp apang5@132.226.148.236:/home/apang5/termProject_exportedData/stops_per_year_export_with_header.txt .
```

---

## B) Race/Ethnicity Breakdown

**Question:** Who is getting pulled over by LAPD?

### 0. Remove old files

```bash
hdfs dfs -rm -r /user/apang5/tmp/stop_race_breakdown_export
```

### 1. Beeline SQL

```sql
USE apang5;

DROP TABLE IF EXISTS stop_race_breakdown_export;

CREATE TABLE stop_race_breakdown_export
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE
LOCATION "/user/apang5/tmp/stop_race_breakdown_export"
AS
SELECT
    descent_description,
    stop_count,
    ROUND((CAST(stop_count AS DOUBLE) / total_count) * 100, 2) AS percent
FROM
(
    SELECT
        descent_description,
        COUNT(*) AS stop_count,
        SUM(COUNT(*)) OVER () AS total_count
    FROM stop_data
    WHERE descent_description IS NOT NULL
      AND TRIM(descent_description) <> ''
    GROUP BY descent_description
) race_counts
ORDER BY stop_count DESC;
```

### 2. Check the HDFS output

```bash
hdfs dfs -ls /user/apang5/tmp/stop_race_breakdown_export
```

### 3. Go to the export folder

```bash
cd ~/termProject_exportedData
```

### 4. Copy the part file(s) from HDFS

```bash
hdfs dfs -get /user/apang5/tmp/stop_race_breakdown_export/00000*_0
```

### 5. Merge output

```bash
cat 00000*_0 > stop_race_breakdown_export.txt
```

### 6. Add the header row

```bash
echo -e "descent_description\tstop_count\tpercent" > stop_race_breakdown_export_with_header.txt

cat stop_race_breakdown_export.txt >> stop_race_breakdown_export_with_header.txt
```

### 7. Copy it back to the local machine

```bash
scp apang5@132.226.148.236:/home/apang5/termProject_exportedData/stop_race_breakdown_export_with_header.txt .
```
