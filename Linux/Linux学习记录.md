# Linux的目录结构

- /bin（重点）
    - 是Binary的缩写, 这个目录存放着最经常使用的命令
    
- /sbin
    - s就是Super User的意思，这里存放的是系统管理员使用的系统管理程序
    
- /home（重点）
    - 存放普通用户的主目录，在Linux中每个用户都有一个自己的目录，一般
      该目录名是以用户的账号命名的
      
- /root（重点）
    - 该目录为系统管理员，也称作超级权限者的用户主目录
    
- /lib
    - 系统开机所需要最基本的动态连接共享库，其作用类似于Windows里的DLL文件。几
      乎所有的应用程序都需要用到这些共享库
      
- /lost+found
    - 这个目录一般情况下是空的，当系统非法关机后，这里就存放了一些文件。
    
- /etc（重点）
    - 所有的系统管理所需要的配置文件和子目录 my.conf
    
- /usr
    - 这是一个非常重要的目录，用户的很多应用程序和文件都放在这个目录下，类似与
      windows下的program files目录
      
- /boot（重点）
    - 存放的是启动Linux时使用的一些核心文件，包括一些连接文件以及镜像文件

- /proc（和Linux内核相关，一般不动）
    - 这个目录是一个虚拟的目录，它是系统内存的映射，访问这个目录来获取系统信息
    
- /srv（和Linux内核相关，一般不动）
    - service缩写，该目录存放一些服务启动之后需要提取的数据

- /sys（和Linux内核相关，一般不动）
    - service缩写，该目录存放一些服务启动之后需要提取的数据
    
- /tmp
    - 这个目录是用来存放一些临时文件的
    
- /dev
    - 类似于windows的设备管理器，把所有的硬件用文件的形式存储
    
- /media（重点）
    - linux系统会自动识别一些设备，例如U盘、光驱等等，当识别后，linux
      会把识别的设备挂载到这个目录下
      
- /mnt
    - 系统提供该目录是为了让用户临时挂载别的文件系统的，我们可以将外部的存储挂载在/mnt/上，然后进入该目录就可以查看里的内容了。 d:/myshare
    
- /opt
    - 这是给主机额外安装软件所摆放的目录。如安装ORACLE数据库就可放到该目录下。
      默认为空
      
- /usr/local（重点）
    - 这是另一个给主机额外安装软件所安装的目录。一般是通过编译源码方式安装的程序
    
- /var（重点）
    - 这个目录中存放着在不断扩充着的东西，习惯将经常被修改的目录放在这个目录下。
      包括各种日志文件
      
- /selinux
    - SELinux是一种安全子系统,它能控制程序只能访问特定文件
    
# Linux目录总结

1. Linux的目录下有且只有一个根目录 /
2. Linux的各个目录存放的内容是规划好的，不要乱放文件
3. Linux是以文件的形式来管理我们的设备，因此Linux系统中，一切皆为文件
4. Linux的各个文件目录下，存放什么内容，必须有一个认识
5. 学习完后，脑海应该有一棵Linux目录树

# Linux vi和vim 编辑器

## vi和vim的三种常见模式

- 正常模式
    > 在正常模式下，我们可以使用快捷键。 以 vim 打开一个档案就直接进入一般模式了(这是默认的模式)。在这个模式中，你可以使用『上 下左右』按键来移动光标，你可以使用『删除字符』或『删除整行』来处理档案内容， 也可以使用 『复制、贴上』来处理你的文件数据

- 插入模式（编辑模式）
    > 在模式下，程序员可以输入内容。 按下 i, I, o, O, a, A, r, R 等任何一个字母之后才会进入编辑模式, 一般来说按 i 即可

- 命令行模式
    > 在这个模式当中， 可以提供你相关指令，完成读取、存盘、替换、离开 vim 、显示行号等的动 作则是在此模式中达成的！
  
  ## 常用快捷键
  
- yy
  - 普通模式下，拷贝当前行。粘贴则按p。
  
- 5yy
    - 普通模式下，拷贝光标后的5行（所以先要移动到要复制的那5行之前的一行先），粘贴也是按p。
    
- dd
    - 普通模式下，删除当前行
    
- 5dd
    - 一次性删除光标下的5行
    
- 查询
    - 命令行模式下，输入/，再输入要查找的字符。例如/hello，就是查找关键字为hello的文本，有多个结果时，按n键移动到下一个。

- 显示行号和取消显示行号（行数很多时使用）
    - 命令行模式下，输入:，再输入set nu显示行号。要取消显示行号则输入:set nonu
    
- 跳转到文本的最末行和最首行
    - 正常模式下，输入G，跳转到最末行。输入gg，则跳转到最首行。

- 回退、撤销功能
    - 在正常模式下，输入u，即可回退上次输入的内容。
    
- 让光标移动到指定行
    - 普通模式下，输入行号（注意：不会有输入产生!），再按shit + g，即可让光标移动，一般会先让行号显示，才知道有多少行。
    
# Linux 系统相关

## 关机和重启命令

- shutdown命令
    - shutdown -h now：表示立即关机
    - shutdown -h 1：表示1分钟过后关机
    - shutdown -r now：表示立即重启
    
- halt
    - 直接使用即可，效果等价于关机
    
- reboot
    - 重启系统
    
- sync
    - 把内存中的数据，同步到磁盘上，执行后是看不到效果的（避免数据丢失，建议在关机之前都执行）
    
## 用户的登录和注销

## 基本介绍

- 登录时尽量少用 root 帐号登录，因为它是系统管理员，最大的权限，避免操作失误。可以利 用普通用户登录，登录后再用”su - 用户名’命令来切换成系统管理员身份。
- 在提示符下输入 logout 即可注销用户。

## 使用细节

- logout 注销命令，在图形界面上的终端运行是无效的，在运行级别3下有效，就是远程登录下进行logout命令注销是有效的

# Linux 用户管理

- 概念：
    - 每个用户，可以有多个组，用组来管理用户，以及用户的权限。
    - 每个用户，有自己一个家目录，每个用户登录时，都会自动进入自己用户的家目录下。用家目录，来规定用户在自己的目录下用。

- 说明：
    - Linux 系统是一个多用户多任务的操作系统，任何一个要使用系统资源的用户，都必须首先向 系统管理员申请一个账号，然后以这个账号的身份进入系统。 
    - Linux 的用户需要至少要属于一个组。
    
## 添加用户

- 语法
    - useradd [选项] 用户名，创建用户
    - useradd -d 目录名 用户名，例如：useradd -d /home/tiger/ xh，意思时创建一个xh用户，并指定它的家目录为 /home/tiger，注意目录不要事先创建，必须目录不存在才行

- 细节
    - 当用户创建后，默认会创建一个和用户名同名的家目录，在/home/目录下可以见到
    
## 指定或修改密码

- 语法
    - passwd 用户名，给指定用户设置密码，需要输入2次密码进行确认
    
