---
dg-publish: true
tags:
  - DE_ZOOMCAMP_2024
  - Snowflake
  - Data_Loading
  - Terraform
  - IaC
---

## What is Terraform?

Terraform is a Infrastructure as Code (IaC) tool developed by HashiCorp. It allows users to provision and control their infrastructure using yaml files. 
It does the same thing that something like an aws cdk script would do (creating, updating, deleting resources, maintaining state), but with much less hassle. It's also platform agnostic, it supports all major cloud providers (AWS, Azure, GCP) as well as services like Snowflake, Docker, Kafka, etc 

Here's a very good TLDR:
<iframe width="560" height="315" src="https://www.youtube.com/embed/tomUWcQ0P3k?si=cOzAcTGnP4bgfV9M" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

## What is Snowflake?

Snowflake is cloud based data warehousing platform, similar in concept to Google's BigQuery and AWS Redshift. It's differentiated by being able to integrate with data stores across all of the major cloud platforms and having a very versatile architecture. 

Snowflake databases are virtual hard drives where the user stores data, while Snowflake Warehouses are virtual compute instances that run the analytical queries. They are largely independent of one another, meaning you can have a lot of storage and little compute or vice versa, no need to get a bigger RDS instance. They are also charged separately. 

Here's a very good TLDR (sadly Fireship hasn't covered it 😢 ... yet!):
<iframe width="560" height="315" src="https://www.youtube.com/embed/9PBvVeCQi0w?si=bZ7oBc-3z_P6G95-" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>


## Let's set up a Data Warehouse

#### The Terraform setup

Make sure you have terraform installed on your local machine. 
The terraform website provides [instructions](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli) for all the supported operating systems. 

