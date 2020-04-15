#### Mac环境变量配置

虽然用Mac很久了，环境变量配置每次都去搜索，很是麻烦，现在记录一下。

- 打开环境变量配置

Mac的环境变量配置保存在共享目录同级的用户目录下（不知道这样讲是不是准确）的**.bash_profile**，或者说下载目录的上一级。

在该目录下，打开终端，输入如下命令，就可以打开该文件：

```
open .bash_profile
```

- 增加环境变量

1. 例如Android开发环境，需要在别的目录调用adb系列命令，就要添加Android目录，默认目录在Users目录下，后面的lizhi是电脑名，根据你的配置来改喔

```
export ANDROID_HOME=/Users/lizhi/Library/Android/sdk
export PATH=${PATH}:${ANDROID_HOME}/tools
export PATH=${PATH}:${ANDROID_HOME}/platform-tools
```

2. Python3开发环境配置，注意每个人安装的版本不一样，目录文件夹就不一样，后面的3.7.7要根据安装的具体版本改喔

```
export PATH=${PATH}:/usr/local/Cellar/python/3.7.7/bin
alias python="/usr/local/Cellar/python/3.7.7/bin/python3"
alias pip="/usr/local/Cellar/python/3.7.7/bin/pip3"
```

- 应用环境变量

改完文件，保存，并不代表应用，需要使用如下命令

```
source ./.bash_profile
```

- 验证

应用完毕后，在终端输入adb，能出现adb命令的帮助。或者python --version，能出现python3版本提示，就代表配置成功了。