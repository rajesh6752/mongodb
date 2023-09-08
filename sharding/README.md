## Set up Sharding using Docker Containers

### Config servers
Start config servers (3 member replica set)
```
docker-compose -f config-server/docker-compose.yaml up -d
```
Initiate replica set
```
mongo mongodb://192.168.1.81:40001
```
```
rs.initiate(
  {
    _id: "cfgrs",
    configsvr: true,
    members: [
      { _id : 0, host : "192.168.1.81:40001" },
      { _id : 1, host : "192.168.1.81:40002" },
      { _id : 2, host : "192.168.1.81:40003" }
    ]
  }
)

rs.status()
```

### Shard 1 servers
Start shard 1 servers (3 member replicas set)
```
docker-compose -f shard1/docker-compose.yaml up -d
```
Initiate replica set
```
mongo mongodb://192.168.1.81:50001
```
```
rs.initiate(
  {
    _id: "shard1rs",
    members: [
      { _id : 0, host : "192.168.1.81:50001" },
      { _id : 1, host : "192.168.1.81:50002" },
      { _id : 2, host : "192.168.1.81:50003" }
    ]
  }
)

rs.status()
```

### Mongos Router
Start mongos query router
```
docker-compose -f mongos/docker-compose.yaml up -d
```

### Add shard to the cluster
Connect to mongos
```
mongo mongodb://192.168.1.81:60000
```
Add shard
```
mongos> sh.addShard("shard1rs/192.168.1.81:50001,192.168.1.81:50002,192.168.1.81:50003")
mongos> sh.status()
```
## Adding another shard
### Shard 2 servers
Start shard 2 servers (3 member replicas set)
```
docker-compose -f shard2/docker-compose.yaml up -d
```
Initiate replica set
```
mongo mongodb://192.168.1.81:50004
```
```
rs.initiate(
  {
    _id: "shard2rs",
    members: [
      { _id : 0, host : "192.168.1.81:50004" },
      { _id : 1, host : "192.168.1.81:50005" },
      { _id : 2, host : "192.168.1.81:50006" }
    ]
  }
)

rs.status()
```
### Add shard to the cluster
Connect to mongos
```
mongo mongodb://192.168.1.81:60000
```
Add shard
```
mongos> sh.addShard("shard2rs/192.168.1.81:50004,192.168.1.81:50005,192.168.1.81:50006")
mongos> sh.status()
```





# Test the Sharding

### Login to mongodb cluster and test the sharding
```
mongo mongodb://127.0.0.1:40001
```
### Create a database
```
mongos> use sharddemo
switched to db sharddemo
mongos> db
sharddemo
mongos> show collections
```

### Create the collections
```
mongos> db.createCollection("movies")
{
   "ok" : 1,
   "$clusterTime" : {
      "clusterTime" : Timestamp(1694026889, 8),
      "signature" : {
         "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
         "keyId" : NumberLong(0)
      }
   },
   "operationTime" : Timestamp(1694026889, 8)
}
mongos> db.createCollection("movies2")
{
   "ok" : 1,
   "$clusterTime" : {
      "clusterTime" : Timestamp(1694026893, 1),
      "signature" : {
         "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
         "keyId" : NumberLong(0)
      }
   },
   "operationTime" : Timestamp(1694026893, 1)
}
mongos> show collections
movies
movies2
mongos> 
```

### Check the db is shored or not
```
mongos> db.movies.getShardDistribution()
Collection sharddemo.movies is not sharded.
mongos>
```
### Enable sharding for the database
```
mongos> sh.enableSharding("sharddemo")
{
   "ok" : 1,
   "$clusterTime" : {
      "clusterTime" : Timestamp(1694027261, 1),
      "signature" : {
         "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
         "keyId" : NumberLong(0)
      }
   },
   "operationTime" : Timestamp(1694027261, 1)
}
mongos>
```

