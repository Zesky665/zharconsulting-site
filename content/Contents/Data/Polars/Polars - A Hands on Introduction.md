---
tags:
  - Python
  - Scripts
  - Polars
dg-publish: true
---

## What is Polars?
Polars is a modern version of Pandas, written completely in Rust. It was meant to address the complaint people had when using pandas, namely it's high memory usage and slow queries. 

Polars biggest differences versus Pandas:
- Polars is internally represented using *Apache Arrow arrays* versus Pandas NumPy arrays. *Apache Arrow* was developed for analytic workflows in columnar in-memory datasets. This means that it's memory efficient and fast.
	- **Warning**: This means that is is efficient at using memory, not that it uses less memory. If you use the `estimated_size()` function to see the memory usage at rest for a polars dataframe and compare it with a pandas dataframe there will not be a significant difference, with the polars dataframe maybe even being bigger. This is expected, not a bug. 
- Polars is made with Rust, which is famous for being memory safe and ideal for parallelization. This means that running parallel jobs in Polars is a breeze. 
- Polars uses lazy evaluation, which means that it optimizes queries before running them, as opposed to Pandas which uses *eager evaluation* meaning that it immediately runs queries. Polars supports both eager and lazy evaluation, eager is the default for most function, but can be overridden using the .lazy() function. 
- Polars due to it's lazy features can be very quick and memory efficient, since it automatically optimizes queries, it will apply batch processing to large data without us having to manually write code to support it. It also has an option to allow for streaming, allowing memory efficient processing of large files. 

### How to set up?
Setting up polars is pretty simple, just install the `polars` package with your package manager of choice. 

### How to create a dataframe?
Creating a dataframe is not much different from how it is in Pandas. 
```
import polars as pldf = pl.DataFrame(  
     {  
         'id': [1,2,3,4,5],  
         'color': ["red", "blue", "red", "yellow", "oragne"],       
     }  
)  
df
```

### How does it compare to Pandas?
From my personal experience it's pretty similar in terms of developer experience. 

Best way to demonstrate is to show how it us used in a common use case, like wrangling data. 

I'm going to be using the [USDA Branded Foods dataset](https://fdc.nal.usda.gov/fdc-app.html#/food-search?type=Branded&query=), because it's big and very normalized. 

This example isn't going to be diving deep into polars, just demonstrating how it's used for doing a relatively simple task. Since that is what pandas is used for most of the time. 

### Setup
Make sure to download the dataset before this, there are instructions in the notebook linked at the bottom of the article, I omitted it here for the sake of brevity. 
```python
import polars as pl

# Read the food table
food = pl.read_csv("brand_file/FoodData_Central_branded_food_csv_2024-04-18/food.csv", infer_schema_length=0)

food.head(5)
_______________
  

['fdc_id', 'data_type', 'description', ... 'food_category_id', 'publication_date', 'market_country', 'trade_channel', 'microbe_data']
```

