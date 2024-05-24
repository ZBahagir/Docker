# Single-node replica set setup
### Here is the docker-compose.yml file to spin up a single-node replica set named rs0:
```bash
version: "3.8"

services:
  mongo1:
    image: mongo:7.0
    command: ["--replSet", "rs0", "--bind_ip_all", "--port", "27017"]
    ports:
      - 27017:27017
    extra_hosts:
      - "host.docker.internal:host-gateway"
    healthcheck:
      test: echo "try { rs.status() } catch (err) { rs.initiate({_id:'rs0',members:[{_id:0,host:'host.docker.internal:27017'}]}) }" | mongosh --port 27017 --quiet
      interval: 5s
      timeout: 30s
      start_period: 0s
      start_interval: 1s
      retries: 30
    volumes:
      - "mongo1_data:/data/db"
      - "mongo1_config:/data/configdb"

volumes:
  mongo1_data:
  mongo1_config:
```
#### We're using the mongo:7.0 image. We're also using the --replSet flag to specify the name of the replica set, rs0. The --bind_ip_all flag is used to bind the MongoDB instance to all IPv4 addresses, and the --port flag is used to specify the port on which the MongoDB instance will be listening. 27017 is the default port for MongoDB. We're also mapping the container port 27017 to the host port 27017 so that we can connect to the MongoDB instance from our host machine. The extra_hosts section is used to map the host.docker.internal hostname to the host machine's IP address.
#### The healthcheck functionality has been repurposed to initialize the replica set in our setup. Replica sets need to be initialized using the rs.initiate() mongosh command. rs.status() is used here because it throws an exception if the replica set is not initialized, so it's convenient to use it to call rs.initiate() until the replica set is initialized.
### Start the containers
```bash
root@srv01:/source/mongo# docker compose up --wait
WARN[0000] /source/mongo/docker-compose.yml: `version` is obsolete 
[+] Running 9/9
 ✔ mongo1 Pulled                                                                                                 406.9s 
   ✔ a8b1c5f80c2d Pull complete                                                                                  200.0s 
   ✔ 408f9504c110 Pull complete                                                                                  200.0s 
   ✔ 03d18b647343 Pull complete                                                                                  200.3s 
   ✔ c24f68d81052 Pull complete                                                                                  200.3s 
   ✔ 1df517147e11 Pull complete                                                                                  200.3s 
   ✔ 77d5ebe2f2e0 Pull complete                                                                                  200.4s 
   ✔ c21b89d414fc Pull complete                                                                                  401.3s 
   ✔ 4138c7eb3b71 Pull complete                                                                                  401.3s 
[+] Running 4/4
 ✔ Network mongo_default         Created                                                                         0.1s 
 ✔ Volume "mongo_mongo1_data"    Created                                                                         0.0s 
 ✔ Volume "mongo_mongo1_config"  Created                                                                         0.0s 
 ✔ Container mongo-mongo1-1      Healthy                                                                         7.8s
```
### Bash into one container
```bash
root@srv01:/source/mongo# docker ps
CONTAINER ID   IMAGE       COMMAND                  CREATED          STATUS                    PORTS                                           NAMES
948ab902e36b   mongo:7.0   "docker-entrypoint.s…"   51 seconds ago   Up 50 seconds (healthy)   0.0.0.0:27017->27017/tcp, :::27017->27017/tcp   mongo-mongo1-1
root@srv01:/source/mongo# docker exec -it mongo-mongo1-1 bash
root@948ab902e36b:/# mongosh                   
Current Mongosh Log ID: 664f781f77c6fd0afe2202d7
Connecting to:          mongodb://127.0.0.1:27017/?directConnection=true&serverSelectionTimeoutMS=2000&appName=mongosh+2.2.5
Using MongoDB:          7.0.9
Using Mongosh:          2.2.5

For mongosh info see: https://docs.mongodb.com/mongodb-shell/


To help improve our products, anonymous usage data is collected and sent to MongoDB periodically (https://www.mongodb.com/legal/privacy-policy).
You can opt-out by running the disableTelemetry() command.

------
   The server generated these startup warnings when booting
   2024-05-23T17:05:27.700+00:00: Using the XFS filesystem is strongly recommended with the WiredTiger storage engine. See http://dochub.mongodb.org/core/prodnotes-filesystem
   2024-05-23T17:05:28.501+00:00: Access control is not enabled for the database. Read and write access to data and configuration is unrestricted
   2024-05-23T17:05:28.502+00:00: vm.max_map_count is too low
------

rs0 [direct: primary] test> rs.status()
{
  set: 'rs0',
  date: ISODate('2024-05-23T17:09:02.132Z'),
  myState: 1,
  term: Long('1'),
  syncSourceHost: '',
  syncSourceId: -1,
  heartbeatIntervalMillis: Long('2000'),
  majorityVoteCount: 1,
  writeMajorityCount: 1,
  votingMembersCount: 1,
  writableVotingMembersCount: 1,
  optimes: {
    lastCommittedOpTime: { ts: Timestamp({ t: 1716484133, i: 1 }), t: Long('1') },
    lastCommittedWallTime: ISODate('2024-05-23T17:08:53.512Z'),
    readConcernMajorityOpTime: { ts: Timestamp({ t: 1716484133, i: 1 }), t: Long('1') },
    appliedOpTime: { ts: Timestamp({ t: 1716484133, i: 1 }), t: Long('1') },
    durableOpTime: { ts: Timestamp({ t: 1716484133, i: 1 }), t: Long('1') },
    lastAppliedWallTime: ISODate('2024-05-23T17:08:53.512Z'),
    lastDurableWallTime: ISODate('2024-05-23T17:08:53.512Z')
  },
  lastStableRecoveryTimestamp: Timestamp({ t: 1716484103, i: 1 }),
  electionCandidateMetrics: {
    lastElectionReason: 'electionTimeout',
    lastElectionDate: ISODate('2024-05-23T17:05:33.435Z'),
    electionTerm: Long('1'),
    lastCommittedOpTimeAtElection: { ts: Timestamp({ t: 1716483933, i: 1 }), t: Long('-1') },
    lastSeenOpTimeAtElection: { ts: Timestamp({ t: 1716483933, i: 1 }), t: Long('-1') },
    numVotesNeeded: 1,
    priorityAtElection: 1,
    electionTimeoutMillis: Long('10000'),
    newTermStartDate: ISODate('2024-05-23T17:05:33.465Z'),
    wMajorityWriteAvailabilityDate: ISODate('2024-05-23T17:05:33.497Z')
  },
  members: [
    {
      _id: 0,
      name: 'host.docker.internal:27017',
      health: 1,
      state: 1,
      stateStr: 'PRIMARY',
      uptime: 215,
      optime: { ts: Timestamp({ t: 1716484133, i: 1 }), t: Long('1') },
      optimeDate: ISODate('2024-05-23T17:08:53.000Z'),
      lastAppliedWallTime: ISODate('2024-05-23T17:08:53.512Z'),
      lastDurableWallTime: ISODate('2024-05-23T17:08:53.512Z'),
      syncSourceHost: '',
      syncSourceId: -1,
      infoMessage: '',
      electionTime: Timestamp({ t: 1716483933, i: 2 }),
      electionDate: ISODate('2024-05-23T17:05:33.000Z'),
      configVersion: 1,
      configTerm: 1,
      self: true,
      lastHeartbeatMessage: ''
    }
  ],
  ok: 1,
  '$clusterTime': {
    clusterTime: Timestamp({ t: 1716484133, i: 1 }),
    signature: {
      hash: Binary.createFromBase64('AAAAAAAAAAAAAAAAAAAAAAAAAAA=', 0),
      keyId: Long('0')
    }
  },
  operationTime: Timestamp({ t: 1716484133, i: 1 })
}
```

