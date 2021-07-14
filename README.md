


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
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.0/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-4.0.asc
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

# 5 - Adding secondary members

```bash
rs.add("kakaroto0002.sou.jeff")
```
```bash
rs.add("kakaroto0003.sou.jeff")
```

# 6 - Start slave service on mongo console (access slaves Mongo Servers)

```bash
vim /etc/mongorc.js
```
```bash
db.getMongo().setSecondaryOk()
```

# 7 - Create a database to test replication

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
# 8 - Adding e new node on replication set MongoDB

On primary node 

```bash
rs.add("kakaroto0004.sou.jeff")
```
On secondary host
```bash
vim /etc/mongorc.js
```
```bash
db.getMongo().setSecondaryOk()
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
umount -l /dev/centos/snapshot
```
```bash
dd if=/dev/centos/snapshot | gzip > /home/bkp/snapshot20210714.gz
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