```python
#Get the needed cloumns
food_df = food.select(["fdc_id", "description"])
```
Then to get the serving size information from a separate csv. 
```python
# Read the branded food table

branded_food = pl.read_csv("brand_file/FoodData_Central_branded_food_csv_2024-04-18/branded_food.csv", infer_schema_length=0)

branded_food_df = branded_food.select(["fdc_id", "brand_owner", "serving_size", "serving_size_unit"])
```
Now we need to merge serving_size and serving_size_unit. 
```python
# Merge serving size with serving measurement
branded_food_df = branded_food_df.with_columns(
	pl.concat_str([
	pl.col("serving_size"),
	pl.col("serving_size_unit")
	], separator="").alias("serving")
)

# Drop the unneeded columns
branded_food_df = branded_food_df.drop(["serving_size", "serving_size_unit"])
```
Now we need to merge the first two dataframes into one using the merge function. 
```python
# Merge the food and branded food tables
merged_df = food_df.join(branded_food_df, on="fdc_id", how="inner")
```
Now that we have the food names and the serving sizes we can start working on the nutrition data. 
```python
# Read the nutrient table
nutrient = pl.read_csv("brand_file/FoodData_Central_branded_food_csv_2024-04-18/nutrient.csv", infer_schema_length=0)
```
We're only looking for macro nutrients so we will filter for those.
```python
# Get only the id and name columns
nutrient_df = nutrient.select(["id", "name"])

# Filter out all of the micronutrients
nutrient_df = nutrient_df.filter(nutrient_df['name'].is_in(["Protein", "Total lipid (fat)", "Carbohydrate, by difference", "Energy"]))

# Remove some junk data
nutrient_df = nutrient_df.filter(nutrient_df['id'] != 1062)
```
The food and nutrient tables are connected via a pivot table called food_nutrients, to merge them we will need that one too. 
```python
# Read the food nutrient table
food_nutrient = pl.read_csv("brand_file/FoodData_Central_branded_food_csv_2024-04-18/food_nutrient.csv", infer_schema_length=0)

# Select the needed columns
food_nutrient_df = food_nutrient.select(["fdc_id", "nutrient_id", "amount"])

# Remove junk data
food_nutrient = food_nutrient.filter(food_nutrient['nutrient_id'] != 1062)
```
We can now start merging all of the tables, first let's merge the nutrients table to the food_nutrients table.
```python
# Merge the nutrient and food nutrient tables
merged_nutrient_df = nutrient_df.join(food_nutrient_df, left_on="id", right_on="nutrient_id", how="inner")
```
Now that we have these merged, a problem becomes obvious. 
```python
merged_nutrient_df.head(5)
___________
shape: (5, 4)

|id|name|fdc_id|amount|
|---|---|---|---|
|str|str|str|str|
|---|---|---|---|
|"1008"|"Energy"|"1105904"|"867.0"|
|"1005"|"Carbohydrate, by difference"|"1105904"|"0.0"|
|"1004"|"Total lipid (fat)"|"1105904"|"93.33"|
|"1003"|"Protein"|"1105904"|"0.0"|
|"1005"|"Carbohydrate, by difference"|"1105905"|"0.42"|
```
The data is in the wrong shape to merge with the food table, we need the different nutrients to be columns. 
We can fix this by pivoting the dataframe using the `.pivot()` function. 
```python
merged_nutrient_df = merged_nutrient_df.pivot(index="fdc_id", columns="name", values="amount", aggregate_function="first").rename({
	"Energy": "Energy",
	"Protein": "Protein",
	"Total lipid (fat)": "Fat",
	"Carbohydrate, by difference": "Carbs"
})

merged_nutrients_df.head(5)
______________________________
shape: (5, 5)

|fdc_id|Energy|Carbs|Fat|Protein|
|---|---|---|---|---|
|str|str|str|str|str|
|---|---|---|---|---|
|"1105904"|"867.0"|"0.0"|"93.33"|"0.0"|
|"1105905"|"4.0"|"0.42"|"0.0"|"0.83"|
|"1105906"|"82.0"|"6.12"|"5.31"|"2.45"|
|"1105907"|"82.0"|"5.31"|"6.12"|"1.22"|
|"1105908"|"4.0"|"0.42"|"0.0"|"0.83"|
```
Now that that's taken care of, we can merge the food and nutrients tables into one big table. 
```python
# Merge the merged_df and merged_nutrient_df tables
final_df = merged_df.join(merged_nutrient_df, on="fdc_id", how="inner")

final_df.head(5)
_______________________
shape: (5, 8)

|fdc_id|description|brand_owner|serving|Energy|Carbs|Fat|Protein|
|---|---|---|---|---|---|---|---|
|str|str|str|str|str|str|str|str|
|---|---|---|---|---|---|---|---|
|"1105904"|"WESSON Vegetable Oil 1 GAL"|"Richardson Oilseed Products (U…|"15.0ml"|"867.0"|"0.0"|"93.33"|"0.0"|
|"1105905"|"SWANSON BROTH BEEF"|"CAMPBELL SOUP COMPANY"|"240.0ml"|"4.0"|"0.42"|"0.0"|"0.83"|
|"1105906"|"CAMPBELL'S SLOW KETTLE SOUP CL…|"CAMPBELL SOUP COMPANY"|"440.0g"|"82.0"|"6.12"|"5.31"|"2.45"|
|"1105907"|"CAMPBELL'S SLOW KETTLE SOUP CH…|"CAMPBELL SOUP COMPANY"|"440.0g"|"82.0"|"5.31"|"6.12"|"1.22"|
|"1105908"|"SWANSON BROTH CHICKEN"|"CAMPBELL SOUP COMPANY"|"240.0ml"|"4.0"|"0.42"|"0.0"|"0.83"|
```
Then we can write it to parquet for later use. 
```python
final_df.write_parquet("usda_branded.parquet")
```