# Three-node replica set setup
```bash
version: "3.8"

services:
  mongo1:
    image: mongo:7.0
    command: ["--replSet", "rs0", "--bind_ip_all", "--port", "27017"]
    ports:
      - 27017:27017
    extra_hosts:
      - "host.docker.internal:host-gateway"
    healthcheck:
      test: echo "try { rs.status() } catch (err) { rs.initiate({_id:'rs0',members:[{_id:0,host:'host.docker.internal:27017',priority:1},{_id:1,host:'host.docker.internal:27018',priority:0.5},{_id:2,host:'host.docker.internal:27019',priority:0.5}]}) }" | mongosh --port 27017 --quiet
      interval: 5s
      timeout: 30s
      start_period: 0s
      start_interval: 1s
      retries: 30
    volumes:
      - "mongo1_data:/data/db"
      - "mongo1_config:/data/configdb"

  mongo2:
    image: mongo:7.0
    command: ["--replSet", "rs0", "--bind_ip_all", "--port", "27018"]
    ports:
      - 27018:27018
    extra_hosts:
      - "host.docker.internal:host-gateway"
    volumes:
      - "mongo2_data:/data/db"
      - "mongo2_config:/data/configdb"

  mongo3:
    image: mongo:7.0
    command: ["--replSet", "rs0", "--bind_ip_all", "--port", "27019"]
    ports:
      - 27019:27019
    extra_hosts:
      - "host.docker.internal:host-gateway"
    volumes:
      - "mongo3_data:/data/db"
      - "mongo3_config:/data/configdb"

volumes:
  mongo1_data:
  mongo2_data:
  mongo3_data:
  mongo1_config:
  mongo2_config:
  mongo3_config:
```
### Start the containers
```bash
root@srv01:/source/mongodb-multinode# docker compose up --wait
WARN[0000] /source/mongodb-multinode/docker-compose.yml: `version` is obsolete 
[+] Running 10/10
 ✔ Network mongodb-multinode_default         Created                                                    0.1s 
 ✔ Volume "mongodb-multinode_mongo2_config"  Created                                                    0.0s 
 ✔ Volume "mongodb-multinode_mongo3_data"    Created                                                    0.0s 
 ✔ Volume "mongodb-multinode_mongo3_config"  Created                                                    0.0s 
 ✔ Volume "mongodb-multinode_mongo1_data"    Created                                                    0.0s 
 ✔ Volume "mongodb-multinode_mongo1_config"  Created                                                    0.0s 
 ✔ Volume "mongodb-multinode_mongo2_data"    Created                                                    0.0s 
 ✔ Container mongodb-multinode-mongo3-1      Healthy                                                    1.8s 
 ✔ Container mongodb-multinode-mongo1-1      Healthy                                                    7.3s 
 ✔ Container mongodb-multinode-mongo2-1      Healthy                                                    1.8s
```
### Check containers
```bash
root@srv01:/source/mongodb-multinode# docker ps
CONTAINER ID   IMAGE       COMMAND                  CREATED         STATUS                   PORTS                                                      NAMES
5882eae498c2   mongo:7.0   "docker-entrypoint.s…"   3 minutes ago   Up 3 minutes (healthy)   0.0.0.0:27017->27017/tcp, :::27017->27017/tcp              mongodb-multinode-mongo1-1
cf4db4516f35   mongo:7.0   "docker-entrypoint.s…"   3 minutes ago   Up 3 minutes             27017/tcp, 0.0.0.0:27018->27018/tcp, :::27018->27018/tcp   mongodb-multinode-mongo2-1
c9108c89bfc8   mongo:7.0   "docker-entrypoint.s…"   3 minutes ago   Up 3 minutes             27017/tcp, 0.0.0.0:27019->27019/tcp, :::27019->27019/tcp   mongodb-multinode-mongo3-1
```
### Connect to replica set
```bash
root@5882eae498c2:/# mongosh --host rs0/mongodb-multinode-mongo1-1:27017,mongodb-multinode-mongo2-1:27017,mongodb-multinode-mongo3-1:27017
Current Mongosh Log ID: 664f7f51859f93d14d2202d7
Connecting to:          mongodb://mongodb-multinode-mongo1-1:27017,mongodb-multinode-mongo2-1:27017,mongodb-multinode-mongo3-1:27017/?replicaSet=rs0&appName=mongosh+2.2.5
Using MongoDB:          7.0.9
Using Mongosh:          2.2.5

For mongosh info see: https://docs.mongodb.com/mongodb-shell/

------
   The server generated these startup warnings when booting
   2024-05-23T17:30:05.867+00:00: Using the XFS filesystem is strongly recommended with the WiredTiger storage engine. See http://dochub.mongodb.org/core/prodnotes-filesystem
   2024-05-23T17:30:07.107+00:00: Access control is not enabled for the database. Read and write access to data and configuration is unrestricted
   2024-05-23T17:30:07.108+00:00: vm.max_map_count is too low
------

rs0 [primary] test> rs.status()
{
  set: 'rs0',
  date: ISODate('2024-05-23T17:39:45.117Z'),
  myState: 1,
  term: Long('1'),
  syncSourceHost: '',
  syncSourceId: -1,
  heartbeatIntervalMillis: Long('2000'),
  majorityVoteCount: 2,
  writeMajorityCount: 2,
  votingMembersCount: 3,
  writableVotingMembersCount: 3,
  optimes: {
    lastCommittedOpTime: { ts: Timestamp({ t: 1716485981, i: 1 }), t: Long('1') },
    lastCommittedWallTime: ISODate('2024-05-23T17:39:41.866Z'),
    readConcernMajorityOpTime: { ts: Timestamp({ t: 1716485981, i: 1 }), t: Long('1') },
    appliedOpTime: { ts: Timestamp({ t: 1716485981, i: 1 }), t: Long('1') },
    durableOpTime: { ts: Timestamp({ t: 1716485981, i: 1 }), t: Long('1') },
    lastAppliedWallTime: ISODate('2024-05-23T17:39:41.866Z'),
    lastDurableWallTime: ISODate('2024-05-23T17:39:41.866Z')
  },
  lastStableRecoveryTimestamp: Timestamp({ t: 1716485941, i: 1 }),
  electionCandidateMetrics: {
    lastElectionReason: 'electionTimeout',
    lastElectionDate: ISODate('2024-05-23T17:30:21.734Z'),
    electionTerm: Long('1'),
    lastCommittedOpTimeAtElection: { ts: Timestamp({ t: 1716485411, i: 1 }), t: Long('-1') },
    lastSeenOpTimeAtElection: { ts: Timestamp({ t: 1716485411, i: 1 }), t: Long('-1') },
    numVotesNeeded: 2,
    priorityAtElection: 1,
    electionTimeoutMillis: Long('10000'),
    numCatchUpOps: Long('0'),
    newTermStartDate: ISODate('2024-05-23T17:30:21.782Z'),
    wMajorityWriteAvailabilityDate: ISODate('2024-05-23T17:30:22.298Z')
  },
  members: [
    {
      _id: 0,
      name: 'host.docker.internal:27017',
      health: 1,
      state: 1,
      stateStr: 'PRIMARY',
      uptime: 580,
      optime: { ts: Timestamp({ t: 1716485981, i: 1 }), t: Long('1') },
      optimeDate: ISODate('2024-05-23T17:39:41.000Z'),
      lastAppliedWallTime: ISODate('2024-05-23T17:39:41.866Z'),
      lastDurableWallTime: ISODate('2024-05-23T17:39:41.866Z'),
      syncSourceHost: '',
      syncSourceId: -1,
      infoMessage: '',
      electionTime: Timestamp({ t: 1716485421, i: 1 }),
      electionDate: ISODate('2024-05-23T17:30:21.000Z'),
      configVersion: 1,
      configTerm: 1,
      self: true,
      lastHeartbeatMessage: ''
    },
    {
      _id: 1,
      name: 'host.docker.internal:27018',
      health: 1,
      state: 2,
      stateStr: 'SECONDARY',
      uptime: 573,
      optime: { ts: Timestamp({ t: 1716485981, i: 1 }), t: Long('1') },
      optimeDurable: { ts: Timestamp({ t: 1716485981, i: 1 }), t: Long('1') },
      optimeDate: ISODate('2024-05-23T17:39:41.000Z'),
      optimeDurableDate: ISODate('2024-05-23T17:39:41.000Z'),
      lastAppliedWallTime: ISODate('2024-05-23T17:39:41.866Z'),
      lastDurableWallTime: ISODate('2024-05-23T17:39:41.866Z'),
      lastHeartbeat: ISODate('2024-05-23T17:39:44.337Z'),
      lastHeartbeatRecv: ISODate('2024-05-23T17:39:43.265Z'),
      pingMs: Long('0'),
      lastHeartbeatMessage: '',
      syncSourceHost: 'host.docker.internal:27017',
      syncSourceId: 0,
      infoMessage: '',
      configVersion: 1,
      configTerm: 1
    },
    {
      _id: 2,
      name: 'host.docker.internal:27019',
      health: 1,
      state: 2,
      stateStr: 'SECONDARY',
      uptime: 573,
      optime: { ts: Timestamp({ t: 1716485981, i: 1 }), t: Long('1') },
      optimeDurable: { ts: Timestamp({ t: 1716485981, i: 1 }), t: Long('1') },
      optimeDate: ISODate('2024-05-23T17:39:41.000Z'),
      optimeDurableDate: ISODate('2024-05-23T17:39:41.000Z'),
      lastAppliedWallTime: ISODate('2024-05-23T17:39:41.866Z'),
      lastDurableWallTime: ISODate('2024-05-23T17:39:41.866Z'),
      lastHeartbeat: ISODate('2024-05-23T17:39:44.331Z'),
      lastHeartbeatRecv: ISODate('2024-05-23T17:39:43.220Z'),
      pingMs: Long('0'),
      lastHeartbeatMessage: '',
      syncSourceHost: 'host.docker.internal:27017',
      syncSourceId: 0,
      infoMessage: '',
      configVersion: 1,
      configTerm: 1
    }
  ],
  ok: 1,
  '$clusterTime': {
    clusterTime: Timestamp({ t: 1716485981, i: 1 }),
    signature: {
      hash: Binary.createFromBase64('AAAAAAAAAAAAAAAAAAAAAAAAAAA=', 0),
      keyId: Long('0')
    }
  },
  operationTime: Timestamp({ t: 1716485981, i: 1 })
}
```