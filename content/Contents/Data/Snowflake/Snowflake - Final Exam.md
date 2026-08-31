---
dg-publish: true
tags:
  - Snowflake
  - Exam
  - Practice
---
## Exam Questions

1. Which of the following terms are associated with the Cloud Services Layer?
	1. Query Processing. ❌ - is a separate layer.
	2. Query Planning. ✅
	3. Query Optimization. ✅
	4. Query Design. ❌ - is not handled by Snowflake.
	5. Query Compilation. ✅
	Answer: The Cloud Services layer contains the Query parsing and optimization service. This service parses, plans, optimizes and compiles queries before passing them to the query processing layer. [Source](https://docs.snowflake.com/en/user-guide/intro-key-concepts#query-processing)

2. Which of the following are performed by the Cloud Services Layer?
	1. Metadata Management.✅
	2. User Authentication.✅
	3. Metadata storage.✅
	4. Data Security.✅
	5. Availability Zone Management. ❌ - Not managed by Snowflake. 
	Answer: The Cloud Service Layer  contains the following services:
		- Authentication.
		- Infrastructure Management.
		- Metadata Management.
		- Query parsing and optimization.
		- Access control.

3. Which of these terms refer to the same layer?
	1. Compute Layer.✅ - Query Processing Layer.
	2. Virtual Warehouse Layer.✅ - Query Processing Layer.
	3. Catalog Layer.❌ - Cloud Services layer.
	4. Metadata Layer.❌ -Storage Layer.
	5. Query Processing Layer.✅ - Query Processing Layer.
	Answer: The Query Processing Layer contains all the services like Compute and Virtual Warehouses . [Source](https://docs.snowflake.com/en/user-guide/intro-key-concepts#query-processing)

4. Which of these terms refer to the same layer?
	1. Metadata Layer.❌ - Cloud Services Layer.
	2. Virtual storage Layer.❌ - Query Processing Layer.
	3. Catalog Layer.❌ - Cloud Services Layer.
	4. Data Layer.✅ - Storage Layer.
	5. Storage Layer.✅ - Storage Layer.
	Answer: The storage layer contains all services that store data: [Source](https://docs.snowflake.com/en/user-guide/intro-key-concepts#database-storage)

5. Which of the following terms are associated with the Compute/Warehouse Layer?
	1. Query Processing. ✅  - The layer that does all the compute.
	2. Query Planning.❌ - Cloud Services Layer.
	3. Query Optimization.❌  - Cloud Services Layer.
	4. Query Design.❌  - Not a Snowflake service.
	5. Query Compilation. ❌  - Cloud Services Layer.
	Answer: This one is obvious.

 6. Which of the following statements about Snowflake are true?
	 1. Storage can increase or decrease without any effect on virtual warehouse sizes.✅ - The Virtual Warehouses are independent of the storage layer.
	 2. Virtual Warehouses have no effect on compute costs. ❌  - Virtual Warehouses are where compute costs comes from. 
	 3. Compute can be scaled up, down, or in, and there is no effect on storage used.✅ - Virtual Warehouse size or number does not affect storage use, since the virtual warehouse splits one copy of needed data between compute processes. So if there are more or less processes the data used doesn't change. 
	 4. Cloud services can be increase to multiple regions without any user intervention.❌  - The user needs to create multiple accounts to move data to multiple regions. 
	 5. Two Virtual Warehouses can access the same data at the same time without causing contention issues.✅ - This is due to the shared data architecture, virtual warehouses use local copies of the data used, hence two or more virtual warehouses can use the same data without conflict.

7. What attributes make Snowflake a true SaaS solution?
	1. No hardware to purchase or configure. ✅ - Since all of the is managed by the cloud provider. 
	2. No creation of user accounts or roles is required. ❌  - This is just not true.
	3. No maintenance upgrades or patches to install. ✅ - This is managed by the cloud provider and Snowflake.
	4. No data storage cost. ❌  - This is just not true.
	5. No query processing cost.❌  - This is just not true.
	6. Transparent releases don't require user intervention. ✅ - This is managed by the  Snowflake team.

8. Which installment option version of Snowflake are available?
	1. Microsoft Cloud Native accounts?❌ - No accounts not hosted by Snowflake.
	2. Snowflake-Hosted Accounts (on Amazon cloud infrastructure)✅ - Snowflake can only be installed on AWS,Azure or GCP cloud infrastructure.
	3. Hybrid On-Premise + Cloud installation.❌ - No on premise installations. 
	4. Enterprise in-House VPC installation.❌ - No on premise installations. 
	5. On-Premise Custom Installation❌ - No on premise installations. 
	6. Snowflake-Hosted Accounts (on Azure cloud infrastructure)✅ - Snowflake can only be installed on AWS,Azure or GCP cloud infrastructure.
	
9. Which of the following terms or phrases can also be used to describe Snowflake?
	1. Native SQL.✅ - yup. 
	2. Hybrid Columnar.✅ - special partition method used by Snowflake.
	3. Built from the ground up for the cloud.✅ -yup.
	4. Hadoop-Compliant.❌ - Nothing to do with hadoop.
	5. Multi-Cluster.✅ - Due to the architecture compute is separated from storage, so multiple clusters can be used when needed. 
	6. Oracle derived. ❌ - Doesn't have any relation to oracle (thank god)
	
10. Which statements are true about storage relationships?
	1. Snowflake Tables are stored within Schemas. ✅
	2. Snowflake Databases are stored within Warehouses.❌
	3. Snowflake Warehouses are stored within Data Marts.❌
	4. Snowflake Schemas are stored within Warehouses.❌
	5. Snowflake Warehouses are stored within Databases.❌
	6. Snowflake Schemas are stored within Databases.✅
	Answer:  User -> Account -> Database -> Schema -> Table
	
11. Which of the following terms describes Snowflake's Architecture?
	1. Shared Disk.❌ - no the disk is not directly shared. 
	2. Shared Nothing.❌ - no the disk is indirectly shared.
	3. Shared Data.✅
	4. Shared Memory.❌ - memory is not shared between all the processes.

12. Snowflake data storage costs are calculated based on:
	1. Uncompressed Size. ❌
	2. Compressed Size. ✅
	3. Amount Stored on Last Day of Month. ❌
	4. Amount Stored on First Day of Month. ❌
	5. Amount Stored - Daily Average. ✅
	Answer: [Source](https://docs.snowflake.com/en/user-guide/cost-understanding-overall#total-cost-example)

13. Snowflake compute costs depend on which of the following?
	1. The number of rows returned in queries. ❌
	2. The amount of time warehouse have run. ✅ - the credits per second.
	3. The total number of warehouses in the account. ❌
	4. The sizes of running warehouses. ✅ - the amount of credits per second. 
14. Which common tasks for traditional on-premise database and IT staff, are not required with Snowflake?
	1. Maintaining metadata. ✅ - automatic in Service Layer.
	2. Maintaining statistics. ✅ - automatic in Service Layer.
	3. Maintaining the physical security of a server room (key cards, door locks, etc.) ✅ - the cloud platform takes care of that.
	4. Maintaining database objects. ❌ - the user handles this.

15. Snowflake Ecosystem Tech Partners are classified into which of the following functional categories?
	1. Data Integration. ✅ 
	2. Business Intelligence. ✅ 
	3. SQL Editors. ✅ 
	4. Security and Governance. ✅ 
	5. Advanced Analytics. ✅ 
	6. Programmatic Interfaces. ✅ 

16. Which statements about Data Integration Tech Partners are true?
	1. Data Integration Tech Partner software can be used to extract data from other systems.  ✅ 
	2. Data Integration Tech Partner software can be used to carry out transformations. ✅ 
	3. Snowflake can carry out transformation after loading files staged by partner software (ELT). ✅ - This is done entirely within the data warehouse. 
	4. Snowflake must be used to extract data from other databases but Data Integration Tech Partner software can load data. ❌ - Snowflake does not extract data from anywhere.
	5. Snowflake can be used to extract data from other databases but Data Integration Tech Partner software must be used to do transformations. ❌ - Snowflake does not extract data from anywhere.
	6. Data Integration Tech Partner software should be used to deliver data to stages, Snowflake is the used to load the data. ✅  - Snowflake loads (not extracts) data from staged files.

17. When a warehouse is resized, which queries make use of the new size?
	1. Both current and subsequent queries. ❌
	2. Only currently running queries. ❌
	3. Only subsequent queries. ✅ 
	Answer: When the virtual warehouse is resized the queries currently running will continue running on the same size warehouse, the ones following will use the new size.

18. Which Tech Partner types are available from in-account menu items?
	1. Partner Connect. ✅ - Can be used from WebUI
	2. Certified ANSI.  ❌ - not a partnership. 
	3. Programmatic Interfaces. ✅  - Can be used via API.
	4. Founding Partnership Companies. ❌ - Not sure what this even is.
	5. HIPAA Certified.  ❌ - not a partnership. 

19. Which of the following have drivers/connectors (or information about where to find them) available via Help -> Download in the Snowflake WebUI?
	1. Go ✅
	2. R ❌
	3. Node.js ✅
	4. JDBC ✅
	5. Hive ❌
	6. Spark ✅

20. What makes a Partner Connect Partner different from other partners?
	1. Can be connected to Snowflake using a streamlined wizard. ✅
	2. Can be connected from within the WebUI. ✅
	3. Includes a streamlines Partner Trial Account Signup. ✅
	4. Includes automated role, user and staging set up. ✅
	5. Requires Enterprise Edition. ❌

21. What are some differences between a Tech Partner and a Solution Partner?
	1. Solution Partners offer consulting and implementation services. ✅
	2. Wipro is an example of a Solution Partner. ✅
	3. Slalom is an example of a Technology Partner. ❌ 
	4. Tech Partners offer software, drovers or interfaces. ✅
	5. Matillion is an example of a Tech Partner. ✅
	6. Tech Partners offer consulting and implementation services. ❌ 

22. If you find a data-related tool that is not listed as part of the Snowflake ecosystem, what industry standard options could you check for as a way to easily connect to Snowflake?
	1. Check to see if the tool can connect to other solutions via ODBC. ✅
	2. Check to see if you can develop a driver and put it on Github. ❌ 
	3. Check to see if there is a petition in the community to create a driver. ❌ 
	4. Check to see if the tool can connect to other solutions via JDBC. ✅

23. How often does Snowflake release new features?
	1. Biannually ❌ 
	2. Yearly ❌ 
	3. Weekly ✅
	4. Never ❌ 

24. Is Snowflake HIPAA compliant?
	1. Yes ✅
	2. No ❌ 

25. What are the names of the three Snowflake Editions offered when signing up for a trial account?
	1. Free-Tier Basic ❌ 
	2. Standard ✅
	3. Ultra ❌ 
	4. Enterprise ✅
	5. Business Critical ✅

26. Which of the following Snowflake Editions automatically store data in an encrypted state?
	1. Standard ✅
	2. Enterprise ✅
	3. Business Critical ✅

27. Which of the following Snowflake Editions encrypt all data transmitted over the network within a Virtual Private Cloud (VPC)?
	1. Standard ❌ 
	2. Premier ❌ 
	3. Enterprise ❌ 
	4. Business Critical ✅

28. Which of the following industry compliance standards has Snowflake been audited and certified for?
	1. SOC 1 ✅
	2. SOC Type 2 ✅
	3. PCI DSS ✅
	4. FedRAMP ✅
	5. Cloud GBDQ ❌
	6. HIPAA ✅

29. What functional category does Matillion fall into?
	1. Data Integration. ✅
	2. Business Intelligence. ❌
	3. SQL Editors. ❌
	4. Security and Governance. ❌
	5. Advanced Analytics. ❌
	6. Programmatic Interfaces. ❌

30. Which cloud infrastructure providers are available as platforms for Snowflake Accounts?
	1. AWS. ✅
	2. Amazon Prime. ❌ - is a delivery subscription service. 
	3. Azure. ✅
	4. Microsoft OneDrive. ❌ - is a desktop sync service.
	5. Google Cloud Platform (GCP). ✅

31. Which cloud infrastructure provider become newly available as a platform Snowflake Accounts in 2020?
	1. AWS. ❌
	2. Amazon Prime. ❌
	3. Azure. ❌
	4. Microsoft OneDrive. ❌
	5. Google Cloud Platform (GCP). ✅

32. When signing up for a new Snowflake Account, enrollees first choose a cloud infrastructure provider and then a region. When choosing Azure, an enrollee might then choose the "Australia East" region. When choosing AWS, the "Australia East" region is not listed. Instead, a region called, "Asia Pacific (Sydney)" is listed. Why?
	1. Snowflake does not allow AWS-based Accounts to use the "Asia Pacific (Sydney)" region. ❌
	2. Snowflake retired the "Australia East" region before AWS-based accounts were in use. ❌
	3. Each cloud provider maintains its own regions and names the regions as they see fit. ✅
	4. The availability zones in "Asia Pacific (Sydney)" are incompatible with Azure partition sizes. ❌

33. When choosing a geographic deployment region, what factors might an enrollee consider?
	1. Proximity to the point of service. ✅
	2. Additional fees charged for regions with geo-political unrest. ❌
	3. End-user perceptions of glamorous or trendy geographic locations. ❌
	4. Number of availability zones within a region. ✅

34. The following questions have to do cloud platforms. Mark all the statements that are true.
	1. A company can use more than one cloud infrastructure provider by setting up several Snowflake accounts. ✅
	2. A company can use its data stored in more than one geographical region by setting up several Snowflake accounts. ✅
	3. A company can use a combination of data sharing and replication to distribute data to various regions and cloud platforms. ✅
	4. A company can use availability zones to distribute data to various regions and cloud platforms. ❌

35. What functional category does Looker fall into?
	1. Data Integration. ❌
	2. Business Intelligence. ✅
	3. SQL Editors. ❌
	4. Security and Governance. ❌
	5. Advanced Analytics. ❌
	6. Programmatic Interfaces. ❌

36. Which of these are Snowflake table types?
	1. Secure. ❌ - View type
	2. Permanent. ✅ - normal table.
	3. External. ✅ - staged file treated like table. No Fail-Safe or time travel.
	4. Materialized. ❌ - View type
	5. Temporary. ✅ - Only exists in current session.
	6. Transient. ✅ - Permanent with no fail-safe.

37. Which of the following are Snowflake view types?
	1. Secure. ✅
	2. Permanent. ❌
	3. External. ❌
	4. Standard. ✅
	5. Materialized. ✅
	6. Transient. ❌

38. What are the Snowflake Stage types?
	1. Secure. ❌
	2. Permanent. ❌
	3. User. ✅
	4. Materialized. ❌
	5. Table. ✅
	6. Named. ✅

39. Named stages come in two varieties, what are they?
	1. Secure. ❌
	2. Permanent. ❌
	3. Internal. ✅
	4. Materialized. ❌
	5. External. ✅

40. Which type of view is most like a table?
	1. Standard. ❌
	2. Secure. ❌
	3. Materialized. ✅
	4. External. ❌

41. What functional category does Node.js fall into?
	1. Data Integration. ❌
	2. Business Intelligence. ❌
	3. SQL Editors. ❌
	4. Security and Governance. ❌
	5. Advanced Analytics. ❌
	6. Programmatic interfaces. ✅

42. Which type of view has an extra layer of protection to hide the SQL code from unauthorized viewing?
	1. Standard. ❌
	2. Secure. ✅
	3. Materialized. ❌
	4. Permanent. ❌

43. In Snowflake account named MX43210, you need to set a user's default namespace to a database called MYDB and the PUBLIC scheme. Which of the following commands would you use?
	1. set default_namespace = MX43210.mydb.public ❌
	2. set default_namespace = public-my-db-MX43210 ❌
	3. set default_namespace = mydb.public  ✅
	4. set default_namespace = mydb.MX43210, default schema = public ❌

44. Select all statements that are true about the Snowflake container hiearchy:
	1. Accounts contain databases which contain schemas. ✅
	2. Schemas contain tables as well as views. ❌
	3. Schemas contain tables as well as views. ✅
	4. Databases contain users which are made up of roles. ❌
	5. Databases contain roles which have tables with sequences. ❌

45. In the Snowflake container hiearchy, what container is represented as a URL (for example: hhtps://HJ54364.snowflakecomputing.com)?
	1. Database ❌
	2. Schema ❌
	3. Role ❌
	4. Account ✅

46. Fail-safe is a seven-day history of data and is automatically available on which table types?
	1. Permanent ✅
	2. Temporary ❌
	3. Transient ❌
	4. External ❌

47. What is the name of the Snowflake-produced Command Line Interface tool? 
	1. SnowCLI ❌
	2. SnowCommand ❌
	3. SnowSQL ✅
	4. SnowSpark ❌

48. Which part of the URL is the region? https://TH77881.us-east-1.azure.snowflakecomputing.com
	1. TH77881 ❌
	2. us-east-1 ✅
	3. azure ❌
	4. snowflakecomputing ❌

49. Each Snowflake account comes with two shared databases. One is a set of sample data and the other contains Account Usage information. Check all true statements about these shared databases.
	1. SNOWFLAKE_SAMPLE_DATA contains a schema called ACCOUNT_USAGE. ❌
	2. SNOWFLAKE contains a table called ACCOUNT_USAGE. ❌
	3. SNOWFLAKE contains a schema called ACCOUNT_USAGE. ✅
	4. SNOWFLAKE_SAMPLE_DATA contains several schemas fromTPC (tpc.org)✅
	5. ACCOUNT_USAGE is a schema filled with external tables.  ❌
	6. ACCOUNT_USAGE is a schema filled with secure views. ✅

50. Which of the following statements are true about Fail-safe?
	1. Only a Snowflake employee can recover data from Fail-safe storage. ✅
	2. Fail-safe is a reliable way to create Dev/Test/QA and other environments. ❌
	3. The data stored as part of Fail-safe is part of storage costs charged to customers. ✅
	4. Fail-safe is not available for tables that have Time Travel. ❌

51. Time travel is available for which table types?
	1. Permanent. ✅
	2. Temporary. ✅
	3. Transient. ✅
	4. External. ❌

52. Which types of stages are automatically available in Snowflake and do not need to be created or configured? 
	1. User. ✅
	2. Table. ✅
	3. Named Internal. ❌
	4. Named External. ❌

53. What is the best way to get the latest ODBC connector for use with Snowflake?
	1. Download it from within the Snowflake WebUI. ✅
	2. Email ODBC@snowflake.com ❌
	3. Compile it in .NET ❌

54. Which table type disappears after the close of the session and therefore has no Fail-safe and no Time travel options after the close of the session?
	1. Permanent. ❌
	2. Temporary. ✅
	3. Transient. ❌
	4. External. ❌

55. You set up a Snowflake account, choosing AWS as your cloud platform provider. What stages can you use to load data files?
	1. USER ✅
	2. TABLE ✅
	3. NAMED INTERNAL ✅
	4. NAMED EXTERNAL - using S3 Buckets ✅
	5. NAMED EXTERNAL - using Azure BLOB storage ✅
	6. NAMED EXTERNAL - using GCS/GCP Buckets ✅

56. Which of the following object types are stored within schemas?
	1. Stages. ✅
	2. File Formats. ✅
	3. Sequences. ✅
	4. Stored Procedures. ✅
	5. User Defined Functions. ✅
	6. Roles. ❌

57. Snowflake data storage costs include which types of data?
	1. Metadata ❌
	2. Persistent data stored in permanent tables ✅
	3. Data retained to enable data recovery (Time Travel and Fail-safe) ✅
	4. Cached results ❌
	5. Semi-structured data - additional fees ❌
	
