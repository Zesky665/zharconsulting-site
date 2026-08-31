---
dg-publish: true
tags:
  - Snowflake
---

## Guidance Questions

1. In what ways does Snowflake protect my data from user errors?
	1. Time-Travel: Preserves all data from the past x-days(1 day by default) depending on edition. This allows users to do the following:
		1. Query data that was recently deleted or updated.
		2. Copy tables, schemas or databases at or before specific points in the past.
		3. Restore tables, schemas and databases that have been dropped.
		4. [Source](https://docs.snowflake.com/en/user-guide/data-time-travel)
	2. Fail-Safe: Non configurable time-travel, envelops data up to 7 days past the time-travel threshold.
		1. Fail-Safe does not allow data to be queried.
		2. Fail-Safe data recovery is done by Snowflake and can take several days. 
		3. [Source](https://docs.snowflake.com/en/user-guide/data-failsafe)
	4. Zero-Copy cloning: Enables testing changes in temporary tables that are not immediately reflected in the db.
		1. Clones only store the changes that are made to them in a separate partition. 
		2. Clones can be cloned.
		3. [Source](https://docs.snowflake.com/en/user-guide/tables-storage-considerations#cloning-tables-schemas-and-databases)
2. Which automatic Snowflake features help to provide business continuity for different types of system failures?
	1. Account object replication. Replicates account level object like user, roles and security settings.
	2. Database replication. Replicates databases and data to multiple geographically isolated areas. 
	3. Client redirect. Allows automatic request rerouting to other databases.
3. What are the differences between Time Travel and Fail-Safe?
	1. Fail-Safe is non configurable, also recovering data from Fail-Safe cannot be done by the account owner, it needs to be done by a Snowflake support employee.
4. How can I use zero-copy cloning to protect data?
	1. Use zero-copy databases to test out alterations before promoting them to the main database.
5. What are key requirements for database replication-fallover-fallback?
	1. Two or more copies of Snowflake objects(users & databases), replication groups and failover groups. 
	2. A replication group is a defined collection of objects from a source account that can be replicated as a unit to one or more target accounts. Replication groups provide read only access to the replicated objects.
	3. Failover groups are like replication group with the addition that they can also fail over. When the primary server fails, the replicated becomes the primary server and is granted write access. 
	4. [Source](https://docs.snowflake.com/en/user-guide/account-replication-intro#label-replication-and-failover-groups)
## Quiz

1. What features automatically protect your Snowflake data without user or system administrator intervention?
	1. Availability zones (AWS, Azure, GCP)
	2. Triple redundancy for critical services.
2. Which activity requires intervention by Snowflake Support to restore data?
	1. Fail-Safe
3. Which statements are true for Snowflake's zero-copy cloning feature?
	1. Data is stored once - only changed micro-partitions are stored and owned exclusively by the clone.
	2. You can instantly promote a cloned and updated table, database or schema to production.
4. Which statements are true for Snowflake's fail-over capabilities?
	1. Must have Snowflake Business Critical Edition or higher. 
	2. Must have an account in the region where you wish to share replicated data.