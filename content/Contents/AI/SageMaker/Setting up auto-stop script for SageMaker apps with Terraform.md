---
dg-publish: true
tags:
  - AI
  - AI/ML
  - AWS
  - MLOps
  - SageMaker
---

### Why is this necessary?

Because the types of instances Data Scientists usually work with like `ml.g4dn.2xlarge` cost \$1.175 per hour. If they accidentally leave it on after their done with their days work, this can result in an extra 19\$ per day. Per month this can add up to 564\$. If the team consists of more than one data scientists the costs can grow even more. An team of for example 4 data scientists regularly failing to turn off their instances would result in upwards of 2256\$ of extra costs per month. 

Of course, turning off the instances is the responsibility of the person using it, but people are fallible and they hopefully have better things to devote their brainpower to than remembering to turn off an instance. 

Luckily there is a relatively easy way to make this problem go away. 
### Auto-Shutdown Script

Before continuing make sure to have `python3` and `boto3` installed and make sure to have a service role/user setup with permissions to access `SageMaker` . 

First, a short explanation of what the script is intended to do. 

AWS SageMaker is a bit of a frankensteins monster of features, it has SageMaker Notebooks, SageMaker Studio Notebooks and SageMaker Studio Classic Notebooks and probably some other notebooks I haven't discovered yet. The difference between these is not going to be obvious to someone who hasn't familiarized themselves with the documentation. For the purposes of this script these differences are important.  

#### SageMaker Notebooks

These are the ones that exist directly on the SageMaker console page. Shutting these off is very easy, you don't even need a script, you just need to configure a lifecycle policy and AWS handles the rest. 

Lifecycle script example:
(Add example here)

### SageMaker (Classic) Studio Notebooks 

These exist under the SageMaker Studio apps, which in turn exist under SageMaker User Profiles which exists under the SageMaker domain. This is all a bit confusing, so let me explain it with a picture. 

