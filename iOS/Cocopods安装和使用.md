#### Cocopods安装和使用

Java中的包管理有Maven，Android中有Gradle，而iOS常用的包管理工具则有Cocopods，如果不使用包管理，则需要手动将库文件手动拷贝，并不好管理包依赖和版本升级。本篇来总结一下Cocopods的安装和使用。

#### Ruby安装

Cocopods是使用Ruby语言编写的，所以要使用Cocopods，需要先安装Ruby环境。

#### 检查Ruby是否安装

- 终端中，查看Ruby版本，如果已安装，则可以跳过

```
ruby -v
```

- 输出以下信息则是安装了

```
ruby 2.3.7p456 (2018-03-28 revision 63024) [universal.x86_64-darwin18]
```

- 如果是输出command not found，则为未安装。

```
-bash: ruby -v:command not found
```

#### 安装Ruby包管理rvm

rvm（ruby version manager），先安装这个rvm包管理，使用rvm包管理去安装Ruby。

#### 检查rvm是否安装

```
rvm -v
```

- 输出以下信息则是安装了

```
rvm 1.29.9 (latest) by Michal Papis, Piotr Kuczynski, Wayne E. Seguin [https://rvm.io]
```

- 如果是输出command not found，则为未安装。

```
-bash: rvm -v:command not found
```

#### 安装rvm

- 终端输入以下命令，等待几分钟即可。

```
curl -L https://get.rvm.io | bash -s stable
```

- 更新环境变量，回车后，什么都不输出则为正常

```
source ~/.rvm/scripts/rvm
```

#### 查找Ruby可用版本

- 终端输入：rvm list known查找Ruby的可用版本

```
rvm list known
```

- 输出以下信息

```
# MRI Rubies
[ruby-]1.8.6[-p420]
[ruby-]1.8.7[-head] # security released on head
[ruby-]1.9.1[-p431]
[ruby-]1.9.2[-p330]
[ruby-]1.9.3[-p551]
[ruby-]2.0.0[-p648]
[ruby-]2.1[.10]
[ruby-]2.2[.10]
[ruby-]2.3[.8]
[ruby-]2.4[.6]
[ruby-]2.5[.5]
[ruby-]2.6[.3] //最新正式版
[ruby-]2.7[.0-preview1] //测试版，一般不安装该版本
ruby-head
```

- 安装Ruby，我这里安装最新的2.6.3，你也可以选择别的版本安装

```
rvm install 2.6.3
```

- 设置刚才安装2.6.3版本为默认选用

```
rvm use 2.6.3 --default
```

- 再次检查Ruby版本，能输出版本信息即可

```
ruby -v
```

#### 正式安装Cocopods

- 安装好Ruby后，开始正式安装我们的Cocopods，终端输入以下命令

```
sudo gem install -n /usr/local/bin cocoapods
```

- 如果安装了多个版本的Xcode，需要再加以下命令选择版本

```
sudo xcode-select --switch /Applications/Xcode.app
```

- 安装本地库

```
pod setup
```

- 安静，等待安装完毕即可

```
Setting up CocoaPods master repo
$ /usr/bin/git clone https://github.com/CocoaPods/Specs.git master --progress
Cloning into 'master'...
remote: Counting objects: 1879515, done.        
remote: Compressing objects: 100% (321/321), done.        
Receiving objects:  21% (404525/1879515), 73.70 MiB | 22.00 KiB/
```

- 接下来，我们来搜索一下AFNetworking，能搜索到则证明没问题啦

