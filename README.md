#### psmongoadmin
#####install
######package content
default location /data/db
######disable last access time
edit /etc/fstab,add
```
/ ext4 defaults,noatime 0 0
```
create data dir
```
md d:\mongoadmin\data
md d:\mongoadmin\log
md d:\mongoadmin\config
```
in config, create mongod.conf (yaml)
```
storage:
  dbPath: "d:/mongoadmin/data"
systemLog:
  destination: file
  path: "d:/mongoadmin/log/mongod.log
```
linux
```
storage:
  dbPath: "/data/db"
systemLog:
  destination: file
  path: "/data/log/mongod.log"
```
######grant permission
```
icacls d:\mondoadmin /grant mongodb:(OI)(CI)F
```

######install service
```
mongod --install --config d:\mongoadmin\config\mongod.conf ==serviceUser mongodb --servicePassword pass
```
verify
```
sc qc mongodb
```
start
```
net start mongodb
```
start mongo:
```
mongo
```
stop:
```
net stop mongodb
mongod --remove
sc qc mongodb
```
######linux
```
sudo service mongod start
```
checklog
```
sudo service mongod stop  //=sig 15
tail -n 10 /var/log/mongod/mongod.log 
```
other way
```
use admin
db.shotdownServer()
```
safe kill -check the lock
```
ls -la /var/lib/mongodb
```
######danger kill in windows
```
taskkill /F /IM mongod.exe
```
######restore: change port
```
mongod --port 20000 --depath d:/mongoadmin/data
```

#####config
######minimal
linux
```
storage:
  dbPath: "/data/db"
systemLog:   (or just one config: destination: syslog)
  logAppend: true
  logRotate: reopen
  destination: file
  path: "/data/log/mongod.log"
```
######storage
engine: mmapv1  //default  or wiredTiger
######journal
```
storage:
  dbPath: "/data/db"
  journal:
    enabled: true
```
######flush interval
2 -300 ms.
```
storage:
  dbPath: "d:/mongoadmin/data"
  journal:
    enabled: true
  engine: mmapv1
  mmapv1: 
    journal:
      commitIntervalMs: 100
```

