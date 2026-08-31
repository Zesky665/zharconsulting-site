---
dg-publish: true
tags:
  - Docker-Compose
  - Docker
  - SQL
  - Business-Intelligence
  - Metabase
aliases:
  - Metabase Introduction
  - Metabase Local Setup
  - Metabase Tutorial
---

## What is Metabase?

Metabase is an open-source business intelligence and data analytics application that allows users to easily connect their data sources and visualize their queries. It's design allows for both technical and non-technical users to create complex queries and charts. These charts can then be organizes into dashboard and shared with others, even those without Metabase accounts. 

## Why Metabase?

Metabase unlike other popular BI tools is open-source and is build from the ground up to be self-hostable. This means that you can run it anywhere you have data, including your local machine. It's also free of any license costs if self-hosted. 

It is also designed to be easy to use, so it is idea for data self-service or for organizations that need data but don't have the resources to hire a full time BI professional. 

## How to set up?

Local set up is actually pretty easy. 

All that is required is a docker-compose file. 

For just Metabase all that is needed is the following.
```yaml
version: '3.8'

services:

metabase:
	image: metabase/metabase:latest
	ports:
		- "3000:3000"
	volumes:
		- metabase_data:/metabase-data

volumes:
	metabase_data:
```

With this we get an instance of Metabase with a local volume for persistence. 

For the purposes of this example I'll also add a Postgres db in the same compose file. 

```yaml
version: '3.8'

services:
 db:
	image: postgres:15
	environment:
		POSTGRES_USER: myuser
		POSTGRES_PASSWORD: mypassword
		POSTGRES_DB: mydatabase
	ports:
		- "5432:5432"
	volumes:
		- postgres_data:/var/lib/postgresql/data
		- ./init:/docker-entrypoint-initdb.d
		- ./data:/data
	
	healthcheck:
		test: ["CMD-SHELL", "pg_isready -U myuser -d mydatabase"]
		
		interval: 10s
		timeout: 5s
		retries: 5

  metabase:
	image: metabase/metabase:latest
	depends_on:
		db:
			condition: service_healthy
	ports:
		- "3000:3000"
	volumes:
		- metabase_data:/metabase-data

volumes:
	postgres_data:
	metabase_data:
```
This is not as simple, so let's walk through it. 

The Postgres instance contains:
- A default postgres user.
- A port.
- A series of volumes.
- A healthcheck. 

The most interesting part here is the volumes.

The `/var/lib/postgresql/data` volume contains the database, this allows the db to be stored outside of the container, allowing it to persist between docker runs. 

The `./init:/docker-entrypoint-initdb.d` volume is linked to a local `init` folder, which contains a series of sql files. These run on init, allowing us to add stuff like tables and users automatically. You can see the exact files [here](https://github.com/Zesky665/Metabase_Project/tree/main/init).

The `./data:/data` volume is also connected to a local `data` folder, this is where we can store data files, like csv files. These can be used by the init scripts to automatically load data into the database.

For the purposes of this example I used some public domain datasets from kaggle, specifically the [MoMa collection inventory](https://www.kaggle.com/datasets/momanyc/museum-collection/data). 

The Metabase instance here waits for the Postgres container to be ready before it starts up. Which is when the Postgres healthcheck passes. 

After you copy the docker compose file, run:
```
docker compose up -d
```

And once it has finished starting, go to http://localhost:3000/. 

There you will need to fill out a set up screen, this is where you create a Metabase user and enter the postgres connection details. 

## How to create dashboard?

Let's start by creating a dashboard.

Start by clicking on the "+ New" button and select "Dashboard". 

![Create New Dashboard](https://i.imgur.com/RGPladV.png)

Then chose a name for the new dashboard and select "Create".

The new dashboard will looks like this.
![New Dashboard View](https://i.imgur.com/5SS0gBu.png)

Now that the dashboard exists we can fill it with charts. 

## How to create charts?

Creating a chart is likewise pretty simple. 

All that is necessary is to select your data source, some simple aggregation logic and your preferred chat type. 

Let's demonstrate this by creating a dashboard for our artists. It will include a total count of the artists, their gender ratio and nationality.

Let's start by adding artist count, this starts by selecting the "+ New" button again and selection "Question".
![Create New Question](https://i.imgur.com/fklyI3w.png)

Then select a primary data source, in this case, it's the artist table we populated in the startup step.

![Pick starting data](https://i.imgur.com/VVLKHyB.png)

Once the question GUI loads, we only need to go to the green "Summarize" section and pick the "Count" function. This will count every row in the artists table. 

![Count artists](https://i.imgur.com/GWz8Z3n.png)

Now we only need to pick the visualization option by clicking the "Visualize" button. 

What pops up now is the default visualization option, which is just a number. 

![Artists count Visualization](https://i.imgur.com/ZeZWPHa.png)

That is it, now we only need to save it and add it to a dashboard. This is done by pressing the "Save" button in the top right corner. 

This pop up will ask if you want to add a description or save it to a specific collection. We are going to leave it as is. But this will be very important later on when we have dozens if not hundreds of questions. 
![Save new Question](https://i.imgur.com/VRmiFRx.png)

The next pop up will ask you if you want to add the question to a dashboard.
![Save to dashboard](https://i.imgur.com/iXJkZef.png)

Select "Yes please!" 

And then select out "Artists Data" dashboard. 
![Select dashboard for new chart](https://i.imgur.com/zJ3jNKJ.png)

The next screen is the dashboard editor, it will contain the new question. 
![Dashboard save](https://i.imgur.com/1p9fUQM.png)

Here you can drag, drop and resize all of the questions. 

The only thing left now is to click "Save". 

The resulting dashboard looks like this.
![Saved dashboard with artist count](https://i.imgur.com/cBJ0LMr.png)

Now, for the sake of brevity, I will only show the query editor and the visualizations options for the next two questions/charts. 

###### Gender Ratio

Query
![Gender Ration Query](https://i.imgur.com/NM1gGKx.png)

Visualization
![Gender Ration Chart](https://i.imgur.com/h1qXekA.png)


###### Nationality

Query
![Artist Nationality Query](https://i.imgur.com/J3gXAP7.png)

Visualization
![Artist nationality visualization](https://i.imgur.com/oEFG9Lm.png)

## How to share charts? 

After following the previous steps you should be left with a dashboard that looks something like this.
![Completed Dashboard](https://i.imgur.com/XhatfIF.png)

Wonderfull, isn't it. 

Now we want to share out wonderful dashboard with our colleagues and customers. We can do that simply by using the share button. 

![Dashboard Share button](https://i.imgur.com/3VxhacS.png)

This will give up a link to a page where the dashboard can be viewed. 

Unfortunately since we are running this on our local machine, only people on out local network will be able to access it. And to enable that you will need to find the local ip address and substitute it for the `localhost` in the share link. 

You can get the local ip address with the following commands based on your OS:
```bash
# Windows 
ipconfig | findstr IPv4 

# Linux 
hostname -I 

# Mac 
ipconfig getifaddr en0
```

The result will looks something like this:
```bash
http://193.169.10.11:3000/public/dashboard/79974b67-273d-6723-816d-3640ff0dcffb
```

If you correctly substituted the ip address you should able to open the dashboard on your phone. 

To enable it to be shared over the internet, you will need to deploy it to the cloud and set up the network to allow for sharing. But that is outside of the scope of this tutorial. 