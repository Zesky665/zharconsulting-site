---
dg-publish: true
tags:
  - Snowflake
  - SQL
---
## Guidance Questions

1. What are the two ways you can set context parameters from within the Worksheets area of the webUI?
	1. Using the context bar and selecting from the dropdown.
	2. Hovering over the desired database/schema, left click and select `SET AS CONTEXT`.
	3. [Source](https://docs.snowflake.com/en/user-guide/ui-worksheet)
	## Notes: 
	1. To use the worksheet you need to select a role first.
	2. The role determines which databases/schemas roles the worksheet has access to.
	3. The active role in Snowsight is the default used when creating a worksheet. Changing the active roles does not change the worksheet role.
	4. The UI does not allow multiple roles. The worksheet has an option to enable multiple roles with the `USE SECONDARY ROLES` command.
	5. Each worksheet exists in it's own unique session tied to the browser tab, it can hold roles independently of the active role. 
	6. [Source](https://docs.snowflake.com/en/user-guide/ui-snowsight-worksheets#change-the-session-context-for-a-worksheet)
		 
2. What is the command keyword used to set context parameters via the SQL Pane (as opposed to the drop menu)?
	1. Using the commands:
		1. `USE WAREHOUSE`
		2. `USE DATABASE`
		3. `USE SCHEMA`
		4. `USE ROLE`
		5. `USE SECONDARY ROLES`
	 1. The objects (db, schema, etc) that can be accessed depends on the active roles selected in the worksheet. 
	 2. A warehouse is required to load any data in the session or run any DML statements.
	 3. If a DB or Scheme are not specified in the session, any references to those objects must be fully qualifies with the name of the database and schema, AKA they need to come with a namespace. 
		 1. Fully Qualified name: `USE dbNAME.schemaNAME;`
		 2. Not fully qualified name: `USE DATABASE dbNAME; USE schemaName;`
	4. [Source](https://docs.snowflake.com/en/sql-reference/sql/use)
3. What command is used with a context function like CURRENT_DATABASE() in order to return the current database name into the results pane?
	1. The database name consists of the warehouse, database and schema so the fully qualifies database name would need to return all of those values.
	2. The Query would look like this:
		1. `SELECT CURRENT_WAREHOUSE(), CURRENT_DATABASE(), CURRENT SCHEMA()`.
	3. [Source](https://docs.snowflake.com/en/sql-reference/functions/current_database)
4. What context function will tell you the cloud infrastructure provider and the geographic region of your Snowflake account?
	1. `SELECT CURRENT_REGION()`
	2. [Source](https://docs.snowflake.com/en/sql-reference/functions/current_region)
5. Which context function can give you a unique id that refers to your current browser tab?
	1. `SELECT CURRENT_CLIENT()`
	2. The result will be unique to each session AKA every separate browser tab.
	3. The result contains the current date + 4 random digits. 
	4. [Source](https://docs.snowflake.com/en/sql-reference/functions/current_client)
6. Which context function gives you a unique id that is roughly equivalent to your current worksheet?
	1. `SELECT CURRENT_SESSION()`
	2. The result will be unique to each worksheet, while the `CURRENT_CLIENT()` will be the same for each worksheet in the same tab.
	3. [Source](https://docs.snowflake.com/en/sql-reference/functions/current_session)

## Quiz

1. You create a new worksheet in the WebUI. You want to set the user role, warehouse, database and schema context that will be used when running the SQL code. 
	1. Run four `USE` commands, setting each context with a separate statement. 
	2. Use the Context drop menu in the upper right corner of the worksheet. 
2. Which of these statements is a valid script-based way to set context in a Worksheet?
	1. `USE database MY_DATABASE;`
	2. `USE schema MY_SCHEMA;`
3. Which of the following are valid context functions?
	1. `CURRENT_REGION()`
	2. `CURRENT_SESSION()`
	3. `CURRENT_CLIENT()`
4. Which context function returns information on the cloud infrastructure provider you chose when you first set up the Snowflake account?
	1. `CURRENT_REGION()`
5. Which command will return information about the current database?
	1. `SELECT CURRENT_DATABASE()`
6. If you click on the "History" in the ribbon, how far back in time are you able to view history?
	1. 14 days.
7. if you want to view query history older than 14days, where can you go to view it? Chose one path and one "term" commonly used.
	1. The Account Usage Share.
	2. SNOWFLAKE(Database) -> ACCOUNT_USAGE(Schema) -> QUERY_HISTORY(Secure View)
8. Which method for viewing query history provides more parameters/fields/attributes?
	1. The QUERY_HISTORY secure view available from the Account Usage Share.