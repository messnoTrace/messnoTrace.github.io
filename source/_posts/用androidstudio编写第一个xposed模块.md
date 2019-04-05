---
title: 用androidstudio编写第一个xposed模块
date: 2019-04-05 10:36:55
tags:
---
<link href="http://cdn.bootcss.com/highlight.js/8.0/styles/monokai_sublime.min.css" rel="stylesheet">  
<script src="http://cdn.bootcss.com/highlight.js/8.0/highlight.min.js"></script>  
<script >hljs.initHighlightingOnLoad();</script>  
前提：你的手机装了xposedinstaller ，已经获取root权限，否则以下内容不用看了。
参考链接:https://blog.csdn.net/mrglaucusss/article/details/50963542

####配置准备工作
我的as版本是3.1的。所以基于此版本做操作
* 新建一个工程包名随便取。我取了com.notrace
* MainActivity代码简单如下:
            
                package com.notrace;
          import android.support.v7.app.AppCompatActivity;
        import android.os.Bundle;
        import android.view.View;
        import android.widget.Toast;

        public class MainActivity extends AppCompatActivity {

        @Override
          protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
            findViewById(R.id.buttonPanel).setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                Toast.makeText(MainActivity.this,hook(),Toast.LENGTH_SHORT).show();
                }
            });
        }
    
        public String hook(){
        return "未被劫持";
        }
      }

很简单，一个button点击弹出toast，显示hook函数返回值,我的目标就是修改这个hook的返回值

* 修改清单文件，在application节点下添加

                      <meta-data
            android:name="xposedmodule"
            android:value="true" />
        <meta-data
            android:name="xposeddescription"
            android:value="你猜猜" />
        <meta-data
            android:name="xposedminversion"
            android:value="54" />


第一个表示是否是xposed模块
第二个是描述，随便你写
第三个是最低的api版本支持

* 修改app/gradle

              repositories {
            jcenter()
        }
        dependencies {
          compileOnly 'de.robv.android.xposed:api:82'
          compileOnly 'de.robv.android.xposed:api:82:sources'
          implementation 'com.android.support:appcompat-v7:27.1.1'
          implementation 'com.android.support.constraint:constraint-layout:1.1.0'
          testImplementation 'junit:junit:4.12'
          androidTestImplementation 'com.android.support.test:runner:1.0.2'
          androidTestImplementation 'com.android.support.test.espresso:espresso-core:3.0.2'
  
        }

注意，上面两个一定要修改为compileOnly 如果你是低版本的as可以改成provided 


#####代码
* 新建HookTest 类：


              package com.notrace;

            import de.robv.android.xposed.IXposedHookLoadPackage;
            import de.robv.android.xposed.XC_MethodHook;
            import de.robv.android.xposed.XposedBridge;
          import de.robv.android.xposed.XposedHelpers;
          import de.robv.android.xposed.callbacks.XC_LoadPackage;

          public class HookTest implements IXposedHookLoadPackage {
              @Override
              public void handleLoadPackage(XC_LoadPackage.LoadPackageParam loadPackageParam)           throws Throwable {


        if(loadPackageParam.packageName.equals("com.notrace")){
            XposedBridge.log("NOTRACE"+loadPackageParam.packageName);

            Class clazz=loadPackageParam.classLoader.loadClass("com.notrace.MainActivity");

            XposedHelpers.findAndHookMethod(clazz, "hook", new XC_MethodHook() {
                @Override
                protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                    super.beforeHookedMethod(param);
                }

                @Override
                protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                    param.setResult("你已经被劫持了");
                }
            });
        }
    }
      }

* 添加xposed入口，新建assets：![微信截图_20180511182235.png](https://upload-images.jianshu.io/upload_images/1453857-7f9d3d644e31ea0b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

xposed_init内容如下：

          com.notrace.HookTest
        

* 至此结束，然后如果你没有禁用instant run 那么你就得打一个带签名的安装包，然后安装到手机上，然后再xposed installer模块中勾选你编写的module，然后手机重启生效。






