---
dg-publish: true
tags:
  - Snowflake
  - SQL
  - History
---

## Guidance Questions

1. Where (in the WebUI) do I go to view recent query results and query profile information?
	1. SnowSight:
		1. Query history is available in the *Query History* page in Snowsight.
		2. In the *Query History* page, you can:
			1. Monitor queries executed by users in your account.
			2. View details about series, including performance data. 
			3. Explore each step of an executed query in the query profile. 
		3. Only includes queries executed in the past 14 days. 
		4. Some queries are not available in the *Query History* page, this can be due to:
			1. The query is still running, when it finishes it will appear in the *Query History*.
			2. Your roles does not have permission to view it.
			3. The query is more than 14 days old.
			4. The query failed to run.
			5. [Source](https://docs.snowflake.com/en/user-guide/ui-snowsight-activity#label-snowsight-query-details-unavailable)
		5. [Source](https://docs.snowflake.com/en/user-guide/ui-snowsight-activity#query-history)
	2. Classic Interface:
		1. The 'History' tab, here you can see the queries executed in the past 14 days.
		2. The following information included about each query includes:
			1. Current status of queries: waiting, running, succeeded or failed.
			2. The SQL text of the query.
			3. The QUERY ID.
			4. The Warehouse that executed the query.
			5. The start and end time as well as duration.
			6. The size of the transaction in bytes.
			7. The number of rows affected.
			8. [Source](https://docs.snowflake.com/en/user-guide/ui-history)
		3. Is based on the `QUERY_HISTORY` view, which: 
			1. Contains the query history of the past 365 days.
			2. Is available only in the reader account view of `READER_ACCOUNT_NAME`.
			3. [Source](https://docs.snowflake.com/en/sql-reference/account-usage/query_history)
2. How far in the past am I able to view query profiles and results using the WebUI?
	1. 14 days of query history in both Classic Interface and SnowSight.
	2. Queries older than 7 days don't show the user that executed the query in Snowsight.
	3. Older queries can be queried but will return no results.
	4. To view older queries the `QUERY_HISTORY` view can be used to view queries up to 365 days old.
	5. [Source-WebUI](https://docs.snowflake.com/en/user-guide/ui-history#overview-of-features)
	6. [Source-SnowSight](https://docs.snowflake.com/en/user-guide/ui-snowsight-activity#query-history)
3. Can I view my co-workers queries?
	1. Yes, but only the results.
	2. In Classic Console you can only view the queries your user has executed, although you can view the queries executed by other users if you have the privileges to do so, but for privacy reasons the results will not be shown. 
	3. [Source](https://docs.snowflake.com/en/user-guide/ui-history#viewing-query-details-and-results)
4. Can I view the queries run by my company's ETL tool that connects using JDBC?
	1. Yes, if you have the correct privileges. 
5. Can I download query results from this morning without having to run the query again?
	1. Yes, the results can be downloaded in the form of a CSV or other similar formats. [Source](https://docs.snowflake.com/en/user-guide/ui-history#viewing-query-details-and-results)
	2. The results persist for 24h(not adjustable). [Source](https://docs.snowflake.com/en/user-guide/querying-persisted-results)
6. Can I download the results of my coworker's query from this morning?
	1. No, you cannon view other peoples results.
	2. [Source](https://docs.snowflake.com/en/user-guide/ui-history#viewing-query-details-and-results)
7. I need to quickly look at this week's queries but only for a particular warehouse, and I want to visually compare queuing time for each query, can I do this from the WebUI?
	1. You can filter queries for the following attributes:
		1. **Status** of Query (waiting, running, successful, failed)
		2. **User**, only users you have permissions to view will appear.
		3. **Time** period in which the query was executed.
		4. **SQL Text**, for example queries that used statements like `GROUP BY`.
		5. **Query ID**, if you need to view a specific query.
		6. **Warehouse**, to view queries executed by specific warehouse.
		7. **Statement Type**, to view queries that used specific statements like `DELETE`, `UPDATE`, `INSERT` or `SELECT`. 
		8. **Duration**, for example to look for long running queries.
		9. **Session ID**, to filter based on specific Snowflake Session. 
		10. **Query Tag**, to view queries with a specific [QUERY TAG](https://docs.snowflake.com/en/sql-reference/parameters#label-query-tag) session parameter. 
	2. [Source](https://docs.snowflake.com/en/user-guide/ui-snowsight-activity#query-history)

## Quiz:
1. What two types of caches were mentioned in the video?
	1. Metadata Cache - the cache holding metadata about the database like size, number of rows, etc.
	2. Results Cache - which holds results of previous queries up for 24h. 
2. Using the History area of the WebUI, how far back in time is query history available? 
	1. 14 days.
3. The columns in the query history include the QueryID, the SQL Text, the Warehouse name, the Warehouse Size, the Session ID and others. Which column is a good indicator of whether a Warehouse was used (and Compute costs incurred) by a query?
	1. Size.
4. FDNIEGE is running queries using SYSADMIN and ACCOUNTADMIN roles. User ALICE is using the PUBLIC Role. 
	1. FDNEIGE can see all of ALICES queries, since she has higher privileges.
	2. ALICE cannon see any other user's queries because PUBLIC is the lowest role.
	3. FDNEIGE can see all other user's queries when her role is et to ACCOUNTADMIN.
	4. * FDNEIGE cannot see ALICE's results, because users cannot see other users results. 
5. A query you initiated is taking too long. You've gone into the History area to check whether this query (which usually runs every hour) is supposed to take a long time.
	1. Information in the History area can be filtered to show a single Warehouse.
	2. Information in the History area can be filtered to show a single User.
	3. Information in the History area can be filtered to show a single Session.
	4. Information in the History area can be filtered to show a single query times.
6. A hour ago, you ran a complex query. You then ran several simple queries from the same worksheet. You want to export the results from the complex query but they are no longer loaded in the Results pane of the worksheet. What is the least costly way to download the results? 
	1. Click on History -> Locate the query -> Click the QueryID -> Use the "Export Result" button.
7. You have a dashboard that connects to Snowflake via JDBC. The dashboard is refreshed hundreds of time per day. The data is very stable, only changing once or twice per day. The query ran by the dashboard connector user never changes. How will Snowflake manage changing and non-changing data?
	1. Snowflake will show the most up-to-date data each time the dashboard is refreshed. 
	2. Snowflake will spin up a warehouse only if the underlying data has changed. 
	3. Snowflake will re-use data from the Result Cache as long as it is still the most up-to-date data available. 
8. When running a SELECT COUNT(\*) on a table, which of the following statements is true?
	1. No Warehouse will be needed because Count statistics are stored in the Metadata cache. 