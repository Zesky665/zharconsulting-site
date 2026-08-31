---
dg-publish: true
tags:
  - Docker
  - Docker-Compose
---

## Step 1: Log in
`docker login`

## Step 2: Build image
Navigate to dockerfile repo then run. 
`docker build -t image_name .`

## Step 3: Tag Image
`docker tag image_name:latest username/image_name:tag`

## Step 4: Push Image
`docker push username/image_name:tag`