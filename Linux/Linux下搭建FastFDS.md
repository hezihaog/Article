# Linux下搭建FastFDS

分布式文件存储，FastDFS是一种选择，它是分布式、高可用的。

# 参考

[学习视频](https://www.bilibili.com/video/BV1ta4y1v7Kw?p=6)

## 大纲

- 安装FastDFS环境
    - 前言
    - 安装 libfastcommon
    - 安装 FastDFS
    - 配置FastDFS跟踪器(Tracker)
    - 配置 FastDFS 存储 (Storage)
    - 文件上传测试
- 配合Nginx
    - 安装Nginx所需环境
    - 安装Nginx
    - 访问文件
- FastDFS配置Nginx模块
    - 安装配置Nginx模块
    
## 安装FastDFS环境

### 前言

系统为`centos7 64位`，ip地址为：`192.168.211.131`

安装FastDFS需要4个安装包，我们都上传到`/usr/local/soft`下。

```
cd usr/local
mkdir soft
```

#### 安装包

- 下载地址：[百度网盘](https://pan.baidu.com/s/1odUUYR2n38ofDam-4U8kBw) 提取码: vjpf

#### 安装环境所需要的依赖

例如c编译器：gcc

```
yum install make cmake gcc gcc-c++
```

### 安装 libfastcommon

`FastDFS`依赖`libfastcommon`公共库，所以先安装它

切换到`/usr/local/soft`目录，解压`libfastcommon`

```
tar -zxvf libfastcommonV1.0.7.tar.gz
```

进入`libfastcommon`目录，执行编译、安装

```
cd /usr/local/soft/libfastcommon-1.0.7/
//编译
./make.sh
//安装
./make.sh install
```

### 安装FastDFS

解压`FastDFS`安装包

```
tar -zxvf FastDFS_v5.05.tar.gz
```

切换目录，执行编译、安装

```
cd /usr/local/soft/fastdfs-5.05
//编译
./make.sh
//安装
./make.sh install
```

### 安装后，目录和文件

#### 服务脚本

在`/etc/init.d/`目录下

```
/etc/init.d/fdfs_storaged
/etc/init.d/fdfs_tracker
```

#### 配置文件

3个作者提供的配置样例，在`/etc/fdfs`目录下

```
/etc/fdfs/client.conf.sample
/etc/fdfs/storage.conf.sample
/etc/fdfs/tracker.conf.sample
```

#### 命令工具

在`/usr/bin`目录下

```
fdfs_appender_test
fdfs_appender_test1
fdfs_append_file
fdfs_crc32
fdfs_delete_file
fdfs_download_file
fdfs_file_info
fdfs_monitor
fdfs_storaged
fdfs_test
fdfs_test1
fdfs_trackerd
fdfs_upload_appender
fdfs_upload_file
stop.sh
restart.sh
```

### 配置FastDFS跟踪器(Tracker)

- 进入解压`FastDFS`的解压目录，再进入到里面的`conf`目录，拷贝`http.conf`和`mine.types`，这2个文件到`etc/fdfs`目录下

```
cd /usr/local/soft/fastdfs-5.05
//进入conf目录
cd conf
//拷贝2个文件到，etc/fdfs目录下
cp http.conf /etc/fdfs
cp mine.types /etc/fdfs
```

- 拷贝`/etc/fdfs`目录下的2个sample文件到桌面，进行修改，并去掉.sample扩展名，为`storage.conf`和`tracker.conf`

```
/etc/fdfs/storage.conf.sample
/etc/fdfs/tracker.conf.sample
```

#### 修改`tracker.conf`

tracker server，只需要配置1个地方

修改`base_path`基础目录，FastDFS启动之后，会在该目录下，生成一些日志文件，修改为：`/opt/fastdfs/tracker`

我们修改为`/opt/fastdfs/tracker`

注意：这个目录必须手动创建，如果不存在，否则启动时会报错

```
#修改前
#base_path=/home/yuqing/fastdfs\

#修改后
base_path=/opt/fastdfs/tracker
```

### 配置 FastDFS 存储 (Storage)

#### 修改`storage.conf`

storage server，需要配置3个地方

1. 修改`base_path`基础路径，也是日志文件存放的位置，修改为：`/opt/fastdfs/storage`

```
#修改前
#base_path=/home/yuqing/fastdfs

#修改后
base_path=/opt/fastdfs/storage
```

2. 修改`store_path0`，真正文件存放目录，修改为：`/opt/fastdfs/storage/files`

```
#修改前
#store_path0=/home/yuqing/fastdfs

#修改后
store_path0=/opt/fastdfs/storage/files
```

3. 修改`tracker_server`，该配置为`tracker server 跟踪器`的地址，修改为你的linux的ip地址

```
#修改前
tracker_server=192.168.209.121:22122

#修改后
tracker_server=192.168.211.131:22122
```

#### 上传配置文件

上传上面修改好的文件`storage.conf`和`tracker.conf`，到linux的`/etc/fdfs`目录中

### 启动FastDFS

#### 创建需要的文件夹

上面说到，我们配置的目录，都需要手动创建，否则启动时会报错

- 创建 tracker server 目录

```
cd /opt
//基础路径
mkdir /opt/fastdfs
mkdir /opt/fastdfs/tracker
```

- 创建 storage server 目录

```
cd /opt
//基础路径
mkdir /opt/fastdfs/storage
//文件存放路径
mkdir /opt/fastdfs/storage/files
```

#### 启动、关闭、重启 tracker server

```
//语法：fdfs_trackerd 配置文件路径 start|stop|restart，如果不传参数，则默认是start启动

//启动
fdfs_trackerd /etc/fdfs/tracker.conf start
//停止
fdfs_trackerd /etc/fdfs/tracker.conf stop
//重启
fdfs_trackerd /etc/fdfs/tracker.conf restart
```

#### 启动、关闭、重启 storage server

```
//语法
fdfs_storaged /etc/fdfs/storage.conf start，如果不传参数，则默认是start启动

//启动
fdfs_storaged /etc/fdfs/tracker.conf start
//停止
fdfs_storaged /etc/fdfs/tracker.conf stop
//重启
fdfs_storaged /etc/fdfs/tracker.conf restart
```

非必要情况，不要使用kill -9 来强制关闭进程，否则在文件上传途中，强制关闭，会导致文件信息不同步的问题

#### 查看是否启动成功

- ps命令查看，能有这样的输出就对了

```
ps -ef | grep fdfs

root      49467      1  0 11:40 ?        00:00:00 fdfs_trackerd /etc/fdfs/tracker.conf start
root      59332      1  9 11:50 ?        00:00:01 fdfs_storaged /etc/fdfs/storage.conf start
root      59652 113978  0 11:50 pts/0    00:00:00 grep --color=auto fdfs
```

- 如果没有，查看log信息，修改完成后，再次启动

```
cd /opt/
cd fastdfs/

//查看tracker，则进入tracker目录
cd tracker/
//如果是查看storage，则进入storage目录
cd storage/

//例如我们storage启动错误，则进入storage目录查看log文件
cd storage/
cd logs/
vim storage.log
```

#### 文件存储目录

文件存放目录在：`/opt/fastdfs/storage/files/data/`，FastDFS会在这里创建256个文件夹，第一层内也有256个，一共2层，所以一共有256*256个文件夹，就是65536个文件夹（6万个）。

目的是将文件分类，不集中于一个文件夹，否则读取会很慢。

### 文件上传测试

将`/etc/fdfs/`下的，`client.conf.sample`文件，上传到桌面，再进行去掉`.sample`拓展名，该文件只在测试时有用

需要修改2个地方！

1. 修改`base_path`基础路径，该目录也是要手动创建的！

```
#修改前
base_path=/home/yuqing/fastdfs

#修改后
base_path=/opt/fastdfs/client
```

2. 修改`tracker_server`跟踪器地址，修改成自己的linux的ip地址

```
#修改前
tracker_server=192.168.0.197:22122

#修改后
tracker_server=192.168.211.131:22122
```

3. 创建文件夹

```
mkdir /opt/fastdfs/client
```

修改完毕，再将`client.conf`上传回linux上即可

#### 创建要上传的文件

我们在当前登录的用户的家目录下，创建一个aa.txt

```
//回到家目录
cd ~
//创建aa.txt
vim aa.txt
//写入内容，wq保存
this is FastDFS test File!
```

#### 上传

- 上传命令

```
语法：fdfs_test 配置文件位置 参数 upload, download, getmeta, setmeta, delete and query_servers

//上传
fdfs_test /etc/fdfs/client.conf upload 要上传的文件路径
```

- 上传成功，输出一大片信息，我们挑一些有用的信息出来

```
[2020-08-31 12:32:26] DEBUG - base_path=/opt/fastdfs/client, connect_timeout=30, network_timeout=60, tracker_server_count=1, anti_steal_token=0, anti_steal_secret_key length=0, use_connection_pool=0, g_connection_pool_max_idle_time=3600s, use_storage_id=0, storage server id count: 0

tracker_query_storage_store_list_without_group: 
        server 1. group_name=, ip_addr=192.168.211.131, port=23000

group_name=group1, ip_addr=192.168.211.131, port=23000
storage_upload_by_filename
group_name=group1, remote_filename=M00/00/00/wKjTg19MfVqAA3YFAAAAGw0ZxeU707.txt
source ip address: 192.168.211.131
file timestamp=2020-08-31 12:32:26
file size=27
file crc32=219792869
example file url: http://192.168.211.131/group1/M00/00/00/wKjTg19MfVqAA3YFAAAAGw0ZxeU707.txt
storage_upload_slave_by_filename
group_name=group1, remote_filename=M00/00/00/wKjTg19MfVqAA3YFAAAAGw0ZxeU707_big.txt
source ip address: 192.168.211.131
file timestamp=2020-08-31 12:32:26
file size=27
file crc32=219792869
example file url: http://192.168.211.131/group1/M00/00/00/wKjTg19MfVqAA3YFAAAAGw0ZxeU707_big.txt
```

1. group_name，组名，文件上传到哪个组里了，这个组名决定文件上传到了哪个机器上面去
2. remote_filename，远程文件名称，这个名称决定了文件存放到了哪个磁盘目录下

- 回头看一下，`/etc/fdfs/`目录下的`client.info`文件

有2个属性，我们要注意：`store_path_count`和`store_path`

1. store_path_count，表示存储的磁盘有多少个，一般我们就一个磁盘，所以是1，除非你有多个磁盘，例如有2个，则改为2
2. store_path，表示文件存放到本地磁盘中的位置，它和磁盘个数对应，有多少个磁盘，就有多少个`store_path`。例如`store_path0`，就是第一块磁盘的存储路径，`store_path1`，就是第二块磁盘的存储路径

- 继续看下输出信息

分解：remote_filename=M00/00/00/wKjTg19MfVqAA3YFAAAAGw0ZxeU707.txt

1. M00，表示第几块磁盘，从0开始，例如只有1块磁盘，那么都是M00。如果有2块，则可能是M00或者M01
2. 00/00，就是那6万个目录，只有第二层目录才是存储文件的
3. wKjTg19MfVqAA3YFAAAAGw0ZxeU707.txt，上传的文件的名字

注意，FastDFS会在我们配置的存储路径上，加上`data/`目录，再进行存储。

以上面信息来看，`M00/00/00/wKjTg19MfVqAA3YFAAAAGw0ZxeU707.txt`，对应的路径就为`/opt/fastdfs/storage/files/data/00/00/wKjTg19MfVqAA3YFAAAAGw0ZxeU707.text`

我们进入到目录，查看文件，发现有4个文件

1. wKjTg19MfVqAA3YFAAAAGw0ZxeU707_big.txt，因为我们是测试命令，加上_big后缀的是备份文件，和下面的`wKjTg19MfVqAA3YFAAAAGw0ZxeU707.txt`，内容是完全相同的，正式使用时，不会有该文件！
2. wKjTg19MfVqAA3YFAAAAGw0ZxeU707_big.txt-m，文件的meta data 信息，记录了一些文件的拓展名、文件大小等信息
3. wKjTg19MfVqAA3YFAAAAGw0ZxeU707.txt，真正上传过来的文件，我们上传的文件是aa.txt，上传过来会被FastDFS重命名，目录是为了防止多个用户上传同一个名称的文件，导致冲突
4. wKjTg19MfVqAA3YFAAAAGw0ZxeU707.txt-m，文件的属性文件，记录文件的拓展名、文件大小、高度、宽度等信息，因为是测试命令，才生成该文件。通常这些信息我们会存储到数据库，而不是这个文件里

- 文件查看

上面的信息中，有一个Url：example file url: http://192.168.211.131/group1/M00/00/00/wKjTg19MfVqAA3YFAAAAGw0ZxeU707.txt

默认这个文件是不能直接访问的，后面我们需要配置一下才能访问！

### 文件下载和删除

#### 下载

```
//命令：fdfs_test 配置文件路径 download 组名 远程文件名称

//例如下载，刚才我们上传的文件
fdfs_test /etc/fdfs/client.conf download group1 M00/00/00/wKjTg19MfVqAA3YFAAAAGw0ZxeU707.txt
```

提示以下信息，则为下载成功了

```
download file success, file size=27, file save to wKjTg19MfVqAA3YFAAAAGw0ZxeU707.txt
```

当前文件夹中，就多了一个`wKjTg19MfVqAA3YFAAAAGw0ZxeU707.txt`文件

#### 删除文件

```
//命令：fdfs_test 配置文件路径 delete 组名 远程文件名称

//例如删除，我们刚上传的文件
fdfs_test /etc/fdfs/client.conf delete group1 M00/00/00/wKjTg19MfVqAA3YFAAAAGw0ZxeU707.txt
```

删除以下信息，则为删除成功了

```
delete file success
```

此时，我们再去查看文件上传目录，`/opt/fastdfs/storage/files/data/00/00`，就会发现文件的确删除了

### 配合Nginx

上面我们访问上传完的地址，是访问不到的，想要访问，我们需要配合Nginx才可以。而除了安装Nginx外，还需要安装一个FastDFS提供的Nginx拓展模块。

上传Nginx和Nginx拓展模块到`usr/local/soft`目录下。

```
//nginx
nginx-1.12.1.tar.gz
//fastdfs提供的nginx拓展模块
fastdfs-nginx-module_v1.16.tar.gz
```

解压这2个安装包

```
//解压nginx
tar -zxvf nginx-1.12.1.tar.gz
//解压拓展模块
tar -zxvf fastdfs-nginx-module_v1.16.tar.gz 
```

进入拓展模块的目录：`fastdfs-nginx-module`，再进入里面的`src`目录

```
cd fastdfs-nginx-module/
cd src/
```

下载`mod_fastdfs.conf`配置文件到桌面上，拷贝`src`完整路径拷贝，待会配置nginx时要用！

```
//获取当前的目录路径
pwd
//输出
/usr/local/soft/fastdfs-nginx-module/src
```

#### 安装nginx

上面的解压nginx，已经做了，就不在赘述

下面进行配置nginx

```
//切换到nginx解压目录下
cd nginx-1.12.1

//配置nginx的安装路径，--prefix表示nginx安装路径，--add-module表示添加一个模块到nginx，这个路径要指定刚刚解压缩的路径，例如/usr/local/soft/fastdfs-nginx-module/src
//如果你的模块路径改了，这里的--add-module，也要记得一起改
./configure --prefix=/usr/local/nginx_fdfs --add-module=/usr/local/soft/fastdfs-nginx-module/src

//编译
make

//安装
make install
```

注意：如果你在make中出现`make： *** 没有规则可以创建“default”需要的目标“build”`

可能还缺少`pcre-devel`、`zlib-devel`、`openssl-devel`，这些依赖库，下载并安装后，再执行make

```
yum install pcre-devel zlib zlib-devel openssl openssl-devel
```

注意：如果你在make时出现`致命错误：fdfs_define.h：没有那个文件或目录`，则需要创建3个文件的软连接，再执行make

```
ln -sv /usr/include/fastcommon /usr/local/include/fastcommon 
ln -sv /usr/include/fastdfs /usr/local/include/fastdfs 
ln -sv /usr/lib64/libfastcommon.so /usr/local/lib/libfastcommon.so
```

#### 配置nginx拓展模块

进入拓展模块的目录：`fastdfs-nginx-module`，再进入里面的`src`目录

```
cd fastdfs-nginx-module/
cd src/
```

下载`mod_fastdfs.conf`配置文件到桌面上，这里如果上面解压nginx拓展模块时已经做了，就不需要继续做了！

1. 修改基础路径，这个目录，我们也要手动创建！

```
# 修改前
base_path=/tmp

# 修改后
base_path=/opt/fastdfs/nginx_mod
```

2. 指定tracker server的地址，改成你的地址

```
# 修改前
tracker_server=tracker:22122

# 修改后
tracker_server=192.168.211.131:22122
```

3. 修改url的请求中，必须包含组名，必须改成true，用于后面nginx拦截时判断的正则表达式所用

```
# 修改前
url_have_group_name = false

# 修改后
url_have_group_name = true
```

4. 指定有几个磁盘存储路径，你有几个就写几个，一般只有1个

```
store_path_count=1
```

5. 指定文件存储路径

```
# 修改前
store_path0=/home/yuqing/fastdfs

# 修改后
store_path0=/opt/fastdfs/storage/files
```

6. （重要）上传配置好的`mod_fastdfs.conf`文件，到`/etc/fdfs/`下

7. （重要）创建`/opt/fastdfs/nginx_mod`目录，否则nginx会启动不起来

```
mkdir /opt/fastdfs/nginx_mod
```

#### 配置nginx

切换到fdfs的nginx目录，再进入到`conf`配置目录下

```
cd /usr/local/nginx_fdfs/
//切换到配置目录
cd conf/
```

将`nginx.conf`文件，nginx的配置文件，上传到桌面进行编辑，我们在`server`节点下，配置一个正则表达式的转发规则

```
server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;

    #access_log  logs/host.access.log  main;

    location / {
        root   html;
        index  index.html index.htm;
    }

    # （重点）配置FastDFS，拦截文件请求给FastDFS提供的nginx拓展模块
    location ~ /group[1-9]/M0[0-9] {
        ngx_fastdfs_module;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }
}
```

最后将这个文件上传到`/usr/local/nginx_fdfs/conf`目录下

#### 启动nginx

```
//-t参数，检查配置文件语法是否有问题
/usr/local/nginx_fdfs/sbin/nginx -c /usr/local/nginx_fdfs/conf/nginx.conf -t

//正式启动
/usr/local/nginx_fdfs/sbin/nginx -c /usr/local/nginx_fdfs/conf/nginx.conf
```

输出以下信息，则为成功

```
ngx_http_fastdfs_set pid=115915
nginx: the configuration file /usr/local/nginx_fdfs/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx_fdfs/conf/nginx.conf test is successful
```

检查进程是否启动成功，输出有2个进程，一个`master`，一个`worker`，就成功了，如果`worker`进程没有启动，则检查日志

```
ps -ef | grep nginx

//输出
root     117133      1  0 14:45 ?        00:00:00 nginx: master process /usr/local/nginx_fdfs/sbin/nginx -c /usr/local/nginx_fdfs/conf/nginx.conf
nobody   117134 117133  0 14:45 ?        00:00:00 nginx: worker process
root     120415 113978  0 14:48 pts/0    00:00:00 grep --color=auto nginx
```

进入nginx目录，里面有一个`logs`目录，存放日志文件

```
cd /usr/local/nginx_fdfs
//进入日志目录
cd logs/

//ls，输出
access.log  error.log  nginx.pid
```

如果这个目录下，没有文件，则去到`/opt/fastdfs/`目录下，进入`nginx_mod`目录，查看是否日志文件

```
cd /opt/fastdfs/
//进入拓展插件目录
cd nginx_mod/
//查看是否有文件
ll
```

一般启动不起来，有2个原因

1. `mod_fastdfs.conf`文件，没有放到`/etc/fdfs`目录中去！
2. 如果放了，但`mod_fastdfs.conf`文件里面有错误，一般错误是`base_path`，基础路径这个目录没有创建，必须自己mkdir手动创建才行！