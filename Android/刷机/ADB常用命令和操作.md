# ADB常用命令和操作

### 常用操作

- 显示当前接入的Android设备

```
adb devices
```

- 开启、关闭adb服务

```
//开启adb
adb start-server
//关闭adb
adb kill-server
```

- 查看日志输出

```
adb logcat
```

- adb安装apk

```
//-r 代表如果apk已安装，重新安装apk并保留数据和缓存文件
adb install -r apk路径
```

- 卸载app

```
//直接删除
adb uninstall apk包名
//卸载 app 但保留数据和缓存文件
adb uninstall -k apk包名
```

- 列出安装的app的包名

```
//列出手机装的所有app的包名
adb shell pm list packages
//列出系统应用的所有包名
adb shell pm list packages -s
//列出除了系统应用的第三方应用包名
adb shell pm list packages -3
```

- 启动应用

```
adb shell am start -n 包名/.MianActivity
```

- 强制停止应用

```
adb shell am force-stop apk包名
```

### 传输相关

- 拉取Android设备中的文件

```
adb pull 要获取的文件路径 存储的文件路径
```

- 推送文件到Android设备中

```
adb push 要上传的文件路径 存储的文件路径
```

### 刷机相关

- 进入fastboot

```
adb reboot bootloader
```

- 刷入recovery

```
fastboot flash recovery twrp-3.0.2-0.img
```

- twrp下，使用adb sideload模式刷机（twrp进入advanced，选择sideload后，使用如下命令）

```
adb sideload 刷机包.zip
```

- fastboot状态下重新启动

```
fastboot reboot
```

- adb进入recovery

```
adb reboot recovery
```