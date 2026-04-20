

## Overview

This folder contains the complete end-to-end pipeline for processing flight delay data from raw CSV through data preparation, feature engineering, database loading, and visualization in Tableau. The workflow spans from Google Cloud Storage to PostgreSQL, enabling scalable analysis and BI dashboards.

---
## Stage 00: Data Repartitioning & GCS Processing

### Purpose

Convert merged CSV files from GCS into compressed, repartitioned format for efficient downstream processing.

### Key Components

- **Source:** Google Cloud Storage (GCS) bucket with merged flight data (MERGED_YYYY.csv)

- **Target:** Google Drive storage with gzip-compressed, repartitioned CSV files (46 partitions)

- **Technology:** PySpark on Google Colab with GCS connector


### Code Example: GCS Authentication & Spark Session


```python

from google.colab import auth, drive

from pyspark.sql import SparkSession

import google.auth

import google.auth.transport.requests

  

# 1. Google Drive authentication

auth.authenticate_user()

  

# 2. Mount Google Drive

drive.mount('/content/drive')

  

# 3. Configure Spark with GCS access

auth.authenticate_user()

credentials, project = google.auth.default()

request = google.auth.transport.requests.Request()

credentials.refresh(request)

access_token = credentials.token

  

spark = SparkSession.builder \

.appName("Team74_Merged_Processing") \

.config("spark.jars.packages", "com.google.cloud.bigdataoss:gcs-connector:hadoop3-2.2.5") \

.config("spark.hadoop.fs.gs.impl", "com.google.cloud.hadoop.fs.gcs.GoogleHadoopFileSystem") \

.config("spark.hadoop.fs.gs.auth.access.token", access_token) \

.getOrCreate()

  

print("spark session ready")

```

  

### Code Example: Repartition & Compress CSV

  

```python

from pyspark.sql.functions import lit

import os

  

project_id = 'cse6242-489611'

bucket_name = 'team74'

year = "2015"

input_path = f"gs://{bucket_name}/merged/MERGED_{year}.csv"

output_drive_path = f"/content/drive/MyDrive/team74_merged_csv_processed/year={year}"

  

# Create output directory

if not os.path.exists(output_drive_path):

os.makedirs(output_drive_path)

  

# Read CSV with inferred schema

df = spark.read \

.option("header", "true") \

.option("inferSchema", "true") \

.csv(input_path)

  

# Add year column as partition key

df = df.withColumn("year", lit(year))

  

# Repartition into 46 files and save with gzip compression

df.repartition(46) \

.write \

.mode("overwrite") \

.option("header", "true") \

.option("compression", "gzip") \

.csv(output_drive_path)

  

print(f"save done: {output_drive_path}")

```

  

### Output Structure

```

/content/drive/MyDrive/team74_merged_csv_processed/

├── year=2015/

│ ├── part-00000-*.csv.gz

│ ├── part-00001-*.csv.gz

│ └── ... (46 partitions)

├── year=2016/

│ └── ... (46 partitions)

└── year=2017/

└── ... (46 partitions)

```

  

---

  

## Stage 01: Silver Layer Creation with Feature Engineering

  

### Purpose

Create a standardized, partitioned Parquet dataset with enriched features from multiple sources (BTS, FAA, NOAA).

  

### Key Components

- **Input:** Repartitioned CSV files from Stage 00 (2015-2017 data)

- **Processing:**

- Schema inference and multi-year data union

- Feature selection (17 core ML features)

- Filtering for specific year (2017 in example)

- Partitioning by MONTH for query optimization

- **Output:** Parquet files partitioned by MONTH

  

### Code Example: Multi-Year CSV Reading

  

```python

from pyspark.sql import SparkSession

  

spark = SparkSession.builder \

.appName("FlightDelay Merged file Corr") \

.config("spark.driver.memory", "10g") \

.config("spark.executor.memory", "10g") \

.config("spark.sql.shuffle.partitions", "46") \

.getOrCreate()

  

# Mount Google Drive

from google.colab import drive

drive.mount('/content/drive')

  

base_path = "/content/drive/MyDrive/team74_merged_csv_processed"

  

# Read all years with wildcard pattern

df = spark.read \

.option("header", "true") \

.option("inferSchema", "true") \

.csv(f"{base_path}/year=*/*.csv.gz")

  

print("Schema:")

df.printSchema()

```

  

