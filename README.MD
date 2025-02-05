# Data Warehouse (Sparkify) 🚀

## 📌 Introduction

### Project Overview
This project builds a **Data Warehouse (DWH) on Amazon Redshift** for **Sparkify**, a music streaming startup. The goal is to **design an ETL pipeline** that extracts user and song activity data from **S3**, stages the data in **Redshift**, and loads it into a **star schema** optimized for analytics.

### Why is this important for Sparkify?
Sparkify collects **massive logs** of user interactions (song plays, searches, skips, etc.). An **OLAP-optimized data warehouse** enables:
- **Efficient query performance** on structured datasets.
- **Advanced analytics** to understand user behavior.
- **Data-driven decisions** to improve engagement and retention.

---

## 📌 Database Schema Design

The schema follows a **Star Schema Model**, which is optimized for **OLAP workloads**.

### 1️⃣ Fact Table
#### `songplays` - Records of song play activity (`NextSong` events).

| Column        | Type                     | Description                 |
|--------------|-------------------------|-----------------------------|
| songplay_id  | `BIGINT IDENTITY(0,1)`  | Unique ID                   |
| start_time   | `TIMESTAMP`             | Timestamp of the song play  |
| user_id      | `INTEGER`               | ID of the user              |
| level        | `VARCHAR`               | User subscription level     |
| song_id      | `VARCHAR`               | ID of the song              |
| artist_id    | `VARCHAR`               | ID of the artist            |
| session_id   | `INTEGER`               | Session ID                  |
| location     | `VARCHAR`               | User location               |
| user_agent   | `VARCHAR`               | User’s device info          |

### 2️⃣ Dimension Tables

#### `users` - Stores user information.

| Column      | Type      |
|------------|----------|
| user_id    | `INTEGER PRIMARY KEY` |
| first_name | `VARCHAR` |
| last_name  | `VARCHAR` |
| gender     | `VARCHAR(1)` |
| level      | `VARCHAR` |

#### `songs` - Stores song information.

| Column     | Type      |
|------------|----------|
| song_id    | `VARCHAR PRIMARY KEY` |
| title      | `VARCHAR` |
| artist_id  | `VARCHAR` |
| year       | `INTEGER` |
| duration   | `FLOAT` |

#### `artists` - Stores artist details.

| Column     | Type      |
|------------|----------|
| artist_id  | `VARCHAR PRIMARY KEY` |
| name       | `VARCHAR` |
| location   | `VARCHAR` |
| latitude   | `FLOAT` |
| longitude  | `FLOAT` |

#### `time` - Stores time-based details.

| Column     | Type      |
|------------|----------|
| start_time | `TIMESTAMP PRIMARY KEY` |
| hour       | `INTEGER` |
| day        | `INTEGER` |
| week       | `INTEGER` |
| month      | `INTEGER` |
| year       | `INTEGER` |
| weekday    | `INTEGER` |

---

## 📌 ETL Pipeline

The ETL pipeline consists of **three main stages**:

1️⃣ **Extract:** Data is loaded from **S3 to staging tables in Redshift**.  
2️⃣ **Transform:** Data is cleaned, filtered, and transformed.  
3️⃣ **Load:** The transformed data is inserted into **fact and dimension tables**.

### 🔹 Loading Data into Staging Tables
```sql
COPY staging_events 
FROM 's3://udacity-dend/log_data'
CREDENTIALS 'aws_iam_role=arn:aws:iam::<AWS_ACCOUNT_ID>:role/MyRedshiftRole'
FORMAT AS JSON 's3://udacity-dend/log_json_path.json'
REGION 'us-west-2';

COPY staging_songs 
FROM 's3://udacity-dend/song_data'
CREDENTIALS 'aws_iam_role=arn:aws:iam::<AWS_ACCOUNT_ID>:role/MyRedshiftRole'
FORMAT AS JSON 'auto'
REGION 'us-west-2';

🔹 Inserting Data into Final Tables

INSERT INTO songplays (start_time, user_id, level, song_id, artist_id, session_id, location, user_agent)
SELECT TIMESTAMP 'epoch' + ts/1000 * INTERVAL '1 second',
       userId, level, song_id, artist_id, sessionId, location, userAgent
FROM staging_events e
LEFT JOIN staging_songs s
ON e.song = s.title AND e.artist = s.artist_name
WHERE e.page = 'NextSong';

📌 Files in Repository
File	Description
dwh.cfg	Stores Redshift cluster and IAM role details
sql_queries.py	SQL queries for table creation, insertion, and data loading
create_tables.py	Drops and creates tables in Redshift
etl.py	Loads data from S3 into Redshift and transforms it
README.md	Documentation for the project
📌 How to Run the Scripts
1️⃣ Setup AWS and Configure Credentials

    Ensure IAM Role has AmazonS3ReadOnlyAccess.
    Store Redshift credentials in dwh.cfg.

2️⃣ Run Table Creation Script

python create_tables.py

This will drop and recreate tables in Redshift.
3️⃣ Run ETL Script

python etl.py

This will load data from S3 to Redshift and transform it.
4️⃣ Verify Data in Redshift

Run the following queries in AWS Redshift Query Editor:

SELECT COUNT(*) FROM songplays;
SELECT COUNT(*) FROM users;
SELECT COUNT(*) FROM songs;
SELECT COUNT(*) FROM artists;
SELECT COUNT(*) FROM time;

📌 Example Analytics Queries
1️⃣ Most Played Songs

SELECT s.title, COUNT(*) AS play_count
FROM songplays sp
JOIN songs s ON sp.song_id = s.song_id
GROUP BY s.title
ORDER BY play_count DESC
LIMIT 10;

2️⃣ Peak Streaming Hours

SELECT hour, COUNT(*) AS plays
FROM time
JOIN songplays ON time.start_time = songplays.start_time
GROUP BY hour
ORDER BY plays DESC
LIMIT 5;

📌 Error Handling & Debugging
Error	Cause	Solution
FATAL: password authentication failed	Wrong DB_USER or DB_PASSWORD in dwh.cfg	Correct credentials
Access Denied (403)	IAM Role doesn't have S3 access	Assign AmazonS3ReadOnlyAccess
relation "songplays" does not exist	Tables were not created	Run create_tables.py
staging_events is empty	COPY failed	Check IAM Role and S3 paths
📌 Future Improvements

    ✅ Data Quality Checks
    ✅ Performance Optimization (DISTKEY & SORTKEY)
    ✅ Automated Scheduling with Apache Airflow
    ✅ Dashboard using Tableau or Power BI

📌 Conclusion

✅ Built a scalable Data Warehouse in Amazon Redshift
✅ Designed an optimized Star Schema for analytics
✅ Implemented a robust ETL pipeline for Sparkify

This project of mines demonstrates key skills in Data Engineering, Cloud Computing (AWS Redshift, S3, IAM), and SQL Query Optimization.
📌 References

    Amazon Redshift Docs
    psycopg2 Docs
    Kimball Data Warehouse

