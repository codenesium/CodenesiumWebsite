+++
title = "Docker"
description = ""
weight = 1
alwaysopen = true
+++




The files required to run the app in a Docker container are in the solution root directory.

We target Alpine Linux by default to keep the image as small as possible. The default is a self contained app. 

For Windows

1. Install DockerToolbok from https://docs.docker.com/toolbox/toolbox_install_windows/
2. Run the Docker QuickStart Terminal app from the start menu.
3. You may want to stop the default machine in Virtual Box and increase the ram. SQL Server requires 2GB ram and the 
default machine created doesn't have great specs.

##### Running on SQL Server
Run DockerWithSQLServer.bat to run with SQL Server. 

If you're using Docker Toolbox

1. The SQL server will be available at http://192.168.99.100:1433
2. The Web app is available at http://192.168.99.100

To run this example 
```
docker-compose -f docker-compose-sqlserver.yml up
```

##### Running on Postgres
Run DockerWithPostgres.bat to run with Postgres. This will create a container for Postgres and link it to your .NET Core image.

If you're using Docker Toolbox

1. The Postgres server will be available at http://192.168.99.100:5433
2. The Web app is available at http://192.168.99.100

To run this example 
```
docker-compose -f docker-compose-postgres.yml up
```

##### Changing database providers
Change the DatabaseProvider setting in app settings or via environment arguments to MSSQL or POSTRESQL. 


##### DockerFile
```
# Build the React application
FROM node:8 AS jsbuild

# The following arguments are passed on the command line of docker-compose with the -e switch

# Used in the React app so it know where the API is. Useful when using a proxy or hosting the API project a different ip than the React app. 
ARG REACT_APP_API_URL
ENV REACT_APP_API_URL="${REACT_APP_API_URL}"
 # Used when the app is not hosted under the root of the domain like http://sitename.com/subdirectory
ARG REACT_APP_HOST_SUBDIRECTORY 
ENV REACT_APP_HOST_SUBDIRECTORY="${REACT_APP_HOST_SUBDIRECTORY}"

# Copy the source code to the container
COPY . ./ 

# Change to the React source code directory
WORKDIR ESPIOT.Api.TypeScript/src/react

# Install npm packages
RUN npm install

# Build the React application
RUN npm run build-docker

# Build the API
FROM microsoft/dotnet:2.2-sdk-alpine AS build

WORKDIR /

# expose port 80 in the container to the host
EXPOSE 80

# Copy the source code to the container
COPY . ./

WORKDIR /

# Publish the API as a self contained application targeting Linux
RUN dotnet publish ESPIOT.Api.Web/ESPIOT.Api.Web.csproj -c Release  -o /publish -r alpine-x64 --self-contained

# Create the runtime containe
FROM microsoft/dotnet:2.2-runtime-deps-alpine AS runtime

WORKDIR /publish
# Copy the output of the API build
COPY --from=build publish .

WORKDIR /publish/app
# Copy the output of the JS build to the app subdirectory 
COPY --from=jsbuild ESPIOT.Api.TypeScript/src/react/build .

WORKDIR /publish
ENTRYPOINT ["./ESPIOT.Api.Web"]
```
##### docker-compose-sqlserver.yml
```
version: '2'

services:
  app:
    container_name: 'espiot-app'
    image: 'espiot-app'
    build:
      context: .
      dockerfile: DockerFile
    environment:
       - ASPNETCORE_URLS=http://+:80
       - ConnectionStrings__ApplicationDbContext=Server=db;Persist Security Info=False;User ID=sa;Password=Passw0rd;Initial Catalog=espiot;MultipleActiveResultSets=True;Connection Timeout=10;
       - DatabaseProvider=MSSQL
       - MigrateDatabase=true
       - DebugSendAuthEmailsToClient=true
       - REACT_APP_API_URL=${REACT_APP_API_URL}
       - REACT_APP_HOST_SUBDIRECTORY=/
    volumes:
      - .:/var/www/app
    ports:
     - "80:80"
    networks:
      - espiot-network

  db:
    image: "microsoft/mssql-server-linux"
    container_name: 'db'
    environment:
       SA_PASSWORD: "Passw0rd"
       ACCEPT_EULA: "Y"
    ports:
      - "1433:1433"
    networks:
      - espiot-network

  watchtower:
    container_name: 'watchtower'
    image: v2tec/watchtower
    restart: on-failure
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    command: --interval 30
    networks:
      - espiot-network

networks:
    espiot-network:
       driver: bridge
```

