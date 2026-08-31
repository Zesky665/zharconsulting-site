---
tags:
  - Spark
  - Python
  - SQL
dg-publish: true
---

### What is Spark?
Apache Spark is an open source analytics framework originally developed in UC Berkeley in 2009 as an extension of Hadoop MapReduce. It's built on the idea of a RDD (Resilient - Distributed - Dataset) as the basic data type, which basically means that the data is not present on a single machine, it is distributed across multiple machines and analyzed and transformed on those other machines. This allowed data processing to take advantage of horizontal scaling, paving the way for Big data. 

### What is it used for?
It was originally designed to enable processing of large datasets across a larger number of smaller machines rather than using a single supercomputer. Nowadays, it has been extended to handle everything data related, notably streaming and ML data. 

### How does it fit in with the tech stack?
##### Extract and Load
Spark has an ecosystem full of connectors for all sorts of data sources and data bases. Using it allows teams to skip that part of the equation. 

##### Streaming
Spark supports both real-time streaming and micro-batching as well as stream to batch with it's Structured Streaming processing engine. This allows spark to ingest streams and run transformations on them the same way it does to batches, it can also just convert stream data int batches. This is enabled by it's fault tolerance and exactly-once stream processing. 

##### Machine Learning
Spark has it's own machine learning library called MLlib, which is analogous to scikit-learn. It's used in the cases where the training data is to large to fit into local memory. 

### How to set it up locally?
You don't need to, the power of spark is in it's ability to leverage the cloud. Which is why all of it's bigger applications are in online environments like Databricks and Google Colab. 

### Example usage
There is a wonderful public google colab available that explain all of Spark's feature in detail available [here](https://colab.research.google.com/drive/1G894WS7ltIUTusWWmsCnF_zQhQqZCDOc#scrollTo=_N5-lspH_N8B) for those who are interested. 

I am however going to provide a super simple example here. 
```python
from pyspark.sql import SparkSession

# Create a spark session
spark=SparkSession.builder.appName('Example').getOrCreate()

# Read some data into a spark dataframe
df_pyspark=spark.read.csv('example.csv')

# Show the type
type(df_pyspark)
	# pyspark.sql.dataframe.DataFrame

# PySpark infers the schema by default
df_pyspark.printSchema()
	# root 
    #  |-- Name: string (nullable = true) 
    #  |-- age: integer (nullable = true) 
    #  |-- Experience: integer (nullable = true) 
    #  |-- Salary: integer (nullable = true)

# Show the first 5 lines of data
df_pyspark.limit(5).show()
# +| Name      |  age | YoE | Salary  |+
#  | Mark      |  35  |  11 |  180000 | 
#  | Christian |  34  |  9  |  100000 | 
#  | Karen     |  33  |  5  |  80000  | 
#  | Paul      |  28  |  2  |  60000  | 
#  | Henry     |  25  |  1  |  70000  | 
# +| Susan     |  27  |  0  |  50000  |+

# Select only the Name and Age columns
df_pyspark.select(['Name','age']).show()
# +| Name      |  age |+
#  | Mark      |  35  | 
#  | Christian |  34  | 
#  | Karen     |  33  |
#  | Paul      |  28  |
#  | Henry     |  25  |  
# +| Susan     |  27  |+

# Filter out the rows with salaries below 100k
df_pyspark.filter("Salary>=100000").show()
# +| Name      |  age | YoE | Salary  |+
#  | Mark      |  35  |  11 |  180000 | 
#  | Christian |  34  |  9  |  100000 | 

# Add column to data frame
df_pyspark=df_pyspark.withColumn('Salary After 2 years',df_pyspark['Salary']*1.1).show()
# +| Name      |  age | YoE | Salary  | Salary After 2 years |+
#  | Mark      |  35  |  11 |  180000 |        198000        |
#  | Christian |  34  |  9  |  100000 |        110000        | 
#  | Karen     |  33  |  5  |  80000  |        88000         | 
#  | Paul      |  28  |  2  |  60000  |        66000         | 
#  | Henry     |  25  |  1  |  70000  |        77000         | 
# +| Susan     |  27  |  0  |  50000  |        55000         |+

# Drop column we just added
df_pyspark=df_pyspark.drop('Salary After 2 years').show()
# +| Name      |  age | YoE | Salary  |+
#  | Mark      |  35  |  11 |  180000 | 
#  | Christian |  34  |  9  |  100000 | 
#  | Karen     |  33  |  5  |  80000  | 
#  | Paul      |  28  |  2  |  60000  | 
#  | Henry     |  25  |  1  |  70000  | 
# +| Susan     |  27  |  0  |  50000  |+

# Rename column
df_pyspark.withColumnRenamed('age','Age').show()
# +| Name      |  Age | YoE | Salary  |+
#  | Mark      |  35  |  11 |  180000 | 
#  | Christian |  34  |  9  |  100000 | 
#  | Karen     |  33  |  5  |  80000  | 
#  | Paul      |  28  |  2  |  60000  | 
#  | Henry     |  25  |  1  |  70000  | 
# +| Susan     |  27  |  0  |  50000  |+

# Writing the dataframe to a file
df.write.option("header",True)
	.option("delimiter",";")
	.format("csv")
	.mode("overwrite")
	.save("data/example_1.csv")
```

### How to test?
Apache spark comes with a built in testing library.  Here's an example from the official [docs](https://spark.apache.org/docs/latest/api/python/getting_started/testing_pyspark.html).
```python
import pyspark.testing
from pyspark.testing.utils import assertDataFrameEqual


# Example 1
df1 = spark.createDataFrame(data=[("1", 1000), ("2", 3000)], schema=["id", "amount"])
df2 = spark.createDataFrame(data=[("1", 1000), ("2", 3000)], schema=["id", "amount"])

assertDataFrameEqual(df1, df2) # pass, DataFrames are identical

# Example 2
df1 = spark.createDataFrame(data=[("1", 0.1), ("2", 3.23)], schema=["id", "amount"])
df2 = spark.createDataFrame(data=[("1", 0.109), ("2", 3.23)], schema=["id", "amount"])

assertDataFrameEqual(df1, df2, rtol=1e-1) # pass, DataFrames are approx equal by rtol
```

These are good for one off tests, but if you are going to be building a whole test suite you will need to incorporate a proper test library like `pytest`. 

Luckily `pytest` integrates directly with PySpark, the biggest plus is being able to write fixtures that automatically spin up and tear down resources. 
```python
import pytest

@pytest.fixture
def spark_fixture():
    spark = SparkSession.builder.appName("Testing PySpark Example").getOrCreate()
    yield spark
```

The rest of the tests can be written in the same way as a regular pytest.
```python
import pytest
from pyspark.testing.utils import assertDataFrameEqual

def test_single_space(spark_fixture):
    sample_data = [{"name": "John    D.", "age": 30},
                   {"name": "Alice   G.", "age": 25},
                   {"name": "Bob  T.", "age": 35},
                   {"name": "Eve   A.", "age": 28}]

    # Create a Spark DataFrame
    original_df = spark.createDataFrame(sample_data)

    # Apply the transformation function from before
    transformed_df = remove_extra_spaces(original_df, "name")

    expected_data = [{"name": "John D.", "age": 30},
    {"name": "Alice G.", "age": 25},
    {"name": "Bob T.", "age": 35},
    {"name": "Eve A.", "age": 28}]

    expected_df = spark.createDataFrame(expected_data)

    assertDataFrameEqual(transformed_df, expected_df)
```
When you run the `pytest` command the tests will run just as with any other pytest test suite. 

