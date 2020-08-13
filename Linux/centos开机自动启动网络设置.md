默认 centos 开机后不自动连接网络，一般我们需要开机时就连接，所以需要配置一下。

- 切换到root用户

```
su - root
```

- 切换到网路配置目录

````
cd /etc/sysconfig/network-scripts/
````

- 找到网卡配置文件，例如ifcfg-ens33（centos重写了网卡得命名规则，可能不是eth0，而是ifcfg-ens+一串数字）

```
ls
```

- 使用vim打开文件，将onboot选项设置为yes，默认为no

```
ONBOOT=yes
```

- :wq保存，重启系统即可