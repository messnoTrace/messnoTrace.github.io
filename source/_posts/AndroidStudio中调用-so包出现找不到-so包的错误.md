---
title: AndroidStudio中调用.so包出现找不到.so包的错误
date: 2019-04-05 11:40:45
tags:
---
<link href="http://cdn.bootcss.com/highlight.js/8.0/styles/monokai_sublime.min.css" rel="stylesheet">  
<script src="http://cdn.bootcss.com/highlight.js/8.0/highlight.min.js"></script>  
<script >hljs.initHighlightingOnLoad();</script>  

![QQ截图20160721125018.png](http://upload-images.jianshu.io/upload_images/1453857-0fb31dfbb366a918.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###出现原因
RT,在Andoridsutdio中调用.so库出现如现标题所示错误，该怎么解决。首先出现这个错误的原因是因为你的cpu架构是64位的。而你的.so库在编译的时候没有支持64位cpu。**  如果这个.so的库是你自己弄的，那你重新生成下支持64位cpu的.so库出来就行了，剩下的文字就不用看了，本文适用于你没法对.so做操作的朋友。** 查看cpu架构，可以通过adb shell 命令。进入/system/目录下有一个build.prop文件，这个里面有一行 ro.product.cpu.abi=xx就是你所要的信息，具体命令如下:

                    adb shell
                    cd /system
                    cat build.prop


apk包在安装的时候，系统会把包中与自己的abi对应的lib目录中的so库文件拷贝到system分区中，32位机器中只有一个目录/system/lib，64位机器中有两个目录/system/lib和/system/lib64，app启动进行链接时，64位机器中会先到/system/lib64目录中去找，这时候肯定找不到。如果没有找到再到/system/lib目录中去找。如果你把32位的so库拷贝到了lib64目录中，会导致链接失败，同样，64位的so库被拷贝到lib目录中也会导致失败，所以so库要和目录一一对应。

###解决方案
首先我的目录结构是这样的：
![QQ截图20160721123538.png](http://upload-images.jianshu.io/upload_images/1453857-be5473a00f8f04a3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
我的.so并没有放在新建的jniLibs目录中，这个关系应该不大，两种方式都可以。
我的cpu架构是 ro.product.cpu.abi=arm64-v8a
而我的只有一个CPU架构就是armebi-v7a,我们要做的就是阻止生成arm64-v8a;
正常情况打开apk的lib结构如下：
![QQ截图20160721124748.png](http://upload-images.jianshu.io/upload_images/1453857-5e5183dd73f1ac09.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

方案如下：
* 首先在project目录下的 gradle.properties中添加一句 
              android.useDeprecatedNdk=true
* 在app的build.gradle中的defaulConfig下添加如下：
            ndk {  
              abiFilters "armeabi", "armeabi-v7a", "x86", "mips"
            }


这时候你解压开生成的apk包，会发现目录lib结构如下：

![QQ截图20160721124539.png](http://upload-images.jianshu.io/upload_images/1453857-df30fc0704288587.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

OK，这样就解决了我的这个问题，网络上有一些别的阻止生成arm64-v8a，但是不好使。