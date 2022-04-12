## 目录

​		线程的实现方式

​		线程同步

​		生产者消费者模型

## 今日目标

```
【应用】能够独立完成实现多线程的三种方式
【应用】能够理解线程安全问题并解决
【应用】能够独立完成生产者消费者案例
```

## 01. 多线程概述

##### 01-多线程概述-初步了解多线程  [时长:2 '29]

[问题]

1. 什么是多线程?
2. 如何保证多个软件能够同时运行?

[答案]

1. 什么是多线程?

   ```
   是指从软件或者硬件上实现多个线程并发执行的技术
   ```
   
2. 如何保证多个软件能够同时运行?

   ```
   每个软件都是一个独立的进程,CPU在多个进程之间进行高速的切换,从而实现多个软件同时运行
   注意:只是看起来是同时运行,但是在同一时刻只有一个软件在真正的运行
   ```


##### 02-多线程概述-并发和并行 [时长:2 '18]

[问题]

1. 什么是并发?
2. 什么是并行?

[答案]

1. 什么是并发?

   ```java
   在同一时刻，有多个指令在单个CPU上[交替]执行
   ```
   
2. 什么是并行?

   ```
   在同一时刻，有多个指令在多个CPU上[同时]执行
   ```


##### 03-多线程概述-进程和线程 [时长:4 '20]

[问题]

1. 什么是进程?

2. 进程有哪些特点?
3. 什么是线程?
4. 进程和线程的关系

[答案]

1. 什么是进程?

   ```java
   正在运行的软件
   ```
   
2. 进程有哪些特点?

   ```
   独立性：进程是一个能独立运行的基本单位，同时也是系统分配资源和调度的独立单位
   动态性：进程的实质是程序的一次执行过程，进程是动态产生，动态消亡的
   并发性：任何进程都可以同其他进程一起并发执行
   ```

3. 什么是线程?

   ```
   是进程中的单个顺序控制流，是一条执行路径
   简单理解:线程是正在运行的软件中的某个独立功能
   ```
   
4. 进程和线程的关系

   ```
   单线程程序：一个进程如果只有一条执行路径(线程)，则称为单线程程序
   多线程程序：一个进程如果有多条执行路径(线程)，则称为多线程程序
   ```

[补充]	

```
多线程的应用场景举例:
	1.多线程处理后台任务
       例如:新游戏开服,有很多玩家到游戏官网注册账号
        	一个玩家在注册的同时,其他的玩家依然可以访问官网,同时也可以进行注册操作
        	多个玩家的操作之间独立且互不干扰

    2.多线程异步处理任务
       例如:博学谷考试,考试结束后需要将答案压缩包进行上传
           	其实,文件上传是很复杂的操作,为了不影响用户体验,先提示用户上传成功
            再在后台去验证文件类型和内容并进行解析上传
            提示用户和后台上传文件不是同步执行

    3.多线程分布式计算
        例如:线程处理的任务过于复杂,消耗资源量很大
            此时,将复杂的任务拆解为多个小的任务,并分配给多条线程进行处理,
            最终将多条线程处理结果进行整合
```

## 02.多线程的实现方式

##### 04-多线程的实现方式-继承Thread类 [时长:4 '15]

[问题]

1. 多线程有哪些实现方式?
2. 描述继承Thread类方式的操作步骤

[答案]

1. 多线程有哪些实现方式?

   ```
   继承Thread类的方式进行实现
   实现Runnable接口的方式进行实现
   利用Callable和Future接口方式实现
   ```

2. 描述继承Thread类实现方式的操作步骤

   ```
   继承Thread类
   	定义一个类MyThread继承Thread类
   	在MyThread类中重写run()方法
   	创建MyThread类的对象
   	启动线程start()
   ```

##### 05-多线程的实现方式-继承Thread类的两个问题 [时长:3 '03]

[问题]

1. 为什么要重写run()方法？ 
2. run()方法和start()方法的区别？ 

[答案]

1. 为什么要重写run()方法？

   ```
   因为run()是用来封装被线程执行的代码
   ```

2. run()方法和start()方法的区别？  

   ```
   run()：封装线程执行的代码，直接调用，相当于普通方法的调用，并[没有开启线程]。
   start()：[启动线程]然后由JVM调用此线程的run()方法
   ```

