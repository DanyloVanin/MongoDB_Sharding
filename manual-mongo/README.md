# MongoDB Sharding

## Description

This guide shows how to configure sharding for default `mongo` Docker image using `docker-compose`. In this guide 2 shards are created and several illustrative query are executed to show off that shards work correctly.

## Instructions

1. Start docker files for mongo

```sh
docker-compose up --detach
```

2.  Get on `mongos` container,  initiate ReplicaSet for `mongo-config`,`mongo-shard1`,`mongo-shard2` using MongoShell `mongosh` command

```sh
# Start MongoShell
docker exec -it manual-mongo-mongos-1 bash

mongosh --host mongo-shard1 --port 27018 --eval "rs.initiate(); rs.status();"
mongosh --host mongo-shard2 --port 27018 --eval "rs.initiate(); rs.status();"
mongosh --host mongo-config --port 27019 --eval "rs.initiate(); rs.status();"

```

Initiate ReplicaSet command: `rs.initiate()`
Verify ReplicaSet status command: `rs.status()`

3. Add shards on `mongos` container:

```sh
# Start MongoShell
mongosh

# Add shards
sh.addShard("shard1/mongo-shard1:27018")
sh.addShard("shard2/mongo-shard2:27018")

# Check status - should see 2 shards
sh.status()

```

4. Enable sharding on `populations` database

```sh
sh.enableSharding("populations")

sh.shardCollection("populations.cities", { "country": "hashed" })
```

5. Fill `populations` database

```sh
use populations

db.cities.insertMany([
    {"name": "Seoul", "country": "South Korea", "continent": "Asia", "population": 25.674 },
    {"name": "Mumbai", "country": "India", "continent": "Asia", "population": 19.980 },
    {"name": "Lagos", "country": "Nigeria", "continent": "Africa", "population": 13.463 },
    {"name": "Beijing", "country": "China", "continent": "Asia", "population": 19.618 },
    {"name": "Shanghai", "country": "China", "continent": "Asia", "population": 25.582 },
    {"name": "Osaka", "country": "Japan", "continent": "Asia", "population": 19.281 },
    {"name": "Cairo", "country": "Egypt", "continent": "Africa", "population": 20.076 },
    {"name": "Tokyo", "country": "Japan", "continent": "Asia", "population": 37.400 },
    {"name": "Karachi", "country": "Pakistan", "continent": "Asia", "population": 15.400 },
    {"name": "Dhaka", "country": "Bangladesh", "continent": "Asia", "population": 19.578 },
    {"name": "Rio de Janeiro", "country": "Brazil", "continent": "South America", "population": 13.293 },
    {"name": "São Paulo", "country": "Brazil", "continent": "South America", "population": 21.650 },
    {"name": "Mexico City", "country": "Mexico", "continent": "North America", "population": 21.581 },
    {"name": "Delhi", "country": "India", "continent": "Asia", "population": 28.514 },
    {"name": "Buenos Aires", "country": "Argentina", "continent": "South America", "population": 14.967 },
    {"name": "Kolkata", "country": "India", "continent": "Asia", "population": 14.681 },
    {"name": "New York", "country": "United States", "continent": "North America", "population": 18.819 },
    {"name": "Manila", "country": "Philippines", "continent": "Asia", "population": 13.482 },
    {"name": "Chongqing", "country": "China", "continent": "Asia", "population": 14.838 },
    {"name": "Istanbul", "country": "Turkey", "continent": "Europe", "population": 14.751 }
])
```

6. Look at data shard distribution and usage, make some queries

```sh

db.cities.getShardDistribution()


# Explain queery that will find all cities
db.cities.find().explain()

# Another query that spans accross both shards 
db.cities.find({"continent": "Europe"}).explain()

# Query with filter based on sharding-column. Should work with only 1 shard
db.cities.find({"country": "Japan"}).explain()
```

7. Shut down all containers
```sh
exit

exit

docker-compose down
```

## Resources

- [How To Use Sharding in MongoDB](https://www.digitalocean.com/community/tutorials/how-to-use-sharding-in-mongodb)
- [Official Mongo Image](https://hub.docker.com/_/mongo)