## 删除用户

- 语法
    - userdel 用户名
    
- 案例
    - 删除xm用户，但保留家目录。userdel xm
    - 删除xm用户，家目录也一起删除，使用-r参数。userdel -r xm

- 细节
    - 一般删除用户时，不会删除家目录
    
## 查询用户

- 基本语法
    - id 用户名
    
- 案例
    - 查询root用户的用户信息：id root，输出uid=(root) gid=0(root) 组=0(root)。表示root用户的id为0，组的id为0，组名为root。
    - 查询xq用户的用户信息：id xq，输出uid=(xq) gid=501(xq) 组=501(xq)
    
- 注意
    - 如果用户不存在，则输出无此用户
    
## 切换用户

当我们在操作Linux时，如果当前用户的权限不够，则可以通过 su - 指令，切换到高权限用户，比如root

- 语法
    - su - 用户名

- 案例
    - 创建一个用户zf，指定密码，然后切换到zf用户
        - useradd zf，创建用户
        - passwd zf，指定用户的密码
        - su - zf，切换到zf用户
        - exit，退出zf用户，切换到原来的用户，例如root

- 细节
    - 从权限高的用户，切换到权限低的用户，不需要密码，反之则需要。
    - 当需要原来的用户时，使用exit指令。
    
## 查看当前用户

- 语法
    - whoami
    
# 用户组

类似于角色，系统可以对有共性的多个用户进行统一的管理

## 新增组（创建组）

- 语法
    - groupadd 组名
    
- 增加用户时，就指定组
    - useradd -g 用户组 用户名

- 案例
    - 增加zwj用户，并且直接指定到wudang组
        - groupadd wudang，新建wudang组
        - useradd -g wudang zwj，新增zwj用户，并指定为wudang组
        - id zwj，查询zwj用户的信息，输出uid=503(zwj) gid=503(wudang) 组=503(wudang)
    
##删除组

- 语法
    - groupdel 组名
    
## 修改用户的组

- 语法
    - usermod -g 用户组 用户名
    
- 案例
    - 创建一个shaolin组，将zwj修改到shaolin组，不在wudang组
        - groupadd shaolin，新增shaolin组
        - usermod -g shaolin zwj，将zwj的组修改为shaolin
        - id zwj，查看zwj的用户信息，uid=503(zwj) gid=504(shaolin) 组=504(shaolin)
        
## /etc/passwd 文件

用户（user）的配置文件，记录用户的各种信息

- root:x:0:0:root:/root:/bin/bash
- zf:x:502:502::/home/zf:/bin/bash
- zwj:x:503:504::/home/zwj:/bin/bash

每行的含义：用户名:口令:用户标识号:组标识号:注释性描述:主目录:登录 Shell

## /etc/shadow 文件

口令的配置文件

- root:$6$PO73HfawEuegVIfS$OxGAx6TzOYSwImLYbyqq.ERoDLZHeAw9OYLseV3H99rTbffNLq/ssiqLG87kX7nSVlAH.qPDX4vT1XAUAOvZc/:18486:0:99999:7:::
- zf:$6$41mu2iCP$jerQRpDcmqQC7XvnQawEu6FaNSCRv8Ot5MlW30q9ai7YKecGHH3.WnLk0N52Q.pc8xbI/SxWJHtqFmSdVSj2k/:18486:0:99999:7:::
- zwj:!!:18486:0:99999:7:::
每行的含义：登录名:加密口令:最后一次修改时间:最小时间间隔:最大时间间隔:警告时间:不活动 时间:失效时间:标志

## /etc/group 文件

组(group)的配置文件，记录 Linux 包含的组的信息

- xm:x:500:
- xq:x:501:
- zf:x:502:
- wudang:x:503:
- shaolin:x:504:

每行含义：组名:口令:组标识号:组内用户列表

# Linux 实用指令

## 指定运行级别

- 0 ：关机
- 1 ：单用户【找回丢失密码】
- 2：多用户状态没有网络服务
- 3：多用户状态有网络服务
- 4：系统未使用保留给用户
- 5：图形界面
- 6：系统重启

1. 常用运行级别是 3 和 5
2. 要修改默认的运行级别可改文件 /etc/inittab 的 id:5:initdefault:这一行中的数字

### 切换到指定的运行级别的指令

- 语法
    - init [012356]，注意4是保留的，不能使用
    
- 案例
    - 通过 init 来切换不同的运行级别，比如从 5 切换到 3（从图形界面切换到命令行界面） ， 然后关机
    - init 3，切换到命令行界面
    - init 5，切换到图形界面
    - init 0，关机

- 面试题
    - 如何找回丢失的root密码
    - 思路：进入到单用户模式，修改root密码。（因为进入到单用户模式，root不需要密码就可以登录）
    - 步骤：开机->在引导时输入 回车键-> 看到一个界面输入 e（edit编辑） -> 看到一个新的界面，选中第二行（编辑 内核）在输入 e -> 在这行最后输入 1（设置运行级别为1，就是单用户模式） ,再输入 回车键 -> 再次输入 b（表示boot重新引导） ,这时就会进入到单用户模式。 这时，我们就进入到单用户模式，使用 passwd 指令来修改 root 密码。
    
- 练习
    - 请设置我们的 运行级别，linux 运行后，直接进入到 命令行界面。（即进入到 3 运行级别 vim /etc/inittab）
    - 答：将 id:5:initdefault:这一行中的数字, 5 这个数字改成对应的运行级别即可。
    
## 帮助指令

当我们对某个指令不熟悉时，我们可以使用 Linux 提供的帮助指令来了解这个指令的使用方法。

### man命令 获取帮助信息

- 语法
    - man [命令或配置文件]（功能描述：获得帮助信息）

- 案例
    - 使用man命令查询ls命令的帮助信息，man ls
    
### help命令

- 语法
    - help 命令 （功能描述：获得 shell 内置命令的帮助信息）
    
- 案例
    - 使用help命令，查询cd命令的帮助信息，help cd
    
## 文件目录类指令

### pwd 指令

- 语法
    - pwd (功能描述：显示当前工作目录的绝对路径)
    
### ls 指令

- 语法
    - ls [选项] [目录或是文件]

- 常用选项
    - `-a` ：显示当前目录所有的文件和目录，包括隐藏的。
    - `-l` ：以列表的方式显示信息
    
- 案例
    - 以列表形式，显示当前目录的所有文件，包括隐藏的：输入ls -al
    
### cd 指令

- 语法
    - cd [参数] (功能描述：切换到指定目录)
    
- 常用参数
    - cd ~ 或者 cd ：回到自己的家目录
    - cd .. 回到当前目录的上一级目录
    
- 案例
    - 使用绝对路径切换到 root 目录：cd /root
    - 当前在/usr/lib，使用相对路径到/root 目录（上一层目录是user，再上一层才是根目录）：../../root
    - 回到当前目录的上一级目录：cd ..
    - 回到家目录：cd ~ 或者直接cd，就回到当前登录用户的家目录
    
