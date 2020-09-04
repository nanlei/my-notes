<h3 align="center"><b>MongoDB 副本集安装和配置(Docker版)</b></h3>

#### 1. 拉取MongoDB镜像
```cmd
# docker pull mongo
Using default tag: latest
Trying to pull repository docker.io/library/mongo ... 
latest: Pulling from docker.io/library/mongo
f08d8e2a3ba1: Pull complete 
3baa9cb2483b: Pull complete 
94e5ff4c0b15: Pull complete 
1860925334f9: Pull complete 
9d42806c06e6: Pull complete 
31a9fd218257: Pull complete 
5bd6e3f73ab9: Pull complete 
f6ae7a64936b: Pull complete 
80fde2cb25c5: Pull complete 
1bec62fe62fc: Pull complete 
2cf4970a1653: Pull complete 
39fac3226e16: Pull complete 
86bca9c64faf: Pull complete 
Digest: sha256:df9eca84736a666d5f7e7a09aeb8a6d8d073698d5b7349400f10ee75812e0e95
Status: Downloaded newer image for docker.io/mongo:latest
```

#### 2. 配置副本集

##### 2.1 配置副本集文件夹

```cmd
# mkdir -p /usr/mongodb/mongo1/data
# mkdir -p /usr/mongodb/mongo2/data
# mkdir -p /usr/mongodb/mongo3/data
```

##### 2.2 生成keyfile
```cmd
# cd /usr/mongodb
# openssl rand -base64 745 > keyfile
# chmod 600 keyfile
# chown 999 keyfile
```

##### 2.3 创建docker-compose.yml
```yml
version: '3.1'
services:
  mongo1:
    image: mongo
    hostname: mongo1
    container_name: mongo1
    restart: always
    networks:
      - mongodb
    ports:
      - 27017:27017
    volumes:
      - /usr/mongodb/keyfile:/data/keyfile
      - /usr/mongodb/mongo1/data:/data/db
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: admin
    command: mongod --auth --keyFile /data/keyfile --bind_ip_all --replSet repls
  mongo2:
    image: mongo
    hostname: mongo2
    container_name: mongo2
    restart: always
    networks:
      - mongodb
    ports:
      - 27018:27017
    volumes:
      - /usr/mongodb/keyfile:/data/keyfile
      - /usr/mongodb/mongo2/data:/data/db
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: admin
    command: mongod --auth --keyFile /data/keyfile --bind_ip_all --replSet repls
  mongo3:
    image: mongo
    hostname: mongo3
    container_name: mongo3
    restart: always
    networks:
      - mongodb
    ports:
      - 27019:27017
    volumes:
      - /usr/mongodb/keyfile:/data/keyfile
      - /usr/mongodb/mongo3/data:/data/db
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: admin
    command: mongod --auth --keyFile /data/keyfile --bind_ip_all --replSet repls
networks:
  mongodb:
    driver: bridge
```

#### 3. 运行docker-compose创建容器
```cmd
# docker-compose -f docker-compose.yml up -d
Creating network "mongodb_mongodb" with driver "bridge"
Creating mongo1 ... done
Creating mongo2 ... 
Creating mongo3 ... 
```

#### 4. 配置副本集

##### 4.1 获取主机IP地址，配置副本集
```cmd
# docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' mongo1
172.18.0.2
```

##### 4.2 设置节点属性
如果使用内网IP进行设置，则外部不可以访问副本集(只能访问master节点)，可以按照各自的内部端口进行配置：
```json
config = {
    _id:"repls",
    members:[
        {_id: 1, host:"172.18.0.2:27017"},
        {_id: 2, host:"172.18.0.3:27017"},
        {_id: 3, host:"172.18.0.4:27017"}
    ]
}
```

如果需要远程访问副本集，则需要使用公网IP进行设置，端口为映射后的公网端口：
```json
config = {
    _id:"repls",
    members:[
        {_id: 1, host:"39.105.58.106:27017"},
        {_id: 2, host:"39.105.58.106:27018"},
        {_id: 3, host:"39.105.58.106:27019"}
    ]
}
```

##### 4.3 登陆到mongo1进行设置
```cmd
# docker exec -it mongo1 /bin/bash
root@mongo1:/# mongo -u admin -p admin
MongoDB shell version v4.4.0
connecting to: mongodb://127.0.0.1:27017/?compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("bcedcab4-0454-4f77-9e69-4e818aad7ee7") }
MongoDB server version: 4.4.0
---
The server generated these startup warnings when booting: 
        2020-09-03T03:00:40.099+00:00: Using the XFS filesystem is strongly recommended with the WiredTiger storage engine. See http://dochub.mongodb.org/core/prodnotes-filesystem
        2020-09-03T03:00:47.225+00:00: /sys/kernel/mm/transparent_hugepage/enabled is 'always'. We suggest setting it to 'never'
        2020-09-03T03:00:47.225+00:00: /sys/kernel/mm/transparent_hugepage/defrag is 'always'. We suggest setting it to 'never'
---
---
        Enable MongoDB's free cloud-based monitoring service, which will then receive and display
        metrics about your deployment (disk utilization, CPU, operation statistics, etc).

        The monitoring data will be available on a MongoDB website with a unique URL accessible to you
        and anyone you share the URL with. MongoDB may use this information to make product
        improvements and to suggest MongoDB products and deployment options to you.

        To enable free monitoring, run the following command: db.enableFreeMonitoring()
        To permanently disable this reminder, run the following command: db.disableFreeMonitoring()
---
> 
```

