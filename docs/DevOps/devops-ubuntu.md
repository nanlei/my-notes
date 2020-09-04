<h3 align="center"><b>Ubuntu 运维</b></h3>

#### 缺失包安装

查看系统版本，发现命令不存在
```cmd
# lsb_release -a
bash: lsb_release: command not found
```

使用apt-get安装，发现无法定位安装包
```cmd
# apt-get install lsb-release
Reading package lists... Done
Building dependency tree       
Reading state information... Done
E: Unable to locate package lsb-release
```

执行apt-get update更新
```cmd
# apt-get update
Ign:1 http://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.4 InRelease                                                                
Hit:2 http://archive.ubuntu.com/ubuntu bionic InRelease                                                                                  
Get:3 http://security.ubuntu.com/ubuntu bionic-security InRelease [88.7 kB]                   
Hit:4 http://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.4 Release                                   
Get:6 http://archive.ubuntu.com/ubuntu bionic-updates InRelease [88.7 kB]           
Get:7 http://security.ubuntu.com/ubuntu bionic-security/main amd64 Packages [1067 kB]
Get:8 http://archive.ubuntu.com/ubuntu bionic-backports InRelease [74.6 kB]
Get:9 http://archive.ubuntu.com/ubuntu bionic-updates/universe amd64 Packages [1421 kB]
Get:10 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 Packages [1362 kB]
Get:11 http://security.ubuntu.com/ubuntu bionic-security/universe amd64 Packages [892 kB]  
Fetched 4994 kB in 7s (692 kB/s)                                                                                                                                      
Reading package lists... Done
```

再次执行安装
```cmd
# apt-get install lsb-release
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following additional packages will be installed:
  distro-info-data file libexpat1 libmagic-mgc libmagic1 libmpdec2 libpython3-stdlib libpython3.6-minimal libpython3.6-stdlib mime-support python3 python3-minimal
  python3.6 python3.6-minimal xz-utils
Suggested packages:
  lsb python3-doc python3-tk python3-venv python3.6-venv python3.6-doc binutils binfmt-support
The following NEW packages will be installed:
  distro-info-data file libexpat1 libmagic-mgc libmagic1 libmpdec2 libpython3-stdlib libpython3.6-minimal libpython3.6-stdlib lsb-release mime-support python3
  python3-minimal python3.6 python3.6-minimal xz-utils
0 upgraded, 16 newly installed, 0 to remove and 5 not upgraded.
Need to get 4704 kB of archives.
After this operation, 28.4 MB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 libpython3.6-minimal amd64 3.6.9-1~18.04ubuntu1.1 [533 kB]
(省略部分内容...)
Get:16 http://archive.ubuntu.com/ubuntu bionic/main amd64 xz-utils amd64 5.2.2-1.3 [83.8 kB]
Fetched 4704 kB in 5s (902 kB/s)   
debconf: delaying package configuration, since apt-utils is not installed
Selecting previously unselected package libpython3.6-minimal:amd64.
(Reading database ... 6772 files and directories currently installed.)
Preparing to unpack .../libpython3.6-minimal_3.6.9-1~18.04ubuntu1.1_amd64.deb ...
(省略部分内容...)
Unpacking lsb-release (9.20170808ubuntu1) ...
Selecting previously unselected package xz-utils.
Preparing to unpack .../6-xz-utils_5.2.2-1.3_amd64.deb ...
Unpacking xz-utils (5.2.2-1.3) ...
Setting up mime-support (3.60ubuntu1) ...
Setting up distro-info-data (0.37ubuntu0.7) ...
Setting up libmagic-mgc (1:5.32-2ubuntu0.4) ...
Setting up libmagic1:amd64 (1:5.32-2ubuntu0.4) ...
Setting up xz-utils (5.2.2-1.3) ...
update-alternatives: using /usr/bin/xz to provide /usr/bin/lzma (lzma) in auto mode
(省略部分内容...)
Setting up libmpdec2:amd64 (2.4.2-1ubuntu1) ...
Setting up libpython3.6-stdlib:amd64 (3.6.9-1~18.04ubuntu1.1) ...
Setting up python3.6 (3.6.9-1~18.04ubuntu1.1) ...
Setting up file (1:5.32-2ubuntu0.4) ...
Setting up libpython3-stdlib:amd64 (3.6.7-1~18.04) ...
Setting up python3 (3.6.7-1~18.04) ...
running python rtupdate hooks for python3.6...
running python post-rtupdate hooks for python3.6...
Setting up lsb-release (9.20170808ubuntu1) ...
Processing triggers for libc-bin (2.27-3ubuntu1.2) ...

```

再次执行命令
```cmd
# lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 18.04.5 LTS
Release:	18.04
Codename:	bionic
```

##### ping命令安装： apt-get install iputils-ping
