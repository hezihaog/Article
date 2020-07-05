#### Linux常用指令

- 切换目录指令：cd

```
//切换到app目录
cd app
//切换上一级目录
cd ..
//切换到系统根目录
cd /
//切换到用户主目录
cd ~
//切换到上一级目录的指定文件夹，如切换到上一级目录下的bb文件夹
cd ../bb
```

- 列出文件列表：ls ll

Linux下，以.开头的文件都是隐藏文件

```
格式：ls[参数] [路径或文件名]
//列出当前目录下所有文件
ls
//显示所有文件或目录（包含隐藏文件）
ls -a
//列出文件详细信息
ls -l （缩写成ll）
```

- 创建目录和删除目录

```
//创建一级目录，第二个参数为文件夹名称
mkdir app
//一次性创建多层文件夹（级联）
mkdir -p app/test

//删除空目录
rmdir app
//强制删除非空目录
rm -rf app
```

- 文件查看

```
//一次性显示文件的所有内容（有大小限制，内容特别多不适合）
cat yum.conf

//more，一般内容多时使用，每次会显示一屏的内容
//按一次回车，显示一行内容。按一个空格，显示下一屏内容
//如果想退出，按Q键进行退出。或者ctrl+c也可以退出。
more yum.conf

//less，和more类似
//按回车一行行显示，按回车一屏一屏显示
//按键盘上、下键可以滚动列表（比more多了这个）
//按Q或ctrl+c退出
less yum.conf

//tail -n xxx，显示文件内容最后的第几行，例如10就是只显示最后10行
//一般查看日志文件时常用，一般看最新的日志内容
tail -10 yum.conf

//（重要）tail命令的-f参数，可以动态更新文件，例如当日志文件不断更新变化时，终端会跟随更新显示
tail -f yum.conf
```

## 文件操作

- 文件删除

```
//删除文件，会询问是否确认删除，y确认删除，n不删除
rm a.txt

//不询问，直接删除
rm -f a.txt

//递归删除（除了可以删除文件，还可以删除文件夹，作用类似rmdir xxx）
rm -r a.txt
//不询问递归删除（慎用！）
rm -rf a.txt

//删除所有文件
rm -rf *

//自杀，删除root目录下的所有文件和文件夹（递归删除）
rm -rf /*
```

- 文件拷贝

```
//拷贝a.txt文件为b.txt
cp a.txt b.txt
//拷贝文件到指定目录下，如将a.txt文件，拷贝到test目录内的app目录下
cp a.txt test/app
//将a.txt拷贝到上一层目录中
cp a.txt ../
```

- 文件移动

```
//将a.txt文件移动到上一层目录中
mv a.txt ../
//将a.txt文件移动到test目录下，并将文件名改为b.txt
mv a.txt test/b.txt

//将文件重命名，将a.txt文件重命名为b.txt
mv a.txt b.txt
```

- 打包和解压

```
//参数
-c：创建一个新tar文件
-v：显示运行过程的信息
-f：指定文件名
-z：调用gzip压缩命令进行压缩
-t：查看压缩文件的内容
-x：解开tar文件

//打包
tar –cvf xxx.tar ./*
//打包并且压缩
tar –zcvf xxx.tar.gz ./*

//解压不带压缩的tar文件
tar –xvf xxx.tar
//解压一个带压缩的tar.gz文件，如不带后面的路径，则会解压到当前路径
tar -zxvf xxx.tar.gz ./dir
//解压一个带压缩的tar.gz文件，并解压到指定目录，-C参数后+目录
tar -zxvf xxx.tar.gz -C /usr/aaa
```

- 查找文件

```
//find命令，格式：find 查找目录 -以什么方式 关键字

//从根目录下，查找文件名称是以ins开头的文件
find / -name “ins*”
//从
find / -name “ins*” –ls
//查找用户itcast的文件
find / –user itcast –ls
//查找用户itcast的目录
find / –user itcast –type d –ls
//查找权限是777的文件
find /-perm -777 –type d-ls
```

- 查找文件内容（查找文件里符合条件的字符串）

```
//grep命令，用法: grep 查找目录 查找关键字

//在文件中查找lang关键字
grep lang anaconda-ks.cfg
//加上--color参数（注意是2个杠），进行高亮显示关键字
grep lang anaconda-ks.cfg --color

//-A参数，表示After，可以显示下一行
grep lang anaconda-ks.cfg --color -A
//-B参数，表示Before，可以显示上一行
grep lang anaconda-ks.cfg --color -B
```

- 其他常见命令

```
//显示当前目录
pwd
//创建一个空文件
touch a.txt
//清屏，或者ctrl+l
clear
```

- vi和vim编辑器

vim相当于vi的升级版，功能更强大。
有3种模式，分别为命令行模式（只能看）、插入模式（可操作文件内容）、底行模式（）

```
//1.切换到命令行模式，按esc键

//2.切换到插入模式，按i、o、a键，解释：
	i 在当前位置前插入
	I 在当前行首插入
	a 在当前位置后插入
	A 在当前行尾插入
	o 在当前行之后插入一行

//3.切换到底行模式，按:键

//打开文件
vim server.xml
//退出，如果有内容修改，则会报错，这时应该使用q!
esc -> :q
//保存并退出
esc -> :wq
//不保存，直接退出
esc -> :q!
//查找指定内容
:/keyword

//快捷键
dd – 快速删除一行
yy - 复制当前行
nyy - 从当前行向后复制几行
p - 粘贴
R – 替换
```

