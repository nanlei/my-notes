<h3 align="center"><b>CentOS 运维</b></h3>

连接到服务器
```cmd
$ ssh root@xxx.xxx.xxx.xxx
root@xxx.xxx.xxx.xxx's password: 
Last login: Fri Aug 14 19:27:34 2020 from 123.185.87.241

Welcome to Alibaba Cloud Elastic Compute Service !

-bash: warning: setlocale: LC_CTYPE: cannot change locale (UTF-8): No such file or directory
```

查看系统版本信息
```cmd
# cat /proc/version
Linux version 3.10.0-1127.18.2.el7.x86_64 (mockbuild@kbuilder.bsys.centos.org) (gcc version 4.8.5 20150623 (Red Hat 4.8.5-39) (GCC) ) #1 SMP Sun Jul 26 15:27:06 UTC 2020
# uname -a
Linux iZ2zedz69evgzeipc297uyZ 3.10.0-1127.18.2.el7.x86_64 #1 SMP Sun Jul 26 15:27:06 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
# lsb_release -a
LSB Version:	:core-4.1-amd64:core-4.1-noarch
Distributor ID:	CentOS
Description:	CentOS Linux release 7.8.2003 (Core)
Release:	7.8.2003
Codename:	Core
```

检查需要更新的软件
```cmd
# yum check-update
Failed to set locale, defaulting to C
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile

NetworkManager.x86_64                                                                  1:1.18.4-3.el7                                                                      base     
...
centos-release.x86_64                                                                  7-8.2003.0.el7.centos                                                               base   
... 
```

升级软件
```cmd
# yum upgrade
Failed to set locale, defaulting to C
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
Resolving Dependencies
--> Running transaction check
---> Package NetworkManager.x86_64 1:1.18.0-5.el7_7.1 will be updated
---> Package NetworkManager.x86_64 1:1.18.4-3.el7 will be an update
...
Transaction Summary
====================================================================================================================================================================================
Install    3 Packages
Upgrade  147 Packages

Total download size: 293 M
Is this ok [y/d/N]: y
Downloading packages:
Delta RPMs disabled because /usr/bin/applydeltarpm not installed.
(1/150): NetworkManager-libnm-1.18.4-3.el7.x86_64.rpm        
...
  util-linux.x86_64 0:2.23.2-63.el7                           yum.noarch 0:3.4.3-167.el7.centos                      yum-plugin-fastestmirror.noarch 0:1.1.31-54.el7_8            

Replaced:
  iwl7265-firmware.noarch 0:22.0.7.0-72.el7                                                                                                                                         

Complete!
```



