<h3 align="center"><b>Debian 运维</b></h3>

#### 获取发行版本信息
```cmd
# cat /etc/os-release
PRETTY_NAME="Debian GNU/Linux 10 (buster)"
NAME="Debian GNU/Linux"
VERSION_ID="10"
VERSION="10 (buster)"
VERSION_CODENAME=buster
ID=debian
HOME_URL="https://www.debian.org/"
SUPPORT_URL="https://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"
```

#### 缺失包安装

发现缺失包
```cmd
# vi /etc/mysql/my.cnf
bash: vi: command not found
```

无法更新
```cmd
# apt-get install vim
Reading package lists... Done
Building dependency tree
Reading state information... Done
E: Unable to locate package vim
```

执行apt-get update更新
```cmd
# apt-get update
Get:1 http://repo.mysql.com/apt/debian buster InRelease [21.5 kB]
Get:2 http://repo.mysql.com/apt/debian buster/mysql-8.0 amd64 Packages [7172 B]
Get:3 http://security.debian.org/debian-security buster/updates InRelease [65.4 kB]
Get:4 http://deb.debian.org/debian buster InRelease [122 kB]
Get:5 http://security.debian.org/debian-security buster/updates/main amd64 Packages [226 kB]
Get:6 http://deb.debian.org/debian buster-updates InRelease [51.9 kB]
Get:7 http://deb.debian.org/debian buster/main amd64 Packages [7906 kB]
Get:7 http://deb.debian.org/debian buster/main amd64 Packages [7906 kB]
Get:7 http://deb.debian.org/debian buster/main amd64 Packages [7906 kB]
Get:8 http://deb.debian.org/debian buster-updates/main amd64 Packages [7868 B]
Fetched 8008 kB in 5min 0s (26.7 kB/s)
Reading package lists... Done
```

再次执行更新安装即可