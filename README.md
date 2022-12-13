<img src="./atuLogo2.jpg" height="200" align="centre"/>

# ATU Distributed Systems
# Lab: Replica Set in MongoDB
Instructions for the Distributed Systems lab **Replica Set in MongoDB**.

## Lab Objectives
In this lab you'll:
- Configure and run a replicated MongoDB cluster
- Verify the fault tolerance of the cluster by injecting failures and testing behaviour using a client app.

## Introduction
MongoDB uses the concept of a replica set to achieve replication. Nodes in a cluster can be added to a replica set. A primary node will then be elected automatically from among the members of the replica set, and data will be replicated to all the nodes. Clients can chose what kind of [write behaviour](https://www.mongodb.com/docs/v5.0/core/replica-set-write-concern/) and what kind of [read behaviour](https://www.mongodb.com/docs/manual/core/read-preference/) they want depending on their consistency and availability requirements.

## Getting Started
1. Log in to your [Azure Lab Services](https://labs.azure.com/) VM.
2. In the VM, open a terminal.

## Creating Node Storage
First we need to create folders where each node in the cluster will store its data
```shell
mkdir ~/dev/mongodb
mkdir -p ~/dev/mongodb/rs0-0
mkdir -p ~/dev/mongodb/rs0-1
mkdir -p ~/dev/mongodb/rs0-2
```

## Launching Cluster Nodes
Now we can start instances using the `mongod` command, which starts a mongodb server process. (It may be necessary to install mongod using the sugggested command):
```shell
mongod --replSet rs0 --port 27017 --bind_ip 127.0.0.1 --dbpath ~/dev/mongodb/rs0-0 --oplogSize 128
mongod --replSet rs0 --port 27018 --bind_ip 127.0.0.1 --dbpath ~/dev/mongodb/rs0-1 --oplogSize 128
mongod --replSet rs0 --port 27019 --bind_ip 127.0.0.1 --dbpath ~/dev/mongodb/rs0-2 --oplogSize 128
```

## Initialise replica set
For now all of these nodes are operating separately. We need to tell mongodb that these nodes should be considered part of a replica set. We do this by running the command `rs.initiate` on one of the nodes, passing in a json object identifying the members of the replica set. (It may be necessary to install the mongo client using the suggested command).
```
mongo --port 27017
rs.initiate({
    _id: "rs0",
    members: [
        {
            _id: 0,
            host: "127.0.0.1:27017"
        },
        {
            _id: 1,
            host: "127.0.0.1:27018"
        },
        {
            _id: 2,
            host: "127.0.0.1:27019"
        }
    ]
})
```
The nodess elect a primary, the rest become secondary. Connect to all three nodes using the `mongo` client (changing the port number for each node) and verify that there is one primary and two secondaries.

## Test Using Client Application
The test application `mongo-school-client` is a sample application to manage course enrolment. Running the command `java -jar ./mongo-school-client.jar physics Michael 25 75.0` attempt to enrol the student Michael in the course physics. His age is 25 and his GPA is 75.0.

This program (source code provided) will attempt to connect to a mongo server and add the data to a collection called `physics`, which doesn't exist. So we need to create it:
```
mongo --port 27017
show dbs
use online-school
db.createCollection("physics")
```

Now rerun the enrolment application to try to enrol Michael and it should report success and show how many students are enrolled. Running the same command another time will not add the student since they're already enrolled.

Now add another student: `java -jar ./mongo-school-client.jar physics Mary 21 79`.

## Test Fault Tolerance
Test the fault tolerance of the cluster by killing and restarting various nodes (including the primary). Verify that:
- a new primary is always elected if the primary node goes down
- the client application continues to be able to write new data to the cluster and no data is lost when nodes go down.


<!--

## Partitioning
- Make dirs for config servers
mkdir config-srv-0
mkdir config-srv-1
mkdir config-srv-2

Start config servers
mongod --configsvr --replSet config-rs --dbpath ~/dev/mongodb/config-srv-0 --bind_ip 127.0.0.1 --port 27020
mongod --configsvr --replSet config-rs --dbpath ~/dev/mongodb/config-srv-1 --bind_ip 127.0.0.1 --port 27021
mongod --configsvr --replSet config-rs --dbpath ~/dev/mongodb/config-srv-2 --bind_ip 127.0.0.1 --port 27022

- Add them to a replica set
```json
rs.initiate({
    _id: "config-rs",
    members: [
        {
            _id: 0,
            host: "127.0.0.1:27020"
        },
        {
            _id: 1,
            host: "127.0.0.1:27021"
        },
        {
            _id: 2,
            host: "127.0.0.1:27022"
        }
    ]
})
```

- Create shard directories
mkdir dev/shard-0
mkdir dev/shard-1

- Start shards
mongod --shardsvr --port 27017 --bind_ip 127.0.0.1 --dbpath ~/dev/mongodb/shard-0 --oplogSize 128
mongod --shardsvr --port 27018 --bind_ip 127.0.0.1 --dbpath ~/dev/mongodb/shard-1 --oplogSize 128

- Start router
mongos --configdb config-rs/127.0.0.1:27020,127.0.0.1:27021,127.0.0.1:27022 --bind_ip 127.0.0.1 --port 27023

- Tell router about shards

-->
