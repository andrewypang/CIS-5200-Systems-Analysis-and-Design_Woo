# Querying from Others
Querying Shared Hive Tables and Exporting Results

This template is for teammates who want to query tables created by others. The following will use `apang5` Hive database as an example and export their own results.

Replace:

```text
teammate_username
```

with your actual username/database.

---

## 1. Test access to the shared table

Open Beeline:

```bash
beeline
```

Then run:

```sql
SELECT * FROM apang5.crime_data LIMIT 5;
```
Note: tables include:  
- apang5.crime_data
- apang5.arrest_data
- apang5.stop_data

If this works, you can query the shared table.

---

## 2. Create an export table in your own database

```sql
USE teammate_username;

DROP TABLE IF EXISTS sample_export;

CREATE TABLE sample_export
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE
LOCATION "/user/teammate_username/tmp/sample_export"
AS
SELECT
    area,
    area_name,
    COUNT(*) AS record_count
FROM apang5.crime_data
GROUP BY area, area_name
ORDER BY record_count DESC;
```

---

## 3. Check the HDFS output

```bash
hdfs dfs -ls /user/teammate_username/tmp/sample_export
```

---

## 4. Go to your export folder

```bash
mkdir -p ~/termProject_exportedData
cd ~/termProject_exportedData
```

---

## 5. Copy the part file(s) from HDFS

```bash
hdfs dfs -get /user/teammate_username/tmp/sample_export/00000*_0
```

---

## 6. Merge or rename output

If there are multiple files:

```bash
cat 00000*_0 > sample_export.txt
```

If there is only one file:

```bash
mv 000000_0 sample_export.txt
```

---

## 7. Add the header row (ignore this step if having a header column is not necessary)

```bash
echo -e "area\tarea_name\trecord_count" > sample_export_with_header.txt

cat sample_export.txt >> sample_export_with_header.txt
```

---

## 8. Copy it back to your local machine

Run this from your local machine:

```bash
scp teammate_username@132.226.148.236:/home/teammate_username/termProject_exportedData/sample_export_with_header.txt .
```

---

# How to Run a Hive SQL Query and Export the Results to a Local Machine

This template shows how to run a Hive query, save the output to HDFS, copy the result to the server, add column headers, and download it to a local machine.

---

## 0. Choose an export name

For each query/export, choose a short name.

Example:

```bash
EXPORT_NAME=sample_export
```

In the examples below, replace:

```text
sample_export
```

with your actual export name.

---

## 1. Remove old HDFS export folder, if needed

Run this in the server shell:

```bash
hdfs dfs -rm -r /user/apang5/tmp/sample_export
```

If the folder does not exist, that is fine.

---

## 2. Run the SQL query in Beeline

Open Beeline:

```bash
beeline
```

Then run:

```sql
USE apang5;

DROP TABLE IF EXISTS sample_export;

CREATE TABLE sample_export
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE
LOCATION "/user/apang5/tmp/sample_export"
AS
SELECT
    area,
    area_name,
    COUNT(*) AS record_count
FROM crime_data
GROUP BY area, area_name
ORDER BY record_count DESC;
```

This example creates a simple export table showing the number of crime records by LAPD area.

---

## 3. Check the HDFS output

Run this in the server shell:

```bash
hdfs dfs -ls /user/apang5/tmp/sample_export
```

You should see one or more files like:

```text
000000_0
000001_0
```

---

## 4. Go to the export folder on the server

```bash
mkdir -p ~/termProject_exportedData
cd ~/termProject_exportedData
```

---

## 5. Copy the part file(s) from HDFS to the server folder

```bash
hdfs dfs -get /user/apang5/tmp/sample_export/00000*_0
```

---

## 6. Merge or rename the output

If there are multiple part files:

```bash
cat 00000*_0 > sample_export.txt
```

If there is only one part file:

```bash
mv 000000_0 sample_export.txt
```

---

## 7. Add the header row (ignore this step if having a header name column is not necessary)

The header row must match the columns in the SQL query.

For the example query above, the columns are:

```text
area, area_name, record_count
```

Because the file is tab-delimited, use `\t` between field names:

```bash
echo -e "area\tarea_name\trecord_count" > sample_export_with_header.txt

cat sample_export.txt >> sample_export_with_header.txt
```

---

## 8. Copy the exported file back to the local machine

Run this from the local machine:

```bash
scp apang5@132.226.148.236:/home/apang5/termProject_exportedData/sample_export_with_header.txt .
```

---

## Notes

- Use tab-delimited output with `FIELDS TERMINATED BY '\t'` because some text fields contain commas.
- The exported Hive part files do not include column headers, so the header row must be added manually.
- The header row must match the SQL `SELECT` fields exactly and in the same order.
- If rerunning the same export, remove the old HDFS folder first.
- If several exports are done in the same folder, remove old part files before copying new ones:

```bash
rm 00000*_0
```
