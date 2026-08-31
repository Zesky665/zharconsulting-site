---
dg-publish: true
tags:
  - Snowflake
---

## Guidance questions:

1. What is Partner Connect? 
	1. A feature that lets you create trial accounts from your Snowflake account for various integrated 3rd party services.
	2. [Source](https://docs.snowflake.com/en/user-guide/ecosystem-partner-connect)
2. How do you access and connect to Partner Connect Partners? 
	1. You login to an admin account. 
	2. Open SnowSight or the Classic Console.
	3. Make and switch to an ACCOUNTADMIN role.
	4. Open Partner Connect Page. 
		1. SnowSight:  Select admin >> Partner Connect.
		2. Classic Console: Partner Connect Tab.
	5. Click on the partner you wish to connect too.
	6. View and close the dialog window detailing the requirements and created database objects.
	7. (Optional) specify one or more existing databases in Snowflake to automatically use with the trial. If you do not specify them now, you can add them manually later.
	8. Click the Connect Button located below the partner description. 
	9. [Source](https://docs.snowflake.com/en/user-guide/ecosystem-partner-connect#connecting-with-a-snowflake-partner)
3. What is SnowSQL? How do you get a copy for installation? 
	1. SnowSQL is a CLI (command line interface) for executing SQL statements in the terminal.
	2. SnowSQL supports DDL and DML operations including loading and unloading data.
	3. Uses the `snowsql` executable, `stdin` or `-f` for batch mode.
	4. You download it from the website.
	5. [Download Link](https://developers.snowflake.com/snowsql/)
4. What drivers, connectors and component collections are available for programmatic interfacing with Snowflake? 
	1. **SnowPark API**: 
		1. Allows you to process data hosted inside Snowflake using Snowflake compute from outside of Snowflake.
		2. Compute handled by snowflake, no need to run large spark instance. 
		3. No egress needed, making it cheaper.
		4. Available in Java, Python and Scala.
		5. [SnowPark](https://docs.snowflake.com/en/developer-guide/snowpark/index)
	2. **Spark-Connector**: 
		1. Allows a Spark instance to read data from and write data to Snowflake. 
		2. [Source](https://docs.snowflake.com/en/user-guide/spark-connector)
	3. **Kafka with Kafka Connect**: 
		1. Needs separate Kafka Connect instance. 
		2. Currently supports one topic to one Snowflake table pipelines.
		3. [Source](https://docs.snowflake.com/en/user-guide/kafka-connector-overview)
	4. **Drivers**: 
		1. [Go Snowflake Driver](https://docs.snowflake.com/en/developer-guide/golang/go-driver) - driver for the Go language.
		2. [JDBC Driver](https://docs.snowflake.com/en/developer-guide/jdbc/jdbc) - for all clients / applications that support JDBC.
		3. [.NET Driver](https://docs.snowflake.com/en/developer-guide/dotnet/dotnet-driver) - driver from Microsoft .NET applications.
		4. [Node.js Driver](https://docs.snowflake.com/en/developer-guide/node-js/nodejs-driver) - native async interface for Node.js.
		5. [ODBC Driver](https://docs.snowflake.com/en/developer-guide/odbc/odbc) - for all clients using ODBC connections.
		6. [PHP PDO Driver for Snowflake](https://docs.snowflake.com/en/developer-guide/php-pdo/php-pdo-driver) - driver for PHP.
		7. [Snowflake Connector for Python](https://docs.snowflake.com/en/developer-guide/python-connector/python-connector) - driver for Python. 
5. Partners are also classified according to their functionality. "Data Integration" is one category, what are the others? 
	1. Business Intelligence.
	2. Machine Learning and Data Science.
	3. Security, Governance and Observability.
	4. SQL Development and Management.
	5. Native Programmatic Interfaces. 
	6. Data Integration.
	7. [Source](https://docs.snowflake.com/en/user-guide/ecosystem)
6. If you were presented with a list of the functional categories, could you recognized them?
	1. Data Integration:
		1. ETL
		2. ELT
		3. Data Migration, Movement and management.
		4. Data Warehouse automation.
	2. Business Intelligence
		1. Data Analysis
		2. Reports
		3. Dashboards
		4. Charts
	3. Machine Learning & Data Science
		1. Advanced Analytics
		2. AI
		3. Big Data
	4. Security, Governance & Observability
		1. Ensure sensitive data is protected
		2. Monitoring
		3. Risk Management
		4. Data Masking/Anonymizations 
		5. Data Health/Quality Checks
	5. SQL Development & Management
		1. Modeling
		2. Development
		3. Deployment
	6. Native Programmatic Interfaces
		1.  See answer 4 to question 4. 
	7. [Ecosystem](https://docs.snowflake.com/en/user-guide/ecosystem)
7. Which functional category do each of the following partners fall into? Matilion? Looker?  Databricks?
	1. Matilion - Data Integration. [ETL](https://docs.snowflake.com/en/user-guide/ecosystem-etl)
	2. Looker - Business Intelligence. [BI](https://docs.snowflake.com/en/user-guide/ecosystem-bi)
	3. Databrick - Machine Learning & Data Science. [Databricks](https://docs.snowflake.com/en/user-guide/ecosystem-analytics)
8. Do Python and Node.js require SnowSQL to connect? Are these connectors used instead of the WebUI or in addition to it?
	1. No, SnowSQL is a wrapper written in Python using the driver. 
	2. In addition to it. For example we can pair the python connector with pandas. [python-connector-pandas](https://docs.snowflake.com/en/developer-guide/python-connector/python-connector-pandas)
9. What is the difference between a Technology Partner and a Solution Partner?
	1. Technology partners are tech companies that provide tools, software, drivers and interfaces.
	2. Solution partners offer consulting and implementation services.
	3. [Source](https://community.snowflake.com/s/question/0D5Do00000H1tFEKAZ/what-are-some-differences-between-a-tech-partner-and-a-solution-partner)

## Quiz

1. Snowflake Ecosystem Tech Partners are classified into which of the following functional categories?
	1. Data Integration
	2. Business Intelligence
	3. SQL Editors
	4. Security and Governance
	5. Advanced Analytics
	6. Programmatic Interfaces
2. Which statements about Data Integration Tech Partners are True?
	1. Data Integration Tech Partner software can be used to extract data from other systems.
	2. Data Integration Tech Partner software can be used to carry out transformations.
	3. Snowflake can carry out transformations after loading files staged by partner software(ELT).
	4. Data Integration Tech Partner software should be used to deliver data to stages, Snowflake is then used to load the data.
3. What functional category does Matilion fall into?
	1. Data Integration
4. What functional category does Looker fall into?
	1. Business Intelligence
5. What functional category does Node.js fall into?
	1. Programmatic Interface
6. What is the name of the Snowflake produced Command Line Interface tool?
	1. SnowSQL
7. What two Tech Partner types are available from in-account menu items?
	1. Partner Connect
	2. Programmatic Interfaces
8. What is the best way to get the latest ODBC connector for use with Snowflake?
	1. Download it from within Snowflake WebUI.
9. Which of the following have drivers/connectors (or information about where to find them) available via Help -> Downloads in the Snowflake WebUI?
	1. Go
	2. Node.js
	3. JDBC
	4. Spark
10. What makes Partner Connect Partner different from other partners?
	1. Can be connected to Snowflake using a streamlined wizard
	2. Can be connected from within the WebUI
	3. Includes a streamlined Partner Trial Account Signup
	4. Includes automated role, user and staging database set-up
11. What are some differences between a Tech Partner and a Solution Partner?
	1. Solution Partners offer consulting and implementation services
	2. Wipro is an example of a Solution Partner
	3. Tech Partners offer software, drivers, or interfaces
	4. Matilion is an example of a Tech Partner
12. If you find a data-related tool that is not listed as part of the Snowflake ecosystem, what industry standard options could you check for as a way to easily connect to Snowflake?
	1. Check to see if the tool can connect to other solutions via ODBC.
	2. Check to see if the tool can connect to other solutions via JDBC.