[练习]

```java
1.操作步骤:
	定义一个类MyThread继承Thread类
	在MyThread类中重写run()方法
	创建MyThread类的对象
	启动线程
2.示例代码:
[MyThread.java]
//继承Thread类
public class MyThread extends Thread{
	//重写run()方法
    @Override
    public void run() {//run()方法内为线程执行的任务
        for (int i = 0; i < 100; i++) {
            System.out.println("线程开启了" + i);
        }
    }
}
[Demo.java]
public class Demo {
    public static void main(String[] args) {
    	//创建线程对象
        MyThread t1 = new MyThread();
        MyThread t2 = new MyThread();
		//开启线程
        t1.start();
        t2.start();
    }
}
```

##### 06-多线程的实现方式-实现Runnable接口 [时长:5 '07]

[问题]

1. 描述实现Runnable接口实现方式的操作步骤

[答案]

1. 描述实现Runnable接口实现方式的操作步骤

   ```
   实现Runnable接口
   	定义一个类MyRunnable实现Runnable接口
   	在MyRunnable类中重写run()方法 -- 线程启动之后执行的任务
   	创建MyRunnable类的对象
   	创建Thread类的对象，把MyRunnable对象作为构造方法的参数
   		目的:将Runnable接口实现类对象[转换]为Thread类对象,为了调用start()方法开启线程
   	启动线程
   ```
```java
   [练习] 
   1.操作步骤
   	定义一个类MyRunnable实现Runnable接口
   	在MyRunnable类中重写run()方法
   	创建MyRunnable类的对象
   	创建Thread类的对象，把MyRunnable对象作为构造方法的参数
   	启动线程
   2.示例代码
   [MyRunnable.java]
   public class MyRunnable implements Runnable{
       @Override
       public void run() {
           for (int i = 0; i < 100; i++) {
               System.out.println(Thread.currentThread().getName() + "---" + i);
           }
       }
   }
   [Demo.java]
   public class Demo {
       public static void main(String[] args) {
       	//创建Runnable实现类对象作为参数传递给Thread类构造方法
           MyRunnable mr = new MyRunnable();
           Thread t1 = new Thread(mr);
           t1.start();
   
           MyRunnable mr2 = new MyRunnable();
           Thread t2 = new Thread(mr2);
           t2.start();
       }
   }
```


##### 07-多线程的实现方式-实现Callable接口 [时长:9 '37]

[问题]

1. 描述实现Callable接口实现方式的操作步骤


[答案]

1. 描述实现Callable接口实现方式的操作步骤

   ```
   实现Callable接口
   	定义一个类MyCallable实现Callable接口
   	在MyCallable类中重写call()方法
   		注意:call方法是有返回值的,返回值类型和实现Callable接口泛型类型一致
   	创建MyCallable类的对象
   	创建Future的实现类FutureTask对象， 把MyCallable对象作为构造方法的参数
   		注意:Thread类构造只能接受Runnable实现类对象,不能接受Callable实现类对象
   		需要转换,转换成FutureTask对象,因为FutureTask实现类Runnable接口
   		同时,FutureTask对象可以调用get()方法获取线程执行之后的结果
   	创建Thread类的对象，把FutureTask对象作为构造方法的参数
   	启动线程,调用Thread类的start()
   	调用get方法,获取线程结束之后的结果
   		注意:get()方法是阻塞方法,如果没有开启线程或者线程没有执行完毕,此时调用get()
   		方法会阻塞(一直等)等线程执行完毕,获取结果
   		
   		一定要先开启线程执行call(),再调用get()
   ```

[练习]