##### docker-compose-postgres.yml
```
version: '2'

services:

  app:
    container_name: 'espiot-app'
    image: 'espiot-app'
    build:
      context: .
      dockerfile: DockerFile
    environment:
       - ASPNETCORE_URLS=http://+:80
       - ConnectionStrings__ApplicationDbContext=Host=espiot-db;Persist Security Info=False;User ID=postgres;Password=test;Database=espiot;
       - DatabaseProvider=POSTGRESQL
       - MigrateDatabase=true
       - DebugSendAuthEmailsToClient=true
       - REACT_APP_API_URL=${REACT_APP_API_URL}
       - REACT_APP_HOST_SUBDIRECTORY=/
    volumes:
      - .:/var/www/app
    ports:
     - "80:80"
    depends_on:
     - "db"
    networks:
      - espiot-network

  db:
    container_name: 'espiot-db'
    image: postgres:9.6-alpine
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: test
      POSTGRES_DB: espiot
    networks:
      - espiot-network
    ports:
     - "5433:5432"
    restart: always

  watchtower:
    container_name: 'watchtower'
    image: v2tec/watchtower
    restart: on-failure
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    command: --interval 30
    networks:
      - espiot-network
volumes:
     pgdata:

networks:
    espiot-network:
       driver: bridge
```

##### Setup for AWS Lightsail
```
This setup is for ubuntu,.NET Core and Postgres on AWS LightSail. This will install and run the espiot docker application.
It will set the docker daemon to run on boot. Your containers should start automatically becuase the docker compose
file has the containers set to restart always. 
Watchtower is also set up. Watchtower is a container that polls docker hub for new images and updates your containers sutomatically.

1. Execute from the AWS CLI. You may get errors if you don't execute them one at a time.
aws lightsail create-instances --instance-name "espiot" --availability-zone "us-east-1b" --blueprint-id "ubuntu_18_04" --bundle-id nano_2_0
aws lightsail create-disk --disk-name espiot-data --availability-zone "us-east-1b" --size-in-gb 8
aws lightsail open-instance-public-ports --port-info fromPort=8000,toPort=8000,protocol=TCP --instance-name="espiot"
aws lightsail attach-disk --disk-name espiot-data --instance-name espiot --disk-path /dev/xvdf
aws lightsail allocate-static-ip --static-ip-name "espiot-ip" 
aws lightsail attach-static-ip --static-ip-name "espiot-ip" --instance-name "espiot"
aws lightsail get-instance --instance-name "espiot"

2. SSH into the created instance and execute the script below. You can get to the terminal from the LightSail web view.
This will create a docker-compose.yml file and a setup.sh script that will set up Docker on the machine and
docker-compose up your container. It will also set up the volume you created earlier so that your database can be persisted.

sudo file -s /dev/xvdf && 
sudo mkfs -t ext4 /dev/xvdf && 
sudo mkdir /database &&
sudo mount /dev/xvdf  /database && 
sudo rmdir /database/lost+found && 
sudo apt-get update && 
sudo apt-get -y -f install docker.io && 
sudo apt-get -y -f install docker-compose && 
sudo apt-get -y -f install unzip && 
sudo service docker start && 
sudo systemctl enable docker
docker pull codenesium/espiot && 
sudo docker-compose up;' >> setup.sh;

chmod 755 setup.sh;
sudo sh ./setup.sh;
```

#### NGROK
If you want to use ngrok to expose your app publicly uncomment this line
```
REM <UNCOMMENT TO EXPOSE CONTAINER WITH NGROK> start ngrok http  192.168.99.100:8000
```
Ngrok is a tunnel service that lets you expose your machine from inside a network to the internet.

https://ngrok.com/
