---
dg-publish: true
tags:
  - Snowflake
---

## Guidance questions:

1) Is the architecture of Snowflake: shared disk? shared nothing? shared data? shared memory? 
	1) Snowflakes architecture is a mix of shared-disk and shared-nothing architecture. It accomplishes this by storing data in a central repository accessible to all compute nodes, this is the shared disk part. The queries are processed using MPP(massively parallel processing), using compute clusters where each node takes a portion of the data set. ([source](https://docs.snowflake.com/en/user-guide/intro-key-concepts#snowflake-architecture))
2) Are databases store within warehouses? Are warehouses stored within databases? 
	1) Data is stored in the storage layers in a shared disk manner, the queries are executed in the compute layer which treats the storage layers are a share nothing disk. ([source](https://docs.snowflake.com/en/user-guide/intro-key-concepts#snowflake-architecture))
	2) The "virtual warehouses" in snowflake are compute clusters with access to the storage layer, they don't store anything. When they process data they take a local copy of the data from the storage layers and divide it amongst the compute nodes, such as no node has access to all of the data. ([source](https://docs.snowflake.com/en/user-guide/intro-key-concepts#snowflake-architecture))
3) Does Snowflake store data with compression? Encryption? Both?
	4) Snowflake reorganizes data into it's own columnar format and compresses it with gzip(unless otherwise stated). ([source](https://docs.snowflake.com/en/user-guide/intro-key-concepts#database-storage))([source](https://docs.snowflake.com/en/user-guide/intro-summary-loading))
	5) Encrypted with a AES 256-bit key, which is provided by Snowflake. This key is automatically rotated(every 30-days). The user can also add a key managed by the host platform called a Master Key. This is called Tri-Secret Secure.([source](https://docs.snowflake.com/en/user-guide/security-encryption-manage)) ([source](https://docs.snowflake.com/en/user-guide/intro-summary-loading))
4) When a warehouse is resized, what queries are affected? Only current? Current and subsequent? Only subsequent? 
	1) Current queries are not affected, they will be allowed to finish. 
	2) Subsequent queries will wait until the change has been made and will then use the new virtual warehouse.
	3) Suspending the warehouse has the same behaviour as altering it. 
	4) [Source](https://docs.snowflake.com/en/sql-reference/sql/alter-warehouse#usage-notes)
5) Cost are broken down into what two major categories? 	
	1) Compute - billed by the second and based on consumed Snowflake credits. 
	2) Storage is prices on a per Terabyte basis and is based on the average number of on-disk bytes stored each day.
	3) Data egress to other cloud platforms or the same platform but different region. 
	4) [Source](https://docs.snowflake.com/en/user-guide/cost-understanding-overall)
6) Storage costs are based on the daily average of stored data. Is this based on the data's compression size or uncompressed size?  
	1) Compresses size and amount stored - daily average. 
	2) The total storage costs is made up of:
		1) Files staged(compressed or uncompressed)
		2) Data tables including historical data for Time Travel.
		3) Fail-safe for database tables. 
		4) Clone tables that reference data deleted in the source table.
		5) [Source](https://docs.snowflake.com/en/user-guide/cost-understanding-data-storage)
7) What things aren't required because Snowflake is true SaaS solution?
	- Requires not physical hardware purchase.
	- No software to be installed.
	- No maintenance required.
	- No user intervention needed.
	- Everything is hosted on publicly available cloud infrastructure.
	- Snowflake can't be hosted on private cloud architecture. 
	- [Source](https://docs.snowflake.com/en/user-guide/intro-key-concepts#data-platform-as-a-self-managed-service)
1) Can Snowflake be hosted on a company's internal cloud? What on-premise options are offered by Snowflake? 
	[None](https://docs.snowflake.com/en/user-guide/intro-key-concepts#data-platform-as-a-self-managed-service).
9) Can Snowflake be purchased for installation on company's internal services or Virtual Private Cloud(VPC)?
	No. But companies that need high level of security can use the Virtual Private Snowflake. Which is not hosted on premise, but is hosted on a completely separate Snowflake environment from all other Snowflake accounts.
	[Source](https://docs.snowflake.com/en/user-guide/intro-editions#virtual-private-snowflake-vps) 
10) Snowflake uses MPP compute clusters. Are these called Virtual Data Marts? or Virtual Warehouses? 
	1) Virtual Warehouses is what Snowflake calls it's MPP compute clusters. 
	2) [Source](https://docs.snowflake.com/en/user-guide/intro-key-concepts#query-processing)
11) Does Snowflake recommend only running no more than 2 warehouses simultaneously to avoid contention? 5?
	1) The Virtual Warehouses are independent. The number of them is irrelevant. 
	2) [Source](https://docs.snowflake.com/en/user-guide/intro-key-concepts#query-processing)
12) Are Snowflake Data Warehouses like Data Marts in that each one is assigned to work on a subset of the data stored in the account?
	1) Snowflake Data Warehouses are not Data Marts, but they can be used to do the same work, thanks to Snowflakes unique architecture. The separated compute layer sidesteps the need for data marts. 
			

## Quiz

1. What attributes make Snowflake a true SaaS solution?
	1. No hardware to purchase or configure.
	2. No maintenance upgrades or patches to install.
	3. Transparent releases don't require user intervention.
2. Which instalment option versions of Snowflake are available?
	1. Snowflake-Hosted Accounts (on Amazon Cloud infrastructure)
	2. Snowflake-Hosted Accounts (on Google Cloud Platform infrastructure)
	3. Snowflake-Hosted Accounts (on Azure Cloud infrastructure)
3. Which ONE of the following terms BEST describes Snowflake's Architecture?
	1. Shared Data.
4. Which of the following terms or phrases can also be used to describe Snowflake?
	1. Native SQL.
	2. Hybrid Columnar.
	3. Build from the ground up for the cloud. 
	4. Multi-Cluster.
5. Which statements are true about storage relationships?
	1. Snowflake Tables are stored within Schemas. 
	2. Snowflake Schemas are stored within Databases. 
6. When a warehouse is resized, which queries make use of the new size?
	1. Only subsequent queries. 
7. What are data storage cost calculations based on in Snowflake?
	1. Compressed Size.
	2. Amount Stored - Daily Average.
8. Snowflake data storage costs include which types of data?
	1. Persistent data sotred in permanent tables.
	2. Data retained to enable data recovery (time travel and fail-safe)
9. Snowflake compute costs depend on which of the following? 
	1. The amount of time warehouses have run.
	2. The size of running warehouses.
10. How often does Snowflake release new features?
	1. Weekly.
11. What common tasks for traditional on-premise databases and IT staff are not required with Snowflake?
	1. Maintaining metadata.
	2. Maintaining statistics.
	3. Maintaining the physical security of a server room (key cards, door locks, etc.)