```java
1.操作步骤
	定义一个类MyCallable实现Callable接口
	在MyCallable类中重写call()方法
	创建MyCallable类的对象
	创建Future的实现类FutureTask对象， 把MyCallable对象作为构造方法的参数
	创建Thread类的对象，把FutureTask对象作为构造方法的参数
	启动线程
	调用get方法,获取线程结束之后的结果
2.示例代码
[MyCallable.java]
public class MyCallable implements Callable<String> {//泛型类型为返回值类型
    @Override
    public String call() throws Exception {
        for (int i = 0; i < 100; i++) {
            System.out.println("跟女孩表白" + i);
        }
        //返回值表示线程运行完毕之后的结果
        return "答应";
    }
}
[Demo.java]
public class Demo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        //创建Callable实现类对象
        MyCallable mc = new MyCallable();
        //将实现类对象转换为FutureTask对象
        FutureTask<String> ft = new FutureTask<>(mc);
        //将转换得到的FutureTask对象作为参数传递给Thread类的构造方法
        Thread t1 = new Thread(ft);
		//开启线程
        t1.start();
        //获取线程执行完毕的结果
        String s = ft.get();
        System.out.println(s);
    }
}	
```

##### 08-三种实现方式的对比 [时长:3 '43]

[问题]

1. 三种线程实现方式的好处和弊端


[答案]

1. 三种线程实现方式的好处和弊端

   |                             | 优点                                   | 缺点                                              |
   | --------------------------- | -------------------------------------- | ------------------------------------------------- |
   | 继承Thread类                | 代码简单，可以直接使用Thread类中的方法 | 扩展性差,不能再继承其他的类                       |
   | 实现Runnable、 Callable接口 | 扩展性强，不影响继承关系使用           | 代码复杂，不能直接使Thread类中的方法,需要进行转换 |



## 03. Thread类相关方法

##### 09-Thread类方法-设置及获取线程名称 [时长:7 '57]

[问题]

1. 如何获取线程名称
2. 如何设置线程名称

[答案]

1. 如何获取线程名称

   ```java
   方法:String getName()：返回此线程的名称
   使用:
   	1.创建Thread类对象,使用对象名.进行调用
   	2.创建Thread类的子类对象,使用对象名.进行调用
   	
   	注意:
   	如果没有设置线程名称,此时获取默认线程名称
   	格式:Thread-编号
   	编码默认从0开始,每创建一条线程,编号+1
   ```

2. 如何设置线程名称

   ```java
   方法:
   	1.void setName(String name)：将此线程的名称更改为参数 name
   		注意:
   			只有Thread类及其子类能直接使用
   	2.通过构造方法设置线程名称
   		new Thread(String name)
   		new Thread的子类(String name)
           注意:子类需要手动给出空参和带有String name参数的构造方法        	
   ```

##### 10-Thread类方法-获得线程对象 [时长:3 '02 ]

[问题]

1. 获得线程对象方法的作用

[答案]

1. 获得线程对象方法的作用

   ```java
   方法:public static Thread currentThread()：返回对当前正在执行的线程对象的引用
   作用:解决多线程的接口实现方式中,无法直接使用Thread类方法的问题
   ```



##### 11-Thread类方法-线程休眠sleep [时长:05' 07]

[问题]

1. 线程休眠方法sleep的作用

[答案]

1. 线程休眠方法sleep的作用

```java
方法:public static void sleep(long time)：让线程休眠指定的时间，单位为毫秒。
	注意:睡眠指定时间之后,自动唤醒线程 -- 计时等待
```



##### 12-Thread类方法-线程优先级 [时长:10 '07]

[问题]

1. 线程的调度方式有哪些,各自有什么特点?
2. Java采用的是哪种线程调度方式?
3. 如何设置及获取线程优先级?
4. 线程优先级的意义

[答案]

1. 线程的调度方式有哪些,各自有什么特点?

   ```
   分时调度模型：所有线程轮流使用CPU的使用权，平均分配每个线程占用 CPU 的时间片
   抢占式调度模型：优先让优先级高的线程使用CPU，如果线程的优先级相同，那么会随机选择一个，优先级高的线程
   获取的CPU时间片相对多一些
   ```

2. Java采用的是哪种线程调度方式?

   ```
   Java使用的是抢占式调度模型
   ```

3. 如何设置及获取线程优先级?

   ```java
   设置线程优先级:public final void setPriority(int newPriority)
   获取线程优先级:public final int getPriority()
   注意:
   	线程优先级默认值为:5
       线程优先级的范围为:[1,10]    
   ```

4. 线程优先级的意义

   ```java
   线程优先级越高,获取CPU执行权的[概率]就越高
   ```

##### 13-Thread类方法-设置守护线程 [时长:6 '04]

