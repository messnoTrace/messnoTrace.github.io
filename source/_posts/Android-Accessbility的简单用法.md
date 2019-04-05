---
title: Android Accessbility的简单用法
date: 2019-04-05 11:33:38
tags:
---
<link href="http://cdn.bootcss.com/highlight.js/8.0/styles/monokai_sublime.min.css" rel="stylesheet">  
<script src="http://cdn.bootcss.com/highlight.js/8.0/highlight.min.js"></script>  
<script >hljs.initHighlightingOnLoad();</script>  
Accessbility 又叫做辅助功能，是Android官方推出帮助身体不便或者操作不灵活的人来辅助操作的，也可以用来干一些别的事，比如自动抢红包啊，静默安装点击啊等已知或者未知的应用。出于某种需求，就研究了下这个功能的简单用法，先做一个模拟按钮点击的效果。
  布局很简单，就不贴代码了，简单描述下，主界面就一个按钮，id随便你取，在Activity中设置点击事件，弹出一个Toast。

#####步骤
1. 创建CheckAccessbilityServices：
 CheckAccessbilityServices 继承自AccessibilityService，并在清单文件applcation节点中配置，并加入权限
    
          <service   
           android:name=".CheckAccessbilityServices"   
           android:enabled="true"        
          android:exported="true"
          android:label="测试点击"
          android:permission="android.permission.BIND_ACCESSIBILITY_SERVICE">  
              <intent-filter>      
                <action android:name="android.accessibilityservice.AccessibilityService" />
                </intent-filter> 
             <meta-data
                   android:name="android.accessibilityservice"
                   android:resource="@xml/check_accessibility_config" />
          </service>'
2.在res目录下创建文件夹xml，并创建步骤一中check_accessibility_config.xml

            <accessibility-service xmlns:android="http://schemas.android.com/apk/res/android"                   android:description="@string/check_click" 
                android:packageNames="com.notrace"      
            android:accessibilityEventTypes=    "typeAllMask|typeViewClicked|typeViewFocused|typeNotificationStateChanged|typeWindowStateChanged"    
              android:accessibilityFlags="flagDefault"   
              android:accessibilityFeedbackType="feedbackSpoken"    
              android:notificationTimeout="100"   
              android:canRetrieveWindowContent="true"    />
这里面有一些常用的属性，简单介绍下
          android:accessibilityEventTypes="typeAllMask"
看属性名也差不多可以明白，这个是用来设置响应事件的类型，typeAllMask当然就是响应所有类型的事件了。当然还有单击、长按、滑动等。 
          android:accessibilityFeedbackType="feedbackSpoken"
设置回馈给用户的方式

          android:notificationTimeout="100"
 响应时间的设置就不用多说了 
          android:packageNames="com.notrace"
可以指定响应某个应用的事件，我的demo包名就叫com.notrace,可以多个，用","隔开。
          android:description="模拟点击"

描述你在系统辅助功能开关中看到的描述

3.CheckAccessbilityServices  实现onAccessibilityEvent和onInterrupt方法
    
    
                    @Override  
                public void onAccessibilityEvent(AccessibilityEvent event) {  
                          //过滤包名
						String pkgName = event.getPackageName().toString();  
                          if(!"com.notrace".equals(pkgName))
                                return;
                        switch (type){  
                              case AccessibilityEvent.TYPE_WINDOW_STATE_CHANGED:      
                          //切换页面的时候此时会触发一个叫TYPE_WINDOW_STATE_CHANGED的事件
                            AccessibilityNodeInfo nodeInfo = getRootInActiveWindow(); 
                           if(nodeInfo!=null)  {
                               if("com.notrace.MainActivity".equals(event.getClassName())){    
                                    List<AccessibilityNodeInfo>     list=
                                    nodeInfo.findAccessibilityNodeInfosByViewId("com.notrace:id/btn_click");
                              if(list!=null&&list.size()>0)  {        
                              list.get(0).performAction(AccessibilityNodeInfo.ACTION_CLICK);       
                               }        
                          }       
                           break;   
                       case AccessibilityEvent.TYPE_WINDOW_CONTENT_CHANGED:   
                         break;
                      }                     
               }  


注意这里有个叫com.notrace:id/btn_click的，就是前面的那个按钮，我取id叫btn_click,这个东西可以通过eclipse提供的一个工具dump查看,如下图：

![QQ截图20160808162014.png](http://upload-images.jianshu.io/upload_images/1453857-78fa62630f5370b0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

微信抢红包的界面你就可以这么看了。
list.get(0).performAction(AccessibilityNodeInfo.ACTION_CLICK);      就是模拟点击事件




至此，模拟点击就已经全部完成了，我们打开手机辅助功能界面会看见：
![QQ截图20160808162331.png](http://upload-images.jianshu.io/upload_images/1453857-2b1084259b04dab0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
点击进去：
![QQ截图20160808162434.png](http://upload-images.jianshu.io/upload_images/1453857-cd8e0447e512c2ba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

打开这个开关就可以了。
至此，当页面切换到MainActivity中就可以弹出toast了，至于别的奇奇怪怪的功能，就需要小伙伴们自己摸索了。
[Demo代码](https://github.com/messnoTrace/Demo_Accessbility.git)