```
pod search AFNetworking

-> AFNetworking (3.2.1)
   A delightful iOS and OS X networking framework.
   pod 'AFNetworking', '~> 3.2.1'
   - Homepage: https://github.com/AFNetworking/AFNetworking
   - Source:   https://github.com/AFNetworking/AFNetworking.git
   - Versions: 3.2.1, 3.2.0, 3.1.0, 3.0.4, 3.0.3, 3.0.2, 3.0.1, 3.0.0,
   3.0.0-beta.3, 3.0.0-beta.2, 3.0.0-beta.1, 2.6.3, 2.6.2, 2.6.1, 2.6.0, 2.5.4,
   2.5.3, 2.5.2, 2.5.1, 2.5.0, 2.4.1, 2.4.0, 2.3.1, 2.3.0, 2.2.4, 2.2.3, 2.2.2,
   2.2.1, 2.2.0, 2.1.0, 2.0.3, 2.0.2, 2.0.1, 2.0.0, 2.0.0-RC3, 2.0.0-RC2,
   2.0.0-RC1, 1.3.4, 1.3.3, 1.3.2, 1.3.1, 1.3.0, 1.2.1, 1.2.0, 1.1.0, 1.0.1,
   1.0, 1.0RC3, 1.0RC2, 1.0RC1, 0.10.1, 0.10.0, 0.9.2, 0.9.1, 0.9.0, 0.7.0,
   0.5.1 [master repo]
   - Subspecs:
     - AFNetworking/Serialization (3.2.1)
     - AFNetworking/Security (3.2.1)
     - AFNetworking/Reachability (3.2.1)
     - AFNetworking/NSURLSession (3.2.1)
     - AFNetworking/UIKit (3.2.1)
```

#### Cocopods基本使用

- 新建一个Xcode功能，cd命令到项目根目录下，创建Podfile文件

```
pod init
```

- 根目录下会见到生成了一个Podfile文件，我们打开这个文件，在文件中添加：(其中targetName为你的项目名，需要你手动更改一下！)

```
platform :ios, '8.0'
target "targetName" do
pod 'AFNetworking'
end
```

- 安装依赖

```
pod install
```

- 等待安装完成即可

```
Analyzing dependencies
Downloading dependencies
Using AFNetworking (3.2.1)
Generating Pods project
Integrating client project
Sending stats
Pod installation complete! There are 1 dependencies from the Podfile and 1 total pods installed.
```

- 新增的.xcworkspace工程文件

现在我们回到项目目录，会发现多了一个.xcworkspace的工程文件，后续我们打开工程是打开这个. xcworkspace文件，而不是之前的.xcodeproj文件，要记住了！

- 判断是否安装成功

我们导入AFNetworking头文件，编译成功即为安装成功！

```
#import "AFNetworking.h"
```

- 显示指定依赖版本

上面我们是直接指定了pod ‘AFNetworking’，它的意思是每次都获取最新版本，如果需要指定目标版本，使用一下语法即可。

```
pod ‘AFNetworking’             //不显式指定依赖库版本，表示每次都获取最新版本
pod ‘AFNetworking’,  ‘2.0’     //只使用2.0版本
pod ‘AFNetworking’, ‘>2.0′     //使用高于2.0的版本
pod ‘AFNetworking’, ‘>=2.0′    //使用大于或等于2.0的版本
pod ‘AFNetworking’, ‘<2.0′     //使用小于2.0的版本
pod ‘AFNetworking’, ‘<=2.0′    //使用小于或等于2.0的版本
pod ‘AFNetworking’, ‘~>0.1.2′  //使用大于等于0.1.2但小于0.2的版本，相当于>=0.1.2并且<0.2.0
pod ‘AFNetworking’, ‘~>0.1′    //使用大于等于0.1但小于1.0的版本
pod ‘AFNetworking’, ‘~>0′      //高于0的版本，写这个限制和什么都不写是一个效果，都表示使用最新版本
```

#### 依赖库升级

- 后续如果依赖库升级版本，我们使用以下命令即可

```
pod update
```

#### 总结

- Cocopods安装还算简单，可能由于不可抗力原因导致安装较慢，耐心等待即可。
- 通过Cocopods，我们可以在github上找到常用的第三方依赖库，例如Masonry、SDWebImage、MJRefresh等，又可以愉快的玩耍啦！