### Code Example: Feature Selection & Parquet Partitioning

  

```python

from pyspark.sql import functions as F

  

base_path = "/content/drive/MyDrive/team74_merged_csv_processed"

output_base_path = "/content/drive/MyDrive/team74_silver_layer_v2"

fact_flights_path = f"{output_base_path}/fact_flights"

  

# Read all years

df = spark.read \

.option("header", "true") \

.option("inferSchema", "true") \

.csv(f"{base_path}/year=*/*.csv.gz")

  

# Define 17 ML features: BTS core + FAA operations + NOAA weather

feature_cols = [

'YEAR', 'MONTH', 'DAY_OF_WEEK',

'OP_UNIQUE_CARRIER', 'ORIGIN', 'DEST', 'DISTANCE',

'DEP_DELAY', 'ARR_DELAY', 'CANCELLED',

'ORIGIN_Total_Operations', 'DEST_Total_Operations',

'ORIGIN_PRCP', 'ORIGIN_AWND', 'ORIGIN_TMAX',

'DEST_PRCP', 'DEST_AWND'

]

  

# Filter for 2017 and select features

df_silver_v2 = df.filter(F.col("YEAR") == 2017).select(*feature_cols)

  

# Save as Parquet partitioned by MONTH for efficient querying

df_silver_v2.write \

.mode("overwrite") \

.partitionBy("MONTH") \

.parquet(fact_flights_path)

  

print(f"Success! Data saved to: {fact_flights_path}")

```

  

### Feature Set Composition

- **Temporal:** YEAR, MONTH, DAY_OF_WEEK

- **Flight Info:** OP_UNIQUE_CARRIER, ORIGIN, DEST, DISTANCE

- **Delays:** DEP_DELAY, ARR_DELAY, CANCELLED

- **FAA Operations:** ORIGIN_Total_Operations, DEST_Total_Operations

- **NOAA Weather:** ORIGIN_PRCP, ORIGIN_AWND, ORIGIN_TMAX, DEST_PRCP, DEST_AWND

  

### Output Structure

```

/content/drive/MyDrive/team74_silver_layer_v2/

└── fact_flights/

├── MONTH=1/

│ ├── part-00000-*.parquet

│ └── ...

├── MONTH=2/

│ └── ...

...

└── MONTH=12/

└── ...

```

  

---

  

## Stage 02: PostgreSQL Data Loading Configuration

  

### Purpose

Establish PostgreSQL database infrastructure and prepare connection details for downstream BI tools (Tableau).

  

### Environment Setup

  

| Parameter | Value |

|-----------|-------|

| Database Type | PostgreSQL |

| Database Name | `flight_db2` |

| User | `postgres` |

| Port | `5432` |

| Host | `localhost` |

  

### Installation: PostgreSQL JDBC Driver

  

The JDBC driver enables Tableau and other BI tools to connect directly to PostgreSQL.

  

**Steps:**

  

