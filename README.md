
# Debezium HOP

## Introduction

This project is a sample to test the integration between Debezium and Apache Hop. 
This same project could be the seed for a more complex project to manage a real life integration
between a database and one or more targets that must be kept aligned to soure database changes.

#### What is Debezium
Debezium is a set of distributed services to capture changes in your databases so that your applications 
can see those changes and respond to them. Debezium records all row-level changes within each database 
table in a change event stream, and applications simply read these streams to see the change events in the 
same order in which they occurred.

[Debezium website](https://debezium.io/)

[Debezium documentation](https://debezium.io/documentation/reference/1.6/index.html)

[Debezium on Github](https://github.com/debezium/)

#### What is Apache Hop
The Hop Orchestration Platform, or Apache Hop (Incubating), aims to facilitate all aspects of data 
and metadata orchestration. Hop is an entirely new open source data integration platform that is easy to use, 
fast and flexible. Hop aims to be the future of data integration.

[Apache Hop website](https://hop.apache.org/)

[Apache Hop documentation](https://hop.apache.org/manual/latest/getting-started/)

[Apache Hop on Github](https://github.com/apache/incubator-hop)

## How this sample works

## Software Prerequisites
This sample runs on all platforms (Win/Mac/Linux) because everything leverages on [Docker Compose](https://docs.docker.com/compose/). If you are on Windows, I strongly suggest you install [cygwin](https://www.cygwin.com/) to properly manage some configuration steps. 

About Apache Hop, I strongly suggest to use the [latest nightly build](https://repository.apache.org/content/repositories/snapshots/org/apache/hop/hop-client/1.0-SNAPSHOT/) so that you can leverage on latest fixes eventually applied to Apache Hop's Kafka transforms.

## Docker Compose
Debezium components are run on a set of docker containers tied together by using Docker Compose. You can find the compose file for this example in the _compose_ directory.

#### Additional links
To fully configure the Docker networking and have the sample proprerly running I strongly suggest to read the following insightful articles.

[Kafka listeners explained](https://www.confluent.io/blog/kafka-listeners-explained/)

[Kafka - Difference between advertised listeners and bootstrap-servers](https://stackoverflow.com/questions/60847050/what-is-the-difference-between-advertised-listeners-and-bootstrap-servers)

A full understand of the illustrated concepts saves you a lot of time. The first time I spent an entire day trying to understand what's wrong and before having these hints available. The sample is already configured by appliying the concepts described in these articles.

## Start SQL Server and Kafka infrastructure
To start the SQL Server instance and the kafka middleware needed by Debezium for this example, go to the _compose_ subdirectory where the _docker-compose.yaml_ file is located. Before starting up everything, please edit the docker-compose.yaml file and dio the following:

* on line 10, replace the label _<my_sa_password>_ with a valid password for your SQL Server's sa user. Remember that SQL Server's a user will be created automatically during the SQL Server container initialization.
* on line 29, replace the label _<your_computer_dns_name>_ with your computer's real DNS name

As soon the changes specified above are set, save the file and type the following command:

    docker-compose -d up

to start SQL Server 2017, Zookeper, Kafka and Kafka Connect.

## Create the SQL Server database for the sample
To initialize correctly a SQL Server database to run Debezium, I strongly suggest to use the [inventory.sql script](https://raw.githubusercontent.com/debezium/debezium-examples/master/sql-server-read-replica/debezium-sqlserver-init/inventory.sql)  you find in github [debezium-examples project](https://github.com/debezium/debezium-examples). Feel free to open it on a SQL Server Management Studio sessoin and run it. As soon as the script's execution completes you will find a new test database named testDB that you can use for your samples.

## Configure the SQL Server Debezium connector to get all the CDC changes
For a full overview on Debezium Connector for SQL Server configuration details, you can read the [documentation provided](https://debezium.io/documentation/reference/1.6/connectors/sqlserver.html) by the Debezium project.

Apart from this, in _debezium/connector/sqlserver_ subdirectory you will find an ready to go configuration file to properly enable the SQL Server Connector for Debezium.
Below you can see the provided configuration file for SQL Server Connector. This file enables the monitoring of changes in the _dbo.customers_ table of the testDB database. 

    {
        "name": "inventory-connector", 
        "config": {
            "connector.class": "io.debezium.connector.sqlserver.SqlServerConnector", 
            "database.hostname": "sqlserver", 
            "database.port": "1433", 
            "database.user": "sa", 
            "database.password": "<my-password>", 
            "database.dbname": "testDB", 
            "database.server.name": "fullfillment", 
            "table.include.list": "dbo.customers", 
            "database.history.kafka.bootstrap.servers": "kafka:29092", 
            "database.history.kafka.topic": "dbhistory.fullfillment" 
        }
    }

Before installing the connector, remember to replace the label _<my_sa_password>_ with the valid password you provided so far for your SQL Server's sa user. Set the password and save the file. To install the connector, go to _debezium/connector/sqlserver_ subdirectory and type the following command:

    curl -i -X POST -H "Accept:application/json" -H "Content-Type:application/json" localhost:8083/connectors/ -d @sql-server.connector.config.json

Debezium will answer with

    [inventory-connector]

that means the connector is installed and fully working.

## Let's rock
* Run Hop Gui and configure a project with project's root directory pointing to _hop/consumer_sample_ (config fils id in _config/project-config.json_). Clear the entry for the parent project and press ok. 
* Load the _debezium-receiver.hpl_ pipeline.
* Open the Get Changes tranform.
* Look at the bootstrap servers field. Replace the label _<your_computer_dns_name>_ with your computer's real DNS name.
* Save the pipeline.
* Run the pipeline; your flow of CDC events will be received from Hop Kafka Consumer transform and you will see the json raw records that describes the changes in _dbo.customers_ table in Hop's log.
* Try to make some changes to _dbo.customers_ and you will see new events fired by Debezium and new records received by hop

### Conclusion
This documentation was written very very quickly therefore I'm sorry if something could be explained better. Please write email me at [sergio.ramazzina@serasoft.it](sergio.ramazzina@serasoft.it) and let me know where I can improve or if there are other sample you would like to see. I hope you found this work useful.