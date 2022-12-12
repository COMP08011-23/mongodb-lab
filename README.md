# Launch a Replica Set
- Create mongo folder
- mkdir ~/dev/mongodb
- Create replicat set folders
mkdir -p ~/dev/mongodb/rs0-0
mkdir -p ~/dev/mongodb/rs0-1
mkdir -p ~/dev/mongodb/rs0-2
- Start instances
mongod --replSet rs0 --port 27017 --bind_ip 127.0.0.1 --dbpath ~/dev/mongodb/rs0-0 --oplogSize 128
mongod --replSet rs0 --port 27018 --bind_ip 127.0.0.1 --dbpath ~/dev/mongodb/rs0-1 --oplogSize 128
mongod --replSet rs0 --port 27019 --bind_ip 127.0.0.1 --dbpath ~/dev/mongodb/rs0-2 --oplogSize 128

- Initialise replica set (once for set)
- Connect to unstance 
sudo apt install mongodb-clients
mongo --port 27017
```json
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

- Nodes elect a primary, rest become secondary
- Connect to all three nodes, check which one is primary

- java -jar ./mongo-school-client physics Michael 25 75.0
- get Invalid Course Physics
- connect to mongo, add collection
mongo --port 27017
show dbs
use online-school
db.createCollection("physics")
- java -jar ./mongo-school-client physics Michael 25 75.0
- Now added
- Repeat
- not added, already enrolled
- Add new one
java -jar ./mongo-school-client.jar physics Mary 21 79
- Get list back

- Kill primary
- Add another: works fine
- Check which is primary


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


