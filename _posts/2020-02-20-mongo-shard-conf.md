---
title: "MongoDB 샤드 구성하기"
category: "ETC"
---

샤드를 구성하려면 크게 세가지가 필요하다.

- Router
- ConfigServer
- ShardServer

![https://docs.mongodb.com/manual/_images/sharded-cluster-production-architecture.bakedsvg.svg](https://docs.mongodb.com/manual/_images/sharded-cluster-production-architecture.bakedsvg.svg)

## Router

라우터는 `mongos` 로도 불린다. 라우터는 외부 요청의 퍼사드다. 외부에서는 몽고DB의 샤딩 환경을 신경쓰지 않고 라우터와 상호작용한다.

라우터는 스스로 데이터를 저장하지 않는다. 실제 데이터는 각 샤드에 있다. 외부 요청에 대해 라우터는 ConfigSever의 설정을 참고하여 각 샤드에 커맨드 & 쿼리를 보내고, 반환된 결과를 클라이언트에 전달한다.

## Config Servers

Config Server는 샤드 클러스터에 대한 메타데이터를 보관한다. 특히 각 샤드들의 샤드키들을 보관하고 있으므로 라우터가 어느 샤드에 쿼리를 보내야 할지 결정할 수 있게 도와준다.

라우터는 Config Servers의 메타데이터를 매번 쿼리하지 않고, 내부에 캐시해 둔다. 메타데이터가 변경되는 경우 (샤드가 추가된다거나, 데이터를 나눈다거나...) Config Server의 데이터를 업데이트한다.

몽고DB는 Config Servers를 Replica할 것을 권장한다. 샤드 키, 샤딩 상태 등 매우 중요한 데이터를 관리하기 때문이다. 각 Config Servers를 복제 환경으로 구성하여 일부분이 제 기능을 할 수 없더라도 전체 시스템에 영향을 줄인다.

## Shard

Shard들은 실제 데이터를 저장하고 관리하는 역할을 수행한다. 동작은 싱글 인스턴스 몽고DB와 유사하다. 단지 라우터와 Config Servers에 의해 같은 데이터베이스, 같은 컬렉션을 나누어 관리하고 있을 뿐이다.

몽고DB는 Shard 역시 Replica할 것을 권장한다.

## Replica Set

복제 세트로 이해할 수 있다. 똑같은 데이터를 여러 서버에 중복 저장한다. RDB의 Master Slave와 거의 유사하다. Master는 읽기쓰기를 지원하고 Slave는 Master의 데이터를 동기화하고 읽기를 지원한다.

몽고DB는 아주 쉽게 Replica Set을 구성할 수 있다. RDB에서는 Replication을 위한 계정, 설정 등의 절차를 거쳐야 하지만 몽고DB는 간단한 admin command로 Replication을 구성할 수 있다.

Replica Set에는 크게 두가지 모델이 있다.

### PSA Primary, Secondary, Arbiter

읽기 쓰기를 수행하는 Primary, Primary 데이터를 동기화하고 읽기를 수행하는 Secondary, Primary 장애시 다음 Primary를 선출하는 Arbiter

### PSS Primary, Secondary, Secondary

읽기 쓰기를 수행하는 Primary, Primary 데이터를 동기화하고 읽기를 수행하는 Secondary 다수.



## 샤드 클러스터 구성하기

샤드 환경 구성은 샤드구성 → Config Server 구성 → Router 구성 순으로 남긴다.

서버 구성은 Router 1대, Config & Shard 1대, Shard 1대 총 3대 구성이다. 공식 지원하는 구성은 아니지만 3대로 구성하는 가장 적절한 방법이라고 판단했다. (받은 서버가 3개였음..)

- Router
- ConfigServer & Shard1
- Shard2

## Shard (Server) 구성

몽고DB는 `mongod` 로 프로세스를 데몬으로 실행하여 띄울 수 있다. 인스턴스 설정을 설정 파일로 설정하는 방법, 쉘에서 설정하는 방법이 있다.

### conf 구성
```
# mongod.conf

systemLog:
    destination: file
    logAppend: true
    path: [PATH_TO_LOG_FILE]

storage:
    dbPath: [PATH_TO_DATA_DIRECTORY]

processManagement:
    fork: true

net:
    port: [PORT_NUMBER]
    bindIpAll: [BOOLEAN]

## 레플리케이션 설정
replication:
    replSetName: "[REPLICA_SET_NAME]"

## 샤드 관련 설정
sharding:
    clusterRole: shardsvr
```

`sudo mongod -f [PATH_TO_CONFIG]` 로 실행한다.

- 프로세스를 포크모드로 수행하지 않으면 현재 쉘에서 프로세스가 뜬다. 백그라운드에서 프로세스를 띄우려면 `fork` 옵션을 `true`로 주어야 한다. `fork` 모드를 사용하려면 `systemLog`가 필수로 설정되어야 한다.
- **단일 인스턴스라도 반드시** Repleication Set이 필요하다.
- `dbpath` 에 존재하지 않는 디렉토리를 전달할 경우, 동작에 실패한다. 디렉토리가 없다면 `mkdir` 하는 코드를 추가해서 컨트리뷰트 해볼 수 있을 것 같다...(빡침)

### Shell Command

```bash
sudo mongod --shardsvr \\
    --replSet [REPLICATION_NAME]
    --dbpath <PATH_TO_DATA_DIRECTORY> \\
    --bind_ip_all \\
    --port [PORT_NUMBER] \\
    --fork --logpath [PATH_TO_LOG_FILE] --logappend
```

### Any Trouble?

실행시 문제가 있을 경우, 쉘에 자세한 로그를 남겨주지 않는다. 대신 에러 넘버를 리턴한다. 에러 넘버는 공식문서에는 안나오고 몽고DB 소스에 나와 있다. [링크](https://github.com/mongodb/mongo/blob/master/src/mongo/shell/servers.js#L926)

```js
MongoRunner.EXIT_ABORT = -6;
MongoRunner.EXIT_CLEAN = 0;
MongoRunner.EXIT_BADOPTIONS = 2;
MongoRunner.EXIT_REPLICATION_ERROR = 3;
MongoRunner.EXIT_NEED_UPGRADE = 4;
MongoRunner.EXIT_SHARDING_ERROR = 5;
MongoRunner.EXIT_SIGKILL = _isWindows() ? 1 : -9;
MongoRunner.EXIT_KILL = 12;
MongoRunner.EXIT_ABRUPT = 14;
MongoRunner.EXIT_NTSERVICE_ERROR = 20;
MongoRunner.EXIT_JAVA = 21;
MongoRunner.EXIT_OOM_MALLOC = 42;
MongoRunner.EXIT_OOM_REALLOC = 43;
MongoRunner.EXIT_FS = 45;
MongoRunner.EXIT_CLOCK_SKEW = 47;  // OpTime clock skew; deprecated
MongoRunner.EXIT_NET_ERROR = 48;
MongoRunner.EXIT_WINDOWS_SERVICE_STOP = 49;
MongoRunner.EXIT_POSSIBLE_CORRUPTION = 60;
MongoRunner.EXIT_NEED_DOWNGRADE = 62;
MongoRunner.EXIT_UNCAUGHT = 100;  // top level exception that wasn't caught
MongoRunner.EXIT_TEST = 101;
```

Permission denied와 같은 문제에는 1, net 설정이 잘못된 경우 48이 나온다.

`1` 은 socket을 열면서 Permission denied된 케이스라 추측한다. `sudo` 키워드를 추가해보자.  
`48` 은 동일 포트를 사용하는 프로세스가 있는 경우다. 포트를 바꾸거나 사용중인 프로세스를 킬하자.

### Replication 활성화

`sudo mongod ~` 로 프로세스를 띄웠다면 레플리케이션을 활성화해줘야 한다.  
`mongo --port [PORT_NUMBER]` 로 방금 띄운 몽고DB에 접속한다.

```
rs.initiate()
```

프롬프트가 `REPLICATION_NAME:OTHER` 로 변경된 것을 확인한다.  
`exit` 으로 mongo 쉘을 종료하고 다시 접속한다.  
프롬프트가 `REPLICATION_NAME:PRIMARY` 로 변경된 것을 확인한다.

## Config Servers

Config Server는 메타데이터만 갖고 있으므로 가볍다. 샤드 클러스터 상태가 변경될 때 Router가 Config Server와 상호작용한다. 샤드 서버 중 하나에 Config Server를 띄운다.

```bash
sudo mongod \\
    --configsvr \\
    --replSet "[REPLICA_SET_NAME]" \\
    --dbpath [PATH_TO_DATA_DIRECTORY] \\
    --bind_ip_all
    --port [PORT_NUMBER] \\
    --fork --logpath [PATH_TO_LOG_DIRECTORY] --logappend
```

- **반드시** 샤드 서버와 다른 포트를 사용해야 한다.
- **반드시** 레플리케이션 이름이 필요하다.

### Replication 활성화

위의 샤드 서버 구성을 참고하여 Replication을 활성화한다.

## Router

라우터는 `mongod` 가 아닌 `mongos` 커맨드로 띄워야 한다.

```
# mongos.conf
    
sharding:
    configDB: "[REPLICATION_NAME]/[CONFIG_SERVER_URL]"

systemLog:
    destination: file
    logAppend: true
    path: [PATH_TO_LOG_FILE]
processManagement:
    fork: true

net:
    port: [PORT_NUMBER]
    bindIpAll: true
```

`sudo mongos -f PATH_TO_CONFIG` 로 라우터를 띄운다.

- 라우터는 데이터를 저장하지 않으므로 `dbpath` 가 필요 없다.
- 지금까지 구성에서 `bindIp` 또는 `bindIpAll` 설정이 잘못되었다면 라우터 동작이 실패한다. 로그를 확인하면 해당 인스턴스로 커넥션을 시도하는데, 샤드 서버나 Config Server들이 `127.0.0.1` 로 바인드되어 있다면 외부 커넥션을 맺을 수 없으므로 타임아웃이 발생한다.
- Config Server가 3대 미만일시 경고문을 출력한다. 프로덕션 레벨에서는 3대 이상 구성할 것을 강력히 권고하고 있다.

### 샤드 활성화

다른 몽고DB와 똑같이 `mongo --port [PORT_NUMBER]` 로 접속한다. 프롬프트가 `mongos` 인 것을 확인한다.

```js
// 샤드 하나 추가
sh.addShard("[REPLICA_SET_NAME]/[SHARD_URL]") 

// 샤드 둘 이상 추가
sh.addShard("[REPLICA_SET_NAME]/SHARD_1, [REPLICA_SET_NAME]/SHARD_2")
```

`sh.status()` 로 현재 구성을 확인할 수 있다.

```js
sh.status()
// --- Sharding Status ---
sharding_version: {
    "_id" : 1,
    "minCompatibleVersion" : 5,
    "currentVersion" : 6,
    "clusterId" : ObjectId("5e4a2e42de76a63ad0f5bedc")
}
shards:
    {  "_id" : "rs1",  "host" : "rs1/HOST_1:27017",  "state" : 1 }
    {  "_id" : "rs2",  "host" : "rs2/HOST_2:27017",  "state" : 1 }
active_mongoses:
    "4.2.3" : 1
autosplit:
    Currently_enabled: yes
balancer:
    Currently_enabled:  yes
    Currently_running:  no
    Failed balancer rounds in last 5 attempts:  0

databases:
    ...
```

### 데이터베이스 샤딩 설정

여기서 말하는 데이터베이스는 몽고DB 내부의 데이터베이스다.

```js
sh.enableSharding("DB_NAME")
```

### 샤드키 설정

컬렉션을 샤드하려면 샤드키가 필요하다. 몽고DB는 Hash 기반 샤드, Range 기반 샤드 모두 지원한다.

`employee` 데이터베이스의 `programmer` 컬렉션을 샤딩한다고 가정한다.

```js
var programmer = {
    _id: ObjectId(123..),
    name: "andole",
    team: "핀테크개발팀",
    ...
}
```

몽고DB에서는 임의의 고유한 필드를 샤드키로 설정할 수 있다.


```js
use employee
db.programmer.createIndex({_id: 1})

sh.shardCollection("DB_NAME.COLLECTION_NAME", { SHARD_FIELD: SHARD_KEY_TYPE })

// Hash 샤드키
sh.shardCollection("employee.programmer", { _id: "hashed" })

// Range 샤드키
sh.shardCollection("employee.programmer", { _id: 1 })
```

`sh.status()` 로 잘 구성되었는지 확인한다.

```js
mongos> sh.status()
--- Sharding Status ---
...
databases:
...
{  "_id" : "employee",  "primary" : "rs1",  ...}
    employee.programmer
        shard_key: { "_id" : 1 }
        unique: false
        balancing: true
        chunks:
                rs1     1
        { "_id" : { "$minKey" : 1 } } -->> { "_id" : { "$maxKey" : 1 } }
```

샤드 구성이 끝났다. 이제부터 `fintech.intern` 은 샤딩되어 저장된다.

`sh.help()` , `rs.help()` , `db.help()` 로 더 많은 명령을 볼 수 있다.