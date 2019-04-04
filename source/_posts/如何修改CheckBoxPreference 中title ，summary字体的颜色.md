以前没怎么关注过PreferenceActivity的一些用法只是简单了解下，然后刚好今天遇到了某个机型上出现了问题：**我明明设置的是黑色的字体，在这个机器上却是白色的**，然后各种摸索得出了个推论，页面的字体颜色什么的，可能是更随系统来的，受系统调控：
我是这么写的，

                <CheckBoxPreference    
                android:defaultValue="true"   
               android:key="weather_use_metric"  
              android:textColor="@color/black"
                android:title="@string/weather_use_metric" />

问题出现 了那没办法，只能想办法改了，好在现在学会了面向google编程，然后搜的一下，找到了个解决方案，先上怎么解决，然后再说为什么这么干。首先建个layout ，名字随便取（我取名custom_preferece_layout.xml），内容如下:

              <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android" 
                 android:layout_width="match_parent"
               android:layout_height="wrap_content"   
               android:minHeight="?android:attr/listPreferredItemHeight" 
               android:gravity="center_vertical" 
               android:paddingRight="?android:attr/scrollbarSize">  
            <RelativeLayout android:layout_width="wrap_content"        
              android:layout_height="wrap_content" android:layout_marginLeft="15dip"   
             android:layout_marginRight="6dip"
             android:layout_marginTop="6dip"      
             android:layout_marginBottom="6dip" 
             android:layout_weight="1">     
            <TextView android:id="@+android:id/title"       
               android:layout_width="wrap_content" 
             android:layout_height="wrap_content"         
             android:singleLine="true" 
              android:textAppearance="?android:attr/textAppearanceLarge"        
              android:ellipsize="marquee" android:fadingEdge="horizontal"         
             android:textColor="@color/black" />    
            <TextView android:id="@+android:id/summary"     
             android:layout_width="wrap_content" 
            android:layout_height="wrap_content"          
            android:layout_below="@android:id/title" 
            android:layout_alignLeft="@android:id/title"           
           android:textAppearance="?android:attr/textAppearanceSmall"        
           android:maxLines="4" />   
       </RelativeLayout>    
    <!-- Preference should place its actual preference widget here. -->  
        <LinearLayout android:id="@+android:id/widget_frame"     
           android:layout_width="wrap_content"
           android:layout_height="match_parent"        
          android:gravity="center_vertical" android:orientation="vertical" />
      </LinearLayout>

在这个布局中修改title，sunmmary的颜色就可以了。
然后再CheckBoxPreference    中加一句：


          <CheckBoxPreference   
           android:defaultValue="true"   
             android:key="weather_use_metric"  
            android:layout="@layout/custom_preferece_layout" 
            android:textColor="@color/black"   
               />
然后，就解决了这个问题，那为什么这么搞呢？进入CheckBoxPreference   顶层父类Preference中看，看到构造中的注释了：


![QQ截图20161110175559.png](http://upload-images.jianshu.io/upload_images/1453857-fca483c577435efc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


这玩意默认的控制了样式，然后再往下看到一个方法：


![QQ截图20161110175911.png](http://upload-images.jianshu.io/upload_images/1453857-54f7fd8444a953dc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

他说大部分情况足够用了，但是这不是出来问题么，于是到android的values中的attr下找，


![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1453857-e5a69f67b3b10c3d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

嗯很贴心，行数都截出来了。顶层默认实现了个布局，在CheckBoxPreference的onBindView方法中出现了个id和一个方法：
![QQ截图20161110181103.png](http://upload-images.jianshu.io/upload_images/1453857-30381fbc2e2d3ff8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

id点进去能找到个布局，然后再父类TwoStatePreference中可以看到这个方法里面写了什么，注意看上面的布局控件的id，你没看错，就是覆盖了preference_material.xml 中的布局：


![QQ截图20161110181315.png](http://upload-images.jianshu.io/upload_images/1453857-c2fb7896c6189aba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


附上答案链接 [http://stackoverflow.com/questions/4469514/how-to-customize-text-color-of-the-checkboxpreference-title](http://stackoverflow.com/questions/4469514/how-to-customize-text-color-of-the-checkboxpreference-title)