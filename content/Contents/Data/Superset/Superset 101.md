---
dg-publish: true
tags:
  - Business-Intelligence
  - Docker-Compose
  - Docker
  - Postgres
  - SQL
aliases:
  - Superset Setup
  - Superset Introduction
  - Superset Tutorial
  - Superset docker-setup
---

## What is Superset?

Apache Superset is an open-source, enterprise-ready business intelligence and data exploration platform. Originally developed at Airbnb, it was later donated to the Apache Software Foundation. Superset provides a feature-rich set of data visualization tools, an interface for SQL queries, and advanced features for data analytics and statistics.

## Why Superset?

Superset unlike other BI tools, comes with a truly overwhelming level of customization. A similar tool like Metabase provides everything needed for data intelligence out of the box in a user friendly package, Superset gives that and everything needed for hyper optimized, use case specific setups. 

This of course comes with a couple of drawbacks, chief of which is the complexity in the setup. 

## How to set up?

Local set up is not as easy as I would like. 

What is required for a very minimal setup is a docker-compose file and a setup-script. 

The following is the content of the docker-compose file.
```yaml
version: '3'
services:
  postgres:
	image: postgres:15
	environment:
	  POSTGRES_USER: myuser
	  POSTGRES_PASSWORD: mypassword
	  POSTGRES_DB: mydatabase
	ports:
	  - "5432:5432"
	volumes:
	  - ./postgres_data:/var/lib/postgresql/data
	  - ./init:/docker-entrypoint-initdb.d
	  - ./data:/data
	
	healthcheck:
	  test: ["CMD-SHELL", "pg_isready -U myuser -d mydatabase"]
	  interval: 10s
	  timeout: 5s
	  retries: 5
	  
  superset:
    init: true
    build:
      context: ./superset
      dockerfile: Dockerfile
      container_name: superset
    volumes:
      - ./superset_data:/app/superset_home
    environment:
      - DATABASE_DB=superset
      - DATABASE_HOST=db
      - DATABASE_PASSWORD=superset
      - DATABASE_USER=superset
      - DATABASE_PORT=5432
    ports:
      - '8080:8088'

  setup:
    image: docker:dind
    container_name: superset_setup
    privileged: true
    depends_on:
      superset:
        condition: service_healthy
    volumes:
      - ./setup.sh:/setup.sh
      - /var/run/docker.sock:/var/run/docker.sock
    command: sh -c "chmod +x /setup.sh && /setup.sh"

volumes:
  postgres_data:
  superset_data:
```

With this we get an instance of Superset with a local volume for persistence. 

This is not a simple setup, so let's walk through it. 

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

The Superset instance contains:
- An init, this helps run the Superset process properly.
- A build setup, this builds the Superset docker image, based on our own dockerfile.
- A volume, this persists the Superset metadata (e.g. charts)
- A port, to enable us to connect to the instance. 

The build step is needed because the Superset images isn't ready to run on it's own, it requires a bit of tweaking before we can run it locally. This is done by running a small script inside of the container before running it. 

Place these locally in a folder named `superset` in the project root.

The dockerfile. 
```dockerfile
FROM apache/superset:3.0.2

USER root
COPY --chmod=777 superset_config.py /app/
ENV SUPERSET_CONFIG_PATH /app/superset_config.py
USER superset

ENTRYPOINT [ "/usr/bin/run-server.sh" ]
```

The python script, `superset_config.py`. 
```python
ENABLE_PROXY_FIX = True
SECRET_KEY = "MyVerySecretKey"
PREVENT_UNSAFE_DB_CONNECTIONS = False
TALISMAN_ENABLED = False
```

The setup container contains:
 - Image, docker-in-docker.
 - Healthcheck, waiting for superset.
 - Volumes, for the setup script.
 - Command, to run the setup script. 

