# Install MongoDB Sharding Cluster

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Frgrebski%2Fazure-quickstart-templates%2Fmaster%2Fmongodb-sharding-centos-dev%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>
<a href="
http://armviz.io/#/?load=https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fmongodb-sharding-centos%2Fazuredeploy.json" target="_blank">
    <img src="http://armviz.io/visualizebutton.png"/>
</a>

This template was created basing on <a href="https://github.com/Azure/azure-quickstart-templates/tree/master/mongodb-sharding-centos">https://github.com/Azure/azure-quickstart-templates/tree/master/mongodb-sharding-centos</a>


This template deploys a MongoDB Sharding Cluster on Ubuntu 16.04. The cluster consists of:  
- 1 router server (has public IP and domain)
- 3 config servers  as replicaSet
- 8 shards (as standalone)

The config server replica set stores sharding cluster metadata. MongoDB suggests to use a replica set for the metadata store in the production environment, in case one of the config server is down, there will still be other 2 config server nodes offer the service.

The nodes are under the same subnet 10.0.0.0/24. Except the router server node (mongos), the other nodes only have private IP address.

##Important Notice
Each VM of the shard uses raid0 to improve performance. The number and the size of data disks(setup raid0) on each shard VM are determined by yourself. However, there is number and size of data disks limit per the VM size. Before you set number and size of data disks, please refer to the link https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-linux-sizes/ for the correct choice.



##After deployment, you can do below to verify if the sharding cluster really works or not:

1. SSH connect to one of the router server, execute below:
  ```
  $mongo -u "<mongouser>" -p "<mongopassword>" "admin"

  db.runCommand( { listshards : 1 } )

  exit
  ```

  Upper db.runCommand( { listshards : 1 } ) command will show the sharding cluster details. 


2. You can "shard" any database and collections you want. SSH connect to one of the router server, execute below:
  ```
  $mongo -u "<mongouser>" -p "<mongopassword>" "admin"

  db.runCommand({enableSharding: "<database>" })

  sh.status()

  sh.shardCollection("<database>.<collection>", shard-key-pattern)

  exit
  ```


3. You can add more shards into this sharding cluster. SSH connect to one of the router server, execute below:
  ```
  $mongo -u "<mongouser>" -p "<mongopassword>" "admin"

  sh.addShard("<replica set name>/<primary ip>:27017")   

  exit
  ```

  Before adding your own replica set into the sharding cluster, you should enable internal authentication in your replica set first, and make sure the replica set is accessiable through this sharding cluster.


##Known Limitations
- The MongoDB version is 3.6.
- This cluster only has 8 shards (standalone, NOT replicas - not recommended for production use), you can add more shards after the deployment. 
- The nodes use internal authentication. So if you want to add your own replica set into this sharding cluster, you should enable the internal authentication in your replica set first. Check any node /etc/mongokeyfile for more details.
- More MongoDB usage details please visit MongoDB website https://www.mongodb.org/ .