[问题]

1. 什么是守护线程?
2. 如何设置守护线程?

[答案]

1. 什么是守护线程?

   ```
   陪伴主线程执行任务的线程
   ```

2. 如何设置守护线程?	

   ```java
   方法:public final void setDaemon(boolean on)：设置为守护线程
   ```

[补充]	

```
Java中的常见守护线程:
	垃圾回收器gc():伴随着执行任务的线程执行,并在CPU空闲的时候进行垃圾回收
				当执行任务的线程执行完毕,垃圾回收器会在执行一会儿再结束
```



## 04.线程安全问题

##### 14-线程安全问题-卖票案例实现 [时长:7 '23]

[需求]

```
某电影院目前正在上映国产大片，共有100张票，而它有3个窗口卖票，请设计一个程序模拟该电影院卖票
```

[问题]

1. 分析多线程卖票案例的实现思路

[答案]

1. 分析多线程卖票案例的实现思路

   ```java
   ① 定义一个类Ticket实现Runnable接口，里面定义一个成员变量： 
   	private int ticketCount = 100;
   ② 在Ticket类中重写run()方法实现卖票，代码步骤如下
       1.判断票数大于0，就卖票，并告知是哪个窗口卖的
       2.票数要减1
       3.卖光之后，线程停止
   ③ 定义一个测试类TicketDemo，里面有main方法，代码步骤如下
       1.创建Ticket类的对象
       2.创建三个Thread类的对象，把Ticket对象作为构造方法的参数，并给出对应的窗口名称
       	注意:此时实现接口的方式,只创建一个参数对象,所以票数量定义为成员变量
       	因为对象只创建一个,三条线程操作的是同一个成员变量的值
       	注意:如果使用继承方式实现多线程,此时共享数据需要定义为
       	private static int ticketCount = 100;
   		原因:继承方式,创建三个Thread子类对象,相当于要初始化三个ticketCount
   		为了让三条线程使用同一个ticketCount,.使用static将数据变成所有对象共享的数据
       3.启动线程
   ```

[练习]

```java
1.操作步骤
① 定义一个类Ticket实现Runnable接口，里面定义一个成员变量： 
	private int ticketCount = 100;
② 在Ticket类中重写run()方法实现卖票，代码步骤如下
    1.判断票数大于0，就卖票，并告知是哪个窗口卖的
    2.票数要减1
    3.卖光之后，线程停止
③ 定义一个测试类TicketDemo，里面有main方法，代码步骤如下
    1.创建Ticket类的对象
    2.创建三个Thread类的对象，把Ticket对象作为构造方法的参数，并给出对应的窗口名称
    3.启动线程
2.示例代码
[Ticket.java]
public class Ticket implements Runnable {//使用多线程第二种实现方式,此时Ticket对象作为参数传递
    //票的数量
    private int ticket = 100;//该成员变量因为只创建一次Ticket对象,所以只会初始化一次,让多条线程使用同一个ticket数据
    @Override
    public void run() {
    	//循环卖票
        while(true){//该循环的作用是:线程执行完毕当次打印任务之后不会消失,继续回到该位置等待
            if(ticket <= 0){//票数<=0证明没有票,结束循环
                break;//所有任务执行完毕,结束循环
            }else{
                ticket--;
                System.out.println(Thread.currentThread().getName() + "在卖票,还剩下" + ticket + "张票");
            }
        }
    }
}
[TicketDemo.java]
public class Demo {
    public static void main(String[] args) {
    	//创建Runnable接口实现类作为参数
        Ticket ticket = new Ticket();
        Thread t1 = new Thread(ticket);
        Thread t2 = new Thread(ticket);
        Thread t3 = new Thread(ticket);
		//设置线程名称
        t1.setName("窗口一");
        t2.setName("窗口二");
        t3.setName("窗口三");
		//开启线程
        t1.start();
        t2.start();
        t3.start();
    }
}
```



##### 15-线程安全问题-卖票案例问题分析 [时长:9 '45]

[问题]

1. 卖票案例出现了哪些问题?
2. 问题产生的原因是什么?
3. 如何解决?

[答案]

1. 卖票案例出现了哪些问题?

   ```
   出现重复票
   出现负数票
   ```

