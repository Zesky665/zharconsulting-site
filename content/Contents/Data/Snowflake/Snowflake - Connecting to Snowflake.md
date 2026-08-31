---
dg-publish: true
tags:
  - Snowflake
  - Python
  - DE_ZOOMCAMP_2024
  - SQL
  - Scripts
---



## Setup

Get the account id and the organization id. They are visible in the admin panel, visible in `Admin` > `Account` > `Org` as seen in the image below. 

![ID Location](https://i.imgur.com/kf5u1Fs.png)

From that panel take the `Ogranization` and `Locator` values. 
The account id is `<Organization>-<Locator>`, i.e. if `Organization` = abc and `Locator` = 123, then the account id would be `abc-123`.

### Via SnowSQL

```Shell
snowsql -a <org_id>-<locator> -u <username>
```

### Via Python

Install python-snowflake connector

```Python
import snowflake.connector as snow

connection = snow.connect(
account='abc-123',
user='username',
password='password',
warehouse='WEEK_2_DE_ZC_SVC_WH',
database='WEEK_2_DE_ZC_DB',
schema='WEEK_2_DE_ZC_DB_SCHEMA'
)
  
# Define a cursor
cursor = connection.cursor()
  
try:
# Execute a SQL query against Snowflake to get the current_version
cursor.execute("SELECT current_version()")
one_row = cursor.fetchone()
  
# Display the version information
print("Connection Successful")
print(one_row[0])

finally:
print("Connection Closed")
cursor.close()

connection.close()
```