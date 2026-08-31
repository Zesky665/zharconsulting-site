---
dg-publish: true
---

### What is Mage?

Mage is an orchestration tool in the same vein as Airflow. What makes it different is that unlike Airflow is a general orchestration tool which is often used to run data pipelines, Mage is an orchestration tool purpose built for data pipelines. 

Mage makes writing data pipelines easy and fun. 

It accomplishes this by making the development experience seamless by having most of the common building blocks built in, so that developers can focus on business logic instead of reinventing the wheel with every pipeline. 

### Who is this for?

This is for people who are new to mage and maybe to data engineering in general. I will try to explain everything as I go along, but I assume you have basic knowledge of python programming, docker, git and terminal usage. 

I have been using it for work for the past 6 months. In that time I have developed a decent idea of what a productive setup looks like. Mage despite it's many qualities comes with a few quirks and the following tutorial is going to help you deal with them. It's also fully applicable to scenarios where you have to deploy a mage server instance on a remote VM. 

### Version Control

Why are we starting with version control? 

Because it's the most important thing, in general, but even more-so when using Mage. 
You see we are going to be running Mage inside of a docker container, which means that any code it generates will be stored by default inside of the docker container, in other words it will be ephemeral. If the container shuts down, all out effort will disappear. 
To prevent this we will need to tell docker to use code that lives outside of itself, this is doable. 
Which leads us to another issue. 
This code will then live only on our machine, which for a throwaway hobby project is fine. But as soon as you need to collaborate with another developer or you need to deploy the code to another machine like a remote server, it becomes much more complicated. 
It also makes it difficult to show it off once you are done. 
There is a simple way to kill all of those birds with one stone. Version control, more specifically git with GitHub. 
Simply create a Github repo, and then clone it to the local directory where you intent to place your Mage project. 

You can copy the command from GitHub, it will look like this:
```
git clone REPO_URL LOCAL_FOLDER_NAME
```

### Docker

Now that you have your repo set up we can proceed with the docker setup.

You can start this by creating a `docker-compose.yml` in the root of the new repo.
The file should look like this:
```yml
services:
  mage:
    image: mageai/mageai:latest
    container_name: mage
    command: mage start ${PROJECT_NAME}
    environment:
      USER_CODE_PATH: /home/src/${PROJECT_NAME}
      ENV: dev
    ports:
      - "6789:6789"
    volumes:
      - .:/home/src/
    restart: on-failure:5
    networks:
      - app-network
    stdin_open: true # docker run -i
    tty: true        # docker run -t

  # Optional: PostgreSQL database for Mage metadata
  postgres:
    image: postgres:14
    container_name: mage-postgres
    restart: on-failure:5
    environment:
      POSTGRES_DB: mage
      POSTGRES_USER: mageuser
      POSTGRES_PASSWORD: magepassword
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - app-network

  # Optional: Redis for caching
  redis:
    image: redis:7-alpine
    container_name: mage-redis
    restart: on-failure:5
    ports:
      - "6379:6379"
    networks:
      - app-network

volumes:
  postgres_data:

networks:
  app-network:
    driver: bridge
```

You will need a `.env` file to go along with it:
```env
# Project configuration
PROJECT_NAME=my_mage_project

# Database configuration (if using PostgreSQL)
POSTGRES_DB=mage
POSTGRES_USER=mageuser
POSTGRES_PASSWORD=magepassword
POSTGRES_HOST=postgres
POSTGRES_PORT=5432

# Redis configuration (if using Redis)
REDIS_HOST=redis
REDIS_PORT=6379
```

Then just run it, the mage image will be downloaded, and it will generate it's boilerplate. 

The directory will look something like this once it's done.
![[Pasted image 20250624230419.png]]

From here you can push the new code to your repo like this:
```
git add .
```
```
git commit -m "Mage init"
```
```
git push
```

And you are done with the setup. 

Here is my [repo](https://github.com/Zesky665/mage_local_tut), it is the end result of the steps taken in this tutorial.