1. **Download** the PostgreSQL JDBC driver from [jdbc.postgresql.org](https://jdbc.postgresql.org/)

- Example: `postgresql-42.7.3.jar`

  

2. **Install to system driver folder:**

- **Windows:** `C:\Program Files\Tableau\Drivers`

- **Mac:** `~/Library/Tableau/Drivers`

- **Linux:** `/opt/tableau/tableau_driver/jdbc`

  

3. **Restart Tableau Desktop** for changes to take effect

  

### Connection String

  

```

Server: localhost

Port: 5432

Database: flight_db2

Username: postgres

Password: [YOUR_PASSWORD]

```

  

### SQL View: Target Encoding & Top-N Grouping

  

To optimize BI performance with high-cardinality categorical fields (e.g., 300+ airports), create a SQL view that performs Top-N grouping and target encoding:

  

```sql

CREATE OR REPLACE VIEW v_flight_analysis AS

WITH TopAirports AS (

SELECT "ORIGIN"

FROM fact_flights

GROUP BY "ORIGIN"

ORDER BY COUNT(*) DESC

LIMIT 10

),

TopCarriers AS (

SELECT "OP_UNIQUE_CARRIER"

FROM fact_flights

GROUP BY "OP_UNIQUE_CARRIER"

ORDER BY COUNT(*) DESC

LIMIT 5

)

SELECT

f.*,

CASE

WHEN f."ORIGIN" IN (SELECT "ORIGIN" FROM TopAirports)

THEN f."ORIGIN"

ELSE 'Others'

END AS origin_grouped,

AVG(f."ARR_DELAY") OVER(PARTITION BY f."ORIGIN") AS origin_target_enc

FROM fact_flights f;

```

  

**Benefits:**

- Reduces Tableau filter cardinality from 300+ to 11 (10 top + Others)

- Pre-computes target encoding (ARR_DELAY by origin) for faster exploration

- Improves dashboard responsiveness without losing information

  

---

  

## Stage 03: Parquet to PostgreSQL Data Transfer

  

### Purpose

Transfer Parquet files from local/cloud storage into PostgreSQL fact and dimension tables.

  

### Prerequisites

- PostgreSQL database and `flight_db2` schema created

- Parquet files downloaded to local machine

- Python environment with `pandas`, `sqlalchemy`, `pyarrow`

  

### Code Example: Connection & Data Upload

  

```python

import pandas as pd

import os

from sqlalchemy import create_engine, text

  

# Create PostgreSQL connection

engine = create_engine('postgresql://postgres:password@localhost:5432/flight_db2')

  

# Define local Parquet directory

base_path = r'C:\data\team74_silver_layer_v2'

  

def upload_to_sql(folder_name, table_name):

"""Upload Parquet folder to PostgreSQL table"""

path = os.path.join(base_path, folder_name)

if not os.path.exists(path):

print(f"No folder: {folder_name} (skipping)")

return

  

print(f"--- {folder_name} Start to read ---")

try:

# PyArrow reads all Parquet files in folder (handles partitions)

df = pd.read_parquet(path, engine='pyarrow')

print(f"Loaded: {len(df):,} rows")

print(f"PostgreSQL '{table_name}' being created and transferring...")

# Use smaller chunksize to manage memory with large datasets

df.to_sql(table_name, engine, if_exists='replace',

index=False, chunksize=50000)

print(f"{table_name} load successful!\n")

except Exception as e:

print(f"{table_name} load error: {e}")

  

# Example: Drop and recreate fact_flights table

with engine.connect() as conn:

conn.execute(text("DROP TABLE IF EXISTS fact_flights CASCADE"))

conn.commit()

  

# Upload fact table (5.7M rows, partitioned by MONTH)

upload_to_sql('fact_flights', 'fact_flights')

```

  

### Upload Strategy

  

| Table | Source Path | Rows | Approach |

|-------|------------|------|----------|

| `fact_flights` | `fact_flights/` | 5.7M | Partitioned by MONTH; use chunksize=50000 |

| `dim_carrier` | `dim_carrier/` | ~30 | Small table; single chunk |

| `dim_weather` | `dim_weather/` | ~1M | Medium table; moderate chunking |

  

### Key Considerations

- **Chunking:** Use `chunksize=50000` to balance memory usage with I/O speed

- **Partitions:** PyArrow automatically merges partitioned Parquet directories

- **Constraints:** Add primary/foreign keys after data loads for BI optimization

- **Indices:** Create indices on frequently-queried columns (ORIGIN, DEST, CARRIER)

  

---

  

## Stage 04: Tableau Visualization & Validation

  

### Purpose

Connect Tableau to PostgreSQL and create exploratory dashboards for data validation and feature correlation analysis.

  

### Tableau Connection Setup

  

1. **Open Tableau Desktop**

2. **Select "PostgreSQL"** from Data Sources

3. **Enter connection details:**

```

Server: localhost

Port: 5432

Database: flight_db2

Username: postgres

Password: [YOUR_PASSWORD]

```

4. **Verify JDBC Driver** is installed (see Stage 02)

  

### Recommended Dashboard Views

  

#### 1. Feature Correlation Dashboard

- **Purpose:** Identify multicollinearity and important predictors

- **Dimensions:** Numerical features (DISTANCE, ARR_DELAY, DEP_DELAY, weather vars)

- **Viz Type:** Heatmap correlation matrix

- **Data Source:** `fact_flights` or `v_flight_analysis` view

  

#### 2. Categorical Feature Distribution

- **Purpose:** Understand high-cardinality field behavior

- **Dimensions:** origin_grouped, OP_UNIQUE_CARRIER (top 5)

- **Measures:** Count of flights, Avg ARR_DELAY

- **Viz Type:** Bar charts, pie charts

- **Query Optimization:** Use `v_flight_analysis` view with pre-grouped origins

  

#### 3. Temporal Patterns

- **Purpose:** Detect seasonal/weekly delay patterns

- **Dimensions:** MONTH, DAY_OF_WEEK

- **Measures:** Avg ARR_DELAY, Cancelled %, Departure Delay

- **Viz Type:** Line chart (trend by month), heatmap (day x month)

  

#### 4. Route Performance

- **Purpose:** Identify problematic origin-destination pairs

- **Dimensions:** ORIGIN, DEST

- **Measures:** Avg ARR_DELAY, On-time rate, Flight count

- **Viz Type:** Matrix (top 20 origin-dest combinations)

  

### Tableau Test Artifacts

- `04_tableau_test.md` - Dashboard design and test results

- `04_tableau_test.pdf` - Correlation heatmap and distribution charts

  

---

  

## End-to-End Data Flow

  

```

┌─────────────────────────────────────────────────────────────────┐

│ UPSTREAM: Data Preparation & Feature Engineering │

│ (01_data_clean, 01_FAA, 02_NOAA, 01_progress) │

└──────────────────────────┬──────────────────────────────────────┘

│

▼

┌──────────────────────────────────────┐

│ 00: CSV Repartitioning (GCS → Drive) │

│ - Read merged CSVs from GCS │

│ - Repartition to 46 files │

│ - Compress with gzip │

└──────────────────┬───────────────────┘

│

▼

┌──────────────────────────────────────┐

│ 01: Silver Layer Creation (Parquet) │

│ - Multi-year union (2015-2017) │

│ - Feature selection (17 features) │

│ - Partition by MONTH │

└──────────────────┬───────────────────┘

│

▼

┌──────────────────────────────────────┐

│ 02: PostgreSQL Configuration │

│ - JDBC driver setup │

│ - Connection string config │

│ - SQL views for optimization │

└──────────────────┬───────────────────┘

│

▼

┌──────────────────────────────────────┐

│ 03: Parquet → PostgreSQL Transfer │

│ - Read Parquet with PyArrow │

│ - Load to fact_flights table │

│ - Create indices & constraints │

└──────────────────┬───────────────────┘

│

▼

┌──────────────────────────────────────┐

│ 04: Tableau BI & Validation │

│ - Connect Tableau to PostgreSQL │

│ - Build exploratory dashboards │

│ - Validate data & correlations │

└──────────────────────────────────────┘

```

  

---

  

## Technical Architecture

  

### Storage & Compute Strategy

  

| Layer | Format | Location | Purpose |

|-------|--------|----------|---------|

| **Raw** | CSV (gzip) | Google Cloud Storage | Sourcing merged flight data |

| **Bronze** | CSV (repartitioned) | Google Drive | Stage 00 output |

| **Silver** | Parquet (partitioned) | Google Drive | Stage 01 output; feature-ready |

| **Gold** | PostgreSQL Tables | Local PostgreSQL | Optimized for BI queries |

  

### Key Technologies

- **PySpark:** Large-scale data processing (Stages 00-01)

- **Pandas + SQLAlchemy:** Data transfer (Stage 03)

- **PostgreSQL:** OLAP database for BI (Stage 02-04)

- **Tableau:** Business intelligence & visualization (Stage 04)

  

### Performance Optimization Techniques

  

1. **Repartitioning (Stage 00):** 46 partitions enable parallel CSV reading in Stage 01

2. **Parquet Compression:** Reduces storage & I/O vs raw CSV

3. **MONTH Partitioning (Stage 01):** Pruning in Tableau queries (e.g., "Q3 2017")

4. **SQL Views (Stage 02):** Top-N grouping + target encoding reduces cardinality

5. **Chunked Loading (Stage 03):** Prevents memory overflow when loading 5.7M rows

6. **Database Indices:** Added on ORIGIN, DEST, MONTH for fast filtering

  

---

  

## File Manifest

  

| File | Stage | Purpose |

|------|-------|---------|

| `00_team74_repartition_v2_merged_csv_into_gz_in_colab.ipynb` | 00 | GCS read → repartition → gzip save |

| `00_team74_repartition_v2_merged_csv_into_gz_in_colab.pdf` | 00 | Notebook export (reference) |

| `01_team74_silver_layer_v2_parquet_with_more_features_colab.ipynb` | 01 | Multi-year union → feature selection → Parquet |

| `01_team74_silver_layer_v2_parquet_with_more_features_colab.pdf` | 01 | Notebook export (reference) |

| `02_data_loading_postgreSQL.md` | 02 | PostgreSQL setup & JDBC config |

| `02_data_loading_postgreSQL.pdf` | 02 | Markdown export (reference) |

| `03_team74_parquet_to_postgres_DB_local.ipynb` | 03 | Parquet → PostgreSQL transfer |

| `04_tableau_test.md` | 04 | Tableau dashboard & correlation tests |

| `04_tableau_test.pdf` | 04 | Markdown export (reference) |

| `05_model_viz.md` | 04 | Integrated pipeline summary |

| `readme.md` | Summary | High-level overview |

| `readme_v2.md` | Summary | Detailed technical guide (this file) |

  

---

  

## Quick Start Guide

  

### For Data Engineers

1. **Configure GCS access** in Stage 00 notebook

2. **Run Stage 00** to repartition merged CSVs (outputs 46 files/year)

3. **Run Stage 01** to create Silver Parquet with features

4. **Download Parquet** to local machine

  

### For Analytics/BI Engineers

1. **Setup PostgreSQL** with Stage 02 configuration

2. **Run Stage 03** to transfer Parquet → PostgreSQL

3. **Configure Tableau** with JDBC driver (Stage 02)

4. **Connect Tableau** to PostgreSQL and build dashboards (Stage 04 reference)

  

### For ML Engineers

- **Use Silver Parquet** (Stage 01) directly for model training

- Features include: temporal, categorical, numerical, weather signals

- Target variable: ARR_DEL15 (0/1 classification)

- Dataset: ~5.7M flights across 12 months of 2017

  

---

  

## Troubleshooting

  

| Issue | Solution |

|-------|----------|

| GCS authentication fails | Re-run `auth.authenticate_user()` and refresh credentials |

| Parquet files too large | Reduce partitions or filter by MONTH before reading |

| PostgreSQL connection refused | Verify PostgreSQL service running; check credentials |

| Tableau cannot find JDBC driver | Restart Tableau after placing .jar in Drivers folder |

| Memory error during Stage 03 | Reduce chunksize (e.g., 25000 instead of 50000) |

  

---

  

## References & Related Work

  

- **Upstream Data Cleaning:** `01_data_clean`, `01_FAA`, `02_NOAA`

- **Modeling & Validation:** `01_progress` (feature engineering & model comparison)

- **Parallel Scripts:** `04_script` (batch processing utilities)

- **Visualization:** `04_tableau_test.md` (dashboard designs)

  

---

  

**Last Updated:** 2026-04-18

**Status:** Production-Ready Pipeline

**Maintainers:** Team 74 (CSE6242 Spring 2026)