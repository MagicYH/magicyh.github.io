---
layout: post
title: Android Smali debug
categories:
  - android
  - smali
tags:
  - android
  - smali
---

本文会比较详细地一步一步说明如何将APK反编译为smali文件，并且进行动态调试、修改，重新打包为可以执行的apk文件

## 相关工具
jdk: 自行检索 <br>
apktool: [https://github.com/iBotPeaches/Apktool/releases](https://github.com/iBotPeaches/Apktool/releases) <br>
android studio: [https://developer.android.google.cn/studio/](https://developer.android.google.cn/studio/) <br>
smalidea: [https://github.com/JesusFreke/smali/wiki/smalidea](https://github.com/JesusFreke/smali/wiki/smalidea) <br>
smali: [https://bitbucket.org/JesusFreke/smali/downloads/](https://bitbucket.org/JesusFreke/smali/downloads/) <br>
baksmali: [https://bitbucket.org/JesusFreke/smali/downloads/](https://bitbucket.org/JesusFreke/smali/downloads/) <br>

## 安装smali相关工具
### 安装方法有两种：

1. 一种是先去上述提供的下载地址上下载最新的`smalidea.zip`压缩包，通过添加本地jar包的方式安装插件`Perference->Plugins`点击右上角的齿轮图标，`Install plugin from disk`找到下载的zip包进行安装
    
    ![smalidea](/images/smali/install_smalidea.png)

2. 还是刚才的对话框，直接在搜索框中搜索`smalidea`就能找到最新的插件

### 反编译APK
下载`smali`，`baksmali`两个jar包，调试前需要使用`baksmali`工具将apk反编译成可供调试的smali文件，使用命令为
```
// 使用方法
java -jar baksmali-2.3.4.jar d 包名
// 使用帮助可以通过如下命令查看
java -jar baksmali-2.4.5.jar -h
```
注意：使用`apktool`也可以得到smali文件，但是并不能通过那样生成的smali文件进行动态调试

### 创建动态调试工程
通过`baksmali`工具得到可供调试的smali文件后，建立一个新文件夹`baksmali/src`把得到的`smali`文件全部放到这个目录下，然后使用`Android Studio`导入一个新工程
- `File -> New -> Import Project` 选择`Create project from existing sources`，入下图所示，之后一路`next`
    ![import_project](/images/smali/import_smali_project.png)

- 创建完项目后右键点击`src`目录，标记为`source root`
    ![mark_source_root](/images/smali/mark_source_root.png)

- 设置安卓SDK`File -> Project Structure`选择`Android SDK`版本
    ![android_sdk](/images/smali/set_sdk.png)

### 开启apk的调试选项
一般来说正式发布的应用这个选项都是关闭的，因此需要调试需要满足一下两个条件之一
1. Apk的 AndroidManifest.xml 中 Application标签 必选包含属性 android:debuggable="true"；
2. /default.prop 中 ro.debuggable 的值为1（可以在 boot.img 镜像文件设置 /default.prop 中 ro.debuggable 的值为 1 来实现）

方便起见，我们选择第一种方式
#### 编辑 AndroidManifest.xml 文件
使用`apktool`解压apk包
```
apktool d xxx.apk
```

#### 添加 debuggable 选项
解包后的 xml文件可以直接编辑，如图所示

![debuggable](/images/smali/debuggable.png)

#### 打包
使用`apktool`重新打包apk，其中 xxx 为解压后的apk目录，这个命令执行后会在 `xxx/dist` 目录下生成新的apk文件
```
apktool b xxx
```

#### 新apk签名
安卓应用没有签名是无法正常安装的，另外这里需要注意的是，部分可能会对签名进行检查，这种情况，即使重新签名后应用也可能会闪退

首先生成RSA秘钥对，输入命令后根据提示操作即可
```
keytool -genkeypair -alias magick.keystore -keyalg RSA -validity 400 -keystore magick.keystore 

// -genkeypair 指定生成密钥对 
// -alias 密钥对的别名 
// -keyalg 密钥对用于的算法，这里用的是RSA 
// -validity 密钥对的有效期，单位为天 
// -keystore 密钥对存储的文件名
```

得到签名后使用`jarsigner`对APK进行签名，命令输入完后会要求输入秘钥对的密码，密码就是生成秘钥对时输入的密码
```
jarsigner -verbose -keystore magick.keystore -signedjar Magick.apk Magick_unsigned.apk magick.keystore 

// -verbose 输出签名详细信息 
// -keystore 指定密钥对的存储路径 
// -signedjar 后面三个参数分别是 签名后的APK包 未签名的APK包 和 密钥对的别名 
```

签名优化，这个步骤是可选的
```
zipalign -f -v 4 Magick.apk Magick_zip.apk 

// -f 强制覆盖已有的文件 
// -v 输出详细内容
```

### 准备ADM工具
ADM工具全称`Android Device Monitor`，该工具已经不被最新版的Android Studio支持了，需要自己找到可执行文件来使用，首先还是通过`File -> Project Structure` 找到本机的SDK路径

![sdk_path](/images/smali/sdk_path.png)

ADM工具就是sdk目录的`tools/monitor`，但一般来说，直接执行会失败，需要先修复
#### 修复ADM
到[Eclipse官网](https://archive.eclipse.org/eclipse/downloads/)下载对应最新的`SWT Binary and Source`

![swt_binary](/images/smali/swt_binary.png)
然后覆盖`tools/tools/lib/monitor-x86_64/plugin/org.eclipse.swt.cocoa.macosx.x86_64_3.100.1.v4236b.jar`包，此时再重启`monitor`即可

### 动态调试
通过查看`AndroidManifest.xml`找到报名和主类名，使用以下命令启动调试，命令成功后，手机界面会弹出窗口，显示`Waiting for debugger`字样

```
// 命令格式： adb shell am start -D -W -n 包名/主Active类名
adb shell am start -D -W -n com.example.myapplication/com.example.myapplication.MainActivity
```

执行命令后打开`monitor`可以看到一个可以debug连接的端口

![monitor_port](/images/smali/monitor_port.png)

此时操作`Android Studio`，`Run -> Configurations`单击"+"按钮，选中"Remote"添加远程调试的配置，如下图所示，根据上图的端口截图，设置连接端口号为：8700，设置调试名称为 debug_apk 

![debug_setting](/images/smali/debug_setting.png)

最后点击`Run -> Debug 'debug_apk'`即可开始调试，记得先打上断点

![run_debug](/images/smali/run_debug.png)

## 总结
整个过程比较长，但是没有遇到太多麻烦，耐心一步一步操作下来还是能搞定的，不过smali调试后如果需要修改代码，smali的语法还是要略微了解一下的，再怎么啰嗦也比汇编强多了，稍微看看的话问题不大，这里再推荐一个关于安卓破解的很好的教程
[https://www.52pojie.cn/thread-408645-1-1.html](https://www.52pojie.cn/thread-408645-1-1.html)

## 参考资料
> [https://blog.csdn.net/xuefu_78/article/details/80228223](https://blog.csdn.net/xuefu_78/article/details/80228223) <br>
> [https://blog.csdn.net/QQ1084283172/article/details/71250622](https://blog.csdn.net/QQ1084283172/article/details/71250622) <br>
> [https://www.jianshu.com/p/cdca73159c5c](https://www.jianshu.com/p/cdca73159c5c) <br>