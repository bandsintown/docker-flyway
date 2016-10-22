# What is Flyway?

![logo](logo.png)

Flyway is an open source database migration tool. It strongly favors simplicity and convention over configuration.

It is based around 6 basic commands: Migrate, Clean, Info, Validate, Baseline and Repair.

Migrations can be written in SQL (database-specific syntax (such as PL/SQL, T-SQL, ...) is supported) or Java (for advanced data transformations or dealing with LOBs).

It has a Command-line client, a Java API (also works on Android) for migrating the database on application startup, a Maven plugin, Gradle plugin, SBT plugin and Ant tasks.

Plugins are available for Spring Boot, Dropwizard, Grails, Play, Griffon, Grunt, Ninja and more.

Supported databases are Oracle, SQL Server, SQL Azure, DB2, DB2 z/OS, MySQL (including Amazon RDS), MariaDB, Google Cloud SQL, PostgreSQL (including Amazon RDS and Heroku), Redshift, Vertica, H2, Hsql, Derby, SQLite, SAP HANA, solidDB, Sybase ASE and Phoenix.

# Usage

## How to use this image


To run a schema migration, you have to mount the folder containing migration scripts in a volume under `/flyway/sql`. 

Assuming your DB is a MySQL Database, running a schema migration is simple as that:  


```
docker run -v ${PWD}:/flyway/sql -it --rm bandsintown/flyway:4.0.3 \
 -url=jdbc:mysql://db -schemas=db_schema -user=db_user -password=db_password migrate
```

## Waiting for the DB


It is common when using tools like Docker Compose to depend on services in other linked containers, 
however oftentimes relying on links is not enough - whilst the container itself may have started, the service(s) within it may not yet be ready - 
resulting in shell script hacks to work around race conditions.

That's typically the case when we use Docker Compose to run migration on a DB managed into a container. 
Whilst the container itself may have started, the database service is not reachable immediately, so the database migration might failed.

That's why this image bundle [Dockerize](https://github.com/jwilder/dockerize). 

Dockerize gives the ability to wait for services on a specified protocol (tcp, tcp4, tcp6, http, and https) before starting your application:

Here is a [sample](sample) using MySQL:

```
version: '2'

services:

  # Database Schema migration
  schema:
    image: bandsintown/flyway:4.0.3
    entrypoint: dockerize -wait tcp://db:3306
    command: flyway -url=jdbc:mysql://db -schemas=sample -user=root -password=root migrate
    volumes:
      - "./db:/flyway/sql"
    depends_on:
      - db

  # Database
  db:
    image: mysql:5.6.23
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_USER: sample
      MYSQL_PASSWORD: sample
      MYSQL_DATABASE: sample

```

# Build

This project is configured as an [automated build in Dockerhub](https://hub.docker.com/r/bandsintown/alpine/).

Each branch give the related image tag.  

# License

All the code contained in this repository, unless explicitly stated, is
licensed under ISC license.

A copy of the license can be found inside the [LICENSE](LICENSE) file.