### Shard the collection
 ```
mongos> sh.shardCollection("sharddemo.movies", {"title": "hashed"})
{
   "collectionsharded" : "sharddemo.movies",
   "ok" : 1,
   "$clusterTime" : {
      "clusterTime" : Timestamp(1694027201, 38),
      "signature" : {
         "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
         "keyId" : NumberLong(0)
      }
   },
   "operationTime" : Timestamp(1694027201, 38)
}
mongos>
```
### Check the sharded collections
```
mongos> db.movies.getShardDistribution()

Shard shard1rs at shard1rs/10.102.120.69:50001,10.102.120.69:50002,10.102.120.69:50003
 data : 0B docs : 0 chunks : 2
 estimated data per chunk : 0B
 estimated docs per chunk : 0

Shard shard2rs at shard2rs/10.102.120.69:50004,10.102.120.69:50005,10.102.120.69:50006
 data : 0B docs : 0 chunks : 2
 estimated data per chunk : 0B
 estimated docs per chunk : 0

Totals
 data : 0B docs : 0 chunks : 4
 Shard shard1rs contains 0% data, 0% docs in cluster, avg obj size on shard : 0B
 Shard shard2rs contains 0% data, 0% docs in cluster, avg obj size on shard : 0B

mongos>
```

### Check other collections as well
```
mongos> db.movies2.getShardDistribution()
Collection sharddemo.movies2 is not sharded.
mongos>
```

### Now insert some records in the shared collection
```
[root@ip-10-102-120-69 sharding]# for i in {1..50}; do echo -e "use sharddemo \n db.movies.insertOne({\"title\": \"Supper Man $i\", \"language\": \"English\"})" | mongo mongodb://10.102.120.69:60000; done
```

### Now check the collections is inserted
```
[root@ip-10-102-120-69 sharding]# mongo mongodb://10.102.120.69:60000
MongoDB shell version v5.0.20
---
mongos> user sharddemo
uncaught exception: SyntaxError: unexpected token: identifier :
@(shell):1:5
mongos> use sharddemo
switched to db sharddemo
mongos> db.movies.find()
{ "_id" : ObjectId("64f8d04c1500c3a8acda2490"), "title" : "Supper Man 2", "language" : "English" }
{ "_id" : ObjectId("64f8d04c10bd71e87fc0e281"), "title" : "Supper Man 3", "language" : "English" }
{ "_id" : ObjectId("64f8d04c4c2266963e1ec689"), "title" : "Supper Man 5", "language" : "English" }
{ "_id" : ObjectId("64f8d04d8bb2fb2330f6c0cd"), "title" : "Supper Man 8", "language" : "English" }
{ "_id" : ObjectId("64f8d04da0a0ee4a549868ba"), "title" : "Supper Man 9", "language" : "English" }
{ "_id" : ObjectId("64f8d04df111e80d3ad1505a"), "title" : "Supper Man 10", "language" : "English" }
{ "_id" : ObjectId("64f8d04df99b8ed8a6c6f4f6"), "title" : "Supper Man 11", "language" : "English" }
{ "_id" : ObjectId("64f8d04daa5ce24b6f2fe49e"), "title" : "Supper Man 12", "language" : "English" }
{ "_id" : ObjectId("64f8d04d874d0d3c91f87078"), "title" : "Supper Man 13", "language" : "English" }
{ "_id" : ObjectId("64f8d04dd176d60ce63a02e3"), "title" : "Supper Man 16", "language" : "English" }
{ "_id" : ObjectId("64f8d04e48d51403aeba76ae"), "title" : "Supper Man 20", "language" : "English" }
{ "_id" : ObjectId("64f8d04e8450c221251eb448"), "title" : "Supper Man 22", "language" : "English" }
{ "_id" : ObjectId("64f8d04ee48db3cc2e288f67"), "title" : "Supper Man 24", "language" : "English" }
{ "_id" : ObjectId("64f8d04ff60b09140b92db85"), "title" : "Supper Man 28", "language" : "English" }
{ "_id" : ObjectId("64f8d04f7737136f562529c2"), "title" : "Supper Man 32", "language" : "English" }
{ "_id" : ObjectId("64f8d04f1f11dea463cee806"), "title" : "Supper Man 33", "language" : "English" }
{ "_id" : ObjectId("64f8d05082b866c0025287e2"), "title" : "Supper Man 35", "language" : "English" }
{ "_id" : ObjectId("64f8d050496526be4c099160"), "title" : "Supper Man 38", "language" : "English" }
{ "_id" : ObjectId("64f8d050cd470a6a1cb34927"), "title" : "Supper Man 39", "language" : "English" }
{ "_id" : ObjectId("64f8d050e1d3e49a4908d629"), "title" : "Supper Man 42", "language" : "English" }
Type "it" for more
mongos>
mongos> db.movies.count()
50
```
### Check the shard details
```
mongos> db.movies.getShardDistribution()

Shard shard2rs at shard2rs/10.102.120.69:50004,10.102.120.69:50005,10.102.120.69:50006
 data : 1KiB docs : 22 chunks : 2
 estimated data per chunk : 756B
 estimated docs per chunk : 11

Shard shard1rs at shard1rs/10.102.120.69:50001,10.102.120.69:50002,10.102.120.69:50003
 data : 1KiB docs : 28 chunks : 2
 estimated data per chunk : 964B
 estimated docs per chunk : 14

Totals
 data : 3KiB docs : 50 chunks : 4
 Shard shard2rs contains 43.96% data, 44% docs in cluster, avg obj size on shard : 68B
 Shard shard1rs contains 56.03% data, 56% docs in cluster, avg obj size on shard : 68B

mongos>
```


