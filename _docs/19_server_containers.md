---
permalink: /server_containers/
title: "Containerized Deployment"
layout: single
toc: false
toc_label: "Contents"
toc_icon: "list"
toc_sticky: true  # Makes the TOC stick on scroll
---

The Sealog-Server and MongoDB database can be deployed as a Docker container. The repo includes a `docker-compose.yaml` and `Dockerfile` to help operators setup deployments.  Utilizing containerization is a great way to evaluatate Sealog or when developing custom code.  Using containers requires [Docker Desktop](https://www.docker.com/products/docker-desktop/) or another container hosting environment.

#### docker-compose.yml
The included `docker-compose.yml.dist` file will build a basic Sealog-Server and MongoDB package ready for evaluation, development or production.  The inline documentation describes how to customize the deployment such as define the access port and setting up TLS (https).

```
services:
  sealog-server:
    build:
      context: .
      dockerfile: Dockerfile
    depends_on:
      - mongo
    environment:
      #- SEALOG_SERVER_PORT=8000
      #- SEALOG_SERVER_FILEPATH_ROOT=/data/sealog-files
      #- SEALOG_SERVER_TLS_PRIVKEY=<privkey.pem>
      #- SEALOG_SERVER_TLS_FULLCHAIN=<fulchain.pem>
      
      # Create SEALOG_SERVER_SECRET with:
      # node -e "console.log(require('crypto').randomBytes(256).toString('base64'));"
      - SEALOG_SERVER_SECRET='<SECRET_TOKEN>'
      
      # NODE_ENV Options are:
      # - production: what to use for realz
      # - development: preloads some events, event templates cruises and lowerings
      # - demo-vessel: preloads some event templates, cruises, and events... good for evaluating Sealog for vessels
      # - demo-vehicle: preloads some event templates, cruises, lowerings and events... good for evaluating Sealog for vehicles
      - NODE_ENV=demo-vessel
      - MONGO_URL=mongodb://mongo:27017/sealogDB
    image: sealog-server
    ports:
      - "8000:8000"
    restart: unless-stopped
    volumes:
      # Make sure this directory exists
      - /data/sealog-files:/data/sealog-files

  mongo:
    image: mongo
    # Uncomment these lines if you need direct access to the database
    # ports:
    #   - "27017:27017"
    volumes:
      - db_data:/data/db

volumes:
  db_data:
```

#### Dockerfile
```
# Use the latest Node.js Alpine image as the base image
FROM node:lts-alpine

# Set the working directory in the container
WORKDIR /usr/src/app

# Copy package.json and package-lock.json to the working directory
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy the rest of the application code to the working directory
COPY . .

# Copy configuration files
COPY config/db_constants.js.dist config/db_constants.js
COPY config/email_settings.js.dist config/email_settings.js
COPY config/manifest.js.dist config/manifest.js
COPY config/server_settings.js.dist config/server_settings.js
COPY config/secret.js.dist config/secret.js

# Expose port 8000 (This needs to match what's in the docker-compose.yml file)
EXPOSE 8000

# Command to run the Node.js application
CMD [ "node", "server.js" ]
```

### Usage
To use/customize the deployment, first copy/rename the distributed versions.
```
cd /opt/sealog-server
cp docker-compose.yaml.dist docker-compose.yaml
cp Dockerfile.yaml.dist Dockerfile.yaml
```

Next you will need to create a `SECRET_TOKEN` and add it to the `docker_compose.yml` file.  The command to create the token is included as a comment in the `docker_compose.yml` file. 

Next build and start the container.
```
docker-compose build
docker-compose up
```

If there were no problems with configuring and building the container then there should be a working instance of Sealog-Server available on port 8000.