I also recommend installing [tfenv](https://github.com/tfutils/tfenv) it's a terraform version manager. Using it you can install different versions of terraform and easily switch between them. 
This is going to be very useful when you encounter errors that require downgrading to fix. 
#### The Snowflake setup

To setup Snowflake to work with terraform, we are going to need to set up a service account. 
This is a must because we are going to be running terraform and therefor also Snowflake via CD pipeline.

To enable this we are going to need to create a Snowflake user and enable that user to log in without us manually logging in. we can do that in one of two ways, by providing terraform with this user with a password or by providing the user with a pair of encryption keys. 

For the purposes of this tutorial I'm going to be using the key pair method because it's less obvious than the password method. 
##### Generating the key pair
Let's start by opening the terminal, then navigate to the directory where the ssh keys are stored. 

```Shell
cd ~/.ssh
```

Once in the ssh directory we can generate the keys, let's start by generating the private key using openssl.

```Shell
openssl genrsa 2048 | openssl pkcs8 -topk8 -inform PEM -out snowflake_tf_snow_key.p8 -nocrypt
```

What this command is doing is generating a 2048 bit private key, converting it to PKCS#8 (industry standard) format, saving it to a file called `snowflake_tf_snow_key.p8` and not encrypting the private key file with a passphrase. 

After this is done we need to use the private key to generate the public key. 
```Shell
openssl rsa -in snowflake_tf_snow_key.p8 -pubout -out snowflake_tf_snow_key.pub
```

This command takes the previously created private key as input and outputs the public key to a file named `snowflake_tf_snow_key.pub`.

##### Creating the Snowflake service account
The service account needs to be created in the Snowflake console, open a SQL worksheet set the user role to `ACCOUNTADMIN`.

To create a service account user enter and run the following script.

```SQL
CREATE USER "tf-snow" RSA_PUBLIC_KEY='RSA_PUBLIC_KEY_HERE' DEFAULT_ROLE=PUBLIC MUST_CHANGE_PASSWORD=FALSE;
```

Replace the `RSA_PUBLIC_KEY_HERE` with the contents of the `snowflake_tf_snow_key.pub` file we generated earlier. 

After creating the service account user, we need to give it the roles it need to create and destroy Snowflake objects.
Run the following SQL commands to assign the `SYSADMIN` and `SECURITYADMIN` roles. 

```SQL
GRANT ROLE SYSADMIN TO USER "tf-snow";
GRANT ROLE SECURITYADMIN TO USER "tf-snow";
```

The service account user is now done, the last set is to retrieve it's account_id and region, this will be needed for authentication and authorization.

The following script will output the account_id and region.
```SQL
SELECT current_account() as YOUR_ACCOUNT_LOCATOR, current_region() as YOUR_SNOWFLAKE_REGION_ID;
```

##### Setting Environmental Variables

To not have to hardcode auth information into the terraform script we need to set the values as environmental variables. We can easily do this in the terminal with the `export` command.
```Shell
export SNOWFLAKE_USER="tf-snow"
export SNOWFLAKE_AUTHENTICATOR=JWT
export SNOWFLAKE_PRIVATE_KEY=`cat ~/.ssh/snowflake_tf_snow_key.p8`
export SNOWFLAKE_ACCOUNT="YOUR_ACCOUNT_LOCATOR"
```

#### The Terraform script

The terraform lives in a file named `main.tf`, it contains the following script.

```YAML
terraform {
	required_providers {
		snowflake = {
			source = "Snowflake-Labs/snowflake"
			version = "~> 0.76"
		}
	}
}

# Providers in this context serve like different roles
provider "snowflake" {
	alias = "sys_admin"
	role = "SYSADMIN"
}
  
# The database (storage)
resource "snowflake_database" "db" {
	provider = snowflake.sys_admin
	name = "TF_DEMO"
}
  
# The warehouse (compute)
resource "snowflake_warehouse" "warehouse" {
	provider = snowflake.sys_admin
	name = "TF_DEMO"
	warehouse_size = "small"
	auto_suspend = 60
}

  
provider "snowflake" {
	alias = "security_admin"
	role = "SECURITYADMIN"
}

# We now add a role that we can assign to out user to allow it your use our resources.

resource "snowflake_role" "role" {
	provider = snowflake.security_admin
	name = "TF_DEMO_SVC_ROLE"
}


resource "snowflake_grant_privileges_to_account_role" "grant" {
	provider = snowflake.security_admin
	on_account_object {
		object_type = "DATABASE"
		object_name = snowflake_database.db.name
	}
	privileges = ["USAGE"]
	account_role_name = snowflake_role.role.name
	with_grant_option = false
}


resource "snowflake_schema" "schema" {
	provider = snowflake.sys_admin
	database = snowflake_database.db.name
	name = "TF_DEMO"
	is_managed = false
}

  
resource "snowflake_schema_grant" "grant" {
	provider = snowflake.security_admin
	database_name = snowflake_database.db.name
	schema_name = snowflake_schema.schema.name
	privilege = "USAGE"
	roles = [snowflake_role.role.name]
	with_grant_option = false
}

  
resource "snowflake_warehouse_grant" "grant" {
	provider = snowflake.security_admin
	warehouse_name = snowflake_warehouse.warehouse.name
	privilege = "USAGE"
	roles = [snowflake_role.role.name]
	with_grant_option = false
}

  
resource "tls_private_key" "svc_key" {
	algorithm = "RSA"
	rsa_bits = 2048
}

  
resource "snowflake_user" "user" {
	provider = snowflake.security_admin
	name = "tf_demo_user"
	default_warehouse = snowflake_warehouse.warehouse.name
	default_role = snowflake_role.role.name
	default_namespace = "${snowflake_database.db.name}.${snowflake_schema.schema.name}"
	rsa_public_key = substr(tls_private_key.svc_key.public_key_pem, 27, 398)
}


resource "snowflake_role_grants" "grants" {
	provider = snowflake.security_admin
	role_name = snowflake_role.role.name
	users = [snowflake_user.user.name]
}
```

#### How to Run the Script

To run the terraform script we need to first initiate the terraform project, we do that running the `init` command in the directory where the `main.tf` file is located. 

```Shell
terraform init
```

This will generate the `.terraform` directory, this is where terraform stores all the resources it needs to run the scripts. It also generates the `terraform.tfstate` file, this is where the state is stored. Make sure to add it to `.gitignore`, the state file contains all the used information including environmental variables and secrets, make sure it's private. 

Before running the script we need to make sure it's correct, we do that by running the `plan` command.

```Shell
terraform plan
```

If everything works (the provider is under-development, the current code might not work), you can use the script by running the `apply`. 

```Shell
terraform apply
```

After this command runs it course, go to the Snowflake console you will be able to see the newly created database and warehouse.

#### The full code

The full code is available [here](https://github.com/Zesky665/de-zoomcamp/blob/feature/terraform/Week_1/terraform/main.tf).

### Next

[[Week 2 - Ingesting Data with Mage]]