### mkdir 指令

mkdir 指令用于创建目录(make directory)

- 语法
    - mkdir [选项] 要创建的目录
    
- 常用选项
    - `-p` ：创建多级目录
    
- 案例
    - 创建一个目录 /home/dog，输入：mkdir /home/dog
    - 创建多级目录 /home/animal/tiger，输入：mkdir -p /home/animal/tiger
    
### rmdir 指令

rmdir 指令删除空目录

- 语法
    - rmdir [选项] 要删除的空目录
    
- 细节
    - rmdir 删除的是空目录，如果目录下有内容时无法删除的。
    - 提示：如果需要删除非空目录，需要使用 rm -rf 要删除的目录。

- 案例
    - 删除一个目录 /home/dog，输入：rmdir /home/dog
    - 删除一个目录，目录内有文件 /home/animal/tiger，输入：rm -rf /home/animal/tiger
    
### touch 指令

touch 指令创建空文件

- 语法
    - touch 文件名称
    
- 细节
    - touch后面可以跟多个文件名，代表一次性创建多个文件
    
- 案例
    - 创建一个hello.txt的空文件，输入：touch hello.txt
    - 一次性创建2个空文件，ok1.text和ok2.text。输入：touch ok1.txt ok2.txt
    
### cp 指令[重要]

cp 指令拷贝文件到指定目录

- 语法
    - cp [选项] source dest，source为源目录，dest为目标目录
    
- 常用选项
    - `-r` ：递归复制整个文件夹

- 细节
    - 当复制时，有文件相同，会询问是否覆盖，将 cp 改为 \cp 即代表为强制覆盖，则不询问了

- 案例
    - 将 /home/aaa.txt 拷贝到 /home/bbb 目录下[拷贝单个文件]，输入：
        - touch aaa.txt，创建aaa.txt文件
        - mkdir bbb，创建bbb文件夹
        - cp aaa.txt bbb/，拷贝aaa.txt文件到当前目录下的bbb目录内
    - 将/home/test 整个目录拷贝到 /home/zwj 目录
        - cp -r test/ zwj/，因为test目录下有文件，拷贝整个test目录到zwj目录时是不可以的，需要加上-r参数进行递归拷贝

### rm 指令

rm 指令移除【删除】文件或目录

- 语法
    - rm [选项] 要删除的文件或目录
    
- 常用选项
    - `-r` ：递归删除整个文件夹
    - `-f` ： 强制删除不提示
    
- 案例
    - 将 /home/aaa.txt 删除，输入：rm aaa.txt（会询问是否删除），不想询问则使用：rm -f aaa.txt
    - 递归删除整个文件夹 /home/bbb，输入：rm -rf bbb/
    
### mv 指令

mv 移动文件与目录或重命名

