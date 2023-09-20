Download the Cluster-to-Cluster Sync .tgz tarball from the following link:
wget https://fastdl.mongodb.org/tools/mongosync/mongosync-amazon2-x86_64-1.5.0.tgz


To extract mongosync, use the tar command in a system shell:
tar -zxvf mongosync-*.tgz


cp /usr/local/bin/mongosync /usr/sbin/
chmod +x /usr/sbin/mongosync




mongosync \
      --cluster0 "mongodb://10.102.124.192:50001,10.102.124.192:50002,10.102.124.192:50003" \
      --cluster1 "mongodb://10.102.124.192:50004,10.102.124.192:50005,10.102.124.192:50006"



curl localhost:27182/api/v1/start -XPOST \
--data '
   {
      "source": "cluster0",
      "destination": "cluster1"
   } '

