---
permalink: /client_containers/
title: "Containerized Deployment"
layout: single
toc: false
toc_label: "Contents"
toc_icon: "list"
toc_sticky: true  # Makes the TOC stick on scroll
---

The Sealog-Client can be deployed as a Docker container. The repo includes a `docker-compose.yaml` and `Dockerfile` to help operators setup deployments.  Utilizing containerization is a great way to evaluatate Sealog or when developing custom code.  Using containers requires [Docker Desktop](https://www.docker.com/products/docker-desktop/) or another container hosting environment.

#### docker-compose.yml
The included `docker-compose.yml.dist` file will build a basic Sealog-Client package ready for evaluation or production.
```
services:
  sealog-client-vessel:
    image: sealog-client-vessel
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8081:80"
```

#### Dockerfile
```
# Use an official Node runtime as a parent image
FROM node:lts-alpine as build

# Set the working directory to /app
WORKDIR /app

# Copy the package.json and package-lock.json to the container
COPY package*.json .

# Install dependencies
RUN npm install

# Copy the rest of the application code to the container
COPY . .
COPY src/client_settings.js.dist ./src/client_settings.js
COPY src/map_tilelayers.js.dist ./src/map_tilelayers.js
COPY webpack.config.js.dist ./webpack.config.js

# Build the React app
RUN npm run build

# Use an official Nginx runtime as a parent image
FROM nginx:stable-alpine

# Copy the ngnix.conf to the container
COPY nginx/nginx.conf.dist /etc/nginx/conf.d/default.conf

# Copy the React app build files to the container
COPY --from=build /app/dist /usr/share/nginx/html/sealog

# Expose port 80 for Nginx
EXPOSE 80

# Start Nginx when the container starts
CMD ["nginx", "-g", "daemon off;"]
```

### Usage
To use/customize the deployment, first copy/rename the distributed versions.
```
cd /opt/sealog-client
cp docker-compose.yaml.dist docker-compose.yaml
cp Dockerfile.yaml.dist Dockerfile.yaml
```

Next build and start the container.
```
docker-compose build
docker-compose up
```

If there were no problems with configuring and building the container then there should be a working instance of the Sealog client available on port 8081.