2. 问题产生的原因是什么?

   ```
   线程的抢占式调度方式导致
   ```

3. 如何解决?	

   ```
   基本思路:让当前线程执行任务时,没有其他线程抢占资源
   使用技术:同步代码块
   ```

##### 16-线程安全问题-同步代码块 [时长:8 '83]

[问题]

1. 同步代码块的使用格式
2. 同步代码块的好处和弊端

[答案]	

1. 同步代码块的使用格式

   ```java
   synchronized(任意对象) {
   	多条语句操作共享数据的代码
   }
   注意:
   	锁对象必须是唯一的,多条线程使用的是同一把锁
   	1.继承Thread类
   		创建多个Thread类的子类对象,需要将成员位置书写的锁对象使用static进行修饰
   		private static Object lock = new Object();
   	2.实现接口
   		由于创建接口实现类一般只创建一个作为参数传递
   		此时锁对象可以定义为private Object lock = new Object();
   ```

2. 同步代码块的好处和弊端	

   ```
   好处：解决多线程操作共享数据的安全问题,保证同一时刻只有一条线程能够操作共享数据
   弊端：当线程数量较多时，每个线程都要去判断同步的锁，耗费资源同时降低程序的运行效率
   ```

##### 17-线程安全问题-锁对象唯一 [时长:7 '18]

[问题]

1. 如何保证锁对象唯一

[答案]

1. 如何保证锁对象唯一	

   ```java
   继承Thread类
   	使用static final修饰锁对象
   	或者使用当前类的字节码文件对象 类名.class
   实现Runnable和Callable接口
   	作为参数传递,只需要创建一个实现类
   	内部定义成员变量锁对象也只会存在一份
   	不需要修改
   	private Object lock = new Object();
   	或者使用this
   ```

##### 18-线程安全问题-同步方法 [时长:9 '50]

[问题]

1. 同步方法的格式
2. 同步方法和同步代码块的区别
3. 同步方法的锁对象是什么?为什么?
4. 同步静态方法的格式
5. 同步静态方法的锁对象是什么?为什么?

[答案]

1. 同步方法的格式

   ```java
   修饰符 synchronized 返回值类型 方法名(方法参数) { }
   ```

2. 同步方法和同步代码块的区别

   ```
   同步代码块可以锁住指定代码,可以指定锁对象
   同步方法只能锁住方法中所有代码,不能指定锁对象
   ```

3. 同步方法的锁对象是什么?为什么?

   ```
   同步方法的锁对象是:this
   限制于使用第二和第三种线程实现方式,将实现类对象作为参数进行传递,保证参数对象唯一,this才是唯一的锁
   ```
   
4. 同步静态方法的格式

   ```java
   修饰符 static synchronized 返回值类型 方法名(方法参数) { }
   ```

5. 同步静态方法的锁对象是什么?为什么?

   ```
   同步静态方法的锁对象是:类名.class
   ```



##### 19-线程安全问题-Lock锁 [时长:4 '40]

[问题]

1. Lock锁的使用步骤	

[答案]

1. Lock锁的使用步骤	

   ```java
   使用方法:
   	1.成员位置定义Lock实现类ReentrantLock的对象
   		注意:ReentrantLock的对象保证唯一
   	2.try{
           ReentrantLock的对象.lock();//加锁
           操作共享数据的代码;
       }catch(异常类型 变量名){
   
       }finally{
           ReentrantLock的对象.unlock();//释放锁
       }
   ```

[练习]	

```java
需求:使用Lock锁解决多线程卖票的数据安全问题
示例代码:
[Ticket.java]
public class Ticket implements Runnable {
    private int ticket = 100;
    private ReentrantLock lock = new ReentrantLock();
    @Override
    public void run() {
        while (true) {
            try {
                lock.lock();
                if (ticket <= 0) {
                    break;
                } else {
                    Thread.sleep(100);
                    ticket--;
                    System.out.println(Thread.currentThread().getName() + "在卖票,还剩下" + ticket + "张票");
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        }
    }
}
[Demo.java]
public class Demo {
    public static void main(String[] args) {
        Ticket ticket = new Ticket();
        Thread t1 = new Thread(ticket);
        Thread t2 = new Thread(ticket);
        Thread t3 = new Thread(ticket);
        t1.setName("窗口一");
        t2.setName("窗口二");
        t3.setName("窗口三");

        t1.start();
        t2.start();
        t3.start();
    }
}
```

