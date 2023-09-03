## Set up MongoDb

### Install MongoDB

Step 1: Update Amazon Linux 2 Server:
```
sudo yum -y update
```
Step 2: Add MongoDB YUM Repositories:
```
# Create a /etc/yum.repos.d/mongodb-org-5.0.repo file so that you can install MongoDB directly using yum:
sudo tee /etc/yum.repos.d/mongodb-org-5.0.repo<<EOL
[mongodb-org-5.0]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/amazon/2/mongodb-org/5.0/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-5.0.asc
EOL
```
```
# Confirm the repository is now available:
sudo yum repolist
```

Step 3: Install MongoDB on Amazon Linux 2:
```
yum install mongodb-org -y 
```
```
# Now, check the installed MongoDB version:
mongod --version
```

Step 4: Start mongodb using below command:
```
mongod --replSet "rs0" --bind_ip 0.0.0.0 --dbpath /var/lib/mongo --logpath /var/log/mongodb/mongod.log --port 27017 &
```

### Configure Replication Set:

Step 1: Configure Mongodb servers (1 Primary, 1 Replica set, 1 Arbiter. Follow the above steps to install mongodb for these three instances.

Step 2: Initiate replica set with 1 Primary, 1 Replica set and 1 Arbiter:
```
mongo mongodb://<mongoIP/DNS>
```
```
mongos> rs.initiate(
  {
    _id: "rs0",
    members: [
      { _id : 0, host : "<Instance1IP>:27017" },
      { _id : 1, host : "<Instance2IP>:27017" },
      { _id : 2, host : "Instance3IP>:27017", arbiterOnly: true }
    ]
  }
)
```

Step 3: Check the status or ReplicaSet:
```
mongos> rs.status()
mongos> rs.printReplicationInfo()
mongos> db.getReplicationInfo()
mongos> rs.conf()
```

Step 4: Test the Replication:
```
# Check the current DB
mongos> db

# Check all the DBs
mongos> show dbs

# Create the collection
mongos> db.createCollection("firstCollection")

# Check created collections
mongos> show collections
```
Step 5: Now login to one of the secondary Db:
```
mongo mongodb://<mongodb2>
```
Step 6: Allow read operations on secondary members for the MongoDB connection:
```
mongos> rs.secondaryOk()
```

Step 6: Test the collection is replicated or not:
```
mongos> show dbs
mongos> show collections
```
### Test failover:
Step 1: Login the primary instance and stop the mongodb service:
```
ps -ef | grep mongo
kill -9 <pidof mongo>
```
Step 2: Login in mongodb replica set cluster:
```
mongo 'mongodb://mongodb0,mongodb1,mongodb2/?replicaSet=rs0'
```
Step 3: Check the replica set status:
```
mongos> rs.status()
mongos> show dbs
mongos> show collections
```
Step 4: Create another collection:
```
mongos> db.createCollection("secondCollection")
mongos> show collections
```
Step 5: Test replication:
```
# login to secondary mongodb and test it:
mongo mongodb://<mongodb2>
```
```
mongos> rs.slaveOk()
mongos> show dbs
mongos> show collections
```
Step 6: Now login to instance of primary mongo which one we had stopped and start the mongodb service:
```
mongod --replSet "rs0" --bind_ip 0.0.0.0 --dbpath /var/lib/mongo --logpath /var/log/mongodb/mongod.log --port 27017 &
```

Step 7: Check what happened since we brought back the failed primary:
```
mongo 'mongodb://mongodb0,mongodb1,mongodb2/?replicaSet=rs0'
```
```
mongos> rs.status()
```

#### Note: connection string only for read replicas
mongo 'mongodb://mongodb0,mongodb1,mongodb2/?replicaSet=rs0&readPrefreference=secondry'