- 语法
    - mv oldNameFile newNameFile (功能描述：重命名）
    - mv /temp/movefile /targetFolder (功能描述：移动文件)
    
- 案例
    - 将 /home/aaa.txt 文件 重新命名为 pig.txt，输入：mv aaa.txt pig.txt
    - 将 /home/pig.txt 文件 移动到 /root 目录下，输入：mv pic.txt /root/
    
- 细节
    - mv指令，如果写的是文件名，则当为是重命名
    - 而如果写的是目录，并且没有存在，则当为移动
    
### cat 指令

cat 查看文件内容，是以只读的方式打开

- 语法
    - cat [选项] 要查看的文件

- 常用选项
    - `-n` ：显示行号
    
- 案例
    - 用cat命令，浏览/etc/profile 文件内容，并显示行号，输入cat -n /etc/profile
    
- 细节
    - cat 只能浏览文件，而不能修改文件，为了浏览方便，一般会带上 管道命令 | more
    - 格式：cat 文件名 | more，按空格进行翻页查看下一页
    
### more 指令

more 指令是一个基于 VI 编辑器的文本过滤器，它以全屏幕的方式按页显示文本文件的内容。more 指令中内置了若干快捷键，详见操作说明

- 语法
    - more 要查看的文件
    
- 案例
    - 采用 more 查看 /etc/profile 文件，输入：more /etc/profile
    
- 操作功能说明

    - `空格键`：代表向下翻页一页
    - `Enter回车键`：代表向下翻一行
    - `q`：表示立即离开more，不再显示文件内容
    - `Ctrl+f`：向下滚动一屏
    - `Ctrl+b`：返回上一屏
    - `=`：输出当前行的行号
    - `:f`：输出文件名和当前行号

### less 指令

less 指令用来分屏查看文件内容，它的功能与 more 指令类似，但是比 more 指令更加强大，支持 各种显示终端。less 指令在显示文件内容时，并不是一次将整个文件加载之后才显示，而是根据显示 需要加载内容，对于显示大型文件具有较高的效率。

- 语法
    - less 要查看的文件
    
- 操作功能说明

    - 空白键：向下翻动一页
    - pagedown：向下翻动一页
    - pageup：向上翻动一页
    - 字串/：向下搜寻字串的的功能，n：向下查找，N：向上查找
    - ?子串：向上搜寻字串的的功能，n：向下查找，N：向上查找
    - q：离开less这个程序
    
### > 指令 和 >> 指令

- `>` 指令 和 `>>` 指令
    - `>` 输出重定向 : 会将原来的文件的内容覆盖
    - `>>` 追加： 不会覆盖原来文件的内容，而是追加到文件的尾部。
    
- 语法
    - ls -l >文件 （功能描述：列表的内容写入文件 a.txt 中（覆盖写））
    - ls -al >>文件 （功能描述：列表的内容追加到文件 aa.txt 的末尾）
    - cat 文件 1 > 文件 2 （功能描述：将文件 1 的内容覆盖到文件 2）
    - echo "内容" >> 文件（功能描述：将输入的内容，追加到文件）
    
- 案例
    - 将 /home 目录下的文件列表 写入到 /home/info.txt 中。输入：ls -l /home/ > /home/info.txt
    - 将当前日历信息 追加到 /home/mycal 文件中。使用cal指令可以获取日历。输入：cal >> /home/mycal
    
### echo 指令

echo 输出内容到控制台。

- 语法
    - echo [选项] [输出内容]
    
- 案例
    - 使用 echo 指令输出环境变量,输出当前的环境路径，输入：echo $PATH
        - 输出：/usr/lib/qt-3.3/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin
    - 使用 echo 指令输出 hello,world!，输入：echo "hello,world!"
    
### head 指令

head 用于显示文件的开头部分内容，默认情况下 head 指令显示文件的前 10 行内容

- 语法
    - head 文件 (功能描述：查看文件头 10 行内容)
    - head -n 5 文件 (功能描述：查看文件头 5 行内容，5 可以是任意行数)
    
- 案例
    - 查看/etc/profile 的前面 5 行代码，输入：head -n 5 /etc/profile
    
### tail 指令

tail 用于输出文件中尾部的内容，默认情况下 tail 指令显示文件的后 10 行内容。

- 语法
    - tail 文件 （功能描述：查看文件后 10 行内容）
    - tail -n 5 文件 （功能描述：查看文件后 5 行内容，5 可以是任意行数）
    - tail -f 文件 （功能描述：实时追踪该文档的所有更新，工作经常使用）
    
- 案例
    - 查看/etc/profile 最后 5 行的代码，输入：tail -n 5 /etc/profile
    - 实时监控 mydate.txt , 看看到文件有变化时，是否看到， 实时的追加日期
        - 输入：tail -f mydate.txt，监听文件内容变化
        - 再另外一个终端追加日期给mydate.txt文件，输入：ls -l >> mydate.txt
        
### ln 指令

软链接也叫符号链接，类似于 windows 里的快捷方式，主要存放了链接其他文件的路径

- 语法
    - ln -s [原文件或目录] [软链接名] （功能描述：给原文件创建一个软链接）
    
- 案例
    - 在/home 目录下创建一个软连接 linkToRoot，连接到 /root 目录，输入：ln -s /root linkToRoot
    - 删除软连接 linkToRoot，输入：rm -rf linkToRoot

- 细节
    - 进入到软连接目录时，用pwd还是显示软连接目录，而不会显示连接到的目录
    - 删除软连接时，不要加上/，否则会删除真实目录内的内容。也有可能会提示设备或资源忙，一定要去掉/。
    
### history 指令

查看已经执行过历史命令,也可以执行历史指令

- 语法
    - history （功能描述：查看已经执行过历史命令）
    
- 案例
    - 显示所有的历史命令，输入：history
    - 显示最近使用过的 10 个指令，输入：history 10，相当于后10个指令
    - 执行历史编号为 5 的指令，输入：!5，一般会先用history查询指令，再通过!号去执行
    
## 时间日期类

### date 指令-显示当前日期

- 语法
    - date （功能描述：显示当前时间）
    - date +%Y （功能描述：显示当前年份）
    - date +%m （功能描述：显示当前月份）
    - date +%d （功能描述：显示当前是哪一天）
    - date "+%Y-%m-%d %H:%M:%S"（功能描述：显示年月日时分秒）
    
- 案例
    - 显示当前时间信息，输入：date，输出：2020年 08月 13日 星期四 17:01:22 CST
    - 显示当前时间年月日，输入：date "+%Y-%m-%d"，输出：2020-08-13。中间是连接符，可以随意使用，例如使用空格，输入：data "+%Y %m %d"，输出：2020 08 13
    - 显示当前时间年月日时分秒，输入：date "+%Y-%m-%d %H:%M:%S"，输出：2020-08-13 17:08:52
    
### date 指令-设置日期

- 语法
    - date -s 字符串时间（和之前不一样的就是加-s参数）
    
- 案例
    - 设置系统当前时间 ， 比如设置成 2018-10-10 11:22:22，输入：date -s "2018-10-10 11:22:22"
    
### cal 指令

查看日历指令

- 语法
    - cal [选项] （功能描述：不加选项，显示本月日历）
    
- 案例
    - 显示当前日历，输入：cal
    - 显示 2020 年的日历，输入 cal 2020
    
# 搜索查找类

## find 指令

find 指令将从指定目录向下递归地遍历其各个子目录，将满足条件的文件或者目录显示在终端

- 语法
    - find [搜索范围] [选项]
   
- 选项说明
    - -name<查询> 按照指令的文件名查找模式查找文件
    - -user<用户名> 查找属于指定用户名所有文件
    - -size<文件大小> 按照指定的文件大小查找文件
    
- 案例
    - 按文件名：根据名称查找/home 目录下的 hello.txt 文件。输入：find /home/ -name hello.txt
    - 按拥有者：查找/opt 目录下，用户名称为 nobody 的文件。输入：find /opt/ -user nobody
    - 查找整个 linux 系统下大于 20m 的文件（+n 大于 -n 小于 n 等于）。输入：find / -size +20M
    - 查询整个 linux 系统下大于 20k 的文件。输入：find / -size 20k
    - 查询 / 目录下，所有 .txt 的文件。输入：find -name *.txt
    
## locate 指令

locate 指令可以快速定位文件路径。 locate 指令利用事先建立的系统中所有文件名称及路径的 locate 数据库实现快速定位给定的文件。Locate 指令无需遍历整个文件系统，查询速度较快。为了保证查询结果的准确度，管理员必须定期更新 locate 时刻。

- 语法
    - locate 搜索文件
    
- 特别说明
    - 由于 locate 指令基于数据库进行查询，所以第一次运行前，必须使用 updatedb 指令创建 locate 数 据库。
    
- 案例
    - 请使用 locate 指令快速定位 hello.txt 文件所在目录。
        - 创建或更新locate数据库：updatedb
        - 查询hello.txt文件：locate hello.txt
        
## grep 指令和 管道符号 |

grep 过滤查找 ， 管道符，“|”，表示将前一个命令的处理结果输出传递给后面的命令处理。

- 语法
    - grep [选项] 查找内容 源文件
    
- 常用选项
    - `-n` 显示匹配行以及行号
    - `-i` 忽略字母大小写
    
- 案例
    - 在 hello.txt 文件中，查找 "yes" 所在行，并且显示行号。输入：cat hello.txt | grep -n yes
    - 在 hello.txt文件中，查找 "no" 所在行，忽略大小写，并且显示行号（忽略大小写、显示行号，有2个参数）。输入：cat hello.txt | grep -ni no
    
# 压缩和解压类

## gzip/gunzip 指令

gzip 用于压缩文件， gunzip 用于解压的

- 语法
    - gzip 文件 （功能描述：压缩文件，只能将文件压缩为*.gz 文件）
    - gunzip 文件.gz （功能描述：解压缩文件命令）
    
- 细节
    - gzip指令进行压缩后，不会保留原来的文件
    - 同理，gunzip指令解压缩后，压缩包也不保留
    
- 案例
    - gzip 压缩， 将 /home 下的 hello.txt 文件进行压缩。输入：gzip hello.txt
    - gunzip 压缩， 将 /home 下的 hello.txt.gz 文件进行解压缩。输入：gunzip hello.txt.gz
    
## zip/unzip 指令

zip 用于压缩文件， unzip 用于解压的，这个在项目打包发布中很有用的

- 语法
    - zip [选项] XXX.zip 将要压缩的内容（功能描述：压缩文件和目录的命令）
    - unzip [选项] XXX.zip （功能描述：解压缩文件）
    
- zip的常用选项
    - -r：递归压缩，即压缩目录
    
- unzip的常用选项
    - -d<目录> ：指定解压后文件的存放目录
    
- 案例
    - 将 /home 下的 所有文件进行压缩成 mypackage.zip。输入：zip -r mypackage.zip /home/
    - 将 mypackge.zip 解压到 /opt/tmp 目录下。输入：unzip -d /opt/tmp mypackage.zip
    
## tar 指令

tar 指令 是打包指令，最后打包后的文件是 .tar.gz 的文件

- 语法
    - tar [选项] XXX.tar.gz 打包的内容 (功能描述：打包目录，压缩后的文件格式.tar.gz)
    
- 选项
    - `-c` 产生.tar打包文件
    - `-v` 显示详细信息
    - `-f` 指定压缩后的文件名
    - `-z` 打包同时压缩
    - `-x` 解包.tar文件
    
- 提示（压缩和解压就相差一个参数，压缩有c，解压则是x）
    - 压缩常用组合：tar -zcvf 压缩后文件名 要压缩的文件（可多个，空格隔开）
    - 解压常用组合：tar -zxvf 要解压的文件名 目标解压目录
    
- 细节
    - 解压时，需要指定到解压目录时，需要加上-C参数，它的作用就是用来指定解压目录的
    
- 案例
    - 压缩多个文件，将 /home/a1.txt 和 /home/a2.txt 压缩成 a.tar.gz。
        - 输入：tar -zcvf a.tar.gz a1.txt a2.txt
    - 将/home 的文件夹 压缩成 myhome.tar.gz
        - 输入：tar -zcvf myhome.tar.gz /home/
    - 将 a.tar.gz 解压到当前目录
        - 输入：tar -zxvf a.tar.gz
    - 将 myhome.tar.gz 解压到 /opt/ 目录下（主要要加-C参数，并且确保解压到的目录事先要存在，否则会报错）
        - 输入：tar zxvf myhome.tar.gz -C /opt/tmp2/
        
# 组管理和权限管理

## Linux 组基本介绍

在 linux 中的每个用户必须属于一个组，不能独立于组外。在 linux 中每个文件 有所有者、所在组、其它组的概念

- 所有者
- 所在组
- 其它组

### 文件/目录 所有者

#### 查看文件的所有者

一般为文件的创建者,谁创建了该文件，就自然的成为该文件的所有者。

- 指令
    - ls -ahl
    
- 案例
    - 创建一个组 police,再创建一个用户 tom,将 tom 放在 police 组 ,然后使用 tom 来创 建一个文件 ok.txt，看看情况如何
        - groupadd police，创建police警察组
        - useradd -g police tom，创建用户tom，并指定组为police组
        - passwd tom，给用户tom指定密码
        - su - tom，切换到tom用户
        - touch ok.txt，创建一个ok.txt文件，在/home/tom下
        - ls -ahl，查看文件，并显示所有者
        
#### 修改文件所有者

- 指令
    - chown 用户名 文件名
    
- 案例
    - 使用 root 创建一个文件 apple.txt ，然后将其所有者修改成 tom
        - su - root，切换到用户root
        - cd /home，切换到home目录
        - touch apple.txt，新建apple.txt文件
        - ls -ahl，查看文件所有者信息，发现现在是root，准备改为tom
        - chown tom apple.txt，将apple.txt文件所有者从root改为tom。（注意：用户名在前，文件件在后）
        - ls -ahl，此时再次查看文件所有者信息，已经是tom了
        
### 组的创建

- 指令
    - groupadd 组名
    
- 案例
    - 创建一个组，monster，再创建一个用户 fox，并放入到 monster 组中
        - groupadd monster，创建monster组
        - useradd -g monster fox，创建用户fox，并指定组为monster
        - id fox，查看用户fox的信息，信息为：uid=1001(fox) gid=1001(monster) 组=1001(monster)

#### 文件/目录 所在组

当某个用户创建了一个文件后，默认这个文件的所在组就是该用户所在的组

##### 查看文件/目录所在组

- 指令
    - ls -ahl
    
##### 修改文件所在的组

- 指令
    - chgrp 组名 文件名
    
- 案例
    - 使用 root 用户创建文件 orange.txt ,看看当前这个文件属于哪个组，然后将这个文件所在组，修改 到 police 组
        - su - root，切换到root用户
        - cd /home，切换到home目录
        - touch orange.txt，创建orange.txt文件
        - ls -ahl，查看文件组信息，现在为root组
        - chgrp police orange.txt，将orange.txt的组改为police组
        - ls -ahl，再次查看文件组信息，已经改为police组
        
### 其它组

除文件的所有者和所在组的用户外，系统的其它用户都是文件的其它组

### 改变用户所在组

在添加用户时，可以指定将该用户添加到哪个组中，同样的用 root 的管理权限可以改变某个用户 所在的组

- usermod –g 组名 目录名
- usermod –d 用户名 用户名 改变该用户登陆的初始目录。

- 案例
    - 创建一个土匪组（bandit）将 tom 这个用户从原来所在的 police（警察） 组，修改到 bandit(土匪) 组
        - groupadd bandit，新建bandit土匪组
        - id tom，查看用户tom的组，现在是在police组。uid=1000(tom) gid=1000(police) 组=1000(police)
        - usermod -g bandit tom，将tom的组，从police组，改为bandit组
        - id tom，再次查看用户tom的组，现在是bandit组。uid=1000(tom) gid=1002(bandit) 组=1002(bandit)
        
## 权限的基本介绍

- ls -l 中显示的内容如下：
    - `-rwxrw-r-- 1` root root 1213 Feb 2 09:39 abc

- 注意
    - 除了第一位是文件类型，后续都是3个为一组。
    - 一共有3组，所有者权限，所属组权限、其他组权限
    - `r`，代表可读
    - `w`，代表可写
    - `x`，代表可执行
    - `-`，代表没有权限
    - 其他组后面的数字（例如1），看文件类型，如果是文件，表示硬连接的数，如果是目录，则表示该目录的子目录个数。
    - 文件大小，如果是目录，则统一都显示4096

- 0-9 位说明
    - 第 0 位确定文件类型(d, - , l , c , b)
        - `-`，普通文件
        - `d`，目录
        - `l`，软链接
        - `c`，字符设备，例如键盘、鼠标
        - `b`，块文件，硬盘
    - 第 1-3 位确定所有者（该文件的所有者）拥有该文件的权限。---User
    - 第 4-6 位确定所属组（同用户组的）拥有该文件的权限，---Group
    - 第 7-9 位确定其他用户拥有该文件的权限 ---Other
    
### rwx 权限详解

- rwx 作用到文件

    - [ r ]代表可读(read): 可以读取,查看
    - [ w ]代表可写(write): 可以修改,但是不代表可以删除该文件，删除一个文件的前提条件是对该 文件所在的目录有写权限，才能删除该文件
    - [ x ]代表可执行(execute):可以被执行

- rwx 作用到目录
    - [ r ]代表可读(read): 可以读取，ls 查看目录内容
    - [ w ]代表可写(write): 可以修改,目录内创建+删除+重命名目录
    - [ x ]代表可执行(execute):可以进入该目录
    
### 文件及目录权限实际案例

- ls -l 中显示的内容如下：(记住)
    - `-rwxrw-r--` 1 root root 1213 Feb 2 09:39 abc
    
- 每部分代表什么作用
    - 第一个字符代表文件类型： 文件 (-),目录(d),链接(l) 其余字符每 3 个一组(rwx) 读(r) 写(w) 执行(x)
    - 第一组 rwx : 文件拥有者的权限是读、写和执行
    - 第二组 rw- : 与文件拥有者同一组的用户的权限是读、写但不能执行
    - 第三组 r-- : 不与文件拥有者同组的其他用户的权限是读不能写和执行
    
- 请描述一下，10 个字符确定不同用户能对文件干什么

    - 1 文件：硬连接数或 目录：子目录数
    - root 用户
    - root 组
    - 1213 文件大小(字节)，如果是文件夹，显示 4096 字节
    - Feb 2 09:39 最后修改日期
    - abc 文件名
    
- 除了用字母来表示，还可以用数字表示: r = 4，w = 2，x = 1 因此 rwx = 4 + 2 + 1 = 7

### 修改权限-chmod

通过 chmod 指令，可以修改文件或者目录的权限

- 第一种方式：+ 、-、= 变更权限（u:所有者，g:所有者，o:其他人 a:所有人，u、g、o的总和）
    - chmod u=rwx,g=rx,o=x 文件或目录名
    - chomd o+w 文件或目录名，代表给文件或目录的其他人，都加上一个可写权限
    - chmod a-x 文件或目录名，代表给文件或目录的所有人，都减去一个执行权限
    
- 案例
    - 给 abc 文件 的所有者读写执行的权限，给所在组读执行权限，给其它组读执行权限
        - touch abc，新建abc文件
        - ls -l，查看abc文件的权限，-rw-r--r--
        - chmod u=rwx,g=rx,o=rx，修改abc的权限，文件所有者可读、可写、可执行，所有组和其他组都是可读、可执行，不可写
        - ls -l，重新查看abc文件的权限，-rwxr-xr-x
    - 给 abc 文件的所有者除去执行的权限，增加组写的权限
        - chmod u-x,g+w abc，修改abc文件的权限
        - ls -l，查看abc文件的权限为：-rw-rwxr-x
    - 给 abc 文件的所有用户添加读的权限
        = chmod a+r abc，修改abc文件的权限
        - ls -l，查看abc的权限为：-rw-rwxr-x
        
### 第二种方式：通过数字变更权限

- 规则：r=4 w=2 x=1 ,rwx=4+2+1=7

- 例如：chmod u=rwx,g=rx,o=x 文件目录名，相当于 chmod 751 文件目录名

- 案例
    - 将 /home/abc.txt 文件的权限修改成 rwxr-xr-x ，使用给数字的方式实现
        - `rwx`，所有者权限，4 + 2 + 1 = 7
        - `r-x`，所有组权限，4 + 1 = 5
        - `r-x`，其他组权限，4 + 1 = 5
        - 所以最终结果是755，命令则是chmod 755 /home/abc.txt
        
## 修改文件所有者-chown

- 指令
    - chown newowner file，改变文件的所有者
    - chown newowner:newgroup file，同时改变用户的所有者和所有组
    - -R参数，如果是目录 则使其下所有子文件或目录递归生效

- 案例
    - 请将 /home/abc .txt 文件的所有者修改成 tom
        - touch abc.txt，创建abc.txt文件
        - chown tom abc.txt，将abc.txt文件的所有者改为tom
    - 请将 /home/kkk 目录下所有的文件和目录的所有者都修改成 tom
        - su - root，切换到root用户
        - cd /home，切换到home目录
        - mkdir kkk，创建kkk目录到home下
        - cd kkk/，进入kkk目录
        - touch a.txt b.txt，在kkk目录下，创建a.txt和b.txt文件
        - cd ..，切换到上一级目录，就是home目录
        - chown -R tom kkk/，将kkk目录下，所有的子文件、子目录下所有文件，递归将文件和目录的所有者从root改为tom
        
## 修改文件所在组-chgrp

- 指令
    - chgrp newgroup file 改变文件的所有组
    
- 案例
    - 请将 /home/abc .txt 文件的所在组修改成 bandit (土匪)
        - chgrp bandit /home/abc.txt，修改/home/abc.txt文件的所在组修改为bandit
    - 请将 /home/kkk 目录下所有的文件和目录的所在组都修改成 bandit(土匪)
        - chgrp -R bandit /home/kkk，递归修改/home/kkk目录下的所有子文件和目录的所在组修改为bandit
        
## 最佳实践-警察和土匪游戏

有2个组，分别是 police警察组 和 bandit土匪组

警察组：jack、jerry
土匪组：xh、xq

- 创建组
    - groupadd police
    - groupadd jerry
    
- 创建用户
    - useradd -g police jack
    - useradd -g police jerry
    - useradd -g bandit xh
    - useradd -g bandit xq
    
- 配置用户的密码，都为123
    - passwd jack
    - passwd jerry
    - passwd xh
    - passwd xq
    
- jack 创建一个文件，自己可以读写，本组人可以读，其它组没人任何权限
    - vim jack01.txt，内容为hello，wq保存并退出，创建文件
    - chmod 640 jack01.txt，修改权限
    - ls -l，查看文件权限为：-rw-r-----
    
- jack 修改该文件，让其它组人可以读, 本组人可以读写
    - chmod o=r,g=rw jack01.txt，修改权限
    - ls -l，查看文件权限为：-rw-rw-r--
    
- xh 投靠 警察，看看是否可以读写，jack目录下的jack01.txt
    - usermod -g police xh，将xh的组从bandit土匪组，改为police警察组
    - cd home/jack，发现权限不够，因为jack目录只能jack自己读写
    - chmod g=rx jack/，开放jack给police组的所有人读写
    - ls -l，查看目录权限为：drwxr-x---
    - logout，注销，再重新登录，权限才会生效
    - cd /home/jack，重新登录后，进入jack的家目录，因为xh已经是polic警察组，所以能进入jack目录
    - ls -l，查看jack01.txt文件的权限为：-rw-rw-r--，jack同组的成员，可以读写该文件
    - vim jack01.txt，修改jack01.txt文件，wq保存并退出，证明是可以读写的
    
# crond任务调度

- 概述

任务调度：是指系统在某个时间执行的特定的命令或程序。 

- 任务调度分类：
    - 系统工作：有些重要的工作必须周而复始地执行。如病毒扫描等
    - 个别用户工作：个别用户可能希望执行某些程序，比如对 mysql 数据库的备份。
    
- 语法
    - crontab [选项]
    
- 常用选项
    - `-e` 编辑crontab定时任务
    - `-l` 查询crontab任务
    - `-r` 删除当前用户所有的crontab任务
    
## 快速入门

### 任务的要求

- 设置任务调度文件：/etc/crontab
- 设置个人任务调度。执行 crontab –e 命令。
- 接着输入任务到调度文件
    - 如：*/1 * * * * ls –l /etc/ > /tmp/to.txt
    - 意思说每小时的每分钟执行 ls -l /etc/ >> /tmp/to.txt 命令

### 步骤

- cron -e，编辑定时任务
- 往文件中，写入`*/ 1 * * * * ls -l /etc >> /tmp/to.txt`
- :wq，写入并退出
- 切换到/tmp/目录下，过1分钟后，ls查询to.txt文件的内容（vim to.txt）

### 5个占位符的说明

| 项目 | 含义 | 范围 |
| ---- | ---- | ---- |
| 第一个* | 一小时当中的第几分钟 | 0-59 |
| 第二个* | 一天当中的第几个小时 | 0-23 |
| 第三个* | 一个月当中的第几天 | 1-31 |
| 第四个* | 一年当中的第几月 | 1-12 |
| 第五个* | 一周当中的星期几 | 0-7（0和7都代表星期日） |

### 特殊符号的说明

| 特殊符号 | 含义 |
| ---- | ---- |
| * | 代表任何时间。比如第一个*，就代表一小时中每分钟都执行一次的意思 |
| , | 代表不连续时间。比如`"0 8,12,16 * * *"`命令，就代表在每天的8点0分，12点0分，16点0分都执行一次命令 |
| - | 代表连续的时间范围。比如`"0 5 * * 1-6"`命令，代表在周一到周六的凌晨5点0分执行命令 |
| */n | 代表每隔多久执行一次。例如`"*/10 * * * *"`命令， 代表每隔10分钟就执行一遍命令|

#### 特定时间执行任务案例

| 时间 | 含义 |
| ---- | ---- |
| 45 22 * * *命令 | 在22点45分执行命令 |
| 0 17 * * 1 命令 | 每周1的17点0分执行命令 |
| 0 5 1,15 * * 命令 | 每月1号和15号的凌晨5点0分执行任务 |
| 40 4 * * 1-5命令 | 每周一到周五的凌晨4点40分执行任务 |
| */10 4 * * *命令 | 每天的凌晨4点，每隔10分钟执行一次命令 |
| 0 0 1,15*1命令 | 每月1号和15号，每周1的0点0分都会执行命令。注意：星期几和几号最好不要同时出现，因为他们定义的都是天。非常容易让管理员混乱 |

#### 应用案例

- 每隔 1 分钟，就将当前的日期信息，追加到 /tmp/mydate 文件 中
    - 先编写一个文件，mytask1.sh，写入：date >> /tmp/mydate
    - 给mytask1.sh文件一个可执行权限，chmod 744 /home/mytask1.sh
    - crontab -e，填入定时规则和脚本位置： */1 * * * * /home/mytask1.sh

- 每隔 1 分钟， 将当前日期和日历都追加到 /home/mycal 文件 中
    - 先编写一个文件，mytask2.sh，写入：cal >> /tmp/mycal
    - 给mytask2.sh文件一个可执行权限，chmod 744 /home/mytask2.sh
    - crontab -e，填入定时规则和脚本位置： */1 * * * * /home/mytask2.sh

- 每天凌晨 2:00 将 mysql 数据库 testdb ，备份到文件中 mydb.bak
    - 先编写一个文件，mytask3.sh，写入：/usr/local/mysql/bin/mysqldump -u root -proot testdb > /tmp/mydb.bak
    - 给mytask3.sh文件一个可执行权限，chmod 744 /home/mytask3.sh
    - crontab -e，填入定时规则和脚本位置： 0 2 * * * /home/mytask3.sh
    
#### crond 相关指令

- crontab –r：终止任务调度。
- crontab –l：列出当前有那些任务调度
- service crond restart [重启任务调度]

## Linux 磁盘分区、挂载

### 分区基础知识

- mbr 分区 
    - 最多支持四个主分区
    - 系统只能安装在主分区
    - 扩展分区要占一个主分区
    - MBR 最大只支持 2TB，但拥有最好的兼容性
    
- gtp 分区
    - 支持无限多个主分区（但操作系统可能限制，比如 windows 下最多 128 个分区）
    - 最大支持 18EB 的大容量（1EB=1024 PB，1PB=1024 TB ）
    - windows7 64 位以后支持 gtp
    
### Linux 分区

#### 原理介绍

- Linux 来说无论有几个分区，分给哪一目录使用，它归根结底就只有一个根目录，一个独立且 唯一的文件结构 , Linux 中每个分区都是用来组成整个文件系统的一部分。
- Linux 采用了一种叫“载入”的处理方法，它的整个文件系统中包含了一整套的文件和目录， 且将一个分区和一个目录联系起来。这时要载入的一个分区将使它的存储空间在一个目录下获得。

#### 硬盘说明

- Linux 硬盘分 IDE 硬盘和 SCSI 硬盘，目前基本上是 SCSI 硬盘
- 对于 IDE 硬盘，驱动器标识符为“hdx~”,其中“hd”表明分区所在设备的类型，这里是指 IDE 硬 盘了。“x”为盘号（a 为基本盘，b 为基本从属盘，c 为辅助主盘，d 为辅助从属盘）,“~”代表分区， 前四个分区用数字 1 到 4 表示，它们是主分区或扩展分区，从 5 开始就是逻辑分区。例，hda3 表示为 第一个 IDE 硬盘上的第三个主分区或扩展分区,hdb2 表示为第二个 IDE 硬盘上的第二个主分区或扩展 分区。
- 对于 SCSI 硬盘则标识为“sdx~”，SCSI 硬盘是用“sd”来表示分区所在设备的类型的，其余则 和 IDE 硬盘的表示方法一样。

#### 使用 lsblk 指令查看当前系统的分区情况

- 指令
    - lsblk -f，查看系统的分区和挂载情况

```
NAME   FSTYPE   LABEL          UUID                                   MOUNTPOINT
sda                                                                   
├─sda1 xfs                     5be0fc66-9f78-4b2e-b847-be4cd83aa7dc   /boot
└─sda2 LVM2_mem                Q3OOXZ-tcDj-cHQ9-aLz7-8gV1-vKsh-RujZZ1 
  ├─centos-root
       xfs                     e156ee8d-fe62-4c5d-b553-ffa0e8ee2a22   /
  └─centos-swap
       swap                    d50ac8f7-8e7a-4a97-9a0f-a5d626bf2156   [SWAP]
sr0    iso9660  CentOS 7 x86_64
                               2020-04-22-00-54-00-00
```

- 各个部分的具体解释
    - sda1、sda2、sda3：分区情况
    - ext4、swap、ext4：分区类型
    - 一长串字符：唯一标识分区的40位不重复的字符串
    - /boot、SWAP：挂载点
    
- 查看每个分区的大小
    - 使用lsblk不加参数的指令即可
    
#### 挂载的经典案例

需求是给我们的 Linux 系统增加一个新的硬盘，并且挂载到/home/newdisk

- 如何增加一块硬盘
    - 虚拟机设置，增加一块新磁盘，配置2G大小，重启虚拟机，再次lsblk -f查看磁盘情况，会发现新磁盘没有分区，接下来进行分区
- 分区
    - 分区命令：fdisk /dev/sdb
    - 开始对/sdb分区
        - 输入m，显示操作列表
        - 输入n，选择新增分区
        - 输入p，再输入1，表示划分主分区的第一块
        - 回车2次确认剩余空间
        - 最后输入w，写入分区并退出，如不保存并退出则输入q
- 格式化
    - 分区后，需要格式化后，才能使用（未格式化时，分区的UUID为空），下面进行格式化
    - 格式化命令：mkfs -t ext4 /dev/sdb1（其中 ext4 是分区类型）
- 挂载
    - 挂载，将一个分区与一个目录联系起来
    - 先创建要挂载的目录：mkdir /home/newdisk
    - 开始挂载：mount /dev/sdb1 /home/newdisk
    - lsblk -f，查看分区信息，sdb1已经挂载到了/home/newdisk
    - 目前是临时挂载的，重启会丢失，需要永久挂载，则需要进行下一步
- 设置自动挂载
    - 当我们重启系统时，硬盘和文件夹的挂载关系会丢失，所以需要建立永久挂载，这样重启系统，仍然可以挂载
    - vim修改/etc/fstab，vim /etc/fstab
    - yy复制一行，再按p粘贴。修改为：/dev/sdb1           /home/newdisk           ext4    defaults        0 0
    - 输入:wq，保存并退出
    - mount -a，自动挂载
    - reboot，重启，再次查看是否会自动挂载
- 拓展，如果想卸载（取消挂载）
    - umount /dev/sdb1，如果出现device is busy的错误，就是你目前就在挂载目录内，必须先退出来，再执行
    - 此时，再进行lsblk -f查看分区信息，则发现已经没有挂载到目录了
- 取消永久挂载
    - 上面的卸载，只是临时性的，因为我们有永久挂载配置到/etc/fstab上
    - 删除掉/etc/fstab配置上的挂载关系即可
    
### 磁盘情况查询

#### 查询系统整体磁盘使用情况

- 语法
    - df -h
    
- 案例
    - 查询系统整体磁盘使用情况，输入df -h
    
```
文件系统                 容量  已用  可用 已用% 挂载点
devtmpfs                 979M     0  979M    0% /dev
tmpfs                    991M     0  991M    0% /dev/shm
tmpfs                    991M  9.6M  981M    1% /run
tmpfs                    991M     0  991M    0% /sys/fs/cgroup
/dev/mapper/centos-root   17G  1.4G   16G    9% /
/dev/sdb1                2.0G  6.0M  1.9G    1% /home/newdisk
/dev/sda1               1014M  137M  878M   14% /boot
tmpfs                    199M     0  199M    0% /run/user/0
```

#### 查询指定目录的磁盘占用情况

查询指定目录的磁盘占用情况，默认为当前目录

- 语法
    - du -h /目录

- 常用参数
    - `-s` 指定目录占用大小汇总
    - `-h` 带计量单位
    - `-a` 含文件
    - `--max-depth=1` 子目录深度
    - `-c` 列出明细的同时，增加汇总值
    
- 案例
    - 查询 /opt 目录的磁盘占用情况，深度为 1
        - 输入：du -ach --max-depth=1 /opt/
        
#### 磁盘情况-工作实用指令

- 统计/home 文件夹下文件的个数
    - ls -l /home | grep "^-" | wc -l
    - 三部曲，先列出所有，再过滤出文件，最后统计
    - grep "^-"，表示以-打头的，就是值统计文件，不统计目录。wc是统计
    
- 统计/home 文件夹下目录的个数
    - ls -l /home | grep "^d" | wc -l
    - 相比第一个，只是过滤掉非目录的，只统计目录。

- 统计/home 文件夹下文件的个数，包括子文件夹里的
    - ls -lR /home | grep "^-" | wc -l
    - 相比第一个，加多了一个-R参数，表示递归统计目录内的所有子文件

- 统计文件夹下目录的个数，包括子文件夹里的
    - ls -lR /home | grep "^d" | wc -l
    - 同样，加上-R参数

- 以树状显示目录结构
    - yum install tree，用yum安装tree命令
    - tree，以树状方式打印
    
## 网络配置

#### Linux 网络配置

目前我们的网络配置采用的是 NAT。虚拟机软件VMWare会在物理机上生成一个虚拟网卡wmnet8，ip默认是192.168.184.x系列，虚拟机的网络和物理机会构成一个网络。
所以虚拟机和物理机能够通信。虚拟机需要和外面公网进行通讯，就需要和网关交互。

#### 查看网络 IP 和网关

- 查看虚拟网络编辑器
    - 点击VMWare，左上角的编辑，选择虚拟网络编辑器。选择vmnet8，窗口底部的子网ip，默认是192.168.184.0，如果更改此值，下次重启系统，则会将ip从设定的开始。
    
- 查看网关
    - 选择wmnet8时，点击NAT设置，可以设置网关ip。
    
#### ping 测试主机之间网络连通

- 语法
    - ping 目的主机 （功能描述：测试当前服务器是否可以连接目的主机）
    
- 案例
    - 测试当前服务器是否可以连接百度
        - ping www.baidu.com
        
#### linux 网络环境配置

- 第一种方法(自动获取)

进入Linux的网络配置页面（centos在顶部栏中选择，系统，首选项，网络连接），选择具体网络，例如eth0，点击编辑，切换到ipv4设置，选择自动（DHCP）。

缺点: linux 启动后会自动获取 IP,缺点是每次自动获取的 ip 地址可能不一样。这个不适用于做服 务器，因为我们的服务器的 ip 需要是固定的。

- 第二种方法(指定固定的 ip)

直接修改配置文件来指定IP,并可以连接到外网（程序员推荐） ，编辑 /etc/sysconfig/network-scripts/ifcfg-eth0

- 要求：将 ip 地址配置的静态的，ip 地址为 192.168.184.130
    - vim /etc/sysconfig/network-scripts/ifcfg-eth0
    - 将ONBOOT配置成yes，即ONBOOT=yes，默认为no，意思为启动时自动开启网络
    - 将BOOTPROTO配置为static，即BOOTPROTO=static，默认为none
    - IPADDR指定为固定的ip，即IPADDR=192.168.184.130
    - GATEWAY指定网关的ip，即GATEWAY=192.168.184.2
    - DNS1指定域名解析器的ip，配置和网关ip一致即可，即DNS1=192.168.184.2
    
- 注意：修改后，一定要 重启服务！
    - service network restart，重启网络服务
    - 或者 reboot 重启系统