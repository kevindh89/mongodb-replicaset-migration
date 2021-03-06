__IMPORTANT:__

* setups should have aligned authorization settings, we should add a keyFile to the security and share these between server to maintain security

# Step 1: Setup replicaset with one secondary

```
docker-compose up -d
```

Temporarily disable mongo authorization in the `web01_mongo.conf` file and 

```
security:
    #authorization: enabled
    #keyFile: /keyfile
    authorization: disabled
```

Create root user:

```
docker-compose restart
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

Re-enable authorization (also `keyFile`) in `web01_mongo.conf` and restart docker containers

```
docker-compose restart
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

NOTE: Make sure all database files of secondary are removed to prepare it to become secondary

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
