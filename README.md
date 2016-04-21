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
#####data in out
mongodump  
by default, it will dump data into /dump folder of root folder
```
mongodump --port 27017 --host x --out d:\..
```
######oplog
can not run on standalone mongo, must create a replica set
```
mongodump --oplog --port 20000
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
