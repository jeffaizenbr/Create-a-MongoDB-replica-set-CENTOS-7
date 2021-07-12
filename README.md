


# 1 - Resolving Requirements 

```bash
vim /etc/hosts
```
```bash
192.168.122.30 kakaroto0001 kakaroto0001.sou.jeff
192.168.122.31 kakaroto0002 kakaroto0002.sou.jeff
192.168.122.32 kakaroto0003 kakaroto0003.sou.jeff
```

# 2 - Edit mongod.conf

```bash
vim /etc/mongod.conf
```

```bash
replication:
  replSetName: "rs0"
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

