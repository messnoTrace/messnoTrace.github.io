---
title: 多线程中wait/notify的一些理解，以及最经典的生产者和消费者
date: 2019-04-05 10:37:28
tags:
---

<link href="http://cdn.bootcss.com/highlight.js/8.0/styles/monokai_sublime.min.css" rel="stylesheet">  
<script src="http://cdn.bootcss.com/highlight.js/8.0/highlight.min.js"></script>  
<script >hljs.initHighlightingOnLoad();</script>  

###前言
说到wait，很容易想到一个常问的面试题，wait和sleep的区别。简单的理解就是，wait是object的方法，调用wait()，线程会进去等待状态，并且会释放出锁，非阻塞的，等待持有锁的线程调用notify()，才会继续执行。sleep是Thread的方法，sleep()会让当前线程暂停一段时间，但是不会释放锁，是阻塞的。

###Wait/notify在线程中的通信
我们先看下面一段代码：
 class Data:

            package com.notrace;
            import java.util.ArrayList;
            import java.util.List;
            public class Data {
	    private List list=new ArrayList();
	      public void add() {
		    list.add("notrace");
	    }
	      public int getSize() {
		return list.size();
	    }
      }


ThreadA：

      package com.notrace;     
      public class ThreadA extends Thread{
	 private Data data;
	 public ThreadA(Data data) {
		super();
		this.data=data;
	}
	
	@Override
	public void run() {
		try {
			for (int i = 0; i < 10; i++) {
				data.add();
				System.out.println("添加了"+(i+1)+"个元素");
				Thread.sleep(1000);
				
			}
		} catch (Exception e) {
		}
	}	
     }


ThreadB:

    package com.notrace;
    public class ThreadB extends Thread{

	private Data data;
	public ThreadB(Data data) {
		super();
		this.data=data;
	}
	
	@Override
	public void run() {
		try {
			while (true) {
		System.out.print("");
				if(data.getSize()==6) {
					System.out.println("==6,线程B要退出了");
					throw new InterruptedException();
				}
			}

		} catch (Exception e) {
			e.printStackTrace();
		}
	}
	
    }

Test:

      package com.notrace;
    public class Test {
	public static  void  main(String []args) {
		Data data=new Data();
		ThreadA threadA=new ThreadA(data);
		threadA.setName("A");
		threadA.start();
		ThreadB threadB=new ThreadB(data);
		threadB.setName("B");
		threadB.start();
	}
    }

