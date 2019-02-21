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
    
    
## Option #1 Authentication by key file     

### Step1 Add internal authentication by key file by editing mongod.conf in each node:

  
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
  
### Step2 Create key file in each node:

    openssl rand -base64 741 > rs0-keyfile
    chmod 400 rs0-keyfile
    scp rs0-keyfile root@defaced2:/home/mongodb/conf/
    scp rs0-keyfile root@defaced3:/home/mongodb/conf/

### Step3 Log onto mongoDB cluser:

    mongo admin -u "admin" -p "admin"

### Step4 Stop mongod in each node(start from secondary node):

    db.shutdownServer()

### Step5 Restart mongod in each node:

    mongod -f mongod.conf

### Step6 Create additional users in mongo shell.

    db.createUser({user: "userAdmin", pwd: "badges", roles: [{ role:"userAdminAnyDatabase", db: "admin" }]})
    db.createUser({user: "reader", pwd: "books", roles: [{ role:"read", db: "acme" }]})


## Option #2 Authentication by X.509 certificate

### Step1 Change the mongod.conf to following content:

    storage:
      dbPath: /home/mongodb/data
    net:
      bindIp: 172.16.155.5,localhost
      port: 27017
      ssl:
          mode: requireSSL
          PEMKeyFile: /home/mongodb/ssl/defaced1.pem
          CAFile: /home/mongodb/ssl/mongoCA.crt
    systemLog:
      destination: file
      path: /home/mongodb/log/mongod.log
      logAppend: true
    security:
      authorization: enabled
      clusterAuthMode: x509
    processManagement:
      fork: true
    replication:
      replSetName: rs0

### Step2 Generate CA key and certificates.

    openssl genrsa -out mongoCA.key -aes256 8192
    openssl req -x509 -new -extensions v3_ca -key mongoCA.key -days 365 -out mongoCA.crt
    
### Step3 Generate server certificates by running following shell (provide hostname).

    #!/bin/bash
    if [ "$1" = "" ]; then
    echo 'Please enter your hostname (CN):'
    exit 1
    fi
    HOST_NAME="$1"
    SUBJECT="/C=us/ST=tn/L=cha/O=bcbst/OU=it/CN=$HOST_NAME"
    openssl req -new -nodes -newkey rsa:4096 -subj "$SUBJECT" -keyout $HOST_NAME.key -out $HOST_NAME.csr
    openssl x509 -CA mongoCA.crt -CAkey mongoCA.key -CAcreateserial -req -days 365 -in $HOST_NAME.csr -out $HOST_NAME.crt
    rm $HOST_NAME.csr
    cat $HOST_NAME.key $HOST_NAME.crt > $HOST_NAME.pem
    rm $HOST_NAME.key
    rm $HOST_NAME.crt


### Step4 Copy all the pem files and CA file to related node.

    First node:
    
    mkdir ssl
    mv conf/defaced1.pem ssl
    cp conf/mongoCA.crt ssl
    chmod 700 ssl
    
    Copy files to second node:
    
    scp conf/defaced2.pem root@defaced2:/home/mongodb/conf/
    scp conf/mongoCA.crt ssl root@defaced2:/home/mongodb/conf/
    
    Then do same as first node:
    
    mkdir ssl
    mv conf/defaced1.pem ssl
    cp conf/mongoCA.crt ssl
    chmod 700 ssl   
    
### Step5 Stop mongod in each node(start from secondary node):
   
    mongo admin -u "admin" -p "admin"
    db.shutdownServer()

### Step6 Start mondod in each node:

    mongod -f mongod.conf
    
### Step7 Logon to the new replica set by mongo shell:

    mongo admin --host defaced1 -u 'admin' -p "admin" --ssl --sslPEMKeyFile ../ssl/defaced1.pem --sslCAFile ../ssl/mongoCA.crt


