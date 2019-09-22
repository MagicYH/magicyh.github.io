---
layout: post
title: Hello xposed
categories:
  - android
  - xposed
tags:
  - android
  - xposed
---

看了不少讲`xposed`框架的文章，大部分都没看懂，主要原因还是本人毫无安卓开发基础……

好在面向小白的文章还是有的，参考一篇文章终于还是摸索出了该如何开发`xposed`插件

### 创建一个新工程
工具就使用`Android Stdudio (AS)`，创建新工程很简单，没有什么特殊要求，但是为了我们展示方便，这里选择`Empty Activity`作为模板

### 导入依赖包
按照经验，在`build.gradle`文件中加入maven相关代码，（不确定是否必要）
```
allprojects {
    repositories {
        google()
        jcenter()
        
    }
}
```
点击`Build -> Edit Libraries and Dependencies`在`Dependencies -> app`项目下添加依赖，点击`+`号，选择`1 Library Dependency`，在输入框检索`de.robv.android.xposed:api:82`添加依赖包
![img](https://s2.ax1x.com/2019/09/22/u9j4SS.png)
![img](https://s2.ax1x.com/2019/09/22/uC98sO.png)
添加依赖后需要再次改动`build.gradle`文件，可以看到，自动添加时，`de.robv.android.xposed:api:82`的模式是`implementation`，需要将`implementation`修改为`compileOnly`
> 注意：这次需要改动的是`app`目录下的`build.gradle`文件，前一次是整个项目的`build.gradle`文件，两者是不一样的

### 配置文件清单
在文件`AndroidManifest.xml`，`application`标签下添加如下代码，如图所示：
```
<meta-data android:name="xposedmodule" android:value="true"/>
<meta-data android:name="xposeddescription" android:value="xposed demo"/>
<meta-data android:name="xposedminversion" android:value="82"/>
```
![img](https://s2.ax1x.com/2019/09/22/u9j5Qg.png)

### 添加目标hook方法
在`MainActivity`类添加方法`MyToast`，作用为弹出一个toast，并且在`onCreate`方法末尾执行
```
public void MyToast(String msg) {
    Toast.makeText(this, msg, Toast.LENGTH_SHORT).show();
}
```
![img](https://s2.ax1x.com/2019/09/22/u9xUgO.png)

### hook
在`java`目录添加一个`hook`包，添加新类`HookMain`添加如下hook代码
```
public class HookMain implements IXposedHookLoadPackage {
    @Override
    public void handleLoadPackage(XC_LoadPackage.LoadPackageParam lpparam) throws Throwable {
        XposedHelpers.findAndHookMethod("com.example.xposeddemo.MainActivity", lpparam.classLoader, "MyToast", String.class, new XC_MethodHook() {
            @Override
            protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                super.beforeHookedMethod(param);
                param.args[0] = "Hook success";
            }

            @Override
            protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                super.afterHookedMethod(param);
            }
        });
    }
}
```
这个exmple，原本的情况下应用启动时应该弹出toast，内容为`Hello world`，启动xposed插件的话，内容因为hook，会被替换成`Hook success`

### 配置xposed入口
最后还需要配置xposed的入口函数，在main目录下创建文件夹`assets`，并且在文件加中创建文件`xposed_init`，其中写入xposed入口类，本例中就是`hook.HookMain`
![img](https://s2.ax1x.com/2019/09/22/uCS9mQ.png)

## 总结
以上工作完成后，编译，安装即可测试

## 参考资料
> [https://www.52pojie.cn/thread-738254-1-1.html](https://www.52pojie.cn/thread-738254-1-1.html) <br>
> [https://github.com/rovo89/XposedBridge/wiki/_pages](https://github.com/rovo89/XposedBridge/wiki/_pages)