The docker-in-docker, a way to run scripts inside of the docker environment, not recommended for production deployments. You can read a more comprehensive explanation [here](https://gopesh3652.medium.com/running-docker-in-docker-dind-a-comprehensive-guide-1fe2e328020). 

The setup script is the following:
```bash
# Wait for Superset to be ready
sleep 10

docker exec superset superset fab create-admin \
--username admin \
--firstname Superset \
--lastname Admin \
--email admin@localhost.com \
--password secret

docker exec superset superset db upgrade &&
docker exec superset superset init

echo "Done"
```

After you copy the docker compose file, run:
```
docker compose up -d
```

And once it has finished starting, go to http://localhost:3000/. 

## How to connect to Database?

Let's start by connecting to the database. We only need to do this once, the connection will persist with the local setup. 

We do this by clicking the "+" button on the top right corner and selecting "Data > Connect a database".
![Connect to Database](https://i.imgur.com/bc4RhUe.png)

Once the "Connect a database" popups select the database type, in this case PostgreSQL. After enter the credentials from the docker-compose file.
![Database Connection Details](https://i.imgur.com/mzD08tC.png)

## How to create Dashboard?

To create a dashboard, to do this we need to navigate to the dashboard tab and click the "+ Dashboard" button.
![Create Dashboard](https://i.imgur.com/qnQGmvF.png)


The new dashboard will looks like this.
![New Dashboard Screen](https://i.imgur.com/vt2dF76.png)
Now that the dashboard exists we can fill it with charts. But before we can do that we need to create a dataset. 

## How to create a dataset?

Datasets are essentially views that get used as sources for analytics and visualizations. 

To create a dataset navigate to the "Datasets" tab and click "+ Dataset". 
![Create Dataset](https://i.imgur.com/P5318tR.png)

Once on the dataset creating page, enter the details for the 'artists' table. 
![Create Artists Dataset](https://i.imgur.com/TQnbCpe.png)
Afterwards, press the "Create Dataset and Create Chart" button on the bottom right.
## How to create charts?

Creating a chart is likewise pretty simple. Especially since the last section left us on the chart creating page. 
![Create New Chart](https://i.imgur.com/5w9vwo6.png)

This first one will be super simple, we will only be looking at the total number of artists. For this we need the "Big Number" chart type and press "Create Chart". 
![Create Big Number Chart](https://i.imgur.com/uUbhSUl.png)

The next screen is the Chart building screen, this is where we can edit every parameter. For this specific case we only need to chose the metric. 
![Big Number Chart Details](https://i.imgur.com/IEogmTG.png)


The just press 'Save' and choose to add it directly to the dashboard we created earlier. 
![Save Chart Screen](https://i.imgur.com/d7HbOlA.png)

And there we have out first chart.
![Dashboard with Big Number](https://i.imgur.com/xlDbHPZ.png)

Now let's add two more charts, gender ratio and nationality. 

#### Gender Ratio

To do this just go to create a chart, select the artist table and select the "Pie Chart" chart type and input the following parameters. 
![Gender Ratio](https://i.imgur.com/ZlmB8l6.png)


#### Nationality

Nationality is going to be a bit more tricky, in the artists dataset the nationality is represented by a demonym, which can't be used to assign values to a map type chart. Luckily we have a demonyms table in the database we can use. 
Once we create a national_demonyms dataset we can merge it with the artists dataset to create a new dataset. 
To do this, first navigate to the "SQL Lab".
![SQL Lab Tab](https://i.imgur.com/1ceyZZ7.png)

And then enter this query into the editor. 
![Custom Dataset](https://i.imgur.com/K2xrL5n.png)

Once this is done, click "Save dataset" and you will be taken to the chart creating screen, here pick "World Map" chary type and enter the following parameters.
![World Map Chart](https://i.imgur.com/vTTUBVn.png)


### How to share charts? 

After following the previous steps you should be left with a dashboard that looks something like this.
![Completed Dashboard](https://i.imgur.com/vxZiCsy.png)

Wonderfull, isn't it. 

Now we want to share out wonderful dashboard with our colleagues and customers. We can do that simply by using the share button. 
![Share permalink to dashboard](https://i.imgur.com/Dqm3yuJ.png)
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
http://193.169.10.11:8080/public/dashboard/79974b67-273d-6723-816d-3640ff0dcffb
```

If you correctly substituted the ip address you should able to open the dashboard on your phone. 

To enable it to be shared over the internet, you will need to deploy it to the cloud and set up the network to allow for sharing. But that is outside of the scope of this tutorial. 