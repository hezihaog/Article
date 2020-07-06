#### mac安装mongodb

因为MongoDB闭源了，网上通过brew和下载dmg安装包的方式已经不可行了，但还有一个方式，就是使用MongoDB自己维护的brew进行安装，速度有点慢，我下了2个小时才下载完， 所以需要耐性等待一下。

- 下载单独维护的brew

```
brew tap mongodb/brew
```

- 下载mongodb-community（社区版）

```
brew install mongodb-community
```

- 启动MongoDB服务

```
brew services start mongodb/brew/mongodb-community
```

#### 配置环境变量

- brew安装完，安装目录是在**/usr/local/Cellar/mongodb-community**下，我的版本是4.2.8

```
/usr/local/Cellar/mongodb-community/4.2.8
```

- 编辑根目录下的**.bash_profile**文件，填入刚才的安装目录，注意你可能需要修改版本

```
export MONGODB_HOME=/usr/local/Cellar/mongodb-community/4.2.8
export PATH=$PATH:$MONGODB_HOME/bin
```

- 保存，关闭，应用环境变量

```
source .bash_profile
```

#### 创建mongoDB的数据库的文件夹

在磁盘根目录创建data文件夹，再在里面创建db文件夹

```
sudo mkdir -p /data/db
```

- 查看版本

```
mongod -version
```

- 进入浏览器，打开**localhost:27017**，出现以下信息就是成功了

```
It looks like you are trying to access MongoDB over HTTP on the native driver port.
```

- 进入MongoDB的命令行客户端，输入exit可退出

```
mongo
```