##### 20-线程安全问题-死锁 [时长:6 '48]

[问题]

1. 死锁产生的原因
2. 如何避免死锁

[答案]

1. 死锁产生的原因

   ```
   由于两个或者多个线程互相持有对方所需要的资源，导致这些线程处于等待状态，无法执行
   ```

2. 如何避免死锁	

   ```
   避免嵌套使用锁
   ```

## 05.生产者和消费者模型

##### 21-生产者和消费者-思路分析 [时长:7 '54]

[问题]

1. 当前案例中存在几个角色?
2. 这些角色做了哪些事情?

[答案]

1. 当前案例中存在几个角色?

   ```
   1.生产者(厨师)
   2.消费者(吃货)
   3.存储数据的容器(桌子)
   ```

2. 这些角色做了哪些事情?	

   ```
   消费者：
       1，判断容器中有没有数据
       2，如果没有,线程等待
       3，如果有,消费数据
       4，消费数据之后，修改容器状态,唤醒等待的生产者生产数据,容器中数据的总数量-1
   生产者：
       1，判断容器中是否有数据
       2，如果有,线程等待，如果没有,生产数据
       3，修改容器状态
       3，唤醒等待的消费者消费数据
   存储数据的容器:
   	1, 记录数据总数量
   	2, 有数据,记录为true
   	3, 没有数据,记录为false
   	4, 消费数据后,总数量-1
   ```

##### 22-生产者和消费者-代码实现 [时长:14 '23]

[问题]

1. 生产者和消费者模型中使用了哪些线程等待唤醒的方法?

[答案]

1. 生产者和消费者模型中使用了哪些线程等待唤醒的方法?	

   | 方法             | 功能                                                         |
   | ---------------- | ------------------------------------------------------------ |
   | void wait()      | 当前线程等待，直到另一个线程调用该对象的 notify()或 notify All()方法 |
   | void notify()    | 唤醒正在等待对象监视器的单个线程                             |
   | void notifyAll() | 唤醒正在等待对象监视器的所有线程                             |

[练习]

```java
示例代码:
[Desk.java 容器(桌子类)]
public class Desk {
/*
存储数据的容器:
	1, 记录数据总数量
	2, 有数据,记录为true
	3, 没有数据,记录为false
	4, 消费数据后,总数量-1
*/
    //定义标记
    //true:桌子上有汉堡包的,此时允许吃货执行
    //false:桌子上没有汉堡包,此时允许厨师执行
    public static boolean flag = false;
    //汉堡包的总数量
    public static int count = 10;
    //锁对象
    public static final Object lock = new Object();
}
[Cooker.java 生产者(厨师类)]
public class Cooker extends Thread {
/*
生产者：
    1，判断容器中是否有数据
    2，如果有,线程等待，如果没有,生产数据
    3，修改容器状态
    3，唤醒等待的消费者消费数据
*/
    @Override
    public void run() {
        while(true){
            synchronized (Desk.lock){
                if(Desk.count == 0){
                    break;
                }else{
                    if(!Desk.flag){
                        //生产数据
                        System.out.println("厨师正在生产汉堡包");
                        Desk.flag = true;
                        Desk.lock.notifyAll();
                    }else{
                        try {
                            Desk.lock.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                }
            }
        }
    }
}
[Foodie.java 消费者(吃货类)]
public class Foodie extends Thread {
/*
消费者：
    1，判断容器中有没有数据
    2，如果没有,线程等待
    3，如果有,消费数据
    4，消费数据之后，修改容器状态,唤醒等待的生产者生产数据,容器中数据的总数量-1
*/
    @Override
    public void run() {
        while(true){
            synchronized (Desk.lock){
                if(Desk.count == 0){
                    break;
                }else{
                    if(Desk.flag){
                        //有数据,消费数据
                        System.out.println("吃货在吃汉堡包");
                        Desk.flag = false;
                        Desk.lock.notifyAll();
                        Desk.count--;
                    }else{
                        //没有数据,等待
                        //使用锁对象调用等待和唤醒的方法.
                        try {
                            Desk.lock.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                }
            }
        }
    }
}
[Demo.java 测试类]
public class Demo {
    public static void main(String[] args) {
        Foodie f = new Foodie();
        Cooker c = new Cooker();

        f.start();
        c.start();
    }
}
```

