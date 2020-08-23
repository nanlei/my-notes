<h3 align="center"><b>MySQL 主从安装和配置(Docker版)</b></h3>

#### 1. 拉取MySQL镜像
```cmd
# docker pull mysql
Using default tag: latest
Trying to pull repository docker.io/library/mysql ... 
latest: Pulling from docker.io/library/mysql
bf5952930446: Pull complete 
8254623a9871: Pull complete 
938e3e06dac4: Pull complete 
ea28ebf28884: Pull complete 
f3cef38785c2: Pull complete 
894f9792565a: Pull complete 
1d8a57523420: Pull complete 
6c676912929f: Pull complete 
ff39fdb566b4: Pull complete 
fff872988aba: Pull complete 
4d34e365ae68: Pull complete 
7886ee20621e: Pull complete 
Digest: sha256:c358e72e100ab493a0304bda35e6f239db2ec8c9bb836d8a427ac34307d074ed
Status: Downloaded newer image for docker.io/mysql:latest
```

#### 2. 配置主从镜像(1主2从)

2.1 配置主从文件夹
```cmd
# mkdir -p /usr/mysql/master
# mkdir -p /usr/mysql/slave1
# mkdir -p /usr/mysql/slave2
```

2.2 在3个目录下创建Dockerfile
```docker
FROM mysql
COPY my.cnf /etc/mysql/
#RUN sed -i "s@http://deb.debian.org@http://mirrors.aliyun.com@g" /etc/apt/sources.list && rm -Rf /var/lib/apt/lists/* && apt-get update
EXPOSE 3306
CMD ["mysqld"]
```

2.3 在3个目录下分别创建my.cnf文件  

master (/usr/mysql/master/my.cnf)
```ini
[mysqld]
log-bin=mysql-bin-master
server-id=11
secure_file_priv=/var/lib/mysql
binlog-format=mixed
binlog-do-db=demo
binlog-ignore-db=mysql
binlog-ignore-db=information_schema
binlog-ignore-db=performance_schema
binlog-ignore-db=sys
```

slave1 (/usr/mysql/slave1/my.cnf)
```ini
[mysqld]
log-bin=mysql-bin-slave1
server-id=21
secure_file_priv=/var/lib/mysql
binlog-format=mixed
replicate-do-db=demo
replicate-ignore-db=mysql
replicate-ignore-db=information_schema
replicate-ignore-db=performance_schema
replicate-ignore-db=sys
```

slave2 (/usr/mysql/slave2/my.cnf)
```ini
[mysqld]
log-bin=mysql-bin-slave2
server-id=22
secure_file_priv=/var/lib/mysql
binlog-format=mixed
replicate-do-db=demo
replicate-ignore-db=mysql
replicate-ignore-db=information_schema
replicate-ignore-db=performance_schema
replicate-ignore-db=sys
```

2.4 在3个目录下创建数据子目录data和配置子目录conf.d
```cmd
# mkdir data
# mkdir conf.d
```

2.5 在3个目录下分别构建Docker镜像　

master
```cmd
# cd /usr/mysql/master
# docker build -t master/mysql .
```

slave1
```cmd
# cd /usr/mysql/slave1
# docker build -t slave1/mysql .
```

slave2
```cmd
# cd /usr/mysql/slave2
# docker build -t slave2/mysql .
```

查看创建好的镜像
```cmd
# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
slave2/mysql        latest              531514c39b89        3 seconds ago       544 MB
slave1/mysql        latest              3cc0f5fb6aab        19 seconds ago      544 MB
master/mysql        latest              a756ddedfa9d        57 seconds ago      544 MB
```

#### 创建容器

master
```cmd
docker run --name mysql-master -p 33306:3306 -v /usr/mysql/master/data:/var/lib/mysql -v /usr/mysql/master/conf.d:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=root -d master/mysql
```

slave1
```cmd
docker run --name mysql-slave1 -p 33307:3306 -v /usr/mysql/slave1/data:/var/lib/mysql -v /usr/mysql/slave1/conf.d:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=root -d slave1/mysql
```

slave2
```cmd
docker run --name mysql-slave2 -p 33308:3306 -v /usr/mysql/slave2/data:/var/lib/mysql -v /usr/mysql/slave2/conf.d:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=root -d slave2/mysql
```

#### 主从库设置

连接到master，进行设置
```cmd
# docker exec -it mysql-master /bin/bash
# mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 8.0.21 MySQL Community Server - GPL

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```

```sql
ALTER USER 'root'@'%' IDENTIFIED BY 'root';
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%';
CREATE USER 'slave' IDENTIFIED WITH mysql_native_password BY 'slave';
--GRANT ALL PRIVILEGES ON *.* TO 'slave'@'%';
GRANT REPLICATION SLAVE ON *.* TO 'slave'@'%';
flush privileges;
show master status;
+-------------------------+----------+--------------+-------------------------------------------------+-------------------+
| File                    | Position | Binlog_Do_DB | Binlog_Ignore_DB                                | Executed_Gtid_Set |
+-------------------------+----------+--------------+-------------------------------------------------+-------------------+
| mysql-bin-master.000003 |     1356 | demo         | mysql,information_schema,performance_schema,sys |                   |
+-------------------------+----------+--------------+-------------------------------------------------+-------------------+
1 row in set (0.00 sec)
```

获取主库主机IP地址 
```cmd
# docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' mysql-master
172.18.0.2
```

配置两个从库，root用户默认所有权限开放
```sql
stop slave;
ALTER USER 'root'@'%' IDENTIFIED BY 'root';
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%';
CREATE USER 'mysql'@'%' IDENTIFIED WITH mysql_native_password BY 'mysql';
GRANT SELECT ON *.* TO 'mysql'@'%';
CHANGE MASTER TO MASTER_HOST='172.18.0.2', MASTER_PORT=3306, MASTER_USER='slave', MASTER_PASSWORD='slave', MASTER_LOG_FILE='mysql-bin-master.000003', MASTER_LOG_POS=1356;
start slave;
show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 172.18.0.2
                  Master_User: slave
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin-master.000003
          Read_Master_Log_Pos: 1356
               Relay_Log_File: 8adaf85d8b70-relay-bin.000002
                Relay_Log_Pos: 331
        Relay_Master_Log_File: mysql-bin-master.000003
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: demo
          Replicate_Ignore_DB: mysql,information_schema,performance_schema,sys
```

```Slave_IO_Running```和```Slave_SQL_Running```均为Yes时表示配置成功

