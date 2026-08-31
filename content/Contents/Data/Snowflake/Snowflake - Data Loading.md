---
dg-publish: true
tags:
  - Snowflake
  - Data_Loading
  - SQL
---

## Guidance Questions

#### Loading Options
1. What are the four supported options for loading data in Snowflake?
	1. WebUI. Drag and drop files into the webUI(Snowsight or Classic Console). 
		1. Limited to 50MB. 
		2. Only supports certain file formats:
			3. Structured(e.g. CSV, TSV)
			4. Semi-Structured(e.g. JSON, Avro, ORC, Parquet or XML)
		3. Can only load file into internal stages.
		4. To load files into externally managed stages, use the platform specific tools. 
		5. [Source](https://docs.snowflake.com/en/user-guide/data-load-local-file-system-stage-ui)
	2. SQL and SnowSQL(Snowflake Client). 
		1. Uses SQL COPY.
		2. SnowSQL is a CLI client.
		3. Can be used to run queries and perform all DDL(Data Definition Language) and DML(Data Manipulation Language) operations. 
		4. [Source](https://docs.snowflake.com/en/user-guide/snowsql)
	3. Snowpipe. Loads data present in the Staged Phase. 
		1. Can be automated.
		2. Can be used to pre-process data.
		3. Can be used to do batch uploading vs the regular bulk uploading. 
		4. Is billed based on Compute used, not Virtual Warehouse time.
		5. [Source](https://docs.snowflake.com/en/user-guide/data-load-snowpipe-intro)
	4. 3rd party tools.
2. Can you briefly describe the significance of each option?
	1. WebUI for limited amounts of data.
	2. SQL & SnowSQL for bulk data loading.
	3. Snowpipe for auto-ingest, better security, potentially cheaper. 
	4. 3rd party etl tools. (dbt, etc)
#### Understanding Stages
1. What is the purpose of stage in data loading within Snowflake? 
	1. Stage is the intermediate step between files being PUT into snowflake and the data being loaded(COPIED) into the data table. The data is compressed and encrypted in this stage.
2. How do internal and external stages differ?
	1. Internal stages are stored within Snowflakes storage layers.
	2. External storage is a 3rd party bucket like s3, gcs or az block storage. The external storage area only stores the file url and credentials of the staged files. 
3. [Source](https://docs.snowflake.com/en/user-guide/data-load-overview#supported-file-locations)
#### Loading Data into Snowflake Tables
1. What are the main steps to load staged data into Snowflake tables?
	1. Send local file to staging with PUT.
	2. Transform and store files in staging by creating a FILE format.
	3. Load into table with COPY INTO(or Snowpipe).
2. Why is the concept of a virtual warehouse relevant in this process?
	1. The Compute resource is only used and charged when the loading is performed. Unless the loading is done with Snowpipe, then it charges based on the amount of compute used rather than the time the virtual warehouse was online. 
	2. [Source](https://docs.snowflake.com/en/user-guide/data-load-overview#bulk-vs-continuous-loading)
#### Internal vs External Stage Types
1. Name and briefly explain the types of internal and external stages.
	1. External: (Snowflake only holds file location and credential)
		1. AWS
		2. GCS
		3. Azure Block Storage.
	2. Internal:
		1. User: Auto created, used for files relevant only to the user.
		2. Table: Auto created for each table in snowflake, used to store files loaded into a table. Cannot be altered or dropped.
		3. Named: Manually created,  ownership controlled by privileges, can be altered and deleted. Created using CREATE STAGE command.
2. How does each type serve a different purpose in the data-loading process?
	1. User: Stores user specific files.
	2. Table: Stores all files used to populate table, serves as backup.
	3. Named: Flexible, can be used to store manually added files. 
3. [Source](https://docs.snowflake.com/en/user-guide/data-load-overview#supported-file-locations)

## Quiz:

1. In a load process, data arrives at these Snowflake objects in what sequence?
	1. Stage -> Pipe -> Table
2. When deciding whether to use bulk loading or Snowpipe, which factor should you consider?
	1. How often you will load the data.
	2. Location of data (local system or cloud)
	3. Number of files you will load one at a time.
3. Snowflake supports landing data into which locations?
	1. Internal stage on cloud storage platform.
	2. External stage on cloud storage platform.
4. When loading data in Snowflake, which statements are true?
	1. You can query a portion of data in external cloud storage without loading into Snowflake.
	2. The Snowflake Data Load wizard has a limitation on file size. (50mb is the max)
5. Who can provide the compute resource used by Snowflake for data loading jobs?
	1. User-managed virtual warehouse
	2. Snowflake-managed serverless compute