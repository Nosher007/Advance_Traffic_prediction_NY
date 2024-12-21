# Automated Traffic Volume Analysis with PySpark

This repository contains a Jupyter Notebook leveraging PySpark to analyze traffic volume data. The project processes traffic volume datasets and provides schema inspection, initial data exploration, and advanced analysis of traffic patterns.

## Contents of the Notebook

The notebook includes the following steps:

### 1. Importing Libraries

```python
from pyspark.sql import SparkSession
from pyspark.sql.types import StructType, StructField, IntegerType, StringType
```

The required PySpark modules are imported to set up the Spark environment and define schema details for the dataset.

### 2. Spark Session Initialization

```python
spark = SparkSession.builder \
    .appName("Traffic Volume Analysis") \
    .getOrCreate()
```

A Spark session is initialized to facilitate distributed data processing.

### 3. Loading the Dataset

```python
data_path = 'Automated_Traffic_Volume_Counts_2024.csv'
data_df = spark.read.option('header', 'true').csv(data_path)
```

The dataset is loaded using Spark's CSV reader. Ensure the dataset path is correctly specified.

### 4. Schema Inspection

```python
data_df.printSchema()
```

#### Output:

```
root
 |-- RequestID: string (nullable = true)
 |-- Boro: string (nullable = true)
 |-- Yr: string (nullable = true)
 |-- M: string (nullable = true)
 |-- D: string (nullable = true)
 |-- HH: string (nullable = true)
 |-- MM: string (nullable = true)
 |-- Vol: string (nullable = true)
 |-- SegmentID: string (nullable = true)
 |-- WktGeom: string (nullable = true)
 |-- street: string (nullable = true)
```

### 5. Initial Data Exploration

```python
data_df.show(5)
```

#### Output:

A preview of the first five rows of the dataset:

| RequestID | Boro    | Vol | SegmentID | WktGeom                 | street         | fromSt              | toSt      | Direction |
|-----------|---------|-----|-----------|-------------------------|----------------|---------------------|-----------|-----------|
| 32970     | Queens  | 0   | 149701    | POINT (997407.099...)  | PULASKI BRIDGE | Newtown Creek Shore | Dead end  | NB        |
| 32970     | Queens  | 1   | 149701    | POINT (997407.099...)  | PULASKI BRIDGE | Newtown Creek Shore | Dead end  | NB        |

### 6. Advanced Analysis

#### Popular Routes (Departure -> Destination)

```python
popular_routes = data_df.groupBy('fromSt', 'toSt').agg(
    count('*').alias('Route_frequency'),
    sum('Vol').alias('Total_Volume')
).orderBy('Route_frequency', ascending=False)

popular_routes.show(10)
```

#### Output:

| fromSt               | toSt                 | Route_frequency | Total_Volume   |
|----------------------|----------------------|-----------------|----------------|
| Dead End            | Dead end            | 71389           | 1.3382925E7    |
| BOROUGH BOUNDARY    | BODY OF WATER       | 17593           | 5107796.0      |

#### Departure Trends

```python
departure_trends = data_df.groupBy("fromSt").agg(
    count("*").alias("Total_departures"),
    sum("Vol").alias("Total_Volume")
).orderBy("Total_departures", ascending=False)

departure_trends.show(10)
```

#### Output:

| fromSt               | Total_departures | Total_Volume   |
|----------------------|------------------|----------------|
| Dead End            | 195077           | 3.3134959E7    |
| BOROUGH BOUNDARY    | 17751            | 5151806.0      |

#### Destination Trends

```python
destination_trends = data_df.groupBy("toSt").agg(
    count("*").alias("Total_arrivals"),
    sum("Vol").alias("Total_Volume")
).orderBy("Total_arrivals", ascending=False)

destination_trends.show(10)
```

#### Output:

| toSt                 | Total_arrivals   | Total_Volume   |
|----------------------|------------------|----------------|
| Dead end            | 202638           | 3.79607387E7   |
| BODY OF WATER       | 32852            | 9315257.0      |

### Inflow and Outflow Hubs

```python
hubs = traffic_discrepancies.withColumn(
    "hub_type",
    when(col("net_flow") > 0, "Outflow Hub")
    .when(col("net_flow") < 0, "Inflow Hub")
    .otherwise("Balanced")
)

hubs.filter(hubs["hub_type"] == "Outflow Hub").show(10)
hubs.filter(hubs["hub_type"] == "Inflow Hub").show(10)
```

#### Output:

- Outflow Hubs: Locations where departures exceed arrivals.
- Inflow Hubs: Locations where arrivals exceed departures.

### Recommendations

```python
for row in hubs.collect():
    if row['hub_type'] == "Outflow Hub":
        print(f"Increase green signal duration for {row['street']}")
    elif row['hub_type'] == "Inflow Hub":
        print(f"Decrease green signal duration for {row['street']}")
```

### Output:

Example recommendations for traffic signal adjustments:

- Increase green signal duration for Dead End
- Decrease green signal duration for Yellowstone Blvd

## Requirements

To run this notebook, ensure you have the following:

- Python 3.8 to 3.10 (for TensorFlow compatibility)
- Apache Spark
- PySpark library
- Jupyter Notebook or Jupyter Lab
- Traffic volume dataset (`Automated_Traffic_Volume_Counts_2024.csv`)

## How to Use

1. Clone this repository:

   ```bash
   git clone https://github.com/yourusername/traffic-volume-analysis.git
   cd traffic-volume-analysis
   ```

2. Install dependencies:

   ```bash
   pip install pyspark tensorflow
   ```

3. Place the traffic volume dataset in the repository directory.

4. Run the notebook:

   ```bash
   jupyter notebook main.ipynb
   ```

## License

This project is licensed under the MIT License. Feel free to use, modify, and distribute it as per the license terms.

## Acknowledgements

- Apache Spark for providing the powerful distributed data processing framework.
- The dataset provider for making the traffic volume data available.
