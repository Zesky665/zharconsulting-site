---
dg-publish: true
tags:
  - Snowflake
  - Mage
  - AI
  - DE_ZOOMCAMP_2024
  - Data_Loading
  - Docker-Compose
  - Docker
  - Python
  - Week_2
  - yaml
---

## What is Mage?

Mage is an orchestration tool in the same vein as Airflow. What makes it different is that unlike Airflow is a general orchestration tool which is often used to run data pipelines, Mage is an orchestration tool purpose built for data pipelines. 

Mage makes writing data pipelines easy and fun. 

It accomplishes this by making the development experience seamless by having most of the common building blocks built in, so that developers can focus on business logic instead of reinventing the wheel with every pipeline. 

For example there are built in [data loading templates](https://docs.mage.ai/design/data-loading) for all the popular data sources.

## What is an ETL?

**ETL** stands for **E**xtract **T**ransform **L**oad, it represents a type of data pipeline where the data is extracted from the source, transformed AKA cleaned and then loaded into a data warehouse. 

**ETL** is the most common type of data pipeline, the other being **ELT** (**E**xtract - **L**oad - **T**ransform) where data is loaded raw and transformed inside of the data warehouse. 

**ETL** is best used when you know what the shape of the needed data looks like, because it does not store all of the incoming data. 

If you don't know what data you might need in the future or if the data needs are likely to change, the **ELT** method is preferable because once the data is loaded in the data warehouse you can manipulate it however you like and still have the raw data available if you decide that you needed one of the values you previously thought was irrelevant.  

## How to run locally?

You can setup a local Mage instance by copying the official template [repo](https://github.com/mage-ai/compose-quickstart). 

I'll add the commands here for easy access. 

```Shell
git clone https://github.com/mage-ai/compose-quickstart.git mage-quickstart \
&& cd mage-quickstart \
&& cp dev.env .env && rm dev.env \
&& docker compose up
```

Since we are going to be using it with Snowflake, we're going to also install the optional mage-ai/Snowflake package. 

```Shell
pip install "mage-ai[snowflake]"
```

You can also add the following to your `requirements.txt` file. 

```Shell
snowflake-connector-python==3.7.0
```


## How to configure?

Mage config are stored in the `io_config.yaml` file in the project directory. 
This file has default setups for every supported service, just add your own values.
I am going to add mine as environmental variables. 

```yaml
# Snowflake
SNOWFLAKE_USER: "{{ env_var('SNOWFLAKE_USERNAME') }}" # required
SNOWFLAKE_PASSWORD: "{{ env_var('SNOWFLAKE_PASSWORD') }}" # required
SNOWFLAKE_ACCOUNT: "{{ env_var('SNOWFLAKE_REGION') }}" # required
SNOWFLAKE_DEFAULT_WH: "{{ env_var('SNOWFLAKE_WAREHOUSE') }}" # required
SNOWFLAKE_DEFAULT_DB: "{{ env_var('SNOWFLAKE_DB') }}" # required
SNOWFLAKE_ROLE: "{{ env_var('SNOWFLAKE_ROLE') }}" # required
```

## How to create a pipeline?

After firing up the mage instance via docker compose, navigate to the http://localhost:6789/.
Once there click on `+ New Pipeline` and then on `Standard(batch)`.
![New Pipeline>Standard](https://i.imgur.com/HKAwW7j.png)

After clicking `Standard(batch)` you will be redirected to the new pipeline edit page.
## How to Load Data?

To load data from a file, you will need one of the built-in templates `Data Loder > Python > Local file`.

![Data Loader > Python > Local file](https://i.imgur.com/ce9oAYT.png)

The following code will be generated. 

```Python
from mage_ai.io.file import FileIO
if 'data_loader' not in globals():
from mage_ai.data_preparation.decorators import data_loader
if 'test' not in globals():
from mage_ai.data_preparation.decorators import test

@data_loader
def load_data_from_file(*args, **kwargs):
"""
Template for loading data from filesystem.
Load data from 1 file or multiple file directories.

For multiple directories, use the following:
FileIO().load(file_directories=['dir_1', 'dir_2'])

Docs: https://docs.mage.ai/design/data-loading#fileio
"""

filepath = 'path/to/your/file.csv'
  
return FileIO().load(filepath)

@test

def test_output(output, *args) -> None:
"""
Template code for testing the output of the block.
"""

assert output is not None, 'The output is undefined'
```
This code will take a local file path and return a FileIO object.

What we need is a block that will read data from a file via a url, load it into a pandas dataframe and return that instead.
From this starting template it's quite easy to add our desired functionality.

```Python
from mage_ai.io.file import FileIO
if 'data_loader' not in globals():
from mage_ai.data_preparation.decorators import data_loader
if 'test' not in globals():
from mage_ai.data_preparation.decorators import test
import pandas as pd


@data_loader
def load_data_from_file(*args, **kwargs):
"""
Template for loading data from urls.
Load data from 1 file via url.

For multiple directories, run the same block multiple
times with single dates.
"""

# Runtime variables are passed via kwargs
month = kwargs.get("month")
year = kwargs.get("year")

date = f'{year}-{month}'
print(date)

url = f'https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_{date}.csv.gz'

# Assigning explicit dtypes greatly reduces memory usage of the dataframe
taxi_dtypes = {
	'VendorID': float,
	'store_and_fwd_flag': str,
	'RatecodeID': float,
	'PULocationID': float,
	'DOLocationID': float,
	'passenger_count': float,
	'trip_distance': float,
	'fare_amount': float,
	'extra': float,
	'mta_tax': float,
	'tip_amount': float,
	'tolls_amount': float,
	'ehail_fee': float,
	'improvement_surcharge': float,
	'total_amount': float,
	'payment_type': float,
	'trip_type': float,
	'congestion_surcharge': float
}

parse_dates = ['lpep_pickup_datetime', 'lpep_dropoff_datetime']

df = pd.read_csv(url, sep=',', compression='gzip', parse_dates=parse_dates)
df = df.astype(taxi_dtypes)

return df
  
@test
def test_output(output, *args) -> None:
"""
Template code for testing the output of the block.
"""

assert output is not None, 'The output is undefined'
assert output.shape[0] > 0, 'The output dataframe is empty'
```

## How to Transform Data?

Adding the transform block works the same as with the data loader block, this time the block we are going to use is `Transformer > Row Actions > Remove Rows`.

![Transformer > Row Actions > Remove Rows](https://i.imgur.com/Cytq8bb.png)

The autogenerated code will look something like this. 

```Python
from mage_ai.data_cleaner.transformer_actions.base import BaseAction
from mage_ai.data_cleaner.transformer_actions.constants import ActionType, Axis
from mage_ai.data_cleaner.transformer_actions.utils import build_transformer_action
from pandas import DataFrame

if 'transformer' not in globals():
from mage_ai.data_preparation.decorators import transformer
if 'test' not in globals():
from mage_ai.data_preparation.decorators import test

@transformer
def execute_transformer_action(df: DataFrame, *args, **kwargs) -> DataFrame:
"""
Execute Transformer Action: ActionType.REMOVE

Docs: https://docs.mage.ai/guides/transformer-blocks#remove-rows
"""

action = build_transformer_action(
	df,
	action_type=ActionType.REMOVE,
	axis=Axis.ROW,
	options={'rows': []}, # Specify indices of rows to remove
)

return BaseAction(action).execute(df)

@test
def test_output(output, *args) -> None:
"""
Template code for testing the output of the block.
"""
assert output is not None, 'The output is undefined'
```

This code is already doing something we want, removing columns. There are multiple different [build_transformer_actions](https://docs.mage.ai/guides/blocks/transformer-blocks#building-transformer-actions) like this and since they output a dataframe you can string multiple together. 

For this specific use case I'm going to stick to using pandas for clarity. 

```Python
from mage_ai.data_cleaner.transformer_actions.base import BaseAction
from mage_ai.data_cleaner.transformer_actions.constants import ActionType, Axis
from mage_ai.data_cleaner.transformer_actions.utils import build_transformer_action
from pandas import DataFrame, to_datetime

if 'transformer' not in globals():
from mage_ai.data_preparation.decorators import transformer
if 'test' not in globals():
from mage_ai.data_preparation.decorators import test

@transformer
def execute_transformer_action(df: DataFrame, *args, **kwargs) -> DataFrame:
"""
Clean up df for new york green taxis. 

Docs: https://docs.mage.ai/guides/transformer-blocks#remove-columns
"""

action_one = build_transformer_action(
	df,
	action_type=ActionType.REMOVE,
	arguments=['extra', 'mat_tax', 'ehail_fee', 'improvement_surcharge', 'congestion_surcharge'],
	axis=Axis.COLUMN,
)

df = BaseAction(action_one).execute(df)

# Rename columns
new_column_names = {
	"VendorID": "vendor_id",
	"lpep_pickup_datetime": "pickup_datetime",
	"lpep_dropoff_datetime": "dropoff_datetime",
	"RatecodeID": "rate_code_id",
	"PULocationID": "pickup_location",
	"DOLocationID": "dropoff_location",
}

df = df.rename(columns=new_column_names)

month = kwargs.get("month")
year = kwargs.get("year")

print(f'{year}-{month}')

# Drop rows with invalid values
df = df[df['pickup_datetime'].dt.year == int(year)]
df = df[df['dropoff_datetime'].dt.year == int(year)]

df = df[df['pickup_datetime'].dt.month == int(month)]
df = df[df['dropoff_datetime'].dt.month == int(month)]

return df


@test
def test_output(output, *args) -> None:
"""
Template code for testing the output of the block.
"""

assert output is not None, 'The output is undefined'
assert isinstance(output, DataFrame), 'The output is not a DataFrame'
assert output.shape[0] > 0, 'The output DataFrame is not empty'
```
## How to Export Data to Snowflake?

Same deal, add export block. This time it's `Data Export > Python > Snowflake`.

![Data Export > Python > Snowflake](https://i.imgur.com/yfPNPcH.png)

The generate code should look like this.

```Python
from mage_ai.settings.repo import get_repo_path
from mage_ai.io.config import ConfigFileLoader
from mage_ai.io.snowflake import Snowflake
from pandas import DataFrame
from os import path

if 'data_exporter' not in globals():
from mage_ai.data_preparation.decorators import data_exporter

@data_exporter
def export_data_to_snowflake(df: DataFrame, **kwargs) -> None:
"""
Template for exporting data to a Snowflake warehouse.
Specify your configuration settings in 'io_config.yaml'.

Docs: https://docs.mage.ai/design/data-loading#snowflake
"""

table_name = 'your_table_name'
database = 'your_database_name'
schema = 'your_schema_name'

config_path = path.join(get_repo_path(), 'io_config.yaml')
config_profile = 'default'

with Snowflake.with_config(ConfigFileLoader(config_path, config_profile)) as loader:
	loader.export(
		df,
		table_name,
		database,
		schema,
		if_exists='replace', # Specify resolution policy if table already exists
)
```

We now have to modify it to suit our needs, in this case we don't need to change much, just the names of the database and the `if_exists` policy. 

```Python
from mage_ai.settings.repo import get_repo_path
from mage_ai.io.config import ConfigFileLoader
from mage_ai.io.snowflake import Snowflake
from pandas import DataFrame
from os import path

if 'data_exporter' not in globals():
from mage_ai.data_preparation.decorators import data_exporter

@data_exporter
def export_data_to_snowflake(df: DataFrame, **kwargs) -> None:
"""
Template for exporting data to a Snowflake warehouse.
Specify your configuration settings in 'io_config.yaml'.

Docs: https://docs.mage.ai/design/data-loading#snowflake
"""

table_name = 'WEEK_2_DE_ZC_TABLE'
database = 'WEEK_2_DE_ZC_DB'
schema = 'WEEK_2_DE_ZC_DB_SCHEMA'

config_path = path.join(get_repo_path(), 'io_config.yaml')
config_profile = 'default'

  
with Snowflake.with_config(ConfigFileLoader(config_path, config_profile)) as loader:
	loader.export(
		df,
		table_name,
		database,
		schema,
		if_exists='append', # append will append data to an exisiting table, replace will replace the table with the new one. 
)
```

If this blocks doesn't work as expected make sure that the `io.config.yaml` is set up correctly. 
## How to run the pipeline?

We are going to trigger the pipeline using a api call to trigger the pipelines, we are going to do this using a simple python script.

#### Creating the Trigger

But first we need to add a trigger, this is done in the the trigger tab. 
![Trigger Tab](https://i.imgur.com/W9KVFL5.png)

Once in the trigger page, click `New Trigger`. 
![New Trigger](https://i.imgur.com/ZBGLoVn.png)

Once the trigger creation menu loads, select the type as api. 

![Trigger API](https://i.imgur.com/ENFOYIF.png)

Then make sure to enable the trigger and to save it.

![Enable and Save Trigger](https://i.imgur.com/ogSJZJb.png)

#### Trigger the pipeline

For this exercise we are going to use the api call to ingest a whole quarter worth of taxi data, so we need to make sure that the script understands what a quarter is and it delivers that information to the triggered pipeline.

```Python
import requests
import sys


def get_quarter(quarter: int):

	if quarter == 1:
		return ["01", "02", "03"]
	elif quarter == 2:
		return ["04", "05", "06"]
	elif quarter == 3:
		return ["07", "08", "09"]
	elif quarter == 4:
		return ["10", "11", "12"]
	else:
		raise ValueError("Invalid quarter")

  
def ingest_month(year: str, month: str):

	url = "http://localhost:6789/api/pipeline_schedules/5/pipeline_runs/72ebbf5d21964c72b4fb48ff63f87e2e"
	
	headers = {
	"Content-Type": "application/json"
	}
	
	data = {
	"pipeline_run": {
		"variables": {
			"year": year,
			"month": month
		}
	}
}

response = requests.post(url, headers=headers, json=data)
response.raise_for_status()

  
if __name__ == "__main__":

	year = sys.argv[1]
	quarter = int(sys.argv[2])
	
	months = get_quarter(quarter)
	
	for month in months:
		print(f"Ingesting data for month {month}")
		ingest_month(year, month)
```

To run this script use the following command. 

```Shell
python3 ingest_quarter.py <year> <quarter>
```

#### The full code

The full code is available [here](https://github.com/Zesky665/de-zoomcamp/tree/feature/week_2/Week_2/mage).

### Next

Week 3 - Snowflake SQL ( coming soon )
