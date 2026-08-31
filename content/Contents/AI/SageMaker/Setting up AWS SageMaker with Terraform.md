---
dg-publish: true
tags:
  - AI/ML
  - IaC
  - MLOps
  - Terraform
  - AWS
  - SageMaker
---

## What is AWS SageMaker?

AWS SageMaker is a fully managed service that is essentially a one-stop shop for all the different applications that are needed to set up Machine Learning development environments, experiments and deploying models. 

The biggest benefit is that it removed the need to have multiple different apps like kubeflow and mlflow set up and all the boilerplate needed to make those work in an MLOPs workflow. 

In this article, I'll explain how to set up a bare-bones development environment in SageMaker using Terraform to orchestrate. 

## Basic setup

It all starts with a `main.tf` file. 

```yaml
terraform {
	required_providers {
		aws = {
			source = "hashicorp/aws"
			version = "5.38.0"
		}
	}
}

### Everything else is down here
```

There are two SageMaker-specific resources that we are going to set up `domain` and `user-profile`.

But before we can do that we need to set up some boilerplate, namely `vpc`, `subnet`, `security_group` and `role`. 

Let's start with a role.
```yaml
// BEGIN: Create a simple role
	resource "aws_iam_role" "simple_role" {
	name = "simple_role"
	path = "/"
	assume_role_policy = data.aws_iam_policy_document.simple_role.json
	managed_policy_arns = ["arn:aws:iam::aws:policy/AmazonSageMakerFullAccess"]
}

  

data "aws_iam_policy_document" "simple_role" {
	statement {
		actions = ["sts:AssumeRole"]
		principals {
			type = "Service"
			identifiers = ["sagemaker.amazonaws.com"]
		}
	}
}
// END: Create a simple role
```

This is a very simplistic role, for the production environment you will need to apply the [principle of least privilege](https://en.wikipedia.org/wiki/Principle_of_least_privilege).

Next, we are adding a `vpc` using the [`aws_default_vpc`](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/default_vpc) resource, this calls the `default vpc` that exists in every account. If the default `vpc` was deleted it will be recreated. 

```yaml
// BEGIN: Create a simple vpc
resource "aws_default_vpc" "simple_vpc" {
	tags = {
		Name = "Default VPC"
	}
}
// END: Create a simple vpc
```

Now that we have the `vpc` we can add the `security groups`, which we are going to set up using default resources. 
```yaml
// BEGIN: Create a simple subnet
resource "aws_default_subnet" "simple_subnet" {
	availability_zone = "eu-central-1c" # substitute for your own region
	tags = {
		Name = "Default subnet for eu-central-1"
	}
}
// END: Create a simple subnet

// BEGIN: Create a simple security group
resource "aws_default_security_group" "simple_security_group" {
name_prefix = "simple-security-group"
vpc_id = aws_default_vpc.simple_vpc.id

	ingress {
		protocol = -1
		self = true
		from_port = 0
		to_port = 0
	}
	
	egress {
		from_port = 0
		to_port = 0
		protocol = "-1"
		cidr_blocks = [aws_default_subnet.simple_subnet.cidr_block]
	}
}
// END: Create a simple security group
```

Again, these are only for show, in production create proper `security groups` that don't allow everything to go in and out. 

Now we can start with setting up SageMaker itself. 

First, we need to create the `domain`, the domain is best through a project folder. It's the top-level entity that contains all the users, apps(notebooks) and models that go into a machine-learning project. 

```yaml
// BEGIN: Create a SageMaker Domain
resource "aws_sagemaker_domain" "simple_domain" {
	domain_name = "simple-domain"
	auth_mode = "IAM"
	vpc_id = aws_default_vpc.simple_vpc.id
	subnet_ids = [aws_default_subnet.simple_subnet.id]
	
	default_user_settings {
			execution_role = aws_iam_role.simple_role.arn
		}
}
// END: Create a SageMaker Domain
```

Now that the domain is created we can add the `user-profiles`.

The `user-profile` is the SageMaker lever `iam-user`, it can use roles and be given exclusive access to certain resources within the SageMaker domain. 

```yaml
// BEGIN: Create a SageMaker UserProfile
resource "aws_sagemaker_user_profile" "simple_user_profile" {
	domain_id = aws_sagemaker_domain.simple_domain.id
	user_profile_name = "simple-user-profile"
	user_settings {
		execution_role = aws_iam_role.simple_role.arn
	}
}
// END: Create a SageMaker UserProfile
```

The SageMaker Studio, now SageMaker Studio Classic is created automatically once the `domain` and `user-profile` are created. The default instance type is `ml.t3.medium`, to use a larger one you have to enable the larger instances in the Service Quotas and then set the default instance type to the one you want. SageMaker supports CPU and GPU clusters, here's the [list](https://docs.aws.amazon.com/sagemaker/latest/dg/notebooks-available-instance-types.html). 

When using those instances make sure to assign it an image that is optimized for the workload, you can find the managed SageMaker images [here](https://docs.aws.amazon.com/sagemaker/latest/dg/notebooks-available-images.html#notebooks-available-images-arn).


The SageMaker > Studio Classic console view should look like this. 

![Studio Classic](https://i.imgur.com/GbXZjgT.jpeg)




Now that we've set up all of the basics we can spin the app up, using `terraform`.

First, we need to initialize the project.
```Shell
terraform init
```

The format it.
```Shell
terraform fmt
```

Run `plan` to stage the changes. 
```Shell
terraform plan
```

Apply to create the resources. 
```Shell
terraform apply
```

To clean up the project, run the `destroy` command.

```Shell
terraform destroy
```


### Notes:

The AWS Provider used in this tutorial wasn't very stable, it's entirely possible that the code above won't work in a couple of weeks. 
I'll check in next month to see if a more stable version exists by that time. 
