# how-to-deploy-basic-MongoDB-replica-set
This is a documentation on how to deploy basic MongoDB replica set on a 3-node cluster.

## Step1 Create an /etc/yum.repos.d/mongodb-enterprise.repo file in each node.

    [mongodb-enterprise]
    name=MongoDB Enterprise Repository
    baseurl=https://repo.mongodb.com/yum/redhat/$releasever/mongodb-enterprise/4.0/$basearch/
    gpgcheck=1
    enabled=1
    gpgkey=https://www.mongodb.org/static/pgp/server-4.0.asc
    
## Step2 Add following 3 lines to /etc/yum.conf and create certificate files for each nodes:    
   
    proxy=http://webproxy.xx.com:80
    proxy_usernamexx
    proxy_password=xx


## Step3 Install MongoDB server.

    sudo yum install -y mongodb-enterprise
    
## Step4 Create mongodb folders in each node and grant the permission to mongod.

    mkdir -p /mongo_install/log /mongo_install/conf /mongo_install/ssl
    Chown -R mongod:mongod /mongo_install/
    Chown -R mongod:mongod /mongo_data/
    
## Step5 Create mongod-noau.conf under /mongo_install/conf/ in each node with no authentication enabled.


    storage:
      dbPath: /mongo_data
    net:
      bindIp: 10.xx.xx.xx,localhost
      port: 27017
    systemLog:
      destination: file
      path: /mongo_install/log/mongod.log
      logAppend: true
    processManagement:
      fork: true
    replication:
      replSetName: xx


## Step6 Start mongod in each node.

    sudo -u mongod mongod -f /mongo_install/conf/mongod.conf

## Step7 Start mongo shell.

    mongo

## Step8 Create replica set on mongo shell:

    Method1:
    
    rs.initiate()
    rs.add(""xx2.xx.com:27017")
    
    Then changed the Ip address of member1 to hostname by:
    
    cfg = rs.conf()
    cfg.members[0].host = "xx1.xx.com:27017"
    rs.reconfig(cfg)

    
    
    Method2:
    
    rs.initiate(
    {
      _id: "myReplSetName",
      version: 1,
      members: [
         { _id: 0, host : "xx1.xx.com:27017" },
         { _id: 1, host : "xx2.xx.com:27017" },
      ]
    }
    )
    

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

### Step1 Change the mongod-x509.conf to following content:

    storage:
      dbPath: /mongo_data
    net:
      bindIp: 10.xx.xx.xx,localhost
      port: 27017
      ssl:
      mode: requireSSL
      PEMKeyFile: /mongo_install/ssl/xx.xx.com.mongo.pem
      CAFile: /mongo_install/ssl/xx.com.ca.pem
    systemLog:
      destination: file
      path: /mongo_install/log/mongod.log
      logAppend: true
    security:
      authorization: enabled
      clusterAuthMode: x509
    processManagement:
      fork: true
    replication:
      replSetName: xx


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
    use admin
    db.shutdownServer()

### Step6 Start mondod in each node:

    sudo -u mongod mongod -f /mongo_install/conf/mongod-x509.conf
    
### Step7 Logon to the new replica set by mongo shell:

    mongo admin --host defaced1 -u 'admin' -p "admin" --ssl --sslPEMKeyFile ../ssl/defaced1.pem --sslCAFile ../ssl/mongoCA.crt


## LDAP integration with AD servers with native LDAP in MongoDB

### Step1  Create mongod-x509-ldap.conf 

we need to create roles before enable LDAP mongod process

  Here is the example configuration on how to enable LDAP anthentication with Aactive Directory servers.
  
    storage:
      dbPath: /mongo_data
    net:
      bindIp: xx.xx.xx.xx,localhost
      port: 27017
      ssl:
          mode: requireSSL
          PEMKeyFile: /mongo_install/ssl/xx.pem
          CAFile: /mongo_install/ssl/xx.ca.pem
    systemLog:
      destination: file
      path: /mongo_install/log/mongod.log
      logAppend: true
    security:
      authorization: enabled
      clusterAuthMode: x509
      ldap:
        servers: "ldap.xx.com"
        userToDNMapping: '[{match: "(.+)",ldapQuery: "dc=xx,dc=com??sub?(sAMAccountName={0})"}]'
        authz:
          queryTemplate: "OU=xx,DC=xx,DC=com??sub?(&(objectClass=group)(member={USER}))"
        bind:
          queryUser: "xx@xx.com"
          queryPassword: "xx"
    setParameter:
        authenticationMechanisms: 'PLAIN'
    processManagement:
      fork: true
    replication:
      replSetName: xx
~
### Step2 Create admin roles linked to AD group before enable LDAP mongod process.


    db.createRole({role: "<AD group DN>", privileges: [], roles: [{ role: "userAdminAnyDatabase", db: "admin" },{ role: "clusterAdmin", db: "admin" }]})


### Step3 Shutdown the existing mongod process and start new mongod with mongod-x509-ldap.conf.

     sudo -u mongod mongod -f mongod-x509-ldap.conf

### Step4 You can use following example to login to the mongodb cluster:

     mongo --host xx -u 'xx' -p  --authenticationMechanism PLAIN --authenticationDatabase '$external' --ssl --sslPEMKeyFile /mongo_install/ssl/xx.pem --sslCAFile /mongo_install/ssl/xx.ca.pem
     
### Step5 Create other required roles from application database, like this:

     db.createRole({role: "<AD group DN>", privileges: [],  roles: [{ role: "readWrite", db: "<application database name>"}]})


 
