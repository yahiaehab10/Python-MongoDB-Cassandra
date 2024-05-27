# Cassandra

## Table of Contents
- [Introduction](#introduction)
- [Installation](#installation)
- [Usage](#usage)
- [Project Structure](#project-structure)
- [Data Collection and Preprocessing](#data-collection-and-preprocessing)
- [Data Analysis](#data-analysis)
- [Visualization](#visualization)
- [License](#license)

## Introduction
This project demonstrates the use of Apache Cassandra for data storage and analysis. The dataset used includes taxi trip data and taxi zone geographic data. The project involves:
- Connecting to a Cassandra database.
- Loading and preprocessing data.
- Storing and retrieving data from a Cassandra database.
- Performing data analysis and visualization.

## Installation
To install the required dependencies, use the following commands:
```sh
pip install cassandra-driver
pip install pandas
pip install matplotlib
```

## Usage
1. Clone the repository.
2. Install the required dependencies as mentioned above.
3. Run the `Cassandra.ipynb` notebook to execute the code.

## Project Structure
- `Cassandra.ipynb`: The main Jupyter Notebook containing all the code for data loading, preprocessing, storing, retrieving, and analyzing data using Cassandra.

## Data Collection and Preprocessing
The project starts with loading and preprocessing the taxi trip data and taxi zone geographic data using Pandas.

### Code Example:
```python
import pandas as pd

# Read data
taxi_trip_data = pd.read_csv('./datasets/taxi_trip_data.csv', nrows=20_000)
taxi_zone_geo_data = pd.read_csv('./datasets/taxi_zone_geo.csv')

# Drop unnecessary columns
taxi_trip_data.drop(columns=["store_and_fwd_flag", "rate_code", "total_amount"], inplace=True)

# Drop rows with missing essential details
essential_columns = [
    "vendor_id",
    "pickup_datetime",
    "dropoff_datetime",
    "passenger_count",
    "trip_distance",
    "payment_type",
    "fare_amount",
    "extra",
    "mta_tax",
    "tip_amount",
    "tolls_amount",
    "imp_surcharge",
    "pickup_location_id",
    "dropoff_location_id",
]
taxi_trip_data.dropna(subset=essential_columns, inplace=True)
taxi_zone_geo_data.dropna(inplace=True)

# Remove IDs in pickup location IDs that are not present in zone IDs
taxi_trip_data = taxi_trip_data[taxi_trip_data['pickup_location_id'].isin(taxi_zone_geo_data['zone_id'])]
```

## Data Analysis
After preprocessing, the data is stored in a Cassandra database, and various analyses are performed.

### Code Example:
```python
from cassandra.cluster import Cluster
from cassandra.auth import PlainTextAuthProvider
import json

# Load credentials and connect to Cassandra
cloud_config= {'secure_connect_bundle': './cassandra_needs/secure-connect-taxi-trip.zip'}
with open("./cassandra_needs/taxi_trip-token.json") as f:
    secrets = json.load(f)
CLIENT_ID = secrets["clientId"]
CLIENT_SECRET = secrets["secret"]

auth_provider = PlainTextAuthProvider(CLIENT_ID, CLIENT_SECRET)
cluster = Cluster(cloud=cloud_config, auth_provider=auth_provider)
session = cluster.connect()

# Verify connection
row = session.execute("select release_version from system.local").one()
if row:
  print(row[0])
else:
  print("An error occurred.")

# Insert data into Cassandra
query = "INSERT INTO taxi_trip_data.trips (...) VALUES (...)"
for index, row in taxi_trip_data.iterrows():
    session.execute(query, (row['column1'], row['column2'], ...))
```

## Visualization
The project includes various visualizations to analyze the data, such as the correlation between trip distance and tip amount.

### Code Example:
```python
import matplotlib.pyplot as plt

# Correlation between trip distance and tip amount
trip_distance_tip_amount = session.execute("SELECT * FROM trip_distance_tip_amount")
trip_distance_tip_amount = pd.DataFrame(trip_distance_tip_amount)
plt.scatter(trip_distance_tip_amount["trip_distance"], trip_distance_tip_amount["tip_amount"])
plt.xlabel("Trip Distance")
plt.ylabel("Tip Amount")
plt.title("Trip Distance vs Tip Amount")
plt.show()
```

## License
This project was managed and provided by the German International University.

# PyMongo

## Table of Contents
- [Introduction](#introduction)
- [Installation](#installation)
- [Usage](#usage)
- [Project Structure](#project-structure)
- [Data Collection and Preprocessing](#data-collection-and-preprocessing)
- [Data Analysis](#data-analysis)
- [Visualization](#visualization)
- [License](#license)

## Introduction
This project demonstrates the use of PyMongo for data manipulation and analysis. The dataset used includes taxi trip data and taxi zone geographic data. The project involves:
- Loading and preprocessing data.
- Storing and retrieving data from a MongoDB database.
- Performing data analysis and visualization.

## Installation
To install the required dependencies, use the following commands:
```sh
pip install pymongo
pip install pandas
pip install matplotlib
```

## Usage
1. Clone the repository.
2. Install the required dependencies as mentioned above.
3. Run the `PyMongo.ipynb` notebook to execute the code.

## Project Structure
- `PyMongo.ipynb`: The main Jupyter Notebook containing all the code for data loading, preprocessing, storing, retrieving, and analyzing data using MongoDB.

## Data Collection and Preprocessing
The project starts with loading and preprocessing the taxi trip data and taxi zone geographic data using Pandas.

### Code Example:
```python
import pandas as pd
from pymongo import MongoClient
import pymongo
from datetime import datetime

# Read data
taxi_trip_data = pd.read_csv('./datasets/taxi_trip_data.csv', nrows=20_000)
taxi_zone_geo_data = pd.read_csv('./datasets/taxi_zone_geo.csv')

# Drop unnecessary columns
taxi_trip_data.drop(columns=["store_and_fwd_flag", "rate_code", "total_amount"], inplace=True)

# Drop rows with missing essential details
essential_columns = [
    "vendor_id",
    "pickup_datetime",
    "dropoff_datetime",
    "passenger_count",
    "trip_distance",
    "payment_type",
    "fare_amount",
    "extra",
    "mta_tax",
    "tip_amount",
    "tolls_amount",
    "imp_surcharge",
    "pickup_location_id",
    "dropoff_location_id",
]
taxi_trip_data.dropna(subset=essential_columns, inplace=True)
taxi_zone_geo_data.dropna(inplace=True)

# Remove IDs in pickup location IDs that are not present in zone IDs
taxi_trip_data = taxi_trip_data[taxi_trip_data['pickup_location_id'].isin(taxi_zone_geo_data['zone_id'])]
```

## Data Analysis
After preprocessing, the data is stored in a MongoDB database, and various analyses are performed.

### Code Example:
```python
# Connect to MongoDB
client = MongoClient('mongodb://localhost:27017/')
db = client['taxi_trip_data']
collection = db['trips']

# Insert data into MongoDB
collection.insert_many(taxi_trip_data.to_dict('records'))

# Query data from MongoDB
data = collection.find({"passenger_count": {"$gte": 1}})
df = pd.DataFrame(list(data))
```

## Visualization
The project includes various visualizations to analyze the data, such as the number of trips per payment type and time of day, the average tip amount per passenger count, and the best locations for drivers to pick up passengers.

### Code Example:
```python
import matplotlib.pyplot as plt

# Number of trips per payment type and time of day
plt.figure(figsize=(10, 6))
df.groupby(['payment_type', df['pickup_datetime'].dt.hour]).size().unstack().T.plot(kind='bar', stacked=True)
plt.xlabel("Hour of the day")
plt.ylabel("Number of trips")
plt.title("Number of trips per payment type and time of day")
plt.legend(["Credit card", "Cash", "No charge", "Dispute", "Unknown", "Voided trip"])
plt.show()

# Average tip amount per passenger count
grouped = df.groupby("passenger_count")["tip_amount"].mean()
grouped.plot(kind='bar')
plt.xlabel("Number of passengers")
plt.ylabel("Average tip amount")
plt.title("Average tip amount per passenger count")
plt.show()
```

## License
This project was managed and provided by the German International University.
