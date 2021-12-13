__IMPORTANT:__

* setups should have aligned authorization settings, in this case easiest would be to temp disable the authorization on both ends

# Step 1: Setup replicaset with one secondary

Disable mongo authorization and create root user:

```
use admin
rs.initiate()
db.createUser(
  {
    user: "webdev",
    pwd: "abc",
    roles: [ { role: "root", db: "admin" } ]
  }
)
```

Auth as admin:

```
db.auth('webdev', 'abc')
```

Create data:

```
use profile2connect
db.profiles.insert({"_id": "test", "username": "Kevin"})
```

Remove all database files of secondary to prepare it to become secondary:

```
rm -rf /var/lib/mongodb/*
```

Add secondary member to rs:

```
rs.add( { host: "mongo_ded01", priority: 0, votes: 0 } )
```

Check rs status:

```
rs.status()
```

Set slaveOk=true to perform queries on secondary:

```
rs.slaveOk()
```

### Successfully running a replicaset with data replication

# Step 2: Disconnect secondary to become standalone server 

Remove the secondary as member of the replicaset

```
rs.remove("mongo_ded01:27017")
```

The secondary now becomes "OTHER" and becomes unavailable, now change the rs config there to set itself as primary:

```
cfg = rs.conf()
cfg.members[0].host = 'mongo_ded01:27017'
rs.reconfig(cfg, {force : true})
```
