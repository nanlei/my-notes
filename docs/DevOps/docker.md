<h3 align="center"><b>Docker</b></h3>

#### 1. 安装和运行
(基于CentOS)
```cmd
# yum install docker -y
Failed to set locale, defaulting to C
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
Resolving Dependencies
--> Running transaction check
...
subscription-manager-rhsm-certificates.x86_64 0:1.24.26-4.el7.centos                           usermode.x86_64 0:1.111-6.el7                                                     
  yajl.x86_64 0:2.0.4-4.el7                                                                     

Complete!
```

检查版本
```cmd
# docker -v
Docker version 1.13.1, build 64e9980/1.13.1
```

启动Docker
```cmd
# systemctl start docker
```

停止Docker
```cmd
# systemctl stop docker
```

设置开机启动
```cmd
# systemctl enable docker
Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.
```

重启Docker
```cmd
# systemctl restart docker
```

查看Docker
```cmd
# docker info
Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
Images: 0
Server Version: 1.13.1
Storage Driver: overlay2
 Backing Filesystem: extfs
 Supports d_type: true
 Native Overlay Diff: true
Logging Driver: journald
Cgroup Driver: systemd
Plugins: 
 Volume: local
 Network: bridge host macvlan null overlay
Swarm: inactive
Runtimes: docker-runc runc
Default Runtime: docker-runc
Init Binary: /usr/libexec/docker/docker-init-current
containerd version:  (expected: aa8187dbd3b7ad67d8e5e3a15115d3eef43a7ed1)
runc version: 66aedde759f33c190954815fb765eedc1d782dd9 (expected: 9df8b306d01f59d3a8029be411de015b7304dd8f)
init version: fec3683b971d9c3ef73f284f176672c44b448662 (expected: 949e6facb77383876aeff8a6944dde66b3089574)
Security Options:
 seccomp
  WARNING: You're not using the default seccomp profile
  Profile: /etc/docker/seccomp.json
Kernel Version: 3.10.0-1127.18.2.el7.x86_64
Operating System: CentOS Linux 7 (Core)
OSType: linux
Architecture: x86_64
Number of Docker Hooks: 3
CPUs: 2
Total Memory: 1.694 GiB
Name: iZ2zedz69evgzeipc297uyZ
ID: FQFX:YM25:JHQE:KBG6:4WHC:DYIC:O5VV:GQGS:F2HI:RKW7:3J3O:NMW6
Docker Root Dir: /var/lib/docker
Debug Mode (client): false
Debug Mode (server): false
Registry: https://index.docker.io/v1/
Experimental: false
Insecure Registries:
 127.0.0.0/8
Live Restore Enabled: false
Registries: docker.io (secure)
```

#### 2. 常用命令

查看镜像
```cmd
docker images
docker images -q #查看所有镜像ID
```

删除镜像
```cmd
# docker rmi master/mysql
# docker rmi $(docker images -q) #删除所有镜像
```

查看容器
```cmd
docker ps
docker ps -a  #列出所有容器(包括未运行的)
docker ps -aq #列出所有容器ID
```

查看容器log
```cmd
# docker logs a930350d7fe1
```

停止容器
```cmd
docker stop a930350d7fe1
docker stop $(docker ps -aq) #停止所有容器
```

删除容器
```cmd
# docker rm a930350d7fe1
# docker rm $(docker ps -aq) #删除所有容器
```

### 问题处理

#### 1. 拉取镜像时报错:lookup production.cloudflare.docker.com: no such host
```cmd
# docker pull mysql
Using default tag: latest
Trying to pull repository docker.io/library/mysql ... 
latest: Pulling from docker.io/library/mysql
bf5952930446: Pulling fs layer 
8254623a9871: Pulling fs layer 
938e3e06dac4: Pulling fs layer 
ea28ebf28884: Waiting 
f3cef38785c2: Waiting 
894f9792565a: Waiting 
1d8a57523420: Waiting 
6c676912929f: Waiting 
ff39fdb566b4: Waiting 
fff872988aba: Waiting 
4d34e365ae68: Waiting 
7886ee20621e: Waiting 
error pulling image configuration: Get https://production.cloudflare.docker.com/registry-v2/docker/registry/v2/blobs/sha256/0d/0d64f46acfd1af4ee6a162f80c6e07e843761bf14d412060023bf0e69e720fb4/data?verify=1597410658-gR3hOa3pYHl9U6iRpV3OykN4STI%3D: dial tcp: lookup production.cloudflare.docker.com: no such host
```

进入 ```/etc/resolv.conf``` 添加
```cmd
nameserver 8.8.8.8
````

#### 2. 下载镜像过慢
手动配置镜像源，```/etc/docker/daemon.json```中添加：
```json
{
  "registry-mirrors" : ["https://xxxxxxxx.mirror.aliyuncs.com"]
}
```