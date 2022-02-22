


# 1 - Resolving Requirements 

```bash
vim /etc/hosts
```
```bash
192.168.122.30 kakaroto0001 kakaroto0001.sou.jeff
192.168.122.31 kakaroto0002 kakaroto0002.sou.jeff
192.168.122.32 kakaroto0003 kakaroto0003.sou.jeff
```

Configure MongoDB Repository 4.0

```bash
[mongodb-org-4.0]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/7/mongodb-org/4.4/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-4.4.asc
```

```bash
sudo yum install mongodb-org -y
```

```bash
sudo systemctl start mongod ; sudo systemctl enable mongod
```

# 2 - Edit mongod.conf

```bash
vim /etc/mongod.conf
```

```bash
replication:
  replSetName: "rs0"
  
  
  
  wiredTiger:
    engineConfig:
     cacheSizeGB: 12

```

```bash
systemctl restart mongod
```

# 3 - Access mongo console

```bash
mongo
```

# 4 - Initiate replication service

```bash
rs.initiate()
```

press ENTER to become "PRIMARY" on mongo console

```bash
rs0:PRIMARY>
```

# 5 - Start slave service on mongo console (access slaves Mongo Servers)

```bash
vim /etc/mongorc.js
```
```bash
db.getMongo().setSecondaryOk()
```

# 6 - Create a database to test replication

```bash
use newdatabase
```

```bash
db.newdatabase.insert({nome: "Ada Lovelace", idade: 205})
WriteResult({{"nInserted" : 1 })
```

```bash
show dbs
```

```bash
db.newdatabase.find()
```
Important, 
to elect a primary node to secondary, type the command below 
```bash
rs.stepDown(120)
```
To view the status of syncronization nodes
```bash
rs.printSecondaryReplicationInfo()
```
# 7 - Adding new cluster hidden mode (priority 0)
A) Access primary MongoDB cluster, and put de command below 

```bash
rs.add({ host: "kakaroto0005:27017", priority: 0, votes: 0, hidden: true })
```
Obs: If you want do remove a cluster member, type the command below : 
```bash
rs.remove("kakaroto0005:27017")
```

B) Test the replication process
```bash
rs.printSlaveReplicationInfo()
```

```bash
rs.status()
```

C) Convert the cluster member from hidden to secondary

```bash
cfg = rs.conf();
```

```bash
cfg.members[4].priority = 0.5
```

```bash
cfg.members[3].votes = 1
```

```bash
cfg.members[3].hidden = false
```

```bash
print(cfg)  OR  printjson(cfg)
```

```bash
rs.reconfig(cfg)
```

# 8 - setting password on mongoDB cluster

create key pass on master

```bash
openssl rand -base64 768 > /var/lib/mongo/mongo-security/keyfile.txt
```

```bash
chmod 400 keyfile.txt ; chown mongod:mongod /keyfile.txt
```
Copy file do slave servers
```bash
scp keyfile.txt root@camus0002:/var/lib/mongo/mongo-security
```
Create user on admin database
```bash
use admin
```
```bash

db.createUser(
   {
       user: "admin", 
       pwd: "123", 
       roles:["root"]
   })
```
OR
```bash
db.createUser(
 {
 user: "admin",
 pwd: "senha",
 roles: [ "userAdminAnyDatabase",
          "dbAdminAnyDatabase",
          "readWriteAnyDatabase"]
 }
)
```
You can add a role to a user that already exist on database

"use DATABASE"
```bash
db.grantRolesToUser('user1', ['readWriteAnyDatabase']);
OR
db.grantRolesToUser('user1', [{ role: 'readWrite', db: 'account' }]);
```

put the line below on /etc/mongod.conf

```bash
security:
   authorization: enabled
   keyFile: /var/lib/mongo/mongo-security/keyfile.txt
```
restart de mongod
```bash
systemctl restart mongod
```
teste de authentication
```bash
mongo --port 27017 -u "USER" -p "SENHA" --authenticationDatabase "admin"
```

# 9 - Backup MongoDB

A) Creating parition the mirror MongoDB volume
```bash
lvcreate --size 500M --snapshot --name mongo-snap /dev/centos/data
```
B) Mounting partition
```bash
mkdir -p /.snapshot
```
```bash
vim /etc/fstab
```
```bash
/dev/centos/mongo-snap   /.snapshot auto  defaults 0 0
```
At this point the volume ".snapshot" will mirror MongoDB partition

C) Making a GZIP file from entire "./snapshot" partition
```bash
umount -l /dev/centos/mongo-snap
```
```bash
dd if=/dev/centos/mongo-snap | gzip > /home/bkp/snapshot20210714.gz
```
D) Restauring the backup “snapshot20210714.gz”

Creating the partition to deploy the backup
```bash
lvcreate --size 650M --name mongorestore centos
```
Initiate restore
```bash
gzip -d -c /home/bkp/snapshot20210714.gz | dd of=/dev/centos/mongorestore
```
Mouting new parition on the same MongoDB volume
```bash
vim /etc/fstab
```
```bash
/dev/centos/mongorestore   /data/mongodb xfs  defaults 0 0 
```
# 10 - monitoring mongo backup

create a script like these
```bash
#!/bin/bash

#Erase files older than 2 days

#find /backup/*.gz -mtime +1 -exec rm -rf {} \;


#Make Mongo Backup

mongodump --db DBNAME --gzip --archive=/backup/DBNAME.gz

#Test integrity file

if gunzip -t /backup/DBNAME.gz
then
echo "0"

else

echo "1"
fi

#Rename file

mv /backup/DBNAME.gz /backup/$(date +%F)DBNAME.gz

```
Copy that line do crontab file
```bash
01 23 * * * bash /etc/batch/administracao/backupmongo.sh > /root/statusmongo.txt
```
Copy that line to zabbix configuration file
```bash
UserParameter=mongodb,cat /root/statusmongo.txt
```

