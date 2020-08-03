# Vagrant配置Centos静态ip

vagrant 中使用的是public_network，而工作网络中，由于桥接了很多路由器，导致ip段位和本机的ip段位不在同一个局域网中

ifconfig之后的结果

```
[root@localhost network-scripts]# ifconfig
eth0      Link encap:Ethernet  HWaddr 08:00:27:5A:FB:02
          inet addr:10.0.2.15  Bcast:10.0.2.255  Mask:255.255.255.0
          inet6 addr: fe80::a00:27ff:fe5a:fb02/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:1269 errors:0 dropped:0 overruns:0 frame:0
          TX packets:780 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:113182 (110.5 KiB)  TX bytes:90192 (88.0 KiB)

eth1      Link encap:Ethernet  HWaddr 08:00:27:65:77:09
          inet addr:192.168.200.102  Bcast:192.168.200.255  Mask:255.255.255.0
          inet6 addr: fe80::a00:27ff:fe65:7709/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:783 errors:0 dropped:0 overruns:0 frame:0
          TX packets:20 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:49649 (48.4 KiB)  TX bytes:2040 (1.9 KiB)
```

主要修改`eth1`，在本机中是`/etc/sysconfig/network-scripts/ifcfg-eth1`，使用vi /etc/sysconfig/network-scripts/ifcfg-eth1，编辑文件

IPADDR是ip地址的配置

```
BOOTPROTO=static
ONBOOT=yes
DEVICE=eth1
IPADDR=192.168.1.119
GATEWAY=192.168.1.1
NETMASK=255.255.255.0
```

重启网络

```
sudo service network restart
```

编辑Vagrantfile文件，将public_network固定一个ip地址

```
config.vm.network "public_network", ip: "192.168.1.119"
```