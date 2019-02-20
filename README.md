# how-to-deploy-basic-MongoDB-replica-set
This is a documentation on how to deploy basic MongoDB replica set on a 3-node cluster.

## Step1 Create an /etc/yum.repos.d/mongodb-enterprise.repo file in each node.

    [mongodb-enterprise]
    name=MongoDB Enterprise Repository
    baseurl=https://repo.mongodb.com/yum/redhat/$releasever/mongodb-enterprise/4.0/$basearch/
    gpgcheck=1
    enabled=1
    gpgkey=https://www.mongodb.org/static/pgp/server-4.0.asc

## Step2 Install MongoDB server.

    yum install -y mongodb-enterprise

## Step3 Create mongodb.conf in each node.


    storage:
      dbPath: /home/mongodb/data
    net:
      bindIp: 172.16.155.5,localhost
      port: 27017
    systemLog:
      destination: file
      path: /home/mongodb/log/mongod.log
      logAppend: true
    processManagement:
      fork: true
    replication:
      replSetName: rs0


## Step4 Create mongodb folders in each node.

    mkdir -p /home/mongodb/log /home/mongodb/data /home/mongodb/conf

## Step5 Start mongod in each node.

    mongod -f mongod.conf

## Step6 Start mongo shell.

    mongo

## Step7 Create replica set on mongo shell:

    rs.initiate()
    rs.add("172.16.178.169:27017")
    rs.add("172.16.180.205:27017")

## step8 Check replica set status by:

    MongoDB Enterprise rs0:PRIMARY> rs.status()
    {
	"set" : "rs0",
	"date" : ISODate("2019-02-20T14:12:29.755Z"),
	"myState" : 1,
	"term" : NumberLong(2),
	"syncingTo" : "",
	"syncSourceHost" : "",
	"syncSourceId" : -1,
	"heartbeatIntervalMillis" : NumberLong(2000),
	"optimes" : {
		"lastCommittedOpTime" : {
			"ts" : Timestamp(1550671946, 1),
			"t" : NumberLong(2)
		},
		"readConcernMajorityOpTime" : {
			"ts" : Timestamp(1550671946, 1),
			"t" : NumberLong(2)
		},
		"appliedOpTime" : {
			"ts" : Timestamp(1550671946, 1),
			"t" : NumberLong(2)
		},
		"durableOpTime" : {
			"ts" : Timestamp(1550671946, 1),
			"t" : NumberLong(2)
		}
	},
	"lastStableCheckpointTimestamp" : Timestamp(1550671886, 1),
	"members" : [
		{
			"_id" : 0,
			"name" : "172.16.155.5:27017",
			"health" : 1,
			"state" : 1,
			"stateStr" : "PRIMARY",
			"uptime" : 204537,
			"optime" : {
				"ts" : Timestamp(1550671946, 1),
				"t" : NumberLong(2)
			},
			"optimeDate" : ISODate("2019-02-20T14:12:26Z"),
			"syncingTo" : "",
			"syncSourceHost" : "",
			"syncSourceId" : -1,
			"infoMessage" : "",
			"electionTime" : Timestamp(1550467457, 1),
			"electionDate" : ISODate("2019-02-18T05:24:17Z"),
			"configVersion" : 3,
			"self" : true,
			"lastHeartbeatMessage" : ""
		},
		{
			"_id" : 1,
			"name" : "172.16.178.169:27017",
			"health" : 1,
			"state" : 2,
			"stateStr" : "SECONDARY",
			"uptime" : 204501,
			"optime" : {
				"ts" : Timestamp(1550671946, 1),
				"t" : NumberLong(2)
			},
			"optimeDurable" : {
				"ts" : Timestamp(1550671946, 1),
				"t" : NumberLong(2)
			},
			"optimeDate" : ISODate("2019-02-20T14:12:26Z"),
			"optimeDurableDate" : ISODate("2019-02-20T14:12:26Z"),
			"lastHeartbeat" : ISODate("2019-02-20T14:12:29.700Z"),
			"lastHeartbeatRecv" : ISODate("2019-02-20T14:12:28.377Z"),
			"pingMs" : NumberLong(0),
			"lastHeartbeatMessage" : "",
			"syncingTo" : "172.16.155.5:27017",
			"syncSourceHost" : "172.16.155.5:27017",
			"syncSourceId" : 0,
			"infoMessage" : "",
			"configVersion" : 3
		},
		{
			"_id" : 2,
			"name" : "172.16.180.205:27017",
			"health" : 1,
			"state" : 2,
			"stateStr" : "SECONDARY",
			"uptime" : 204480,
			"optime" : {
				"ts" : Timestamp(1550671946, 1),
				"t" : NumberLong(2)
			},
			"optimeDurable" : {
				"ts" : Timestamp(1550671946, 1),
				"t" : NumberLong(2)
			},
			"optimeDate" : ISODate("2019-02-20T14:12:26Z"),
			"optimeDurableDate" : ISODate("2019-02-20T14:12:26Z"),
			"lastHeartbeat" : ISODate("2019-02-20T14:12:28.052Z"),
			"lastHeartbeatRecv" : ISODate("2019-02-20T14:12:28.889Z"),
			"pingMs" : NumberLong(0),
			"lastHeartbeatMessage" : "",
			"syncingTo" : "172.16.178.169:27017",
			"syncSourceHost" : "172.16.178.169:27017",
			"syncSourceId" : 1,
			"infoMessage" : "",
			"configVersion" : 3
		}
	],
	"ok" : 1,
	"operationTime" : Timestamp(1550671946, 1),
	"$clusterTime" : {
		"clusterTime" : Timestamp(1550671946, 1),
		"signature" : {
			"hash" : BinData(0,"pwDMCs0ahzkZWaUii2XXM2iU0l0="),
			"keyId" : NumberLong("6659198646141059074")
		}
	  }
      }


## Step9 Create First admin user:

    use admin
    db.createUser({user: 'admin', pwd: 'admin', roles: ['root']})

## Step10 Add internal authentication by key file by editing mongod.conf in each node:

  
    storage:
      dbPath: /home/mongodb/data
    net:
      bindIp: 172.16.155.5,localhost
      port: 27017
    security:
      authorization: enabled
      keyFile: /home/mongodb/conf/rs0-keyfile
    systemLog:
      destination: file
      path: /home/mongodb/log/mongod.log
      logAppend: true
    processManagement:
      fork: true
    replication:
    replSetName: rs0
  
## Step11 Create key file in each node:

    openssl rand -base64 741 > rs0-keyfile
    chmod 400 rs0-keyfile
    scp rs0-keyfile root@defaced2:/home/mongodb/conf/
    scp rs0-keyfile root@defaced3:/home/mongodb/conf/

## Step12 Log onto mongoDB cluser:

    mongo admin -u "admin" -p "admin"

## Step13 Stop mongod in each node(start from secondary node):

    db.shutdownServer()

## Step14 Restart mongod in each node:

    mongod -f mongod.conf

## Step15 Create additional users in mongo shell.

    db.createUser({user: "userAdmin", pwd: "badges", roles: [{ role:"userAdminAnyDatabase", db: "admin" }]})
    db.createUser({user: "reader", pwd: "books", roles: [{ role:"read", db: "acme" }]})