##### 23-生产者和消费者-代码改写 [时长:9 '09]

```java
示例代码:
[Desk.java]
public class Desk {
    private boolean flag;//标记
    private int count;//数据总数
    private final Object lock = new Object();//锁对象
    public Desk() {
    }
    public Desk(boolean flag, int count) {
        this.flag = flag;
        this.count = count;
    }
    public boolean isFlag() {
        return flag;
    }
    public void setFlag(boolean flag) {
        this.flag = flag;
    }
    public int getCount() {
        return count;
    }
    public void setCount(int count) {
        this.count = count;
    }
    public Object getLock() {
        return lock;
    }
}
[Cooker.java]
public class Cooker extends Thread {
    private Desk desk;
    public Cooker(Desk desk) {
        this.desk = desk;
    }
    @Override
    public void run() {
        while(true){
            synchronized (desk.getLock()){
                if(desk.getCount() == 0){
                    break;
                }else{
                    if(!desk.isFlag()){
                        System.out.println("厨师正在生产汉堡包");
                        desk.setFlag(true);
                        desk.getLock().notifyAll();
                    }else{
                        try {
                            desk.getLock().wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                }
            }
        }
    }
}
[Foodie.java]
public class Foodie extends Thread {
    private Desk desk;
    public Foodie(Desk desk) {
        this.desk = desk;
    }
    @Override
    public void run() {
        while(true){
            synchronized (desk.getLock()){
                if(desk.getCount() == 0){
                    break;
                }else{
                    if(desk.isFlag()){
                        System.out.println("吃货在吃汉堡包");
                        desk.setFlag(false);
                        desk.getLock().notifyAll();
                        desk.setCount(desk.getCount() - 1);
                    }else{
                        try {
                            desk.getLock().wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                }
            }
        }
    }
}
[Demo.java]
public class Demo {
    public static void main(String[] args) {
        Desk desk = new Desk();
        Foodie f = new Foodie(desk);
        Cooker c = new Cooker(desk);

        f.start();
        c.start();
    }
}
```

##### 24-生产者和消费者-阻塞队列基本使用 [时长:6 '12]

[问题]

1. 阻塞队列的分类
2. 阻塞队列的常用方法	

[答案]

1. 阻塞队列的分类

   ```java
   ArrayBlockingQueue:底层是数组,有界
   LinkedBlockingQueue:底层是链表,无界
   ```

2. 阻塞队列的常用方法	

   ```
   ArrayBlockingQueue<E>(容量):根据当前容量创建阻塞队列
   put(元素) :添加元素,当添加元素的数量超过阻塞队列长度,不能继续添加,进行阻塞
   E take() :获取元素,当阻塞队列中没有元素时,不能再继续取出,进行阻塞
   ```

##### 25-生产者和消费者-阻塞队列实现 [时长:7 '05]

```java
示例代码:
[Cooker.java]
public class Cooker extends Thread {
    private ArrayBlockingQueue<String> bd;
    public Cooker(ArrayBlockingQueue<String> bd) {
        this.bd = bd;
    }
    @Override
    public void run() {
        while (true) {
            try {
                bd.put("汉堡包");
                System.out.println("厨师放入一个汉堡包");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

[Foodie.java]
public class Foodie extends Thread {
    private ArrayBlockingQueue<String> bd;
    public Foodie(ArrayBlockingQueue<String> bd) {
        this.bd = bd;
    }
    @Override
    public void run() {
        while (true) {
            try {
                String take = bd.take();
                System.out.println("吃货将" + take + "拿出来吃了");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

[Demo.java]
public class Demo {
    public static void main(String[] args) {
        ArrayBlockingQueue<String> bd = new ArrayBlockingQueue<>(1);
        Foodie f = new Foodie(bd);
        Cooker c = new Cooker(bd);

        f.start();
        c.start();
    }
}
```































