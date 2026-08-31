
### Overview

This section covers the basics of S3 and AWS Glue. 

#### S3 Buckets

- The buckets have unique global names across all global accounts and regions/ availability zones. 
- Names must consist of between 3 and 63 characters. 
- Only lowercase letters, numbers, dots (.) and hyphens (-).
- Each object inside of the buckets also has a unique name, which can include the '/'. This denotes a pseudo directory structure. 
- The buckets come in different tiers based on cost and availability:
	- Standard: Default, high-availability, frequently accessed data. 
	- Intelligent-Tiering: Automatically moves data between tiers based on access patterns.
	- Standard-IA (Infrequent Access): Lower cost, but charges for retrieval.
	- One Zone-IA: Like Standard-IA but stored in only one AZ.
	- Glacier: Very low cost, for archival with retrieval times from minutes to hours. 
	- Glacier Deep Archive: Lowest cost, fro long-term archival with retrieval times up to 12 hours. 
- Security is implemented via multiple different policies:
	- Bucket Policies: JSON documents that define who can access the bucket and what actions they can take. 
	- IAM Policies: Attached to users/roles to grant permissions. 
	- Access Control Lists (ACLs): Legacy method, still available but bucket policies are preferred. 
	- Block Public Access settings: Additional layer of protection against unintended public access. 
- Data Management Features:
	- Versioning: When enabled, keeps multiple versions of objects.
	- Lifecycle Rules: Automate moving objects between storage classes or deleting them.
	- Replication: Copy objects automatically between buckets, even across regions. 
	- Transfer Acceleration: Fast file transfers using cloudFront's edge locations. 
	- Object Lock: Prevent objects from being deleted or overwritten. 
- Other important configurations:
	- Static Website Hosting: Buckets can host static websites with custom domain names. 
	- Server Access Logging: Track all requests made to the bucket. 
	- Event Notifications: Trigger Lambda functions, SQS, or SNS when bucket events occur.
	- Requester Pays: Make the requester pay for data transfer costs.
	- Tags: Add metadata tags for cost allocation and organization. 
- Performance and Scalability:
	- S3 automatically scales. 
	- Use randomized prefixes to distribute load. Each prefix has a limit of 5k GET requests per second. 
	- Use parallel operations for better throughput. 
	- Consider using Transfer Acceleration for faster uploads.
	- Use multipart uploads for files over 100MB. 
- Cost Considerations:
	- Storage used per month.
	- Data transfer OUT of S3.
	- Request pricing (PUT, GET, LIST, etc.)
	- Management features used (like replication or analytics)
	- Storage class chosen.

#### Streaming vs Batch Ingestion

##### Streaming
- Enables real-time ingestion.
- Ideal for time-sensitive data.
- More expensive and intricate.

##### Batch
- Ingests data periodically in batches.
- Typically large volumes.
- Cost effective and efficient. 


#### AWS Glue

- Fully managed ETL service.
- Designed to load and transform data.
- Visual interface, easy to use.
- Integrated with various services like S3, Redshift and Amazon RDS. 
- Autogenerates Spark Python script based on selected options. 
- Serverless. 
- Pay-as-you-go.

##### AWS Glue Data Catalog

- Stores Data Catalog: schemas and metadata.
- Allows data to be queried using Athena, Redshift, etc.
- Used Glue Crawlers.
- Automatically detects data schema. 
- Can be scheduled.
- Can be crawled incrementally. 