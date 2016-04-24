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


#####security
######surface area
default port:  
standalone:27017  --shardsvr: 27018 --configsvr:27019  mongos:27017


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
