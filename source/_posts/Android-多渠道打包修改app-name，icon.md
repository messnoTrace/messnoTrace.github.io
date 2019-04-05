---
title: Android 多渠道打包修改app name，icon
date: 2019-04-05 11:44:02
tags:
---
<link href="http://cdn.bootcss.com/highlight.js/8.0/styles/monokai_sublime.min.css" rel="stylesheet">  
<script src="http://cdn.bootcss.com/highlight.js/8.0/highlight.min.js"></script>  
<script >hljs.initHighlightingOnLoad();</script>  
搞过多渠道打包的都知道，我们只需要在 module 的build.gradle中配置相应的渠道号就行:

                   
                    productFlavor{
                              baidu {
                              }
                            xiaomi{}
                             wandoujia{}
                        }
像这样，就可以了，然后将友盟的chanelvalue修改下就可以了，需要不同的渠道，配置不同的applicationid，也不是什么难事。不过今天我遇到的需求是，不同的渠道配置不同的appname和icon。一开始我想着用之前的哪种方法应该可行，然后就试了下，一试不知道，试过就蛋疼了，appname是可以换掉，但是icon呢，，咋整，，，找了一圈方法，都木有找到，郁闷，string类型的，都可以通过常用的那种替换占位符的方式来改，或者是resValue(这个没有试，，但是似乎是可以的)。然后各种搜资料，有什么设置 useOldManifestMerger false ，不过自己没试成功（不能怪我，理论是这个道理的，只不过gradle版本不一样，Google升级了，导致我懵逼了）偶然间Google了一下  how to change app icon for diffrent productFlavors in android，点开第一个[链接](http://stackoverflow.com/questions/22875948/how-to-provide-different-android-app-icons-for-different-gradle-buildtypes),然后答主的答案给了我一些启发：


![QQ截图20160811184619.png](http://upload-images.jianshu.io/upload_images/1453857-3cbf6c005c8c592c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


然后搜啊搜，又找到了[这个](http://stackoverflow.com/questions/25981156/tools-replace-not-replacing-in-android-manifest)




![QQ截图20160811185123.png](http://upload-images.jianshu.io/upload_images/1453857-0c8656280ca0a302.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


然后经过自己的摸索，搞了出来了，简单来说：
大家看第一张图你的目录结构就会发现，如果我们在src/main/文件夹下新建以我们去渠道名为名的，目录结构和main一样的文件目录，当我们打包的时候就会读取这里面的资源文件，所以呢，我就新建了个清单文件，将头部换成图二所示，在根目录添加

                  xmlns:tools="http://schemas.android.com/tools"

然后application
  
          tools:replace="android:icon" 
          android:icon="@drawable/icon_all"

替换你需要换的icon就行，ok，至此搞定，至于为什么这么搞，我看到了官方的一篇文章[清单合并](http://tools.android.com/tech-docs/new-build-system/user-guide/manifest-merger#TOC-Manifest-files-ordering),另外还有篇国内[译文](http://blog.csdn.net/maosidiaoxian/article/details/42671999)


剩下的就靠小伙伴么自己摸索了。。。。