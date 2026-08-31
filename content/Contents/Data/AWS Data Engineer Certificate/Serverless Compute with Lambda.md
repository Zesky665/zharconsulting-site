
### Overview

AWS Lambda is a serverless compute service.

#### AWS Lambda
- Lets you run code without managing servers. 
- It provisions separate standalone instances based on demand, scales very well. 
- Supports scripts in various programming languages:
	- Python
	- Java
	- Node.ja
	- Go
	- etc.
- Use cases:
	- Data Processing tasks on data in Amazon S3 or DynamoDB.
	- Event-Driven ingestion:
		- S3
		- DynamoDB
		- Kinesis
	- Automation: Automate workflows based on event triggers. 
- Advantages:
	- **Scalable**: Scales automatically based on workload.
	- **Cost efficient**: Pay only for what you use. 
	- **Simplicity**: No need to manage infrastructure. 
###### AWS Lambda Layers
Layers are dependency packages that can be added alongside function code. It essentially works like custom libraries/dependencies that can be called from inside the main lambda function. 

These can be added as a layer, which is a separate module from the function itself, allowing them be to shared across multiple functions. 
It can also potentially reduce the size of the package by letting the user only import the code that is needed, this speeding up the function. 