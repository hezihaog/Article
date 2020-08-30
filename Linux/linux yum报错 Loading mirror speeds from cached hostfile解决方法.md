# linux yum报错 Loading mirror speeds from cached hostfile 解决方法

一般是yum没有配置源，我们配置上163源就可以了。

配置前，先将网络连通，例如刚安装好centos，网络默认是不会开机启动的，就需要配置一下开机启动，一般要连同静态ip一起配。

只要网络能 ping www.baidu.com，能通就可以了。

将下列每一句命令都执行即可，再执行yum相关命令，就发现不会报错了

```
cd /etc/yum.repos.d
mv CentOS-Base.repo CentOS-Base.repo.backup
wget http://mirrors.163.com/.help/CentOS6-Base-163.repo
mv CentOS6-Base-163.repo CentOS-Base.repo
yum clean all
```