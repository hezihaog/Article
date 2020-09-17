# Mac下安装RabbitMQ

## 步骤

### 更新homebrew

```
brew update
```

### 安装RabbitMQ

```
brew install rabbitmq
```

安装成功会输出以下信息

```
Man pages can be found in:
  /usr/local/opt/erlang/lib/erlang/man

Access them with `erl -man`, or add this directory to MANPATH.
==> rabbitmq
Management Plugin enabled by default at http://localhost:15672

Bash completion has been installed to:
  /usr/local/etc/bash_completion.d

To have launchd start rabbitmq now and restart at login:
  brew services start rabbitmq
Or, if you don't want/need a background service you can just run:
  rabbitmq-server
```

### 配置环境变量

RabbitMQ的命令被安装在`/usr/local/sbin`，我们想在任意目录下都可以访问它的命令，则需要配置环境变量。

修改`~/.bash_profile`文件，使用`vi ~/.bash_profile`命令打开文件，在末尾添加以下配置

```
export PATH=$PATH:/usr/local/sbin
```

马上刷新配置

```
source ~/.bash_profile
```

### 启动RabbitMQ

- 前台运行 `rabbitmq-server`
- 后台运行 `brew service start rabbitmq`

### 访问后台

启动后，浏览器访问`http://localhost:15672`，即可进入RabbitMQ的可视化后台，用户名和密码都是`guest`，登录后可在admin那一栏，添加自定义用户。