# Question 2: Crime Data Analysis

**Author:** Daniel (dramir182)
**Dataset:** `apang5.crime_data` (3,138,031 records)

---

## Questions Answered

- What's the number of crimes per year? Is there more and more crime nowadays?
- What are the victims of the crime?
- What's the age range of the victims?
- What's the Race/Ethnicity of the victims?
- What's the type of weapon being used in these crimes?

---

## Data Source

- **Website Source:** [LAPD Crime Data - data.lacity.org](https://data.lacity.org/Public-Safety/Crime-Data-from-2020-to-Present/2nrs-mtv8)
- **CSV Google Drive:** Shared folder `CIS 5200 - Woo - Term Project > dataset C...`
  - `crimeData.csv` — 775.3 MB

---

## Prerequisites

SSH access to the cluster and read access to `apang5.crime_data`.

```bash
ssh dramir182@132.226.148.236
```

> Replace `dramir182` with your own username in all commands below.

---

## Step 1: Verify the Date Format

Before querying, check what the date field looks like:

```sql
SELECT date_occ FROM apang5.crime_data
WHERE date_occ IS NOT NULL AND TRIM(date_occ) <> ''
LIMIT 10;
```

**Result format:** `2010 Feb 20 12:00:00 AM`

The year is in positions 1–4, so use `SUBSTR(date_occ, 1, 4)` to extract it.

---

## A) Crimes Per Year

**Question:** What's the number of crimes per year? Is there more and more crime nowadays?

### 0. Remove old files (if any)

```bash
hdfs dfs -rm -r /user/dramir182/tmp/crimes_per_year_export
```

### 1. Beeline SQL

```sql
USE apang5;

DROP TABLE IF EXISTS crimes_per_year_export;

CREATE TABLE crimes_per_year_export
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE
LOCATION "/user/dramir182/tmp/crimes_per_year_export"
AS
SELECT
    SUBSTR(date_occ, 1, 4) AS year,
    COUNT(*) AS crime_count
FROM apang5.crime_data
WHERE date_occ IS NOT NULL AND TRIM(date_occ) <> ''
GROUP BY SUBSTR(date_occ, 1, 4)
ORDER BY year;
```

### 2. Check the HDFS output

```bash
hdfs dfs -ls /user/dramir182/tmp/crimes_per_year_export
```

### 3. Go to the export folder

```bash
mkdir -p ~/termProject_exportedData
cd ~/termProject_exportedData
```

### 4. Copy the part file(s) from HDFS

```bash
hdfs dfs -get /user/dramir182/tmp/crimes_per_year_export/00000*_0
```

### 5. Merge output

```bash
cat 00000*_0 > crimes_per_year.txt
```

### 6. Add the header row

```bash
echo -e "year\tcrime_count" > crimes_per_year_with_header.txt
cat crimes_per_year.txt >> crimes_per_year_with_header.txt
```

### 7. Copy it back to the local machine

```bash
scp dramir182@132.226.148.236:/home/dramir182/termProject_exportedData/crimes_per_year_with_header.txt .
```

---

## B) Victim Age Groups

**Question:** What's the age range of the victims?

### 0. Remove old files (if any)

```bash
hdfs dfs -rm -r /user/dramir182/tmp/victim_age_export
```

### 1. Beeline SQL

```sql
USE apang5;

DROP TABLE IF EXISTS victim_age_export;

CREATE TABLE victim_age_export
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE
LOCATION "/user/dramir182/tmp/victim_age_export"
AS
SELECT
    CASE
        WHEN vict_age < 18 THEN 'Under 18'
        WHEN vict_age BETWEEN 18 AND 25 THEN '18-25'
        WHEN vict_age BETWEEN 26 AND 35 THEN '26-35'
        WHEN vict_age BETWEEN 36 AND 50 THEN '36-50'
        WHEN vict_age BETWEEN 51 AND 65 THEN '51-65'
        WHEN vict_age > 65 THEN '65+'
        ELSE 'Unknown'
    END AS age_group,
    COUNT(*) AS count
FROM apang5.crime_data
WHERE vict_age IS NOT NULL
GROUP BY
    CASE
        WHEN vict_age < 18 THEN 'Under 18'
        WHEN vict_age BETWEEN 18 AND 25 THEN '18-25'
        WHEN vict_age BETWEEN 26 AND 35 THEN '26-35'
        WHEN vict_age BETWEEN 36 AND 50 THEN '36-50'
        WHEN vict_age BETWEEN 51 AND 65 THEN '51-65'
        WHEN vict_age > 65 THEN '65+'
        ELSE 'Unknown'
    END
ORDER BY count DESC;
```

### 2. Check the HDFS output

```bash
hdfs dfs -ls /user/dramir182/tmp/victim_age_export
```

### 3. Go to the export folder

```bash
cd ~/termProject_exportedData
```

### 4. Copy the part file(s) from HDFS

```bash
hdfs dfs -get /user/dramir182/tmp/victim_age_export/00000*_0
```

### 5. Merge output

```bash
cat 00000*_0 > victim_age.txt
```

### 6. Add the header row

```bash
echo -e "age_group\tcount" > victim_age_with_header.txt
cat victim_age.txt >> victim_age_with_header.txt
```

### 7. Copy it back to the local machine

```bash
scp dramir182@132.226.148.236:/home/dramir182/termProject_exportedData/victim_age_with_header.txt .
```

---

## C) Victim Race/Ethnicity

**Question:** What's the Race/Ethnicity of the victims?

### 0. Remove old files (if any)

```bash
hdfs dfs -rm -r /user/dramir182/tmp/victim_race_export
```

### 1. Beeline SQL

```sql
USE apang5;

DROP TABLE IF EXISTS victim_race_export;

CREATE TABLE victim_race_export
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE
LOCATION "/user/dramir182/tmp/victim_race_export"
AS
SELECT
    vict_descent,
    COUNT(*) AS count
FROM apang5.crime_data
WHERE vict_descent IS NOT NULL AND TRIM(vict_descent) <> ''
GROUP BY vict_descent
ORDER BY count DESC;
```

### 2. Check the HDFS output

```bash
hdfs dfs -ls /user/dramir182/tmp/victim_race_export
```

### 3. Go to the export folder

```bash
cd ~/termProject_exportedData
```

### 4. Copy the part file(s) from HDFS

```bash
hdfs dfs -get /user/dramir182/tmp/victim_race_export/00000*_0
```

### 5. Merge output

```bash
cat 00000*_0 > victim_race.txt
```

### 6. Add the header row

```bash
echo -e "vict_descent\tcount" > victim_race_with_header.txt
cat victim_race.txt >> victim_race_with_header.txt
```

### 7. Copy it back to the local machine

```bash
scp dramir182@132.226.148.236:/home/dramir182/termProject_exportedData/victim_race_with_header.txt .
```

**Race/Ethnicity Code Reference:**

| Code | Meaning |
|------|---------|
| H | Hispanic |
| W | White |
| B | Black |
| O | Other |
| X | Unknown |
| A | Asian |
| K | Korean |
| F | Filipino |
| C | Chinese |
| J | Japanese |

---

## D) Top 15 Weapons Used

**Question:** What's the type of weapon being used in these crimes?

### 0. Remove old files (if any)

```bash
hdfs dfs -rm -r /user/dramir182/tmp/weapon_export
```

### 1. Beeline SQL

```sql
USE apang5;

DROP TABLE IF EXISTS weapon_export;

CREATE TABLE weapon_export
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE
LOCATION "/user/dramir182/tmp/weapon_export"
AS
SELECT
    weapon_desc,
    COUNT(*) AS count
FROM apang5.crime_data
WHERE weapon_desc IS NOT NULL AND TRIM(weapon_desc) <> ''
GROUP BY weapon_desc
ORDER BY count DESC
LIMIT 15;
```

### 2. Check the HDFS output

```bash
hdfs dfs -ls /user/dramir182/tmp/weapon_export
```

### 3. Go to the export folder

```bash
cd ~/termProject_exportedData
```

### 4. Copy the part file(s) from HDFS

```bash
hdfs dfs -get /user/dramir182/tmp/weapon_export/00000*_0
```

### 5. Merge output

```bash
cat 00000*_0 > weapons.txt
```

### 6. Add the header row

```bash
echo -e "weapon_desc\tcount" > weapons_with_header.txt
cat weapons.txt >> weapons_with_header.txt
```

### 7. Copy it back to the local machine

```bash
scp dramir182@132.226.148.236:/home/dramir182/termProject_exportedData/weapons_with_header.txt .
```

---

## Results Summary

| Query | Output Rows | Key Finding |
|-------|-------------|-------------|
| Crimes Per Year | 15 rows (2010–2024) | Peak in 2016 (283,798); 2024 is partial year |
| Victim Age Groups | 6 rows | Ages 36–50 most affected (705,264) |
| Victim Race/Ethnicity | 20 rows | Hispanic victims largest group (1,028,452) |
| Top 15 Weapons | 15 rows | Strong-arm (hands/fists) #1 at 609,573 |
