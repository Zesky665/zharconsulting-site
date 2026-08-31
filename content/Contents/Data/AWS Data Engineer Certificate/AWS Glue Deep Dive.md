
### Overview

This section is a deep dive into all things AWS Glue.

#### Costs

###### Crawlers
- Hourly based on the number of DPUS used. 
- Billed by second with a 10min minimum. Yikes. 
- What are DPUs?
	- Data Processing Units
	- 1 DPU = 4 vCPU and 16bg ram. 

###### Data Catalog
- Up to one million objects for free.
- $1.00 per 100,00 objects over a million, per month. 

###### ETL Jobs
- Hourly based on number of DPUs used.
- Billed by seconds with a 10 min minimum.
- AWS Glue version 2.0 and later have 1 min minimum.

###### How many DPUs are used?
- Apache Spark: Minimum of 2 DPUs - Default: 10 DPUs. Up to 100 DPUs, suitable for large-scale data processing. 
- Spark Streaming: Minimum of 2DPUs - Default 2 DPUs. Up to 100 DPUs. 
- Ray job (ML/AI): Minimum of 2 M-DPUs (high memory). Default: 6 M-DPUs
- Python Shell job (flexible & simple): 1 DPU or 0.0625 DPU. Default 0.0625 DPU. 
- Notebooks / Interactive Sessions: Minimum of 2 DPUs. Default of 5 DPUs. 

###### Cost of DPUs
- $0.44 per DPU-Hour (may differ and depend on region)


#### AWS Budgets

##### Alarms:
Allows budget to be set and an notification to be sent when it is exceeded. 

###### Actual & Forecasted
Helps to manage costs.

###### Budget Types:
- Cost budget: Alerts when cost threshold is crossed.
- Usage budget: Alerts when usage threshold is crossed. 
- Saving plans budget: Alerts when savings plan capacity is exceeded by some set amount. 
- Reservation plans budget: Alerts when usage falls bellow the set amount. Meant ot detect underutilization of reserved resources. 

###### Budget Cost
2 action enabled budgets are free, then it's $0.10/day. 
Non action enabled budgets are completely free. 


#### Stateful vs. Stateless

###### Stateful
Remembers past runs.
Enabled by setting the bookmark value to "Enabled". It will remember files that have been processed and skip them during ETLs. 

###### Stateless
Does not remember past runs. 


#### AWS GLUE - Extract Transform Load

###### Extract
From:
- Amazon RDS, Aurora, DynamoDB.
- Amazon Redshift.
- Amazon S3, Kinesis. 

###### Transform
Using:
- Filtering: Stripping out unneeded data. 
- Joining: Combining data.
- Aggregation: Summarizing data. 
- FindMatches ML: Deduplication.
- Detect PII: Find Personally Identifiable Information.
- CSV <--> Parquet <--> JSON <--> XML

###### Load
- Amazon RDS, Aurora, DynamoDB.
- Amazon Redshift.
- Amazon S3, Kinesis.

#### AWS Glue - Glue Workflows

- Orchestrate multi-step data processing jobs, manage executions and monitoring of jobs/crawlers. 
- Ideally used for managing AWS Glue operations. 
- Provides visual interface. 
- You can create workflows manually or with AWS Glue Blueprints.
- Triggers initiate jobs and crawlers:
	- Schedule Triggers: Starts workflow on regular intervals. 
	- On-Demand Triggers: Manually start the workflow from AWS Console. 
	- EventBridge Event: Launches the workflow based on specific events captured by Amazon EventBridge. 


#### AWS Glue - Partitioning

- Enhances the performance of AWS Glue. 
	- Provides better query performance.
	- Reduces I/O operations. 
- AWS Glue can skip over large segments within partitioned data.
- AWS Glue can process each partition independently.
- Provides cost efficiency by reducing query efforts. 
- In AWS Glue, define partitioning as part of ETL job scripts. Also possible within Glue Data Catalog. 

#### AWS Glue DataBrew
- Data preparation tool with visual interface.
- Cleaning and data format processes. 
- 250+ pre-built transformations. 
- Consists of:
	- Project: Where you configure the transformation task.
	- Step: The transformations to be applied.
	- Recipe: Set of transformation steps, can be saved and reused. 
	- Job: Execution of a recipe on a dataset, output to locations such as S3.
	- Schedule: Schedule jobs to automate transformation.
	- Data Profiling: Understand quality and characteristics of your data. 
- Transformations:
	- NEST_TO_MAP: converts columns into a map.
	- NEST_TO_ARRAY: converts columns into an array.
	- NEST_TO_STRUCT: like NEST_TO_MAP but retains exact data type and order. 
	- UNNEST_ARRAY: expands array to multiple columns. 
	- PIVOT: Pivot column and pivot values to rotate data from rows into columns.
	- UNPIVOT: Rotates columns into rows. (attribute + value)
	- TRANSPOSE: Switch columns and rows. 
	- JOIN: Combine two datasets. 
	- SPLIT: Split a column into multiple columns based on a delimiter.
	- FILTER: Apply conditions to only specific rows in your dataset. 
	- SORT: Arrange the rows in your dataset in ascending or descending order. 
	- DATE/TIME CONVERSIONS: Convert strings to data.time formats or change between different date/time formats. 
	- COUNT DISTINCT: Calculate the number of unique entries in that column. 