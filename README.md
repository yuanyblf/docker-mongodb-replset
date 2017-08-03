#Mongodb Relica Set副本集方式 Docker部署

Mongodb的Replica Set即副本集方式主要有两个目的，一个是数据冗余做故障恢复使用，当发生硬件故障或者其它原因造成的宕机时，可以使用副本进行恢复。

另一个是做读写分离，读的请求分流到副本上，减轻主（Primary）的读压力。

**1.Primary和Secondary搭建的Replica Set**

![](http://images2015.cnblogs.com/blog/524341/201609/524341-20160929155630438-1271486804.png)

Replica Set是mongod的实例集合，它们有着同样的数据内容。包含三类角色：

（1）主节点（Primary）

接收所有的写请求，然后把修改同步到所有Secondary。一个Replica Set只能有一个Primary节点，当Primary挂掉后，其他Secondary或者Arbiter节点会重新选举出来一个主节点。默认读请求也是发到Primary节点处理的，需要转发到Secondary需要客户端修改一下连接配置。

（2）副本节点（Secondary）

与主节点保持同样的数据集。当主节点挂掉的时候，参与选主。

（3）仲裁者（Arbiter）

不保有数据，不参与选主，只进行选主投票。使用Arbiter可以减轻数据存储的硬件需求，Arbiter跑起来几乎没什么大的硬件资源需求，但重要的一点是，在生产环境下它和其他数据节点不要部署在同一台机器上。

注意，一个自动failover的Replica Set节点数必须为奇数，目的是选主投票的时候要有一个大多数才能进行选主决策。

（4）选主过程

其中Secondary宕机，不受影响，若Primary宕机，会进行重新选主：

![](http://images2015.cnblogs.com/blog/524341/201609/524341-20160929160048547-1867921030.png)

**2.使用Arbiter搭建Replica Set**

 偶数个数据节点，加一个Arbiter构成的Replica Set方式：
 ![](http://images2015.cnblogs.com/blog/524341/201609/524341-20160929160144938-2112435000.png)

#Docker 配置步骤

##步骤1

部署3个mongodb，需要配置 ‘--replSet rs1’ 参数

说明：参数值rs1，可以任意设定；要求3个mongodb都要设置一样的值；


##步骤2

1、登录其中一台mongo

```
mongo 101.37.163.6:27017

```

2、输入

```
var config = {
     "_id" : "rs1",
     "members" : [
         { "_id": 0, "host": "101.37.163.6:27017"},
         { "_id": 1, "host": "101.37.163.6:27018"},
         { "_id": 2, "host": "101.37.163.6:27019"}
     ] 
 }
```

####说明：
- _id:rs1 值是在mongo启动时配置的参数值
- members：分别是我们启动的3个mongo服务地址

3、执行命令

```
rs.initiate(config);
```

4、输出结果（成功）

```
{ "ok" : 1 }
```

5、查看配置状态

```
rs.status();
```
输出

```
{
	"set" : "rs1",
	"date" : ISODate("2017-08-03T15:48:52.257Z"),
	"myState" : 1,
	"term" : NumberLong(1),
	"heartbeatIntervalMillis" : NumberLong(2000),
	"optimes" : {
		"lastCommittedOpTime" : {
			"ts" : Timestamp(1501775329, 1),
			"t" : NumberLong(1)
		},
		"appliedOpTime" : {
			"ts" : Timestamp(1501775329, 1),
			"t" : NumberLong(1)
		},
		"durableOpTime" : {
			"ts" : Timestamp(1501775329, 1),
			"t" : NumberLong(1)
		}
	},
	"members" : [
		{
			"_id" : 0,
			"name" : "101.37.163.6:27017",
			"health" : 1,
			"state" : 1,
			"stateStr" : "PRIMARY",
			"uptime" : 325,
			"optime" : {
				"ts" : Timestamp(1501775329, 1),
				"t" : NumberLong(1)
			},
			"optimeDate" : ISODate("2017-08-03T15:48:49Z"),
			"infoMessage" : "could not find member to sync from",
			"electionTime" : Timestamp(1501775248, 1),
			"electionDate" : ISODate("2017-08-03T15:47:28Z"),
			"configVersion" : 1,
			"self" : true
		},
		{
			"_id" : 1,
			"name" : "101.37.163.6:27018",
			"health" : 1,
			"state" : 2,
			"stateStr" : "SECONDARY",
			"uptime" : 94,
			"optime" : {
				"ts" : Timestamp(1501775329, 1),
				"t" : NumberLong(1)
			},
			"optimeDurable" : {
				"ts" : Timestamp(1501775329, 1),
				"t" : NumberLong(1)
			},
			"optimeDate" : ISODate("2017-08-03T15:48:49Z"),
			"optimeDurableDate" : ISODate("2017-08-03T15:48:49Z"),
			"lastHeartbeat" : ISODate("2017-08-03T15:48:50.284Z"),
			"lastHeartbeatRecv" : ISODate("2017-08-03T15:48:50.642Z"),
			"pingMs" : NumberLong(0),
			"syncingTo" : "101.37.163.6:27017",
			"configVersion" : 1
		},
		{
			"_id" : 2,
			"name" : "101.37.163.6:27019",
			"health" : 1,
			"state" : 2,
			"stateStr" : "SECONDARY",
			"uptime" : 94,
			"optime" : {
				"ts" : Timestamp(1501775329, 1),
				"t" : NumberLong(1)
			},
			"optimeDurable" : {
				"ts" : Timestamp(1501775329, 1),
				"t" : NumberLong(1)
			},
			"optimeDate" : ISODate("2017-08-03T15:48:49Z"),
			"optimeDurableDate" : ISODate("2017-08-03T15:48:49Z"),
			"lastHeartbeat" : ISODate("2017-08-03T15:48:50.284Z"),
			"lastHeartbeatRecv" : ISODate("2017-08-03T15:48:50.643Z"),
			"pingMs" : NumberLong(0),
			"syncingTo" : "101.37.163.6:27017",
			"configVersion" : 1
		}
	],
	"ok" : 1
}

```

##docker 编排模板yml

###mongodb-set-1.yml

```
mongo-relica-set-1:
  image: 'mongo:latest'
  mem_limit: 0
  environment:
    - GPG_KEYS=0C49F3730359A14518585931BC711F9BA15703C6
    - MONGO_PACKAGE=mongodb-org
    - MONGO_REPO=repo.mongodb.org
    - MONGO_MAJOR=3.4
  kernel_memory: 0
  memswap_reservation: 0
  restart: always
  shm_size: 0
  volumes:
    - '/mongo/config:/data/configdb-set-1'
    - '/mongo/data:/data/db-set-1'
  ports:
    - '27017:27017/tcp'
  memswap_limit: 0
  command:
    - '--replSet'
    - rs1
  labels:
    aliyun.scale: '1'
    
```

###mongodb-set-2.yml

```
mongo-relica-set-2:
  image: 'mongo:latest'
  mem_limit: 0
  environment:
    - GPG_KEYS=0C49F3730359A14518585931BC711F9BA15703C6
    - MONGO_PACKAGE=mongodb-org
    - MONGO_REPO=repo.mongodb.org
    - MONGO_MAJOR=3.4
  kernel_memory: 0
  memswap_reservation: 0
  restart: always
  shm_size: 0
  volumes:
    - '/mongo/config:/data/configdb-set-2'
    - '/mongo/data:/data/db-set-2'
  ports:
    - '27018:27017/tcp'
  memswap_limit: 0
  command:
    - '--replSet'
    - rs1
  labels:
    aliyun.scale: '1'
    
```

###mongodb-set-3.yml

```
mongo-relica-set-3:
  image: 'mongo:latest'
  mem_limit: 0
  environment:
    - GPG_KEYS=0C49F3730359A14518585931BC711F9BA15703C6
    - MONGO_PACKAGE=mongodb-org
    - MONGO_REPO=repo.mongodb.org
    - MONGO_MAJOR=3.4
  kernel_memory: 0
  memswap_reservation: 0
  restart: always
  shm_size: 0
  volumes:
    - '/mongo/config:/data/configdb-set-3'
    - '/mongo/data:/data/db-set-3'
  ports:
    - '27019:27017/tcp'
  memswap_limit: 0
  command:
    - '--replSet'
    - rs1
  labels:
    aliyun.scale: '1'
    
```

##Robomongo client 连接

