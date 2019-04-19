+++
title = "Docker"
description = ""
weight = 1
alwaysopen = true
+++




The files required to run the app in a Docker container are in the solution root directory.

We target Alpine by default to keep the image s small as possible. The default is a self contained app. 

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

# to run this with docker
# build a container named ESPIOT with 
# docker build -t ESPIOT .
# run with
# docker run -d -p 8000:8000  --add-host="localhost:10.0.2.2" --rm --name espiotapp ESPIOT
# this will run your app exposed on the docker network and will forward port 8000 on the local machine to port 8000 in the container.
# On this machine it's http://192.168.99.100:8000 but it could be different for you.
# with the rm option the container will be deleted when it exits
# running this command will also expose your local network to the docker container so that localhost from
# inside the container will point to your local machine.
# see https://stackoverflow.com/questions/30495905/accessing-host-machine-as-localhost-from-a-docker-container-thats-also-inside-a for
# info about why 10.0.2.2 makes localhost accessible from the container.


FROM microsoft/dotnet:2.2-sdk-alpine AS build
WORKDIR /app
EXPOSE 80

# copy csproj and restore as distinct layers
COPY . ./

WORKDIR /app
RUN dotnet publish ESPIOT.Api.Web/ESPIOT.Api.Web.csproj -c Release  -o /app/out -r alpine-x64 --self-contained


FROM microsoft/dotnet:2.2-runtime-deps-alpine AS runtime
WORKDIR /app
COPY --from=build /app/out .
ENTRYPOINT ["./ESPIOT.Api.Web"]
```
##### docker-compose-sqlserver.yml
```
version: '2'

services:

  web:
    container_name: 'espiot-app'
    image: 'espiot'
    build:
      context: .
      dockerfile: DockerFile
    environment:
       - ASPNETCORE_URLS=http://+:80
       - ConnectionStrings__ApplicationDbContext=Server=db;Persist Security Info=False;User ID=sa;Password=Passw0rd;Initial Catalog=espiotMultipleActiveResultSets=True;Connection Timeout=10;
       - DatabaseProvider=MSSQL
       - MigrateDatabase=true
    volumes:
      - .:/var/www/app
    ports:
     - "80:80"
    depends_on:
     - "db"
    networks:
      - espiot-network
    extra_hosts:
      - "localhost:10.0.2.2"

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

##### docker-compose-sqlserver.yml
```
version: '2'

services:

  web:
    container_name: 'espiot-app'
    image: 'espiot'
    build:
      context: .
      dockerfile: DockerFile
    environment:
       - ASPNETCORE_URLS=http://+:80
       - ConnectionStrings__ApplicationDbContext=Host=espiot-db;Persist Security Info=False;User ID=postgres;Password=test;Database=espiot;
       - DatabaseProvider=POSTGRESQL
       - MigrateDatabase=true
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
You have to presee enter after pasting it to the terminal. 

sudo echo "version: '2'
services:

  web:
    container_name: 'espiot-app'
    image: 'codenesium/espiot'
    build:
      context: .
      dockerfile: DockerFile
    environment:
       - ASPNETCORE_URLS=http://+:80
       - ConnectionStrings__ApplicationDbContext=Host=espiot-db;Persist Security Info=False;User ID=postgres;Password=test;Database=espiot;
       - DatabaseProvider=POSTGRESQL
       - MigrateDatabase=true
    volumes:
      - .:/var/www/app
    ports:
     - "80:80"
    restart: always
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
    volumes:
         - /database/espiot:/var/lib/postgresql/data
  watchtower:
    container_name: 'watchtower'
    image: v2tec/watchtower
    restart: on-failure
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    command: --interval 30

networks:
    espiot-network:
       driver: bridge" >> docker-compose.yml;
   
sudo echo '#!/bin/bash
sudo file -s /dev/xvdf && 
sudo mkfs -t ext4 /dev/xvdf && 
sudo mkdir /database &&
sudo mkdir /database/espiot &&
sudo mount /dev/xvdf  /database/espiot && 
sudo rmdir /database/espiot/lost+found && 
sudo apt-get update && 
sudo apt-get install -y docker && 
sudo apt-get install -y docker-compose && 
sudo service docker start && 
sudo systemctl enable docker  &&
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
