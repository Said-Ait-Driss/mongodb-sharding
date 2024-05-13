# Set up Sharding using Docker Containers

setting up a mongo sharding by using docker compose to simulate a real world clustering envirment

## Config servers

config servers will start with 3 member replica set (40001,40002,40003)

## Initiate replica set for config servers

### log to one of the instance example :

```bash
    mongo mongodb://192.168.1.81:40001
```

### setting up replica set

```bash
rs.initiate(
  {
    _id: "config_servers",
    configsvr: true,
    members: [
      { _id : 0, host : "192.168.1.81:40001" },
      { _id : 1, host : "192.168.1.81:40002" },
      { _id : 2, host : "192.168.1.81:40003" }
    ]
  }
)
```

### check replica set status to confirm configuration

```bash
rs.status()
```

## Shard 1 servers

Shard servers will start with 3 member replica set (50001,50002,50003)

## Initiate replica set for Shard 1 servers

### log to one of the instance example :

```bash
    mongo mongodb://192.168.1.81:50001
```

### setting up replica set

```bash
rs.initiate(
  {
    _id: "shard_1_servers",
    members: [
      { _id : 0, host : "192.168.1.81:50001" },
      { _id : 1, host : "192.168.1.81:50002" },
      { _id : 2, host : "192.168.1.81:50003" }
    ]
  }
)
```

### check replica set status to confirm configuration

```bash
rs.status()
```

## Mongos Router server

    mongos query router will start on port 60000 with db configuration servers : cfgrs/192.168.1.81:40001,192.168.1.81:40002,192.168.1.81:40003

## Adding shard to the cluster

### 1 - Connect to mongos router server

```bash
mongo mongodb://192.168.1.81:60000
```

### 2 - Add shard

```bash

sh.addShard("shard1rs/192.168.1.81:50001,192.168.1.81:50002,192.168.1.81:50003")

```

### confirm configuration

```bash
sh.status()
```

## To add another shard

    example adding shard_2_server to the cluster

### connect to one of the shard_2_server

```bash
mongo mongodb://192.168.1.81:50004
```

### setting up replica set

```bash
rs.initiate(
  {
    _id: "shard_2_servers",
    members: [
      { _id : 0, host : "192.168.1.81:50004" },
      { _id : 1, host : "192.168.1.81:50005" },
      { _id : 2, host : "192.168.1.81:50006" }
    ]
  }
)
```

### check replica set status to confirm configuration

```bash
rs.status()
```

### Then adding shard to the cluster

#### 1 - Connect to mongos router server

```bash
mongo mongodb://192.168.1.81:60000
```

#### 2 - Add shard

```bash

sh.addShard("shard1rs/192.168.1.81:50004,192.168.1.81:50005,192.168.1.81:50006")

```

#### confirm configuration

```bash
sh.status()
```

## Sharding a collection

1 - Connect to mongos router server

```bash
mongo mongodb://192.168.1.81:60000
```

2 - creating or swithing to mongo db database

```bash
    use dbtest()
```

2 - creating mongo db collection

```bash
    db.createCollection('testcollection')
```

3 - gather facts about collection ( to see if sharding is enabled on it or not )

```bash
    use db.testcollection.getShardDistribution()
```

4 - enable sharding for database first

```bash
    sh.enableSharding('dbtest')
```

5 - sharding collection

```bash
    sh.shardCollection("dbtest.testcollection", {"title":"hashed"})
```

6 - gather facts about collection ( to see if sharding is enabled on it or not )

```bash
    use db.testcollection.getShardDistribution()
```

## Sharding a collection that contains data

1 - Creating index fisrt

```bash
    use db.testcollection.createIndex( { "title" : "hashed" } )
```

2 - gather facts about collection ( to see if sharding is enabled on it or not )

```bash
    use db.testcollection.getShardDistribution()
```
