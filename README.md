tasks 30

FROM python:3.9-slim

ENV PYTHONUNBUFFERED=1

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY FILE 

CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]

version: '3.7'

services:
  web:
    build: .
    ports:
      - "8000:8000"
    depends_on:
      - db

  db:
    image: postgres:latest
    environment:
      POSTGRES_USER: myuser
      POSTGRES_PASSWORD: mypassword
      POSTGRES_DB: mydatabase
    ports:
      - "5432:5432"

docker-compose up -d


tasks 31 

FROM node:16-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY FILE 

RUN npm run build

FROM nginx:latest

COPY --from=build /app/dist/angular-app /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]

docker build -t angular-app .
docker run -d -p 80:80 angular-app

tasks 32

FROM openjdk:11-jre-slim

WORKDIR /app

COPY target/*.jar app.jar

CMD ["java", "-jar", "app.jar"]

docker build -t spring-boot-app .
docker run -d -p 8080:8080 spring-boot-app

tasks 33

FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
WORKDIR /app

COPY *.csproj ./
RUN dotnet restore

COPY src ./src
COPY FILE 

WORKDIR /app/src/
RUN dotnet publish -c Release -o out

FROM mcr.microsoft.com/dotnet/aspnet:6.0
WORKDIR /app
COPY --from=build /app/src/out .

ENTRYPOINT ["dotnet", "app.dll"]


FROM ruby:2.7

WORKDIR /app

COPY Gemfile Gemfile.lock ./

RUN bundle install

COPY FILE 

RUN rails db:create db:migrate

EXPOSE 3000

CMD ["puma", "-C", "config/puma.rb"]

version: '3.7'

services:
  web:
    build: .
    ports:
      - "80:80"
    depends_on:
      - app

  app:
    build: .
    ports:
      - "3000:3000"

networks:
  default:
    driver: bridge

    docker-compose up -d

    docker build -t dotnet-app .
docker run -d -p 5000:5000 dotnet-app


tasks 34 

FROM golang:1.17

WORKDIR /app

COPY FILE 

RUN go build -o main

CMD ["/app/main"]

package main

import (
  "fmt"
  "net/http"
)

func main() {
  http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello from Go!")
  })

  fmt.Println("Server listening on port 8080")
  http.ListenAndServe(":8080", nil)
}

docker build -t go-app .
docker run -d -p 8080:8080 go-app

tasks 35

FROM node:16-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY FILE 

RUN npm run build

CMD ["npm", "start"]

docker build -t vue-app .
docker run -d -p 8080:8080 vue-app

tasks 36

FROM openjdk:11-jre-slim

WORKDIR /app

COPY target/*.jar app.jar

CMD ["java", "-jar", "app.jar"]


docker build -t spring-boot-app .
docker run -d -p 8080:8080 spring-boot-app

tasks 37

FROM node:16-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

RUN npm run build

FROM nginx:latest

COPY --from=build /app/dist/angular-app /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]

docker build -t angular-app .
docker run -d -p 80:80 angular-app

tasks 38 

FROM node:16-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY FILE 

RUN npm run build

CMD ["npm", "start"]

docker build -t vue-app .
docker run -d -p 8080:8080 vue-app

tasks 39

FROM tomcat:9.0-jre8

COPY target/*.war /usr/local/tomcat/webapps/

docker build -t java-app .
docker run -d -p 8080:8080 java-app

tasks 40

FROM ubuntu:latest

# Install Java
RUN apt-get update && \
    apt-get install -y openjdk-11-jre-headless

# Set working directory
WORKDIR /opt/airsonic

# Download Airsonic-Advanced
RUN wget -nv https://github.com/airsonic-advanced/airsonic-advanced/releases/download/v10.11.3/airsonic-advanced-10.11.3.war -O airsonic-advanced.war

# Configure environment variables
ENV AIRSONIC_HOME /opt/airsonic
ENV AIRSONIC_LOG_LEVEL INFO
ENV AIRSONIC_DATABASE_URL jdbc:mysql://mysql:3306/airsonic?useUnicode=true&characterEncoding=utf8
ENV AIRSONIC_DATABASE_USER airsonic
ENV AIRSONIC_DATABASE_PASSWORD airsonic

# Install MySQL client
RUN apt-get install -y mysql-client

# Copy the startup script
COPY airsonic-startup.sh /opt/airsonic/

# Expose the web server port
EXPOSE 8080

# Start Airsonic-Advanced
CMD ["sh", "/opt/airsonic/airsonic-startup.sh"]

#!/bin/bash

# Start MySQL server if not already running
if ! pgrep -f "mysqld"; then
  echo "Starting MySQL server..."
  /etc/init.d/mysql start
fi

# Wait for MySQL to be available
while ! mysql -u root -h localhost -e 'SELECT 1'; do
  echo "Waiting for MySQL to become available..."
  sleep 1
done

# Create the database and user
mysql -u root -h localhost -e "CREATE DATABASE IF NOT EXISTS airsonic;"
mysql -u root -h localhost -e "GRANT ALL ON airsonic.* TO airsonic@localhost IDENTIFIED BY 'airsonic';"

# Run Airsonic-Advanced
java -Djava.awt.headless=true -jar airsonic-advanced.war &
wait

   mkdir airsonic-docker
   cd airsonic-docker
   
   docker build -t my-airsonic .
   
   docker run -d -p 8080:8080 --name my-airsonic my-airsonic
   