> Another Example, what if we want shard collections which already has the data

### check the existing records
```
mongos> db.movies2.find()
```

### Insert the some records
```
[root@ip-10-102-120-69 sharding]# for i in {1..10}; do echo -e "use sharddemo \n db.movies2.insertOne({\"title\": \"Supper Man $i\", \"language\": \"English\"})" | mongo mongodb://10.102.120.69:60000; done
```
### Now loging and check the data
```
[root@ip-10-102-120-69 sharding]# mongo mongodb://10.102.120.69:60000
MongoDB shell version v5.0.20
mongos> use sharddemo
switched to db sharddemo
mongos>
mongos> db.movies2.find()
{ "_id" : ObjectId("64f8d21d731749b0e6d7a737"), "title" : "Supper Man 1", "language" : "English" }
{ "_id" : ObjectId("64f8d21eb51a5b9b6ebb9730"), "title" : "Supper Man 2", "language" : "English" }
{ "_id" : ObjectId("64f8d21e026a5aaa03cc6b84"), "title" : "Supper Man 3", "language" : "English" }
{ "_id" : ObjectId("64f8d21e38fadb5d6e945423"), "title" : "Supper Man 4", "language" : "English" }
{ "_id" : ObjectId("64f8d21ec3aeb811b028afbd"), "title" : "Supper Man 5", "language" : "English" }
{ "_id" : ObjectId("64f8d21e5c7275fd58f5b5c0"), "title" : "Supper Man 6", "language" : "English" }
{ "_id" : ObjectId("64f8d21ee8fbee381f456200"), "title" : "Supper Man 7", "language" : "English" }
{ "_id" : ObjectId("64f8d21e3f4ed4389960a525"), "title" : "Supper Man 8", "language" : "English" }
{ "_id" : ObjectId("64f8d21ec5f8ad6970ec0da1"), "title" : "Supper Man 9", "language" : "English" }
{ "_id" : ObjectId("64f8d21e4b15474c42857494"), "title" : "Supper Man 10", "language" : "English" }
mongos>
```
### Veryfi collections is not sharded
```
mongos> db.movies2.getShardDistribution()
Collection sharddemo.movies2 is not sharded.
mongos>
```
### Now try the shard collection but you will get below error
```
mongos> sh.shardCollection("sharddemo.movies2", {"title": "hashed"})
{
   "ok" : 0,
   "errmsg" : "Please create an index that starts with the proposed shard key before sharding the collection. ",
   "code" : 72,
   "codeName" : "InvalidOptions",
   "$clusterTime" : {
      "clusterTime" : Timestamp(1694028673, 9),
      "signature" : {
         "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
         "keyId" : NumberLong(0)
      }
   },
   "operationTime" : Timestamp(1694028673, 9)
}
mongos>
```
### To solve this error need to create Index
```
mongos> db.movies2.createIndex({"title": "hashed"})
{
   "raw" : {
      "shard2rs/10.102.120.69:50004,10.102.120.69:50005,10.102.120.69:50006" : {
         "numIndexesBefore" : 1,
         "numIndexesAfter" : 2,
         "createdCollectionAutomatically" : false,
         "commitQuorum" : "votingMembers",
         "ok" : 1
      }
   },
   "ok" : 1,
   "$clusterTime" : {
      "clusterTime" : Timestamp(1694028792, 7),
      "signature" : {
         "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
         "keyId" : NumberLong(0)
      }
   },
   "operationTime" : Timestamp(1694028792, 7)
}
mongos>
```
### Now shard the collection again
```
mongos> sh.shardCollection("sharddemo.movies2", {"title": "hashed"})
{
   "collectionsharded" : "sharddemo.movies2",
   "ok" : 1,
   "$clusterTime" : {
      "clusterTime" : Timestamp(1694028898, 26),
      "signature" : {
         "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
         "keyId" : NumberLong(0)
      }
   },
   "operationTime" : Timestamp(1694028898, 26)
}
mongos>
mongos>
mongos> db.movies2.getShardDistribution()

Shard shard2rs at shard2rs/10.102.120.69:50004,10.102.120.69:50005,10.102.120.69:50006
 data : 681B docs : 10 chunks : 1
 estimated data per chunk : 681B
 estimated docs per chunk : 10

Totals
 data : 681B docs : 10 chunks : 1
 Shard shard2rs contains 100% data, 100% docs in cluster, avg obj size on shard : 68B

mongos>
```
### Check the Shard details 
```
mongos> sh.status()
--- Sharding Status ---
  sharding version: { "_id" : 1, "clusterId" : ObjectId("64f8c9795c26349596127d65") }
  shards:
        {  "_id" : "shard1rs",  "host" : "shard1rs/10.102.120.69:50001,10.102.120.69:50002,10.102.120.69:50003",  "state" : 1,  "topologyTime" : Timestamp(1694026394, 2) }
        {  "_id" : "shard2rs",  "host" : "shard2rs/10.102.120.69:50004,10.102.120.69:50005,10.102.120.69:50006",  "state" : 1,  "topologyTime" : Timestamp(1694026569, 2) }
  active mongoses:
        "7.0.1" : 1
  autosplit:
        Currently enabled: yes
  balancer:
        Currently enabled: yes
        Currently running: no
  databases:
        {  "_id" : "config",  "primary" : "config",  "partitioned" : true }
                config.system.sessions
                        shard key: { "_id" : 1 }
                        unique: false
                        balancing: true
                        chunks:
                                shard1rs  1024
                        too many chunks to print, use verbose if you want to force print
        {  "_id" : "sharddemo",  "primary" : "shard2rs",  "partitioned" : false,  "version" : {  "uuid" : UUID("7b04b65d-79eb-4f2a-a4e9-538b80757191"),  "timestamp" : Timestamp(1694026888, 1),  "lastMod" : 1 } }
mongos>
mongos> db.movies2.getShardDistribution()

Shard shard2rs at shard2rs/10.102.120.69:50004,10.102.120.69:50005,10.102.120.69:50006
 data : 681B docs : 10 chunks : 1
 estimated data per chunk : 681B
 estimated docs per chunk : 10

Totals
 data : 681B docs : 10 chunks : 1
 Shard shard2rs contains 100% data, 100% docs in cluster, avg obj size on shard : 68B

mongos>
mongos>
mongos> db.movies.getShardDistribution()

Shard shard2rs at shard2rs/10.102.120.69:50004,10.102.120.69:50005,10.102.120.69:50006
 data : 1KiB docs : 22 chunks : 2
 estimated data per chunk : 756B
 estimated docs per chunk : 11

Shard shard1rs at shard1rs/10.102.120.69:50001,10.102.120.69:50002,10.102.120.69:50003
 data : 1KiB docs : 28 chunks : 2
 estimated data per chunk : 964B
 estimated docs per chunk : 14

Totals
 data : 3KiB docs : 50 chunks : 4
 Shard shard2rs contains 43.96% data, 44% docs in cluster, avg obj size on shard : 68B
 Shard shard1rs contains 56.03% data, 56% docs in cluster, avg obj size on shard : 68B

mongos>
```
