# linux 出现Name or service not known的解决方法

当配置完静态ip后，可能会ping不通公网的域名，例如www.baidu.com。

## 前置准备

### 确认基础配置

首先确认你配置了`BOOTPROTO`为static，`IPADDR`固定的ip地址，网关`GAREWAY`，这3个是必须的。

```
//打开配置文件
vi /etc/sysconfig/network-scripts/ifcfg-ens33

//确认是否有以下配置
BOOTPROTO=static
IPADDR=192.168.211.133
GAREWAY=192.168.211.2
```

### 重启网络服务

```
service network restart
```

### 测试

```
ping www.baidu.com
```

如果配置完后，:wq保存并写入，重启网络服务，如果还是ping 不通 baidu，则进行下面的DNS配置。

## DNS配置

### 打开配置文件

```
vi /etc/resolv.conf
```

### 添加DNS地址

```
nameserver 8.8.8.8
```

### 重启网络服务

```
service network restart
```

### 测试

```
ping www.baidu.com
```