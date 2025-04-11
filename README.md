# Data-Driven-Strategy-for-New-Ride-Hailing-Venture

![WhatsApp Image 2025-04-11 at 09 04 56-2](https://github.com/user-attachments/assets/c09e19c5-4bd7-4c62-a8b8-a1d8f5d80117)


### Problem Statement
A new ride-hailing startup aims to launch operations in New York City but lacks data-driven insights to optimize its service strategy. To compete effectively, the company must leverage Uber’s publicly available TLC (Taxi & Limousine Commission) trip data to:

i. Demand & Traffic Analysis – Identify peak demand periods and high-traffic zones to optimize driver allocation and reduce wait times.

ii. Revenue & Payment Trends – Analyze fare patterns by payment type (cash/credit) and passenger count to maximize profitability and pricing strategies.

iii. Scalable Data Infrastructure – Build a robust, real-time analytics pipeline to support dynamic decision-making as the business scales.

### Business Questions

1.	What are the peak demand hours and locations for ride demand? 
   
2.	What is the total fair by Vendor ID ? 
   
3.	What is the average tip amount by payment type ? 
   
4.	What are the peak demand periods and most profitable locations? -geo map 
   

### Key Metrics

•	Average Tip Amount: $1.9.


•	Average Trip Distance: 3.0.

•	Top Pickup Cluster.

•	Top dropoff Cluster.

•	Total Revenue: $1.6M

•  Average no of Trips: 100K

•	Average Fair Amount: 13.3.


### Methodology
The project followed a systematic approach to data modeling, transformation, and analysis using Google Cloud Platform (GCP) tools

### 1. Data Ingestion 
The uber trip data was gotten from (https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page)

### 2. Data Modeling:
- Drafted out my data model

<img width="949" alt="image" src="https://github.com/user-attachments/assets/f2765658-5c70-4a0f-9399-2b5725406478" />

- Went ahead to clean my data and create my transformative code that i will use in creating my pipeline in my jupiter notebook

here is the code:
    
    # 1. DATETIME DIMENSION 
    datetime_dim = (
        df[datetime_cols]
        .drop_duplicates()
        .assign(
            pick_hour=lambda x: x['tpep_pickup_datetime'].dt.hour,
            pick_day=lambda x: x['tpep_pickup_datetime'].dt.day,
            pick_month=lambda x: x['tpep_pickup_datetime'].dt.month,
            pick_year=lambda x: x['tpep_pickup_datetime'].dt.year,
            pick_weekday=lambda x: x['tpep_pickup_datetime'].dt.weekday,
            drop_hour=lambda x: x['tpep_dropoff_datetime'].dt.hour,
            drop_day=lambda x: x['tpep_dropoff_datetime'].dt.day,
            drop_month=lambda x: x['tpep_dropoff_datetime'].dt.month,
            drop_year=lambda x: x['tpep_dropoff_datetime'].dt.year,
            drop_weekday=lambda x: x['tpep_dropoff_datetime'].dt.weekday,
            datetime_id=lambda x: x.index
        )
        [['datetime_id', 'tpep_pickup_datetime', 'pick_hour', 'pick_day', 
          'pick_month', 'pick_year', 'pick_weekday', 'tpep_dropoff_datetime',
          'drop_hour', 'drop_day', 'drop_month', 'drop_year', 'drop_weekday']]
    )

    # 2. PASSENGER COUNT DIMENSION
    passenger_count_dim = (
        df[['passenger_count']]
        .drop_duplicates()
        .assign(passenger_count_id=lambda x: x.index)
        [['passenger_count_id', 'passenger_count']]
    )

    # 3. TRIP DISTANCE DIMENSION
    trip_distance_dim = (
        df[['trip_distance']]
        .drop_duplicates()
        .assign(trip_distance_id=lambda x: x.index)
        [['trip_distance_id', 'trip_distance']]
    )

    # 4. RATE CODE DIMENSION 
    rate_code_type = {
        1: "Standard rate",
        2: "JFK",
        3: "Newark",
        4: "Nassau or westchester",
        5: "Negotiated fare",
        6: "Group ride"
    }
    rate_code_dim = (
        df[['RatecodeID']]
        .drop_duplicates()
        .assign(
            rate_code_id=lambda x: x.index,
            rate_code_name=lambda x: x['RatecodeID'].map(rate_code_type)
        )
        [['rate_code_id', 'RatecodeID', 'rate_code_name']]
    )

    # 5. LOCATION DIMENSIONS 
    location_cols = ['pickup_longitude', 'pickup_latitude', 
                    'dropoff_longitude', 'dropoff_latitude']
    
    pickup_loc = (
        df[['pickup_longitude', 'pickup_latitude']]
        .drop_duplicates()
        .assign(pickup_location_id=lambda x: x.index)
    )
    
    dropoff_loc = (
        df[['dropoff_longitude', 'dropoff_latitude']]
        .drop_duplicates()
        .assign(dropoff_location_id=lambda x: x.index)
    )

    # 6. PAYMENT TYPE DIMENSION
    payment_type_name = {
        1: "Credit card",
        2: "Cash",
        3: "No charge",
        4: "Dispute",
        5: "Unknown",
        6: "Voided trip"
    }
    payment_type_dim = (
        df[['payment_type']]
        .drop_duplicates()
        .assign(
            payment_type_id=lambda x: x.index,
            payment_type_name=lambda x: x['payment_type'].map(payment_type_name)
        )
        [['payment_type_id', 'payment_type', 'payment_type_name']]
    )

    # 7. FACT TABLE
    fact_table = (
        df
        .merge(passenger_count_dim, on='passenger_count')
        .merge(trip_distance_dim, on='trip_distance')
        .merge(rate_code_dim, on='RatecodeID')
        .merge(
            pickup_loc[['pickup_longitude', 'pickup_latitude', 'pickup_location_id']],
            on=['pickup_longitude', 'pickup_latitude']
        )
        .merge(
            dropoff_loc[['dropoff_longitude', 'dropoff_latitude', 'dropoff_location_id']],
            on=['dropoff_longitude', 'dropoff_latitude']
        )
        .merge(
            datetime_dim[['tpep_pickup_datetime', 'tpep_dropoff_datetime', 'datetime_id']],
            on=['tpep_pickup_datetime', 'tpep_dropoff_datetime']
        )
        .merge(payment_type_dim, on='payment_type')
        [[
            'VendorID', 'datetime_id', 'passenger_count_id', 'trip_distance_id',
            'rate_code_id', 'store_and_fwd_flag', 'pickup_location_id',
            'dropoff_location_id', 'payment_type_id', 'fare_amount', 'extra',
            'mta_tax', 'tip_amount', 'tolls_amount', 'improvement_surcharge',
            'total_amount'
        ]]
    )

   

### 3. Connected to my Google Cloud Project (GCP):

• Created a new project

• Cteated a bucket 

• Uploaded my Uber data 

• Created a an instance in my compute engine 

• Connected my SHH (VM), and downloaded the necessary packages needed to start mage 

<img width="1013" alt="Pasted Graphic 24" src="https://github.com/user-attachments/assets/b455a6ef-03b0-4197-969e-8ae9d61b4a3a" />


### 4. Data Pipeline Architecture

•	Initialized a Mage project and configured firewall rules to expose port 6789 for the Mage UI.

• Pipeline Steps:

-Data Loader: Fetched raw data from GCS via URL.

-Transformer: Executed Python code (adapted from Jupyter Notebook) (check the above mentioed tranformation code

- Exporter: Pushed data to BigQuery using service account credentials (JSON key)

  ![b1155375-7a8f-469e-9848-27a3076cf741](https://github.com/user-attachments/assets/4558ab52-3cfe-4b34-af81-cd966c17138e)



### 5. Data Warehouse & Analytics

• BigQuery Setup:

- Created a dataset (uber_data_engineering) and tables via Mage’s exporter.
- SQL Queries:

  i. Analytics Table Creation

  <img width="283" alt="image" src="https://github.com/user-attachments/assets/2e13ce6b-6e67-4283-92ea-2f486e476471" />



Here is the code:

CREATE OR REPLACE TABLE `my-test-project-kimenendu.uber_data_engineering_kimenendu.tbl_analytics` AS
SELECT
    f.VendorID,
    d.tpep_pickup_datetime,
    d.tpep_dropoff_datetime,
    p.passenger_count,
    t.trip_distance,
    r.rate_code_name,
    pick.pickup_latitude,
    pick.pickup_longitude,
    drop.dropoff_latitude,
    drop.dropoff_longitude,
    py.payment_type_name,  
    f.fare_amount,
    f.extra,
    f.mta_tax,
    f.tip_amount,
    f.tolls_amount,
    f.improvement_surcharge  
FROM
    `my-test-project-kimenendu.uber_data_engineering_kimenendu.fact_table` f
JOIN `my-test-project-kimenendu.uber_data_engineering_kimenendu.datetime_dim` d 
    ON f.datetime_id = d.datetime_id
JOIN `my-test-project-kimenendu.uber_data_engineering_kimenendu.passenger_count_dim` p 
    ON p.passenger_count_id = f.passenger_count_id
JOIN `my-test-project-kimenendu.uber_data_engineering_kimenendu.trip_distance_dim` t 
    ON t.trip_distance_id = f.trip_distance_id
JOIN `my-test-project-kimenendu.uber_data_engineering_kimenendu.rate_code_dim` r 
    ON r.rate_code_id = f.rate_code_id
JOIN `my-test-project-kimenendu.uber_data_engineering_kimenendu.pickup_location_dim` pick 
    ON pick.pickup_location_id = f.pickup_location_id
JOIN `my-test-project-kimenendu.uber_data_engineering_kimenendu.dropoff_location_dim` drop 
    ON drop.dropoff_location_id = f.dropoff_location_id
JOIN `my-test-project-kimenendu.uber_data_engineering_kimenendu.payment_type_dim` py 
    ON py.payment_type_id = f.payment_type_id; 

ii. This SQL shows the Average fare amount by VendorID
SELECT 
    VendorID, 
    AVG(fare_amount) AS avg_fare_amount
FROM `my-test-project-kimenendu.uber_data_engineering_kimenendu.fact_table` 
GROUP BY VendorID;

iii. This SQL Query shows the Total fare amount by VendorID
SELECT 
    VendorID, 
    SUM(fare_amount) AS total_fare_amount
FROM `my-test-project-kimenendu.uber_data_engineering_kimenendu.fact_table` 
GROUP BY VendorID;

iv. This SQL Query shows Total tip amount by payment type
SELECT 
    b.payment_type_name, 
    SUM(a.tip_amount) AS total_tip_amount 
FROM `my-test-project-kimenendu.uber_data_engineering_kimenendu.fact_table` a
JOIN `my-test-project-kimenendu.uber_data_engineering_kimenendu.payment_type_dim` b
    ON a.payment_type_id = b.payment_type_id
GROUP BY b.payment_type_name;

v. This SQL Query shows Average tip amount by payment type
SELECT 
    b.payment_type_name, 
    AVG(a.tip_amount) AS avg_tip_amount 
FROM `my-test-project-kimenendu.uber_data_engineering_kimenendu.fact_table` a
JOIN `my-test-project-kimenendu.uber_data_engineering_kimenendu.payment_type_dim` b
    ON a.payment_type_id = b.payment_type_id
GROUP BY b.payment_type_name;

vi. This SQL Query shows Top 10 pickup locations based on number of trips
SELECT 
    pl.pickup_latitude,
    pl.pickup_longitude,
    COUNT(*) AS trip_count
FROM `my-test-project-kimenendu.uber_data_engineering_kimenendu.fact_table` f
JOIN `my-test-project-kimenendu.uber_data_engineering_kimenendu.pickup_location_dim` pl
    ON f.pickup_location_id = pl.pickup_location_id
GROUP BY pl.pickup_latitude, pl.pickup_longitude
ORDER BY trip_count DESC
LIMIT 10;

vii. This SQL Query shows Total number of trips by passenger count
SELECT 
    pc.passenger_count,
    COUNT(*) AS trip_count
FROM `my-test-project-kimenendu.uber_data_engineering_kimenendu.fact_table` f
JOIN `my-test-project-kimenendu.uber_data_engineering_kimenendu.passenger_count_dim` pc
    ON f.passenger_count_id = pc.passenger_count_id
GROUP BY pc.passenger_count
ORDER BY pc.passenger_count;

<img width="311" alt="Pasted Graphic" src="https://github.com/user-attachments/assets/92c756bf-c49d-4918-b565-49858954ff4c" />


viii. This SQL Query shows Average fare amount by hour of the day 
SELECT 
    EXTRACT(HOUR FROM PARSE_TIMESTAMP('%Y-%m-%dT%H:%M:%S', dt.tpep_pickup_datetime)) AS hour_of_day,
    AVG(f.fare_amount) AS avg_fare_amount
FROM `my-test-project-kimenendu.uber_data_engineering_kimenendu.fact_table` f
JOIN `my-test-project-kimenendu.uber_data_engineering_kimenendu.datetime_dim` dt
    ON f.datetime_id = dt.datetime_id
GROUP BY hour_of_day
ORDER BY hour_of_day;

### 4. Data Visualization & Dashboard Insights (Looker Studio)

Dashboard Overview
The Looker Studio dashboard synthesizes key metrics from tbl_analytics into actionable visuals. 

Key sections:

1. Filters Panel
Interactive Controls:
Vendor ID, Payment Type, Rate Code, Trip Distance

Purpose: Dynamically filter data across all visuals (e.g., compare revenue by vendor or payment type).

2. Summary Metrics
Metric	Value	Business Insight
Total Revenue	$1.6M	Baseline for financial performance.
Total Trips	100K	Demand volume benchmark.
Avg Trip Distance	3.0 mi	Optimize driver allocation for short trips.
3. Geographic Analysis
Pickup/Drop-off Heatmaps:

>Data Source: Custom fields for latitude/longitude from pickup_location_dim and dropoff_location_dim.

Key Findings:

> Top Pickup Cluster: Coordinates near 40.774, -73.873 (Manhattan) had 25% of all trips.

> Drop-off Spread: Wider distribution across states (IDAHO, COLORADO), suggesting interstate trips.

Pickup Locations Map Example of high-density pickup zones.

4. Payment & Revenue Trends
Charts:

>Avg Tip by Payment Type: Credit cards (
2.50)outperformedcash(0.80).

>Fare Trends by Hour: Peaks at 8 AM and 5 PM (aligns with commute times).

>Funnel by Trip Distance: 60% of trips were under 3 miles (highlighting urban demand).

5. Rate Code Breakdown
Legend:
Standard rate (70% of trips), JFK (15%), Negotiated fare (10%).

### Actionable Insight: 

> Prioritize marketing for high-margin rate codes (e.g., JFK airport trips).

### Technical Implementation
Custom Fields: Created calculated fields in Looker Studio to:
-Aggregate coordinates for map visuals.
-Normalize payment types for consistent labeling.

### Dynamic Filtering:
Linked all visuals to the filter panel for cross-analysis.


















