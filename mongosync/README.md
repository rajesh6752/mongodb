# MongoDB Cluster-to-Cluster Sync:
MongoDB Cluster-to-Cluster Sync connects MongoDB clusters and provides a way to synchronize data between them. The tool that makes the connection is mongosync. 

### Setup:
Cluster-to-Cluster Sync syncs data between two clusters. To run mongosync, you will need to:

1. Define a source and a destination cluster with replicaset.
2. Define administrative users.
3. Download and install mongosync.
4. Connect the clusters.

### Installation:
> Download the Cluster-to-Cluster Sync .tgz tarball from the following link:
```
wget https://fastdl.mongodb.org/tools/mongosync/mongosync-amazon2-x86_64-1.5.0.tgz
```

> To extract mongosync, use the tar command in a system shell:
```
tar -zxvf mongosync-*.tgz
```
> Copy the binary into a directory listed in your PATH variable and add executable permission:
```
cp /usr/local/bin/mongosync /usr/sbin/
chmod +x /usr/sbin/mongosync
```

### Connect the Clusters:
Determine the hostname or IP address and port for your source and destination clusters. You will use this information and the user authentication details to construct the connection strings.

> Your connections strings will resemble:
```
mongosync \
      --cluster0 "mongodb://10.102.124.192:50001,10.102.124.192:50002,10.102.124.192:50003" \
      --cluster1 "mongodb://10.102.124.192:50004,10.102.124.192:50005,10.102.124.192:50006"
```

### Synchronize Data Between Clusters:
```
curl localhost:27182/api/v1/start -XPOST \
--data '
   {
      "source": "cluster0",
      "destination": "cluster1"
   } '
```

### Validate synchronization:
> Login the source cluster and insert some records in the collections:
```
mongo mongodb://10.102.124.192:50001
db
show dbs
show collections 
db.createCollections(“firstCollection”)
db.firstCollection.insert( { item: "card", qty: 15 } )
db.firstCollection.insert( { item: "books", qty: 23 } )

```
> Now login to Destination cluster and check data is synced or not:
```
mongo mongodb://10.102.124.192:50004
db
show dbs
show collections 
db.firstCollection.find()
```
