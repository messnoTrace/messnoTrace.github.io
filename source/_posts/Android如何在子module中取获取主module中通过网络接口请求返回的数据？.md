---
title: Android如何在子module中取获取主module中通过网络接口请求返回的数据？
date: 2019-04-05 10:35:18
tags:
---
<link href="http://cdn.bootcss.com/highlight.js/8.0/styles/monokai_sublime.min.css" rel="stylesheet">  
<script src="http://cdn.bootcss.com/highlight.js/8.0/highlight.min.js"></script>  
<script >hljs.initHighlightingOnLoad();</script>  
如题：今天遇到一个需求，在子module中获取主module中通过网络接口异步返回的数据？嗯。。绕了一下午把自己绕进去了，简单来说，就两个回调完事。一个request，一个response。但是胡思乱想了一下午，很是尴尬，废话不多说，咱直接上代码。
###首先一个request接口：
            public interface IRequest {
    public void request(IResponse response);
      }

###然后一个response接口

            public interface IResponse {
    void response(String result);
      }

###一个通信管理
            public class BridgeManager {
    private IRequest request;

    public void registerCallback(IRequest request) {
        this.request = request;
    }

    public IRequest getRequest() {
        return request;
    }

    public void setRequest(IRequest request) {
        this.request = request;
    }
    }
###为了方便扩展，我们再来一个通信管理的list的管理容器
            public class BrigeListManager {
    private static class SingletonHolder {

        private static final BrigeListManager sInstance = new BrigeListManager();
    }

    public static BrigeListManager getInstance() {
        return SingletonHolder.sInstance;
    }

    private BridgeManager bridgeManager;

    public BridgeManager getBridgeManager() {
        return bridgeManager;
    }

    public void setBridgeManager(BridgeManager bridgeManager) {
        this.bridgeManager = bridgeManager;
    }
     }

###   我们再来写一个主工程模拟网络请求的东东

          public class MainModuleImpl extends BridgeManager{
    public MainModuleImpl(){

        registerCallback(new IRequest() {
            @Override
            public void request(IResponse response) {

                try {
                    Thread.sleep(3000);

                    System.out.print("耗时任务执行完毕");
                    response.response("给子module返回结果");
                }catch (Exception e){

                }
            }
        });
    }
       }


###  然后随便在写一个主工程中初始化的地方,具体情况具体考察，我只模拟一下下
            public class MainModule {
    MainModule(){
        BrigeListManager.getInstance().setBridgeManager(new MainModuleImpl());
    }
     }


### 完事了，我们模拟一下子module中如何获取

              public class Test {
    public static void main(String args[]) {


        MainModule mainModule=new MainModule();
        IResponse response = new IResponse() {
            @Override
            public void response(String result) {
                System.out.print(result);
            }
        };
        BrigeListManager.getInstance().getBridgeManager().getRequest().request(response);

    }
       }


###### end

   ![WX20181129-212144@2x.png](https://upload-images.jianshu.io/upload_images/1453857-b084f81966313e54.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

嗯，，，美滋滋吧，虽说代码不难，但是不知道为什么，却让我想的很是难受，恼火，可能年纪大了，😔
 


