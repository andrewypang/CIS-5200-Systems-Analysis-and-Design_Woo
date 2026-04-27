# Question 4: Comparative Analysis

## A) Crime to Arrest

**Question:** Which areas have high crime counts but relatively low arrest counts?

### 0. Remove old files (if any)

```bash
hdfs dfs -rm -r /user/apang5/tmp/crime_to_arrest_ratio_export
```

### 1. Beeline SQL

```sql
USE apang5;

DROP TABLE IF EXISTS crime_to_arrest_ratio_export;

CREATE TABLE crime_to_arrest_ratio_export
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE
LOCATION "/user/apang5/tmp/crime_to_arrest_ratio_export"
AS
SELECT
    c.area AS division_id,
    c.area_name,
    c.crime_count,
    COALESCE(a.arrest_count, 0) AS arrest_count,
    CASE
        WHEN COALESCE(a.arrest_count, 0) > 0
        THEN CAST(c.crime_count AS DOUBLE) / a.arrest_count
        ELSE NULL
    END AS crime_to_arrest_ratio
FROM
(
    SELECT
        area,
        area_name,
        COUNT(*) AS crime_count
    FROM crime_data
    GROUP BY area, area_name
) c
LEFT JOIN
(
    SELECT
        area_id,
        area_name,
        COUNT(*) AS arrest_count
    FROM arrest_data
    GROUP BY area_id, area_name
) a
ON c.area = a.area_id
ORDER BY crime_to_arrest_ratio DESC;
```

### 2. Remove old files

```bash
hdfs dfs -rm -r /user/apang5/tmp/crime_to_arrest_ratio_export
```

### 3. Check the HDFS output

```bash
hdfs dfs -ls /user/apang5/tmp/crime_to_arrest_ratio_export
```

### 4. Go to the export folder

```bash
cd ~/termProject_exportedData
```

### 5. Copy the part file(s) from HDFS

```bash
hdfs dfs -get /user/apang5/tmp/crime_to_arrest_ratio_export/00000*_0
```

### 6. Merge or rename output

If there are multiple files:

```bash
cat 00000*_0 > crime_to_arrest_ratio_export.txt
```

If there is only one file:

```bash
mv 000000_0 crime_to_arrest_ratio_export.txt
```

### 7. Add the header row

```bash
echo -e "division_id\tarea_name\tcrime_count\tarrest_count\tcrime_to_arrest_ratio" > crime_to_arrest_ratio_export_with_header.txt

cat crime_to_arrest_ratio_export.txt >> crime_to_arrest_ratio_export_with_header.txt
```

### 8. Copy it back to the local machine

```bash
scp apang5@132.226.148.236:/home/apang5/termProject_exportedData/crime_to_arrest_ratio_export_with_header.txt .
```

---

## B) Stop to Arrest

**Question:** Which areas have high stop counts but relatively low arrest counts?

### 0. Remove old files (if any)

```bash
hdfs dfs -rm -r /user/apang5/tmp/stop_to_arrest_ratio_export
```

### 1. Beeline SQL

```sql
USE apang5;

DROP TABLE IF EXISTS stop_to_arrest_ratio_export;

CREATE TABLE stop_to_arrest_ratio_export
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE
LOCATION "/user/apang5/tmp/stop_to_arrest_ratio_export"
AS
SELECT
    s.division_id,
    s.area_name,
    s.stop_count,
    COALESCE(a.arrest_count, 0) AS arrest_count,
    CASE
        WHEN COALESCE(a.arrest_count, 0) > 0
        THEN CAST(s.stop_count AS DOUBLE) / a.arrest_count
        ELSE NULL
    END AS stop_to_arrest_ratio
FROM
(
    SELECT
        officer_1_division_number AS division_id,
        division_description_1 AS area_name,
        COUNT(*) AS stop_count
    FROM stop_data
    WHERE officer_1_division_number IS NOT NULL
    GROUP BY officer_1_division_number, division_description_1
) s
LEFT JOIN
(
    SELECT
        CAST(area_id AS STRING) AS division_id,
        area_name,
        COUNT(*) AS arrest_count
    FROM arrest_data
    GROUP BY area_id, area_name
) a
ON s.division_id = a.division_id
ORDER BY stop_to_arrest_ratio DESC;
```

### 2. Check the HDFS output

```bash
hdfs dfs -ls /user/apang5/tmp/stop_to_arrest_ratio_export
```

### 3. Go to the export folder

```bash
cd ~/termProject_exportedData
```

### 4. Copy the part file(s) from HDFS

```bash
hdfs dfs -get /user/apang5/tmp/stop_to_arrest_ratio_export/00000*_0
```

### 5. Merge or rename output

If there are multiple files:

```bash
cat 00000*_0 > stop_to_arrest_ratio_export.txt
```

If there is only one file:

```bash
mv 000000_0 stop_to_arrest_ratio_export.txt
```

### 6. Add the header row

```bash
echo -e "division_id\tarea_name\tstop_count\tarrest_count\tstop_to_arrest_ratio" > stop_to_arrest_ratio_export_with_header.txt

cat stop_to_arrest_ratio_export.txt >> stop_to_arrest_ratio_export_with_header.txt
```

### 7. Copy it back to the local machine

```bash
scp apang5@132.226.148.236:/home/apang5/termProject_exportedData/stop_to_arrest_ratio_export_with_header.txt .
```
