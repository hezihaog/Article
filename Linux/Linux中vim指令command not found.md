# Linux中vim指令command not found

我的是centos，使用vim指令，提示command not found

## 查看系统是否安装完整vim

输入以下指令

```
rpm -qa|grep vim
```

如果已正确安装的话，会输出以下3行

```
vim-enhanced-7.0.109-7.el5
vim-minimal-7.0.109-7.el5
vim-common-7.0.109-7.el5
```

## 安装vim

- 如果少了其中一条，则可以单独安装，例如vim-enhanced

```
yum -y install vim-enhanced
```

- 如果条都没有返回，则可以一次性安装所有的

```
yum -y install vim*
```