check
```
db.serverStatus().dur
```
######each dict per db
```
  journal:
    enabled: true
  directoryPerDB: true
```
######ip
getIp:
```
netsh interface ip show address
```
add net section
```
storage:
  dbPath: "d:/mongoadmin/data"
  journal:
    enabled: true
net:
  bindIp: add1,add2..
  port:27017
systemLog:
```
######enable http gui
```
net:
  bindIp: add1,add2..
  port:27017
  http:
    enabled: true
```
#####query crash
save will update old value, insert will not.
######temporal
```
ObjectId("123123123").getTimestamp() //get time portion
```
######cursor
```
var x =db.x.find()
x.next()
```
#####data in out (--db = -d  --collection=-c,--quert=-q,also can use,--sort,--limit...)
[see here](https://docs.mongodb.org/manual/reference/program/mongoexport/)
mongodump  
```
dir /s /b
```
by default, it will dump data into /dump folder of current folder
```
mongodump --port 27017 --host x --out d:\..
```
######oplog
can not run on standalone mongo, must create a replica set
```
mongodump --oplog --port 20000  //20000 is replica port
```
######dump specific db
by default mongodump will dump all db
```
mongodump --db dbname
mongodump --db dbname --collection tbname
```
######credentials
```
mongodump --username a --password b --out d:\..
```
######mongorestore
```
mongorestore \dump
```
###### drop
restore from back files, will drop existing files on server (fullrestore)
```
mongorestore --drop  \dump
```
######restore collection
```
mongorestore --drop --collection tbname --db dbname \dump\tbname\a.bson
```
######restore to a new database
```
mongorestore --db newdb \dump
```
###### --oplogReplay
######mongoimport
```
mongoimport --db dbname --collection collectionnaeme file.json
```

######upsert
mongoimport again (overwrite pprevious one)
```
mongoimport --db dbname --collection collectionname --upsert file.json
```
######upsertFields (need to review)
For example, in database we have no id field, we can use another filed to substitute. *
sourse file:
```
{ a: x, b:y}
```
mongodb:
```
{_id:x,a:x,b:y}
```
to import we can use
```
mongoimport --db dbname --collection collectionname --upsert --upsertFields a file.json
```
######csv
to tell mongodb a csv file has headerline,
```
mongoimport --type csv --headerline --db name --collection name file.csv
```
second way: tell columns of csv, (suitable if csv has no headerline)
```
mongoimport --type csv --fields _id,a,b --db ....
```
######--fieldFile
filename:
```
_id
a
b
```
import
```
mongoimport --type csv --fieldFile filename --db ....
```
######mongoexport
```
mongoexport --db xx --collection xx
mongoexport --db xx --collection xx --out xx.json
```
if export to csv, will not include id,and you must specify field names
```
mongoexport --db xx --collection xx --type=csv --fields a
```
######complex document
if b is a object
```
mongoexport --db xx --collection xx --type=csv --fields a,b.a,b.a
```
######selective export
```
mongoexport --db xx --collection xx --query "{_id:{$gt:2}}"
```
#####indexing
######_id
```
db.cl.getIndexes()
db.system.indexes.find()
db.cl.dropIndex('f')
```
######explain
Ex:this query
```
db.col.find(...
```
check winningPlan->stage = "COLLSCAN"  not good,it means need to scan every doc in the collection..  
More detail:
```
.explain('executionStats')
```
check executionStats->totalDocsExamined  
nReturned->matching  
If created index, if should be stage->FETCH, inputStage usage->IXSCAN

######multifield index match
```
db.cl.createIndex({'a':1,'b':1,'c':1})
```
match must from leftmost.
```
db.cl.find({'a':x,'b':x}).sort({c:1}).explain()
db.cl.find({'a':x}).sort({c:1}).explain()   //sort will use slow scan
```
######sparse index
sparse index cannot be properly sorted, must use hint
```
db.cl.createIndex({sparseidx:1},{sparse:true})
db.cl.find().sort({sparseidx:1}).hint({sparseidx:1}).explain('executionStats')
```
######unique index(doc with same key valuecannot insert twice)
```
db.cl.createIndex({uniqidx:1},{unique:true})
```
###### TTL index (one ttl per collection)
```
db.cl.createIndex({field:[-1|1]},{expireAfterSeconds:1000});
```
######rebuild
rebuild all indexes on a collection
```
db.cl.reIndex()
```
or
```
db.runCommand({compact:'cl'})
```



#####replica sets
######capped coll
when write to a server, it will convert to oplog and write to another secondaries

######simple rep
```
mongod --dbpath . --replSet r1 --oplogSize 1  // how large megabyte
```
######reconfig
```
var conf = rs.config()
rs.reconfig(conf)
```
#######more
```
mongod --dbpath . --port (1-3) --replSet r1
mongo --port 1
rs.initiate()
rs.add(127.0:30002)
rs.add(127.0:30002,true) //will be arbiter, do not store any data
```
connect to another server
```
db= connect("mysserver:30002/test")
db.setSlaveOk()
```
######failover
our current cluster has 2 server,arbiter. If a server and arbiter die, the cluster will die
######priority
```
var conf = rs.config()
conf.members[0].priority=10
```
```
db.getMongo()
```
######stepdown
```
rs.stepDown(20) //shut down temperarily   //rs.stepDown()  noparam=60 sec
```
######freeze
let a second server not to become primary server for some time
```
rs.freeze(40)
```
######hidden
cannot become primary forever
```
cfg.members[2].priority=0
cfg.members[2].hidden=true
```

######chaining
dc2 pull data from dc1
allow: pull from secondary
disallow: pull from primary
######durability

######writeconcern
```
db.cl.insert({x:'hi'},{writeConcern:{w:0}})  //0 member need to ackowledge
db.cl.insert({x:'hi'},{writeConcern:{j:true}})  //normal
db.cl.insert({x:'hi'},{writeConcern:{w:1}}) 
```
######timeout
```
db.cl.insert({x:'hi'},{writeConcern:{w:1,wtimeout:1000}}) 
```


#####sharding (sharding:scalibility replicaset:durability)
[see here for version 3.2](https://docs.mongodb.org/manual/tutorial/deploy-shard-cluster/)
structure:  
application->mongos(router) ,a cinfig server,some shard server
######simplistic topology
start configserver
```
start mongod --configsvr --dbpath test/cfg1
```
start mongos routing based on above
```
start mongos --configdb myserver
```
start two mongod server for shards
```
md test/s1
md test/s2
mongod --port 30001 --dbpath test/s1
mongod --port 30002 --dbpath test/s2
```
by default mongos will run on 27017. start:
```
mongo
```
after log in it should be mingis prompt
```
sh.addShard('myserver:30001')
sh.addShard('myserver:30002')
```
ebable sharing
```
sh.enableSharding('dbname')
sh.shardCollection('dbname.cl',{'field.a':1,'field.b':1})
```
######using sharded cluster
```
db.chunks.find().pretty()
```
save doc
```
doc
db.cl.save(doc)
```
######production topology
we use 3 config server+3 shard server
```
start mongos --configdb myserver:40001 myserver:40002 myserver:40003
```
syntax to add a replica server
```
sh.addShard("r1/myserver:27018")  //replicaname/servername :port
```


######evaluation
hashed key- only on a single field
```
sh.shardCollection("dbname.cl",{"fn":"hashed"})
```
######tag aware demo
refer  `production topology 4:37`
```
sh.status()
```
will get
```
shards:{"_id":"shard0000","host":"localhost:30000"}
....
```
add tag
```
sh.addShardTag('shard0000",'tagname');
```
add range definition: syntax: keyname, from(inclusive) to(exclusive),tag
```
sh.addTagRange("db.cl",{country:"01"},{country:"02"},"newtagname")
sh.addTagRange("db.cl",{country:"01"},{country:MaxKey},"newtagname2")
```
enable
```
sh.shardCollection('db.cl',{country:1})
```
######move chunk
```
sh.moveChunk("db.cl",{country:"01},"shard0003"}
```
######verify
```
connect('localhost:30000/db').messages.find()
```

#####monitering
######log file
```
db.setLogLevel(0,'query')   //verboity 0-5, topic (query, accessControl,command,control,geo,index,network,replication,storage,joirnal,write)
```
-1 means inherit the default
```
db.getLogComponents()
```
show log
```
show log global
```
######settting verbosity configu
mongod.conf
```
systemLog:
  destination: file
  path: ".mongod.log"
  component:
    command:
      verbosity: 2
```
######query profiler
```
show profile
```
set profile,0(no profiling)-2, threshold(ms), if use 1 , any operations take longer than threshold will be logged
```
db.setProfilingLevel(0,100)
```
######query slow queries
```
db.system.profile.findOne({op:'query':ns:'dbname.cl'})
```
######mongostat
basic
```
mongostat --host xx --port xx --rowcount 2  //show how many times
```
######mongotop
show time spend
```
mongotop --host xx --port xx --rowcount 2  //show how many times
```
######db.stats()
```
db.stats(1024) //in kb
```
diagnostic: mongostat->db.stats()->db.col.stats()->explain()
######db.serverStatus()
```
db.serverStatus();
db.runCommmand({serverStatus:1})
```





#####security
######surface area
default port:  
standalone:27017  --shardsvr: 27018 --configsvr:27019  mongos:27017
######ssl
```
net:
  ssl:
    mode:requireSSL
    PEMKeyFile: ..pem
    PEMKeyPassword: 
    CAFile:
```
connect:
```
mongo --ssl -sslCAFile x.pem --host hostname --sslPEMKeyPassword pass
```
######replica set with SSL
m1.conf
```
storage:
  dbPath: ../m1
replication:
  replSetName: r1
net:
  port: 30001
  ssl:
    mode:requireSSL
    PEMKeyFile: ..pem
    PEMKeyPassword: 
    CAFile:
```
m2.conf
```
storage:
  dbPath: ../m2
replication:
  replSetName: r1
net:
  port: 30002
  ssl:
    mode:requireSSL
    PEMKeyFile: ..pem
    PEMKeyPassword: 
    CAFile:
```
start both
```
mongod --config 1.conf
mongod --config 2.conf
```
```
mongo --port 30001 --ssl -sslCAFile x.pem --host hostname --sslPEMKeyPassword pass
```
when connected, use
```
cfg
```
to check replSet
```
rs.initiate(cfg)
rs.status()
```
######roles
root, userAdminAnyDatabase
######creating a user
conf:
```
storage:
  dbPath: 
secirity:
  authorization: enabled
```
start
```
mongod --config .conf
```
first time login, no password required
```
use admin
usr
db.createUser(usr)
```
login again
```
mongo --username superUserAdmin --password 1234 myserver/admin
```
create new user
```
var n ={user:'reporter',pwd:'123',roles:[{role:'read',db:'db'}]}
var app={user:'app',pwd:'123',roles:[{role:'readWrite',db:'db'}]}
db.createUser(n)
```
current user is superUserAdmin,
```
show collections //will fail
db.grantRolesToUser("superUserAdmin",['readWrite"])
```
```
db.system.users.findOne()
```
other api
```
db.dropUser("superUserAdmin")
db.revokeRolesFromUser("superUserAdmin",["read"])
```
######logging in
```
db.logout()
```
re log in
```
db.auth('app','1234')
```
```
use dbname
db.createUser({'user':'a','pwd':'p','roles':['read']})
```
show all users
```
use admin
db.system.users.find({},{credentials:0}),pretty()
```
grant a user to another database
```
db.grantRolesToUser("a",[{role:'read',db:'other'}])
```
login user must using authenticationDatabase
```
mongo --username a --password 123 --authenticationDatabase db
```

######key file in cluster
```
security:
  keyFile:
  clusterAuthMode:
```

#####misc
######meomry
working set is the amount of memory that mongo needs in order to satisfy an operation
######wiredTiger
speed: mmapv1->wiredTiger->wiredTiger+compression  
in cmd
```
start mongod --storageEngine mmapv1 --port 11111
```
by default, wiredTiger is compressed. If not compress, 
``
start mongod --storageEngine wiredTiger --wiredTigerCollectionBlockCompressor none --wiredTigerJournalCompressor none --wiredTigerIndexPrefixCompression 0 --port 30002 --dbpath .
######directory for indexes
```
start mongod --storageEngine wiredTiger --wiredTigerDirectoryForIndexes --port ..
```
