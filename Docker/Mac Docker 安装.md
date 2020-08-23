# Mac Docker 安装

## 安装

安装有2种，一种是命令安装，一种是下载dmg文件安装，2种都可以。

- 命令安装

```
brew cask install docker
```

- dmg下载

官方下载速度太慢，可使用阿里云的镜像地址下载

```
//docker官网下载
https://download.docker.com/mac/stable/Docker.dmg
//阿里云镜像
https://links.jianshu.com/go?to=http%3A%2F%2Fmirrors.aliyun.com%2Fdocker-toolbox%2Fmac%2Fdocker-for-mac%2F
```

## 配置网易源

官方的仓库速度太慢了，好在网易提供了镜像源

安装好后，点击顶部栏的Docker图标，选择Preferences => Docker Engine，复制粘贴以下json进去

```
{
  "registry-mirrors": [
    "https://hub-mirror.c.163.com"
  ],
  "experimental": false,
  "debug": true
}
```

保存并应用即可

## 检查

docker命令生效即可

```
docker -v
//输出
Docker version 19.03.12, build 48a66213fe
```