#### adb install安装失败之INSTALL_FAILED_TEST_ONLY

问题现象

- 使用adb install命令，安装debug包，可能会出现**INSTALL_FAILED_TEST_ONLY**错误

```
$ adb install app-debug.apk 
Failed to install app-debug.apk: Failure [INSTALL_FAILED_TEST_ONLY: installPackageLI]
```

产生原因

- AS Run 出来的 Apk，之所以无法安装，是因为其携带了 FLAG_TEST_ONLY 这个 Flag，它会阻止我们使用正常的方式安装。想要安装，可以通过 adb install -t 来解决。

- 虽然这个 Flag 初始于 API Level 4，但是它在 AS 3.0 中，才被默认加入。想要去掉可以通过增加 android.injected.testOnly=false 来实现。

解决方法

- 方式一：增加-t参数进行安装（适合没有项目，只有一个debug包）

```
adb install -t app-debug.apk
```

- 方式二：项目根目录的gradle.properties增加testOnly配置，testOnly默认为true（适合有项目，每次使用AndroidStudio进行run安装时使用）

```
android.injected.testOnly=false
```

参考文章

[为什么我把 Run 出来的 Apk 发给老板，却装不上！](https://mp.weixin.qq.com/s/Ro3HuC31S8asxdE0fQCkjg)