##### 4.4 使用rs.initiate()进行副本集设置
```
> config = {
...     _id:"repls",
...     members:[
...         {_id: 1, host:"172.18.0.2:27017"},
...         {_id: 2, host:"172.18.0.3:27017"},
...         {_id: 3, host:"172.18.0.4:27017"}
...     ]
... }
{
	"_id" : "repls",
	"members" : [
		{
			"_id" : 1,
			"host" : "172.18.0.5:27017"
		},
		{
			"_id" : 2,
			"host" : "172.18.0.6:27017"
		},
		{
			"_id" : 3,
			"host" : "172.18.0.7:27017"
		}
	]
}
> rs.initiate(config)
{ "ok" : 1 }
```
返回`{ "ok" : 1 }` 说明配置成功，可以使用`rs.status()`查看副本集的状态


#### 使用MongoDB Shell进行连接

在MacOS系统，使用brew进行安装，需要先增加MongoDB的存储库
```cmd
brew tap mongodb/brew
==> Tapping mongodb/brew
Cloning into '/usr/local/Homebrew/Library/Taps/mongodb/homebrew-brew'...
remote: Enumerating objects: 33, done.
remote: Counting objects: 100% (33/33), done.
remote: Compressing objects: 100% (27/27), done.
remote: Total 377 (delta 12), reused 12 (delta 6), pack-reused 344
Receiving objects: 100% (377/377), 78.91 KiB | 20.00 KiB/s, done.
Resolving deltas: 100% (167/167), done.
Tapped 10 formulae (38 files, 145KB).
```

之后即可安装MongoDB Shell
```cmd
$ brew install mongodb-community-shell
Updating Homebrew...
==> Installing mongodb-community-shell from mongodb/brew
==> Downloading https://fastdl.mongodb.org/osx/mongodb-shell-macos-x86_64-4.2.0.tgz
######################################################################## 100.0%
🍺  /usr/local/Cellar/mongodb-community-shell/4.2.0: 8 files, 45MB, built in 4 seconds
```

使用Mongo Shell进行连接
```cmd
$ mongo mongodb://{usrname}:{password}@39.105.58.106:27017,39.105.58.106:27018,39.105.58.106:27019/?replicaSet=repls
MongoDB shell version v4.2.0
connecting to: mongodb://39.105.58.106:27017,39.105.58.106:27018,39.105.58.106:27019/?compressors=disabled&gssapiServiceName=mongodb&replicaSet=repls
2020-09-03T17:13:13.528+0800 I  NETWORK  [js] Starting new replica set monitor for repls/39.105.58.106:27017,39.105.58.106:27018,39.105.58.106:27019
2020-09-03T17:13:13.530+0800 I  CONNPOOL [ReplicaSetMonitor-TaskExecutor] Connecting to 39.105.58.106:27018
2020-09-03T17:13:13.531+0800 I  CONNPOOL [ReplicaSetMonitor-TaskExecutor] Connecting to 39.105.58.106:27017
2020-09-03T17:13:13.531+0800 I  CONNPOOL [ReplicaSetMonitor-TaskExecutor] Connecting to 39.105.58.106:27019
2020-09-03T17:13:13.722+0800 I  NETWORK  [ReplicaSetMonitor-TaskExecutor] Confirmed replica set for repls is repls/39.105.58.106:27017,39.105.58.106:27018,39.105.58.106:27019
Implicit session: session { "id" : UUID("e475dce7-ce80-4ecd-b85f-32ed13c80d52") }
MongoDB server version: 4.4.0
WARNING: shell and server versions do not match
Server has startup warnings: 
{"t":{"$date":"2020-09-03T03:00:40.099+00:00"},"s":"I",  "c":"STORAGE",  "id":22297,   "ctx":"initandlisten","msg":"Using the XFS filesystem is strongly recommended with the WiredTiger storage engine. See http://dochub.mongodb.org/core/prodnotes-filesystem","tags":["startupWarnings"]}
{"t":{"$date":"2020-09-03T03:00:47.225+00:00"},"s":"W",  "c":"CONTROL",  "id":22178,   "ctx":"initandlisten","msg":"/sys/kernel/mm/transparent_hugepage/enabled is 'always'. We suggest setting it to 'never'","tags":["startupWarnings"]}
{"t":{"$date":"2020-09-03T03:00:47.225+00:00"},"s":"W",  "c":"CONTROL",  "id":22181,   "ctx":"initandlisten","msg":"/sys/kernel/mm/transparent_hugepage/defrag is 'always'. We suggest setting it to 'never'","tags":["startupWarnings"]}
---
Enable MongoDB's free cloud-based monitoring service, which will then receive and display
metrics about your deployment (disk utilization, CPU, operation statistics, etc).

The monitoring data will be available on a MongoDB website with a unique URL accessible to you
and anyone you share the URL with. MongoDB may use this information to make product
improvements and to suggest MongoDB products and deployment options to you.

To enable free monitoring, run the following command: db.enableFreeMonitoring()
To permanently disable this reminder, run the following command: db.disableFreeMonitoring()
---

repls:PRIMARY> 
```

#### 使用MongoDB Compass连接

下载MongoDB Compass并安装
https://www.mongodb.com/try/download/compass

创建连接，进行设置：
![](img/compass-connection-host.png)

在更多选项中设置副本集：
![](img/compass-connection-more.png)

连接成功，即可看到数据库：
![](img/compass-database.png)