运行test，结果如下：![1.png](https://upload-images.jianshu.io/upload_images/1453857-21d6e937cb766da6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

两个线程之间实现了通信，但是有一个弊端，就是线程B，要不停的循环，这样会很浪费资源，那么有没有什么好的办法去实现多个线程通信呢，答案是肯定的，就是wait/notify的机制。



###wait/notify机制简单介绍
举个栗子，就说我们日常生活中的餐厅吧，厨师和服务员之间的交互，
厨师做完菜，放到传递台上，服务员来传递台上取，往桌上送。
 1）厨师做完一道菜的时间是不确定的，所以把菜放到传递台上的时间也是不确定的。
2）服务员取菜的时间，取决于厨师，所以，厨师做完菜之前，服务员，就有了等待(wait)的状态。
3）服务员是怎么取到菜呢？这取决于厨师，简单点来说，厨师做完菜，放到传递台上，然后大喊一声：“红烧肉好了，来上菜。”然后服务员就过来取了，这个大喊一声，就相当于notify的过程。
4）这个情景中就有wait/notify的机制了，那么如何用代码来实现呢？
####wait/notify机制简单实现
方法wait（）的作用是使当前执行代码的线程进行等待，wait（）方法是Object类的方法，该方法用来将当前线程置入“预执行队列”中，并且在wait（）所在的代码行处停止执行，直到接到通知或被中断为止。在调用wait（）之前，**线程必须获得该对象的对象级别锁，即只能在同步方法或同步块中调用wait（）方法。在执行wait（）方法后，当前线程释放锁。** 在从wait（）返回前，线程与其他线程竞争重新获得锁。如果调用wait（）时没有持有适当的锁，则抛出IllegalMonitorStateException，它是RuntimeException的一个子类，因此，不需要try-catch语句进行捕捉异常。
**方法notify（）也要在同步方法或同步块中调用，即在调用前，线程也必须获得该对象的对象级别锁。如果调用notify（）时没有持有适当的锁，也会抛出IllegalMonitorStateException。**该方法用来通知那些可能等待该对象的对象锁的其他线程，如果有多个线程等待，则由线程规划器随机挑选出其中一个呈wait状态的线程，对其发出通知notify，并使它等待获取该对象的对象锁。需要说明的是，在执行notify（）方法后，当前线程不会马上释放该对象锁，呈wait状态的线程也并不能马上获取该对象锁，要等到执行notify（）方法的线程将程序执行完，也就是退出synchronized代码块后，当前线程才会释放锁，而呈wait状态所在的线程才可以获取该对象锁。当第一个获得了该对象锁的wait线程运行完毕以后，它会释放掉该对象锁，此时如果该对象没有再次使用notify语句，则即便该对象已经空闲，其他wait状态等待的线程由于没有得到该对象的通知，还会继续阻塞在wait状态，直到这个对象发出一个notify或notifyAll。
   用一句话来总结一下wait和notify：wait使线程停止运行，而notify使停止的线程继续运行。
代码测试如下：
ThreadA:
        
          package com.notrace;

    public class ThreadA extends Thread {
	private Object lock;

	public ThreadA(Object lock) {
		super();
		this.lock = lock;
	}

	@Override
	public void run() {
		try {
			synchronized (lock) {
				System.out.println("begin wait time" +       System.currentTimeMillis());
				lock.wait();
				System.out.println("end wait time" + System.currentTimeMillis());
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

    }

ThreadB:

            package com.notrace;

    public class ThreadB extends Thread {

	private Object lock;

	public ThreadB(Object lock) {
		super();
		this.lock = lock;
	}

	@Override
	public void run() {

		try {

			synchronized (lock) {
				System.out.println("begin notify time:" + System.currentTimeMillis());
				lock.notify();
				System.out.println("end nofity time:" + System.currentTimeMillis());
			}

		} catch (Exception e) {
			e.printStackTrace();
			System.out.println(e.toString());
		}
	}

    }

Test:

            package com.notrace;

public class Test {
	public static void main(String[] args) {

		try {
			Object lock = new Object();
			ThreadA threadA = new ThreadA(lock);
			threadA.setName("A");
			threadA.start();

			Thread.sleep(3000);
			ThreadB threadB = new ThreadB(lock);
			threadB.setName("B");
			threadB.start();
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}

	}

      }

运行结果如下图：
  ![2.png](https://upload-images.jianshu.io/upload_images/1453857-070a66f33dd162b9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从打印结果来看，3S后线程别nofity唤醒.
我们再来实现下之前的上面那个例子：
ThreadA:

            package com.notrace;

        public class ThreadA extends Thread {
	private Object lock;

	public ThreadA(Object lock) {
		super();
		this.lock = lock;
	}

	@Override
	public void run() {
	    try {
	        synchronized (lock) {
	        	if(Data.getSize()!=6)
	            System.out.println("begin wait time" +       System.currentTimeMillis());
	            lock.wait();
	            System.out.println("end wait time" + System.currentTimeMillis());
	        }
	    } catch (Exception e) {
	        e.printStackTrace();
	    }
	}
    }


ThreadB:

            package com.notrace;
      public class ThreadB extends Thread{

        private Object  lock;
      public ThreadB(Object lock) {
        super();
        this.lock=lock;
    }

    @Override
    public void run() {
        try {

        synchronized (lock) {
        	for (int i = 0; i < 10; i++) {
				Data.add();
				if(Data.getSize()==6) {
					lock.notify();
					System.out.println("发出通知");
				}
				System.out.println("添加了"+(i+1)+"个元素");
				Thread.sleep(1000);
			}        
          
        }

    } catch (Exception e) {
        e.printStackTrace();
        System.out.println(e.toString());
    }
    }

    }


Data:

            package com.notrace;

      import java.util.ArrayList;
      import java.util.List;

      public class Data {
	private static List list = new ArrayList();

	public static void add() {
		list.add("notrace");
	}

	public static int getSize() {
		return list.size();
	}
        }


Test:

          package com.notrace;

        public class Test {
	public static void main(String[] args) {

		try {
			Object lock = new Object();
			ThreadA threadA = new ThreadA(lock);
			threadA.setName("A");
			threadA.start();

			Thread.sleep(50);
			ThreadB threadB = new ThreadB(lock);
			threadB.setName("B");
			threadB.start();
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}

	}

    }

运行结果如下:
![3.png](https://upload-images.jianshu.io/upload_images/1453857-584bcf34b652abb9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

日志说明了一个问题,notify()调用之后，并不会立即释放锁，而是会接着执行后面的代码，执行完了，才会释放锁。

synchronized可以将任何一个Object对象作为同步对象来看待，每个Object都实现了wait()和notify()。他们必须用被用在synchronized同步的区域内，wait()让区域内的线程进入等待状态，释放同步锁，notify()可以唤醒一个因调用了wait()而处于等待状态中的线程。被唤醒的线程继续执行wait()后面的代码。如果notify之后没有处于阻塞等待中的线程，就会被忽略。
关于释放锁，有以下几个结论，
1）执行完同步代码块，就会释放锁。
2）执行代码块的过程中，如果遇到异常而导致线程终止，也会释放锁。
3）调用wait()这个线程会释放锁。

另外还有一点，需要提一下。notify()只会唤醒一个线程，所以在多线程的时候，可能会出现一些问题，所以，尽量用notifyAll()唤醒所有等待的线程 比较好。下面来看看另一个经典问题。
###生产者/消费者模式
###一个生产者，一个消费者
废话不多说，直接看code.
生产者P:

                package com.notrace;

      //生产者
      public class P {
	private String lock;
	public P(String lock) {
		super();
		this.lock=lock;
	}
	public void setValue() {
		try {
			synchronized (lock) {
				if(!ValueObj.value.equals("")) {
					lock.wait();
				}
				String value=System.currentTimeMillis()+"";
				System.out.println("set的值："+value);
				ValueObj.value=value;
				lock.notify();
			}
		} catch (Exception e) {
			// TODO: handle exception
		}
	}
    }


消费者C:
        
      package com.notrace;

        //消费者
        public class C {
	private String lock;
	public C(String lock) {
		super();
		this.lock=lock;
	}
	public void getValue() {
	try {
		synchronized (lock) {
			if(ValueObj.value.equals("")) {
				lock.wait();
			}
			System.out.println("get的值:"+ValueObj.value);
			ValueObj.value="";
			lock.notify();
		}
	} catch (Exception e) {
		// TODO: handle exception
	}
	}

    }

线程P：

        package com.notrace;

    public class ThreadP extends Thread{

	private P p;
	public ThreadP(P p) {
		super();
		this.p=p;
	}
	@Override
	public void run() {
		while(true) {
			p.setValue();
		}
	}
    }
线程C：

                package com.notrace;

      public class ThreadC extends Thread{
	private C c;
	public ThreadC(C c) {
		super();
		this.c=c;
	}
	@Override
	public void run() {
		while(true) {
			c.getValue();
		}
	}
    }



Run：

              package com.notrace;

    public class Run {
	public static void main(String []args) {
		String lock=new String("");
		P p=new P(lock);
		C c=new C(lock);
		ThreadP tP=new ThreadP(p);
		ThreadC tC=new ThreadC(c);
		tP.start();
		tC.start();
	}

    }

运行结果：
![4.png](https://upload-images.jianshu.io/upload_images/1453857-eb6739d745cb9929.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看出来，是生产者，消费者，交替运行的。

接下来变化一下:
######多生产者，多消费者

P:

        package com.notrace;

      //生产者
      public class P {
	private String lock;
	public P(String lock) {
		super();
		this.lock=lock;
	}
	public void setValue() {
		try {
			synchronized (lock) {
				while(!ValueObj.value.equals("")) {
					System.out.println("生产者:"+Thread.currentThread().getName()+"watting*");
					lock.wait();
				}
				System.out.println("生产者:"+Thread.currentThread().getName()+"run*");
				String value =System.currentTimeMillis()+"";
				ValueObj.value=value;
				lock.notify();
			}
		} catch (Exception e) {
			// TODO: handle exception
		}
	}
    }

C:
      
      package com.notrace;

    //消费者
      public class C {
	private String lock;
	public C(String lock) {
		super();
		this.lock=lock;
	}
	public void getValue() {
	try {
		synchronized (lock) {
			while(ValueObj.value.equals("")) {
				System.out.println("消费者:"+Thread.currentThread().getName()+"waitting");
				lock.wait();
			}
			System.out.println("消费者:"+Thread.currentThread().getName()+"run");
			System.out.println("get的值:"+ValueObj.value);
			ValueObj.value="";
			lock.notify();
		}
	} catch (Exception e) {
		// TODO: handle exception
	}
	}

    }

threadP，threadC一样:

Run:

            package com.notrace;

      public class Run {
	public static void main(String []args) {
		try {
			String lock=new String("");
			P p=new P(lock);
			C c=new C(lock);
			
			ThreadP[] pThreadPs=new ThreadP[2];
			ThreadC[]cThreadCs=new ThreadC[2];
			for(int i=0;i<2;i++) {
				pThreadPs[i]=new ThreadP(p);
				pThreadPs[i].setName("生产者"+(i+1));
				cThreadCs[i]=new ThreadC(c);
				cThreadCs[i].setName("消费者"+(i+1));
				pThreadPs[i].start();
				cThreadCs[i].start();
			}

			Thread.sleep(5000);	
		} catch (Exception e) {
			// TODO: handle exception
		}
		
		
	}

      }


执行完：
![5.png](https://upload-images.jianshu.io/upload_images/1453857-c2d2686d93ecd77d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


两个都是等待，而且还没执行完，是不是很奇怪。。。产生这种情况是因为唤醒的早了点， 生产者1唤醒了生产者2，然而生产者2发现产品并未被消费，所以生产者2也是wait状态。另外，还有种原因，就是连续唤醒同类。解决这种问题，就需要将P和C中的notify改成notifyAll()，将生产者和消费者一起唤醒，就ok了。


咱接着变
######一生产一消费：来操作栈
这个场景就是生产者向堆栈List中放数据，消费者从List中取数据。list最大容量是1:
p:

            package com.notrace;

      //生产者
      public class P {
	
	
	private MyStack stack;
	public P(MyStack stack) {
		super();
		this.stack=stack;
		
	}
	public void pushService() {
		stack.push();
	}
    }


C:
  
            package com.notrace;

      //消费者
    public class C {
	
	private MyStack stack;
	public C(MyStack stack) {
		super();
		this.stack=stack;
	}
	
	public void popService() {
		System.out.println("pop="+stack.pop());
	}
      }


MyStack:


            package com.notrace;

      import java.util.ArrayList;
      import java.util.List;

      public class MyStack {
      private List list=new ArrayList<>();
      synchronized public void push() {
	try {
		if(list.size()==1) {
			this.wait();
		}
		list.add("notrace:"+Math.random());
		this.notify();
		System.out.println("push="+list.size());
	} catch (Exception e) {
		// TODO: handle exception
	}
      }

          synchronized public String pop() {
	String value="";
	try {
		if(list.size()==0) {
			System.out.println("pop操作"+Thread.currentThread().getName());
			this.wait();
		}
		value=""+list.get(0);
		list.remove(0);
		this.notify();
		System.out.println("pop"+list.size());
	} catch (Exception e) {
		// TODO: handle exception
	}
	return value;
      }
    }


ThreadP:

            package com.notrace;

      public class ThreadP extends Thread{

	private P p;
	public ThreadP(P p) {
		super();
		this.p=p;
	}
	@Override
	public void run() {
		while(true) {
			System.out.print("");
			p.pushService();
    //			p.setValue();
			
		}
	}
    }


ThreadC:

            package com.notrace;

    public class ThreadC extends Thread{
	private C c;
	public ThreadC(C c) {
		super();
		this.c=c;
	}
	@Override
	public void run() {
		while(true) {
			System.out.print("");
			c.popService();
      //			c.getValue();
		}
	}
      }

Run:

            package com.notrace;

public class Run {
	public static void main(String []args) {
		
		
		MyStack stack=new MyStack();
		P p=new P(stack);
		C c=new C(stack);
		ThreadP threadP=new ThreadP(p);
		ThreadC threadC=new ThreadC(c);
		threadP.start();
		threadC.start();
    }
    }

执行结果：
![6.png](https://upload-images.jianshu.io/upload_images/1453857-eb44c5bbfbef7611.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

程序的结果是size不会大于1|。

###一生产多消费：来操作栈
代码还是上面的那些，我们改一下Run：
            
            package com.notrace;

    public class Run {
	public static void main(String []args) {
		
		
		MyStack stack=new MyStack();
		P p=new P(stack);
		C c1=new C(stack);
		C c2=new C(stack);
		C c3=new C(stack);
		C c4=new C(stack);
		C c5=new C(stack);
		ThreadP threadP=new ThreadP(p);
		ThreadC threadC1=new ThreadC(c1);
		ThreadC threadC2=new ThreadC(c2);
		ThreadC threadC3=new ThreadC(c3);
		ThreadC threadC4=new ThreadC(c4);
		ThreadC threadC5=new ThreadC(c5);
		threadP.start();
		threadC1.start();
		threadC2.start();
		threadC3.start();
		threadC4.start();
		threadC5.start();
      }
      }
执行结果：![7.png](https://upload-images.jianshu.io/upload_images/1453857-e4a1b50023c36141.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

假死状态了，多次执行。还可能会出现indexoutofbounce，这是因为条件改变时并没有得到及时的响应，引起了list.remove(0)报错，解决办法是将MyStack中的if()判断改为while()。多消费一生产，多消费多生产，也是一样的。改一下run里面的代码就行了

好了，基本完事。



  





  



                



