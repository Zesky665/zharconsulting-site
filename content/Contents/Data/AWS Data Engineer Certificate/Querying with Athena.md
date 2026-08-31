
### Overview

This section covers AWS Athena.

Athena is a serverless interactive query service, that let's us query data from files stored in S3.

Some use cases for Athena:
- Log Analysis.
- Ad-Hoc Analytics.
- Data Lake Analytics.
- Source for QuickSight.

The queries themselves look just like regular SQL queries where the source is the name of the specific file being queried. 

###### Federated Queries

Allows Athena to query data from other data sources, such as:
- Relational Databases, such as Amazon RDS.
- Object data sources.
- Custom data sources. 

To use a new source with federated queries, you need to use an existing middleware or code your own. 
The built in data sources include:
- Amazon cloudWatch Logs.
- Amazon DynamoDB.
- Amazon DocumentDB.
- Amazon RDS.


###### Cost
- Pay as you go - only pay for data scanned. 
- Reserved capacity can be bought for cost savings. 

###### Performance
- Use partitions, don't scan irrelevant data. 
- Use partition-projection, this automates partition management. 
- AWS Glue Partition Indexes, like partition-projection, but happens outside of query context. Removes needs for pruning. 
- Query Result Reuse, uses stored results from past queries instead of repeating queries. Only use with slow data. 
- Data Compression, reduced data scanned. 
- Format Conversion, by converting row based data formats like .csv to column based formats like Parquet or OCR, the query performance can be massively improved. 

###### Workgroups
Allows queries and settings to be segregated into different groups based on:
- Teams
- Use cases
- Applications

This segregation allows for liner grained control over access and permissions as well as much finer cost monitoring. It can also be used to specify different query engines (Athena SQL vs Apache Spark). 

Up to 1000 workgroups are allowed per region. 
Each account has a primary workgroup as a default. 