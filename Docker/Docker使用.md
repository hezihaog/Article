#### Docker使用

Docker越来越常用，Docker学习势在必行！

## 安装Docker

>Docker官方建议在Ubuntu中安装，因为Docker是基于Ubuntu发布的，而且一般Docker出现的问题Ubuntu是最先更新或者打补丁的。在很多版本的CentOS中是不支持更新最新的一些补丁包的。
>> 如果使用的是CentOS，建议安装在CentOS 7.x以上的版本，在CentOS 6.x的版本中，安装前需要安装其他很多的环境而且Docker很多补丁不支持更新。

- yum 包更新到最新

```
sudo yum update
```

- 安装需要的软件包， yum-util 提供yum-config-manager功能，另外两个是devicemapper驱动依赖的

```
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

- 设置yum源为阿里云（默认连的是国外站点，速度很慢）

```
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

- 安装docker

```
sudo yum install docker-ce
```

- 安装后查看docker版本

```
docker -v
```

### 设置ustc的镜像

ustc是老牌的linux镜像服务提供者了，还在遥远的ubuntu 5.04版本的时候就在用。ustc的docker镜像加速器速度很快。ustc docker mirror的优势之一就是不需要注册，是真正的公共服务。

[ustc地址](https://lug.ustc.edu.cn/wiki/mirrors/help/docker)

- 编辑daemon.json文件

```
vi /etc/docker/daemon.json
```

- 在该文件中输入如下内容

```
{
"registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]
}
```

### Docker的启动与停止

**systemctl**命令是系统服务管理器指令

- 启动docker

```
systemctl start docker
```

- 停止docker

```
systemctl stop docker
```

- 重启docker

```
systemctl restart docker
```

- 查看docker状态

```
systemctl status docker
```

- 设置为开机启动

```
systemctl enable docker
```

- 查看docker概要信息

```
docker info
```

- 查看docker帮助文档

```
docker --help
```

## 常用命令

### 镜像相关命令

- 查看镜像

镜像都是存储在Docker宿主机的/var/lib/docker目录下

```
docker images

//输出的表格，每一列的参数意义
REPOSITORY：镜像名称
TAG：镜像标签
IMAGE ID：镜像ID
CREATED：镜像的创建日期（不是获取该镜像的日期）
SIZE：镜像大小
```

- 搜索镜像

如果你需要从网络中查找需要的镜像，可以通过以下命令搜索

```
docker search 镜像名称

//输出的表格，每一列的参数意义
NAME：仓库名称
DESCRIPTION：镜像描述
STARS：用户评价，反应一个镜像的受欢迎程度
OFFICIAL：是否官方
AUTOMATED：自动构建，表示该镜像由Docker Hub自动构建流程创建的
```

- 拉取镜像

拉取镜像就是从中央仓库中下载镜像到本地

```
docker pull 镜像名称

//例如，要下载centos7镜像
docker pull centos:7
```

- 删除镜像

按镜像ID删除镜像

```
docker rmi 镜像ID
```

- 删除所有镜像

```
docker rmi `docker images -q`
```

### 容器相关命令

#### 查看容器

- 查看正在运行的容器

```
docker ps
```

- 查看所有容器

```
docker ps -a
```

- 查看最后一次运行的容器

```
docker ps -l
```

- 查看停止的容器

```
docker ps -f status=exited
```

#### 创建与启动容器

创建容器常用的参数说明

```
创建容器命令：docker run
    -i：表示运行容器
    -t：表示容器启动后会进入其命令行。加入这两个参数后，容器创建就能登录进去。即分配一个伪终端。
    --name :为创建的容器命名。
    -v：表示目录映射关系（前者是宿主机目录，后者是映射到宿主机上的目录），可以使用多个－v做多个目录或文件映射。注意：最好做目录映射，在宿主机上做修改，然后共享到容器上。
    -d：在run后面加上-d参数,则会创建一个守护式容器在后台运行（这样创建容器后不会自动登录容器，如果只加-i -t两个参数，创建后就会自动进去容器）。
    -p：表示端口映射，前者是宿主机端口，后者是容器内的映射端口。可以使用多个-p做多个端口映射
    （1）交互式方式创建容器
```

- 交互式方式创建容器（创建就进入登录容器，但是一退出，容器就会关闭）

```
docker run -it --name=容器名称 镜像名称:标签 /bin/bash

//退出当前容器
exit
```

- 守护式方式创建容器（需要单独登录容器，但是退出并不会关闭容器）

```
docker run -di --name=容器名称 镜像名称:标签
```

- 登录守护式容器方式

```
docker exec -it 容器名称 (或者容器ID)  /bin/bash
```

#### 停止与启动容器

- 停止容器

```
docker stop 容器名称（或者容器ID）
```

- 启动容器

```
docker start 容器名称（或者容器ID）
```

#### 文件拷贝

如果我们需要将文件拷贝到容器内，可以使用cp命令

- 拷贝文件到容器中

```
docker cp 需要拷贝的文件或目录 容器名称:容器目录
```

- 从容器中，拷贝文件出来

```
docker cp 容器名称:容器目录 需要拷贝的文件或目录
```

#### 目录挂载（映射）

我们可以在创建容器的时候，将宿主机的目录与容器内的目录进行映射，这样我们就可以通过修改宿主机某个目录的文件从而去影响容器。

- 创建容器时，就指定目录挂载

创建容器 添加-v参数 后边为宿主机目录:容器目录，例如：

```
docker run -di -v /usr/local/myhtml:/usr/local/myhtml --name=mycentos3 centos:7
```

如果你共享的是多级的目录，可能会出现权限不足的提示。
这是因为CentOS7中的安全模块selinux把权限禁掉了，我们需要添加参数  --privileged=true  来解决挂载的目录没有权限的问题。

#### 查看容器IP地址

我们可以通过以下命令查看容器运行的各种数据

```
docker inspect 容器名称（容器ID）
```

也可以直接执行下面的命令直接输出IP地址

```
docker inspect --format='{{.NetworkSettings.IPAddress}}' 容器名称（容器ID）
```

#### 删除容器

- 删除指定的容器

```
docker rm 容器名称（容器ID）
```

### 应用部署

#### MySQL部署

- 拉取mysql镜像

```
docker pull centos/mysql-57-centos7
```

- 创建容器

```
docker run -di --name=tensquare_mysql -p 33306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql

//参数解释
* -p 代表端口映射，格式为  宿主机映射端口:容器运行端口
* -e 代表添加环境变量  MYSQL_ROOT_PASSWORD  是root用户的登陆密码
* 远程登录mysql
连接宿主机的IP,指定端口为33306
```

#### tomcat部署

- 拉取镜像

```
docker pull tomcat:7-jre8
```

- 创建容器

创建容器，-p表示地址映射

```
docker run -di --name=mytomcat -p 9000:8080 
-v /usr/local/webapps:/usr/local/tomcat/webapps tomcat:7-jre8
```

#### Nginx部署

- 拉取镜像

```
docker pull nginx
```

- 创建Nginx容器

```
docker run -di --name=mynginx -p 80:80 nginx
```

#### Redis部署

- 拉取镜像

```
docker pull redis
```

- 创建容器

```
docker run -di --name=myredis -p 6379:6379 redis
```

### 迁移与备份

- 容器保存为镜像

我们可以通过以下命令将容器保存为镜像

```
docker commit mynginx mynginx_i
```

- 镜像备份

我们可以通过以下命令将镜像保存为tar 文件

```
docker  save -o mynginx.tar mynginx_i
```

- 镜像恢复与迁移

首先我们先删除掉mynginx_img镜像  然后执行此命令进行恢复

```
docker load -i mynginx.tar

//参数解释
-i 输入的文件
执行后再次查看镜像，可以看到镜像已经恢复
```