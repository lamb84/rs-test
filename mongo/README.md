# Deployment Guide Overview

Due to the small data footprint in dev, MongoDB cluster is deployed differently from Production.
It does not incorporate any sharding for any DB.  The cluster is a replica set.
For more info about replica set, please refer to [Mongo Replication](https://docs.mongodb.com/manual/replication/)
For more info about building a replica set, please refer to [Deploy a Replica Set](https://docs.mongodb.com/manual/tutorial/deploy-replica-set/)

This guide provides details about the `forestry-dev` replica set deployment 

## Hostnames and Replica Set name
| name          | value                 | decleration location   | description         |
| ------------- |:---------------------:|:----------------------:|--------------------:|            
| Replica Set Name   | dev-gcp-replica-set   | mongo-config configmap yaml file | alos used when initializing the replica set    |
| mongo-1 hostname             | mongo-1:27017         | mongo-1 service yaml file        | Only used in replice set communication |
| mongo-2 hostname             | mongo-2:27017         | mongo-2 service yaml file        | Only used in replice set communication |
| mongo-3 hostname             | mongo-3:27017         | mongo-3 service yaml file        | Only used in replice set communication |   
| replica set cluster hostname | mongo:27017         | mongo service yaml file        | Used for external services to communicate with replica set cluster |

## Deployment Instruction

1. Apply svc, configmap and deployment yaml files
```
$ kubectl apply -f cumul8-platforms/forestry/dev/forestry-dev-new/services/mongo
$ kubectl apply -f cumul8-platforms/forestry/dev/forestry-dev-new/configs/mongo_config.yaml
$ kubectl apply -f cumul8-platforms/forestry/dev/forestry-dev-new/deployments/mongo
```
2. Get into shell of any one of the Mongodb pods
```
kubectl -n forestry-dev exec -ti mongodb-config-rsa-78869d45fc-6j648 /bin/bash
```
3. In the shell, invoke mongo shell by typing `mongo` in BASH shell
4. Initialize replica set by running the command below in `mongo shell`
```
rsconf = {
  _id: "dev-gcp-replica-set",
  members: [
    {
     _id: 1,
     host: "mongo-1:27017"
    },
    {
     _id: 2,
     host: "mongo-2:27017",
	 priority: 100
    },
    {
     _id: 3,
     host: "mongo-3:27017"
    }
   ]
}
```
Important: please note the `priority` value on mongo-2.  This key ensures that mongo-2 is always the primary of the replicaset.
Only primary member can handle write. Please make sure that the service is always pointing to the primary.
 
```
rs.initiate( rsconf )
```
5. Check replica set status by running the command below in `mongo shell`
```
dev-gcp-replica-set:PRIMARY> rs.config()
{
	"_id" : "dev-gcp-replica-set",
	"version" : 1,
	"protocolVersion" : NumberLong(1),
	"members" : [
		{
			"_id" : 1,
			"host" : "mongo-1:27017",
			"arbiterOnly" : false,
			"buildIndexes" : true,
			"hidden" : false,
			"priority" : 1,
			"tags" : {
				
			},
			"slaveDelay" : NumberLong(0),
			"votes" : 1
		},
		{
			"_id" : 2,
			"host" : "mongo-2:27017",
			"arbiterOnly" : false,
			"buildIndexes" : true,
			"hidden" : false,
			"priority" : 100,
			"tags" : {
				
			},
			"slaveDelay" : NumberLong(0),
			"votes" : 1
		},
		{
			"_id" : 3,
			"host" : "mongo-3:27017",
			"arbiterOnly" : false,
			"buildIndexes" : true,
			"hidden" : false,
			"priority" : 1,
			"tags" : {
				
			},
			"slaveDelay" : NumberLong(0),
			"votes" : 1
		}
	],
	"settings" : {
		"chainingAllowed" : true,
		"heartbeatIntervalMillis" : 2000,
		"heartbeatTimeoutSecs" : 10,
		"electionTimeoutMillis" : 10000,
		"catchUpTimeoutMillis" : -1,
		"catchUpTakeoverDelayMillis" : 30000,
		"getLastErrorModes" : {
			
		},
		"getLastErrorDefaults" : {
			"w" : 1,
			"wtimeout" : 0
		},
		"replicaSetId" : ObjectId("5bd7972923c1dbbb38017fae")
	}
}
```