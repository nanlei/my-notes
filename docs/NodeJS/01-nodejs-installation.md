<h3 align="center"><b>NodeJS 安装(@MacOS)</b></h3>

## 1. 安装`nvm`(Node Version Manager)

nvm的官方地址为：https://github.com/nvm-sh/nvm

找到当前版本的安装脚本，执行即可：
```cmd
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.37.2/install.sh | bash
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 13527  100 13527    0     0  34773      0 --:--:-- --:--:-- --:--:-- 34773
=> Downloading nvm from git to '/Users/nanlei/.nvm'
=> Cloning into '/Users/nanlei/.nvm'...
remote: Enumerating objects: 333, done.
remote: Counting objects: 100% (333/333), done.
remote: Compressing objects: 100% (283/283), done.
remote: Total 333 (delta 38), reused 151 (delta 25), pack-reused 0
Receiving objects: 100% (333/333), 177.16 KiB | 530.00 KiB/s, done.
Resolving deltas: 100% (38/38), done.
=> Compressing and cleaning up git repository

=> Appending nvm source string to /Users/nanlei/.bash_profile
=> Appending bash_completion source string to /Users/nanlei/.bash_profile
=> Close and reopen your terminal to start using nvm or run the following to use it now:

export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
```

查看`.bash_profile`文件，可以看到`nvm`已经写入：
```shell
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
```

重启Terminal，即可验证`nvm`安装：
```cmd
nvm --version
0.37.2
```

## 2. 安装`Node.js`

获取可供安装的`Node.js`版本：
```cmd
nvm ls-remote
        v0.1.14
        v0.1.15
        ...
        v4.9.1   (Latest LTS: Argon)
        ...
        v6.17.1   (Latest LTS: Boron)
        ...
        v8.17.0   (Latest LTS: Carbon)
        ...
        v10.23.0   (Latest LTS: Dubnium)
        ...
        v12.20.0   (Latest LTS: Erbium)
        ...
        v14.15.1   (Latest LTS: Fermium)
        ...
```

选择LTS(Long Term Support)版本安装即可：
```cmd
nvm install 14.15.1
Downloading and installing node v14.15.1...
Downloading https://nodejs.org/dist/v14.15.1/node-v14.15.1-darwin-x64.tar.xz...
################################################################################################################################################################################ 100.0%
Computing checksum with shasum -a 256
Checksums matched!
Now using node v14.15.1 (npm v6.14.8)
Creating default alias: default -> 14.15.1 (-> v14.15.1)
```

检查安装结果：
```cmd
node -v
v14.15.1
```

查看`Node.js`安装路径：
```cmd
which node
/Users/nanlei/.nvm/versions/node/v14.15.1/bin/node
```

查看`Node.js`已安装版本：
```cmd
nvm ls
->     v14.15.1
default -> 14.15.1 (-> v14.15.1)
iojs -> N/A (default)
unstable -> N/A (default)
node -> stable (-> v14.15.1) (default)
stable -> 14.15 (-> v14.15.1) (default)
lts/* -> lts/fermium (-> v14.15.1)
lts/argon -> v4.9.1 (-> N/A)
lts/boron -> v6.17.1 (-> N/A)
lts/carbon -> v8.17.0 (-> N/A)
lts/dubnium -> v10.23.0 (-> N/A)
lts/erbium -> v12.20.0 (-> N/A)
lts/fermium -> v14.15.1
```

若`nvm`执行速度过慢，可以使用`NVM_NODEJS_ORG_MIRROR`设置镜像来提高下载速度，比如：
```cmd
NVM_NODEJS_ORG_MIRROR=https://npm.taobao.org/mirrors/node nvm install 14.15.1
```

切换`Node.js`版本，可以安装一个新版本(这里安装12)
```cmd
nvm install 12
Downloading and installing node v12.20.0...
Downloading https://nodejs.org/dist/v12.20.0/node-v12.20.0-darwin-x64.tar.xz...
################################################################################################################################################################################ 100.0%
Computing checksum with shasum -a 256
Checksums matched!
Now using node v12.20.0 (npm v6.14.8)
```

此时系统使用v12.20.0版本，切回v14.15.1可以使用如下命令：
```cmd
nvm use 14
Now using node v14.15.1 (npm v6.14.8)
```

查看`Node.js`全局目录和缓存位置：
```cmd
npm config get prefix
/Users/nanlei/.nvm/versions/node/v14.15.1
npm config get cache
/Users/nanlei/.npm
```

如果要使用其他版本安装的全局模块，可以使用如下命令(从v12版本复制)
```cmd
nvm reinstall-packages 12
```

## 3. 安装`nrm`(Node Registry Manager)

使用`npm`直接安装即可：
```cmd
npm install -g nrm
npm WARN deprecated request@2.88.2: request has been deprecated, see https://github.com/request/request/issues/3142
npm WARN deprecated mkdirp@0.3.5: Legacy versions of mkdirp are no longer supported. Please update to mkdirp 1.x. (Note that the API surface has changed to use Promises in 1.x.)
npm WARN deprecated coffee-script@1.7.1: CoffeeScript on NPM has moved to "coffeescript" (no hyphen)
npm WARN deprecated har-validator@5.1.5: this library is no longer supported
/Users/nanlei/.nvm/versions/node/v14.15.1/bin/nrm -> /Users/nanlei/.nvm/versions/node/v14.15.1/lib/node_modules/nrm/cli.js
+ nrm@1.2.1
added 494 packages from 893 contributors in 16.63s
```

查看`nrm`的源：
```cmd
nrm ls

* npm -------- https://registry.npmjs.org/
  yarn ------- https://registry.yarnpkg.com/
  cnpm ------- http://r.cnpmjs.org/
  taobao ----- https://registry.npm.taobao.org/
  nj --------- https://registry.nodejitsu.com/
  npmMirror -- https://skimdb.npmjs.com/registry/
  edunpm ----- http://registry.enpmjs.org/

```

其中，带`*`号的表示当前使用的源。使用`nrm`测速，确定哪个源更快
```cmd
nrm test

* npm ---- 339ms
  yarn --- 334ms
  cnpm --- 995ms
  taobao - 319ms
  nj ----- Fetch Error
  npmMirror  1245ms
  edunpm - Fetch Error

```

确定要使用哪个源之后，可以切换源来获得最佳体验，比如切换到`taobao`：
```cmd
nrm use taobao
Registry has been set to: https://registry.npm.taobao.org/
```

如果本地搭建了私有镜像，也可以向nrm添加源：
```cmd
nrm add registryname http://registry.npm.youraddress.com
```
