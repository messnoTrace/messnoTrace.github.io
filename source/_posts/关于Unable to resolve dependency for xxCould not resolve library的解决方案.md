---
title: 关于Unable to resolve dependency for xxCould not resolve library的解决方案
date: 2019-04-05 10:39:39
tags:
---
先介绍下背景，环境是Androidstudio3.1.3，没错，截止目前是最新的，在不久之前，我用的是3.1.2，然后新建项目，敲上代码，implemention 各种顺手的library，毛问题都没有，升级了3.1.3之后，我想加一个库，结果出现了如下的各种问题。![WX20180626-110232.png](https://upload-images.jianshu.io/upload_images/1453857-4489eca9ee9892f6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


一开始我以为是库的问题，然后一顿google搜索：
![WX20180626-110400@2x.png](https://upload-images.jianshu.io/upload_images/1453857-b767224e5beb26ec.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

眼睛一亮，看到第二个答案直接出现关键词androidstudio3.0，心里顿时一阵鸡冻，点进去看看答案，直接找认同最多的答案：
![WX20180626-110636@2x.png](https://upload-images.jianshu.io/upload_images/1453857-2d2ff13fcdfab998.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

意思是啥呢，大概就是说在andoridsutio2.x向3.x迁移的时候需要注意修改的东西，然并卵啊，我是直接上的3.X的新项目，没有迁移一说。随后又是一阵搜索，这下就尴尬了，答案基本上都是类似这样 的和不勾选gradle offline work这个选项，各种搜索各种的贼鸡儿蛋疼，年轻人你感觉得到绝望么，我在想，如果时光可以倒流，我肯定不手贱升级AndroidStudio，以前的2.x用起来，是毛问题都没有啊，google都救不了我了，我怕是没救了，绝望。添加不了任何库，这个问题，是真的真的很蛋疼啊，啥都要自己写，换你，你愿意么？反正我是不愿意。。
接着又是一波操作猛如虎，中间过程就不重要了，直接跳结果吧。
打开Terminal选项卡，输入 ./gradlew build（windows下是gradlew build提示没这个命令的，那就是你的gradle环境变量没配置）-> 回车

![WX20180626-111908.png](https://upload-images.jianshu.io/upload_images/1453857-6a68fddb44522e5f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

重点来了，`Connect to 127.0.0.1:1080 [/127.0.0.1] failed: Connection refused (Connection refused)
` mmp 搜索：
![WX20180626-112122@2x.png](https://upload-images.jianshu.io/upload_images/1453857-e4706390e41dfdf8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

第一个点进去 [链接](https://blog.csdn.net/Rainminism/article/details/79713788)
进去一看说我们使用了代理，要修改gradle.properties 文件，然后默默的打开我们的工程此文件一看，![WX20180626-112620.png](https://upload-images.jianshu.io/upload_images/1453857-23b4e09470d702f9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

意不意外，惊不惊喜，里面毛都没有啊，然后有没有人跟我一样，有点怀疑人生了，怎么办怎么办，继续搜索啊，果然，在一个答案中发现了自己是有多傻逼
![WX20180626-113000@2x.png](https://upload-images.jianshu.io/upload_images/1453857-f646707a7e7906d8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

重点来了，第一句，`Go to gradle scripts` ，卧槽，我想起来个东西，我们通常用的都是project视图啊，新建项目都是Android视图啊，要是你曾经看过Andorid视图，或者说你曾经解决过Andoridstudio遇到的坑，那你就明白了我说的是啥，吓得我赶紧切换到Android视图：
![WX20180626-113441.png](https://upload-images.jianshu.io/upload_images/1453857-834fb634600acea8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

果然，`globle properties` 
打开一看，mmp，果然存在：![WX20180626-113550.png](https://upload-images.jianshu.io/upload_images/1453857-5b2fc6c929a87506.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
果断注释，重新编译，老铁，真的没毛病了。😌 mmp







