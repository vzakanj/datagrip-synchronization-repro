# DataGrip synchronization behavior repro

This simple example demonstrates an issue with DataGrip synchronization when schema is changed outside of DataGrip.

## Versions
* Postgres: 9.5

* DataGrip: 2018.1.5

* Docker: Docker for Windows
    * Docker Engine: 18.03.1-ce
    * Docker Compose: 1.21.1
    * Compose file version: 3.4

## Scenario description

Postgres runs in a container and the scripts directory is mounted as a volume in the container at /docker-entrypoint-initdb.d/. Postgres container runs all scripts it finds in that directory when the container is created which can be effectively used to to create and seed the database.

## Reproduce

1. clone this repository

2. navigate to the repository, make sure your localhost port 5432 is available (or change the binding in docker-compose.yml) and run 
```
docker-compose up
```

This will create a test database with a single table person, having columns id and first_name.

3. Connect to the container db using the following information:

* Host: localhost
* Port: 5432
* Database: repro
* User: postgres
* Pass: postgres

Verify that the database and the person table are indeed created and that the table has columns id and first_name.

4. stop the container with ctrl+c

5. open scripts/02_create_person_table.sql and uncomment the line specifying last_name column, save the file

6. remove the container by running
```
docker-compose rm
``` 

press y to confirm when prompted to delete the existing container

7. recreate the container which should now seed the changed create person script
```
docker-compose up
```

8. once the container is up again, check the opened database in DataGrip and it will not have the last_name column visible in the database window even though it will be shown in the query results window when a query is made (e.g. SELECT * FROM person;). 

## Current behavior
After going through the reproduction steps, clicking the synchronize button on any level (data source, database, schema, table) and auto sync doesn't help as well. This gets even more painful to deal with when more complex changes are made like altering some columns and dropping others.

## Expected behavior
Clicking the synchronize button (at least on the data source) should show the changes (i.e. in this case show the last_name column)

## Workaround
The only workaround I've found for this issue is to delete the data source from the Database window in DataGrip and then recreate it.