- 重定向输出>和>>

将命令的输出，或文件内容，重定向输出到指定的文件中

```
//重定向输出，覆盖原有内容；>> 重定向输出，又追加功能。示例：
//将输出定向到a.txt中
cat /etc/passwd > a.txt
//输出并且追加
cat /etc/passwd >> a.txt
//将ifconfig命令的输出，重定向到aa.txt文件中
ifconfig > aa.txt
```

- 系统管理命令

```
//ps命令，正在运行的某个进程的状态
//查看所有进程
ps -ef
//查找某一进程，例如：查找包含java的进程
ps -ef | grep ssh
//杀掉2868编号的进程
kill 2868
//强制杀死进程
kill -9 2868
```

- 管道 |

管道是Linux命令中重要的一个概念，其作用是**将一个命令的输出用作另一个命令的输入**

```
//分页查询帮助信息
ls --help | more
//查询名称中包含java的进程
ps –ef | grep java

//分页查看ifconfig命令的输出
ifconfig | more
cat index.html | more
ps –ef | grep aio
```

- Linux的权限命令

```
//格式
- rwx rw- rw-

	1.第一个，代表文件类型
		* -，代表是文件
		* d，代表是文件夹
		* l，代表是链接（类似快捷方式）

//权限分为10个部分，每个部分代表一种权限，除了第一个是文件类型外，从第二个开始，3个为一组。
//每一部分，都有3个字符组成，分别是，他们都有一个特定的数值来代表
	* r，read，读权限			4
	* w，write，写权限			2
	* x ，excute，可执行权限		1

- 第一部分，当前用户的所具有权限
- 第二部分，当前组内用户所具有的权限
- 第三部分，其他组所具有的权限

//修改权限（比较完整的写法，u代表当前用户，g代表组内用户，o代表其他组用户）
chmod u=rwx,g=rx,o=rx a.txt
//修改权限（更快捷的方式，用上面提到的数值相加的方式）

//1.全部都只可读
chmod 444
//2.全部可读可写
chmod 666
//3.当前用户可读、可写、可执行，其他组可读和可执行
chmod 755
//4.最大权限，可读、可写、可执行
chmod 777
```

## Linux上常用网络操作

- 主机名配置

```
//查看主机名
hostname
//修改主机名 重启后无效
hostname xxx
如果想要永久生效，可以修改/etc/sysconfig/network文件
```

- IP地址配置

```
//查看(修改)ip地址(重启后无效)
ifconfig
//修改ip地址
ifconfig eth0 192.168.12.22
//如果想要永久生效，修改 /etc/sysconfig/network-scripts/ifcfg-eth0文件

//网卡名称
DEVICE=eth0
//获取ip的方式(static/dhcp/bootp/none)
BOOTPROTO=static
//MAC地址
HWADDR=00:0C:29:B5:B2:69
//IP地址
IPADDR=12.168.177.129
//子网掩码
NETMASK=255.255.255.0
//网络地址
NETWORK=192.168.177.0
//广播地址
BROADCAST=192.168.0.255
//系统启动时是否设置此网络接口，设置为yes时，系统启动时激活此设备。
NBOOT=yes
```

- 网络服务管理

```
//查看指定服务的状态
service network status
//停止指定服务
service network stop
//启动指定服务
service network start
//重启指定服务
service network restart

//查看系统中所有后台服务
service --status–all
//查看系统中网络进程的端口监听情况
netstat –nltp

- 防火墙设置，防火墙根据配置文件/etc/sysconfig/iptables来控制本机的”出”、”入”网络访问行为。

//查看防火墙状态
service iptables status
//关闭防火墙
service iptables stop
//启动防火墙
service iptables start
//禁止防火墙自启
chkconfig  iptables off
```

## Linux上软件安装

- 安装JDK
- 安装MySQL
- 安装Tomcat
- 安装Redis
- 部署项目到Linux

### 安装JDK

1. 上传JDK到Linux的服务器

```
* 上传JDK
* 卸载open-JDK
# 查看jdk版本
java –version
# 查看安装的jdk信息
rpm -qa | grep java
# 卸载jdk
rpm -e --nodeps java-1.6.0-openjdk-1.6.0.35-1.13.7.1.el6_6.i686
rpm -e --nodeps java-1.7.0-openjdk-1.7.0.79-2.5.5.4.el6.i686
```

2. 在Linux服务器上安装JDK

```
* 通常将软件安装到/usr/local
* 直接解压就可以
tar –xvf  jdk.tar.gz  -C 目标路径  
```

3. 配置JDK的环境变量

```
① vi /etc/profile
② 在末尾行添加
#set java environment
JAVA_HOME=/usr/local/jdk/jdk1.7.0_71
CLASSPATH=.:$JAVA_HOME/lib.tools.jar
PATH=$JAVA_HOME/bin:$PATH
export JAVA_HOME CLASSPATH PATH
③保存退出
④source /etc/profile  使更改的配置立即生效
```
 
#### 部署项目到Linux

- 使用maven给项目打包，文件名xxx.war
- 将xxx.war上传到tomcat的webapps目录下
- 重启tomcat
- 导出mysql的表结构数据到linux上的mysql