![SageMaker Diagram](https://i.imgur.com/OQCAT8Z.png)

So shutting down these notebooks isn't as straightforward as just issuing a single command. 

First we need to figure out which domain we are targeting, then we need to figure out which apps are the type we want to shut down. In the case of the Studio Notebook, the app type is unintuitively named `KernelGateway`. The meaning behind this is pretty self explanatory once you learn what exactly they do. 

The `KernelGateway` provisions a compute instance, then sends commands for the studio notebook to the instance and returns the results to the notebook, essentially it's the entity doing all of the heavy lifting. The interesting thing here is that these instances don't persist any information, so shutting them off is not going to result in data loss. 

Below is a very simple script capable of stuffing down all `KernelGateway` apps in a specified domain. 

```python
import boto3
import json

session = boto3.Session(profile_name='default')
sagemaker = session.client('sagemaker')

domain = "default"

# First, get the domain_id
def get_domain(domain_name):
    response = sagemaker.list_domains()

    for domain in response['Domains']:
        if domain['DomainName'] == domain_name:
            print(domain['DomainId'])
            return domain['DomainId']
    print("Domain not found")
    return 0

# Then get a list of KernelGateway apps
def get_sm_apps(domain_id):
    resp = sagemaker.list_apps(
        MaxResults=10,
        SortOrder='Ascending',
        DomainIdEquals=domain_id
            )

    apps = []

    for app in resp.get('Apps'):
       if app['AppType'] == 'KernelGateway' and app['Status'] == 'InService':
          app = {
                "domain_id": app['DomainId'],
                "user_profile": app['UserProfileName'],
                "app_type": app['AppType'],
                "app_name": app['AppName'],
                  }
          apps.append(app)

    return apps

# Then just delete them
def get_sm_apps(domain_id):
    resp = sagemaker.list_apps(
        MaxResults=10,
        SortOrder='Ascending',
        DomainIdEquals=domain_id
            )

    apps = []

    for app in resp.get('Apps'):
       if app['AppType'] == 'KernelGateway' and app['Status'] == 'InService':
          app = {
                "domain_id": app['DomainId'],
                "user_profile": app['UserProfileName'],
                "app_type": app['AppType'],
                "app_name": app['AppName'],
                  }
          apps.append(app)

    return apps

if __name__ == '__main__':

    print("Get Domain Id")
    domain_id = get_domain(domain)

    print("Get Apps")
    apps = get_sm_apps(domain_id)

    print("Shutdown App")
    for app in apps:
        print(app)
        shutdown_sm_app(app['domain_id'], app['user_profile'], app['app_type'], app['app_name'])
```

If you run this script on your PC, with proper credentials, all of the instances used in your studio apps in the domain you specified will be shut deleted in ~3 minutes. 

This script can easily be set to run periodically inside of a job orchestrator like Mage AI or Airflow. But here we are going to run it using a lambda function, orchestrated with terraform, this will help keep all of the logic in one place. 

### What is Lambda?

Lambda is a service offered by AWS, which when given a code/image will provision a server, run the code and delete itself down once the function exits. This is called serverless computing. 
The user can specify the size and maximum run duration of the lambda. As well as things like runtime and which files it should run from. Alternatively it can also take a docker image hosted on ecr. 

The user is only charged for the time the lambda runs, including 1 million requests and 400k of compute(seconds) as a free tier. 

One of the other major benefits is that it can run inside of a AWS VPC and take on roles like any other AWS service. Which makes it ideal for carrying out simple tasks since unlike scripts being run inside of an ec2 instance, the scripts in Lambda don't need to account for roles with stuff like temporary credentials, since they terminate after running.
### Terraform

Here is my simple setup a lambda function. 

```yaml
// BEGIN: Create the IAM Role for the Lambda Function
resource "aws_iam_role" "simple_lambda_role" {
	name = "simple_lambda_role"
	assume_role_policy = data.aws_iam_policy_document.simple_lambda_role.json
	managed_policy_arns = ["arn:aws:iam::aws:policy/AmazonSageMakerFullAccess"]
}

data "aws_iam_policy_document" "simple_lambda_role" {
statement {
	actions = ["sts:AssumeRole"]
	principals {
	type = "Service"
	identifiers = ["lambda.amazonaws.com"]
		}
	}
}
// END: Create the IAM Role for the Lambda Function


// BEGIN: Create a Lambda Function

data "archive_file" "lambda" {
	type = "zip"
	source_dir = "${path.module}/lambda"
	output_path = "${path.module}/lambda/index.zip"
}

  

resource "aws_lambda_function" "test_lambda" {
	function_name = "auto-shutdown-lambda"
	filename = "${path.module}/lambda/index.zip"
	role = aws_iam_role.simple_lambda_role.arn
	runtime = "python3.10"
	handler = "index.lambda_handler"
	timeout = 60
	source_code_hash = data.archive_file.lambda.output_base64sha256
	depends_on = [aws_iam_role.simple_lambda_role]
}
// END: Create a Lambda Function
```

Once all of this is set up all that is left is to run the terraform script. 
```shell
terraform init
terraform plan
terraform apply
```

Congratulations you have a lambda. 🎉
## So how do we run it?

Lambdas don't run on their own, they need to be triggered somehow. 
Triggers can be things like a manual api call, a cron job or an event (like an new file being added to an s3 bucket). 

In this tutorial I'm going to use a scheduled trigger (cron job) that will be set to run the script every day at 6pm. 

```yaml
resource "aws_cloudwatch_event_rule" "this" {
	name = "data-quality-lambda-schedule"
	description = "Trigger to run data quality checks every day at 20:00 UTC"
	schedule_expression = "cron(0 20 * * ? *)"
}

resource "aws_cloudwatch_event_target" "this" {
	rule = aws_cloudwatch_event_rule.this.name
	target_id = "data-quality-lambda"
	arn = aws_lambda_function.this.arn
}

resource "aws_lambda_permission" "this" {
	statement_id = "AllowExecutionFromCloudWatch"
	action = "lambda:InvokeFunction"
	function_name = aws_lambda_function.this.function_name
	principal = "events.amazonaws.com"
	source_arn = aws_cloudwatch_event_rule.this.arn
}
```

This will run the lambda function every day at 6pm, so hopefully after everyone is done with work for the day. 

If you want to check if the lambda works, you can run it manually with a functional url. To create one head to the lambda in the aws console, select the `Functional URL` tab and press `Create Functional URL`. 

![Functional URL Console Screen](https://i.imgur.com/EBZZtgf.png)

I chose the No Auth option for simplicity, you should chose the auth option if you are using this is a professional environment. 

Once the URL is created, you can trigger it using a API call. 

```bash
aws lambda invoke \
    --function-name auto-shutdown-lambda \
    --cli-binary-format raw-in-base64-out \
    --payload '{ "hello": "World!" }' \
    response.json
```

________

You might be interested in these notes:
- [Setting up AWS Sagemaker](https://www.zharconsulting.com/contents/ai/sage-maker/setting-up-aws-sage-maker-with-terraform/)
- [Setting up Training Pipelines in AWS SageMaker]() (coming soon)
- [Setting up Data Quality Tests with Lambda]()(coming soon)