---
dg-publish: true
tags:
  - Process
  - Scripts
---

## What is Make?

GNU Make is a command line tool which was developed to provide an easy way to compile and recompile code in the era of c/c++. It is still used to compile more complicated applications written in compiled programming languages. 

It does this by providing an easy way to create alias functions that can encompass multiple different commands, with variables and logic built in. 

All these commands can simply be written to a `Makefile` and triggered with the `make` command. 

#### Hello World? 

For example, you can use it to run echo. So let's try writing a simple hello world. 

```Bash
hello_world:
	@echo "Hello World!"
```

To run it:
```Bash
make 
```
or 
```Shell
make hello_world
```

The `make` command executes the first command in the file if not told which command to execute. By adding the specific command it triggers only the specified one.

## Why use it?

Well, it doesn't just help compile code, it executes pretty much any command that can run in the terminal. 

In my projects, I use it to replace more verbose commands that I need to regularly run. 

For example, I use two docker-compose files, one for production and one for testing, the one used for testing is named `docker-compose-test.yml` and to run it I would need to run the following command. 

```Shell
docker-compose -f docker-compose-test.yml up -d --build
```

This is annoying to type and a bit of an eyesore, I'd rather just run `make test` instead. 

```
hello_world:
	@echo "Hello World!"
test:
	docker-compose -f docker-compose-test.yml up -d --build
```

And that's that, problem solved. 
## Exactly how useful is it? 

If the above example wasn't convincing enough I did a little experiment to show the real-world effects of replacing all of my common commands using make. 

Here's my personal `Makefile` that I use for work, it includes most of the commands that I commonly use at least once a day. 

```Bash
plan:
	terraform validate
	terraform fmt
	terraform plan

apply:
	terraform apply

destroy:
	terraform destroy

build:
	docker build -t image_name .

run:
	docker run -p 8080:8080 image_name
	
test:
	docker build -t image_name .
	docker run -p 8080:8080 image_name & \
```

This makes development more ergonomic and saves me a little bit of time. 

I did the math to check approximately how much time this saves me every month. This past week I have been doing a lot of work with Terraform, so I have had to use the `validate`, `fmt` and `plan` commands a lot. I tried counting but I lost tracks somewhere around 20. 

I wondered how much time I saved by having the make file versus using the commands separately, so I measured it. 

Results:
-  `make plan`  =~ 12s
-  `terraform validate/fmt/plan` =~ 20s

So assuming I use them an average of 20 times in a working day that is around 3 minutes per day. Weekly that is 15 minutes, monthly around 45 minutes. That's an extra hour of work every month done by a super simple script you can create in 5 minutes.  