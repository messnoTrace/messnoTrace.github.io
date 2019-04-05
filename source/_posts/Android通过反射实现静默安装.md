---
title: Android通过反射实现静默安装
date: 2019-04-05 10:35:18
tags
---
本文讲解通过反射Android pm  instal来调用隐藏api， 来静默安装，至于原理什么的，就不多说了，网络上一搜一大把，下面是亲测可行的；
先上[Demo](https://github.com/messnoTrace/SlienceInstall),其中的libs文件夹下的class.jar是主角。
本文有一个大前提，那就是你的apk是放在系统/system/priv-app目录下，也就是说，rom是你们自己搞的，手动滑稽=。= 
如果你是用AndroidStudio版本的，也是链接中的[DemoSilenceInstall](https://github.com/messnoTrace/SlienceInstall/tree/master/DemoSilenceInstall)
步骤如下：
 * 建一个你自己的工程 ，将class.jar放入到libs目录下。
 * 然后projectStructure(按F4直接进)，进入dependence点击+号，选择FileDependence 记得把 scope置为Provided，如下图:
![1.png](http://upload-images.jianshu.io/upload_images/1453857-d74f2be33a1b022c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 工程目录下的gradle文件修改：
              allprojects {  
                repositories {      
                          jcenter()   
                       }  
                          gradle.projectsEvaluated {             
                                   tasks.withType(JavaCompile) {     
                                                 options.compilerArgs.add('-Xbootclasspath/p:app\\libs\\class.jar')                  
                                }  
                               }
                          }

     **看好名字,不要眼瞎，后面是你起的jar包的名字，这个名字随意取，两者对应就行，**

    以上的操作的目的有两个：
      1.就是将class.jar以eclipse中那种userlib的形式导入，
      2.调整jar包的优先级
* 再将demo中install 包下的代码放到你的工程中.如果这个时候你的PM.java这个文件不报错，那么恭喜，你的操作就算完成了
* 这个步骤我的代码出现了问题，不知道你的会不会，就是Android中65535问题了，也就是解决这个问题做的操作，AS下好解决，这个不做过多解释。


剩下的，就是将应用打包签名，放到/system/priv-app这个目录下 就ok了，

下面说下Eclipse版本的操作，代码都是一样的，jar包也是一样，就两点，jar包通过userlibrary的形式导入，直接放图吧，多图慎入：
![
![Uploading 4_049957.png . . .]
](http://upload-images.jianshu.io/upload_images/1453857-e9e8d436fbd4ddde.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![3.png](http://upload-images.jianshu.io/upload_images/1453857-d5b134f0db69b9a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![4.png](http://upload-images.jianshu.io/upload_images/1453857-cecd54290fd90360.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![

![Uploading 6_086779.png . . .]
](http://upload-images.jianshu.io/upload_images/1453857-e6cde6675deb959b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![6.png](http://upload-images.jianshu.io/upload_images/1453857-16490fdfd7ab0de6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)






![7.png](http://upload-images.jianshu.io/upload_images/1453857-73ac54c47095b33b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



![8.png](http://upload-images.jianshu.io/upload_images/1453857-c524e6d1cb374503.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![9.png](http://upload-images.jianshu.io/upload_images/1453857-c57017127dc439c4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



![
![Uploading 10_162291.png . . .]
](http://upload-images.jianshu.io/upload_images/1453857-d0b9e66477c90d60.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![11.png](http://upload-images.jianshu.io/upload_images/1453857-8fd7af66b59e3784.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)















至此，，搞定了，╮(╯▽╰)╭，继续搬砖。如果你要问这个class怎么来的，你找做rom的大神们给你编译一份源码就行，反正我不会。OTZ.... 然而这个地方在我写完文章后又出现了一个实战问题，那就是用multydex分包处理后，还是会出现 [Too many classes in --main-dex-list, main dex capacity exceeded](http://stackoverflow.com/questions/32721083/too-many-classes-in-main-dex-list-main-dex-capacity-exceeded)，，目前一个最简单暴力的方法就是把minsdkversion 调成21+，我在想想办法，，，，解决了更新文章。


(⊙o⊙)…，暂时没解决，不过上了一个新版本，用aidl实现的，也是可以用[这里的](https://github.com/messnoTrace/SlienceInstall/tree/master/MyApplication)...东西都差不多，jar包换了个精简的，然后改用aidl实现的注意，aidl的包名不要动，，，然后配置下gradle里面的aidl