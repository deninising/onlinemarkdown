<center>
    <h1>
        Java多线学习
    </h1>
</center>

# 阶段一：java多线程基础知识
## 快速认识线程
- **线程的生命周期**
    ![](https://raw.githubusercontent.com/deninising/onlinepicture/master/blog/20200223225714.png)
    - NEW：用new关键字创建一个Thread对象，此时并没有线程创建，仅仅创建了表示一个线程的Java对象，在未执行start()方法之前线程并不存在。
    - RUNNABLE：当Thread对象调用start()方法时，即在JVM中创建一个真正的线程，此时该线程位于可执行队列中，等待着CPU的调度，所以称之为可执行状态。**严格来讲，RUNNABLE状态的线程只能意外终止或是进入RUNNING状态，即使对线程调用sleep()、wait()或是其他block的OI操作也必须等到CPU对其进行调度后（进入到RUUNNING状态）才能显示阻塞**
    - BLOCKED：在RUNNING状态下，当执行到sleep()、wait()或是IO操作亦或是执行到对锁资源的获取时，线程将进入到阻塞状态。
    - RUNNING：CPU从任务队列中选中了线程后，线程将会执行逻辑代码。
    - TERMINATED：线程的最终状态，该状态下不能切换到其何其他状态，标志着一个线程的生命周期的结束。
## 线程start（）方法解析
- 源码：
![](https://raw.githubusercontent.com/deninising/onlinepicture/master/blog/20200224164201.png)

该方法为线程安全的，可看见其内部调用start0()本地方法，该方法中会使JVM去调用当前线程的逻辑执行单元run()方法，以到达最终目的。
## 模板方法在Thread中的应用(重写thread的run()方法)
- **案例：**
    ```java
        package com.dennis.conccurency.chapter01;

        /**
        * 描述：通过直接重写Thread的run()方法来创建一个线程
        *
        * @author Dennis
        * @version 1.0
        * @date 2020/2/23 20:43
        */
        public class TryConcurrency {
            public static void main(String[] args) {

                // 创建一个Thread匿名对象，重修其run()方法,调通start方法启动线程,其内部将调用本地方法start0，start0()底层将调用当前线程的执行单元run()
                // 其整个执行逻辑框架为以典型的模板方法模式

                new Thread("writeThread") {
                    @Override
                    public void run() {
                        for (int i = 0; i < 1000; i++) {
                            System.out.println("write to file > > >");
                            try {
                                Thread.sleep(490);
                            } catch (InterruptedException e) {
                                e.printStackTrace();
                            }
                        }
                    }
                }.start();

                new Thread("readThread") {
                    @Override
                    public void run() {
                        for (int i = 0; i < 1000; i++) {
                            System.out.println("read from file > > >");
                            try {
                                Thread.sleep(500);
                            } catch (InterruptedException e) {
                                e.printStackTrace();
                            }
                        }
                    }
                }.start();

                for (int i = 0; i < 100; i++) {
                    System.out.println(i);
                    try {
                        Thread.sleep(250);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }
    ```
- **类比：**
    ```java
        package com.dennis.conccurency.chapter01;

        /**
        * 描述：模板方法：类比通过通过重写thread对象的run()方法来创建线程
        *
        * @author Dennis
        * @version 1.0
        * @date 2020/2/23 19:50
        */
        public class TemplateMethod {

            // 类似于Thread.start()方法,内部流程执行框架固定
            public final void print(String message) {
                System.out.println("+++++++++++++++++++++");
                wrapPrint(message);
                System.out.println("+++++++++++++++++++++");
            }

            // 类似于Thread.run()方法,具体的业务逻辑执行单元
            protected void wrapPrint(String message) {
            }

            public static void main(String[] args) {
                TemplateMethod templateMethod = new TemplateMethod() {
                    @Override
                    protected void wrapPrint(String message) {
                        System.out.println(">" + message + "<");
                    }
                };
                templateMethod.print("do things in one way");

                TemplateMethod templateMethod1 = new TemplateMethod() {
                    @Override
                    protected void wrapPrint(String message) {
                        System.out.println(">>" + message + "<<");
                    }
                };
                templateMethod1.print("do things in another way");
            }
        }
    ```
## 策略模式在Thread中的应用（实现Runnable接口中的run()方法）
- **案例：**
    ```java
        package com.dennis.conccurency.chapter02;

        /**
        * 描述：重构：采用Runnable接口来封装原本thread.run()方法中执行的业务逻辑单元，本质上是一种策略模式的体现：把业务逻辑提取成一个策略接口-->达到线程控制和业务逻辑执行的解耦效果
        *
        * @author Dennis
        * @version 1.0
        * @date 2020/2/23 22:34
        */
        public class TicketWindowRunnable implements Runnable {

            /**
            * 最大处理业务量
            */
            private static final Long MAX = 50L;

            /**
            * 当前序号
            */
            private static int index = 1;

            @Override
            public void run() {
                while (index <= MAX) {
                    try {
                        Thread.sleep(50);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println("柜台：" + Thread.currentThread().getName() + "--当前号码：" + index++);
                }
            }
        }
    ```
    ```java
        package com.dennis.conccurency.chapter02;

        public class AppDemo {
            public static void main(String[] args) {
        //        TicketWindow window01 = new TicketWindow("一号柜台");
        //        TicketWindow window02 = new TicketWindow("二号柜台");
        //        TicketWindow window03 = new TicketWindow("三号柜台");
        //        TicketWindow window04 = new TicketWindow("四号柜台");
        //
        //        window01.start();
        //        window02.start();
        //        window03.start();
        //        window04.start();


                TicketWindowRunnable task = new TicketWindowRunnable();
                Thread t1 = new Thread(task, "一号柜台");
                Thread t2 = new Thread(task, "二号柜台");
                Thread t3 = new Thread(task, "三号柜台");
                Thread t4 = new Thread(task, "四号柜台");

                t1.start();
                t2.start();
                t3.start();
                t4.start();
            }
        }
    ```
    - **说明：** 定义Runnable的子类并实现其中的run()方法，将业务逻辑执行单元置于run()方法中，调用带Runnable参数的Thread（Runnable r）构造函数创建线程对象，当JVM调用该线程的run()方法时便会检验线程的target成员变量，若不为空则执行target中的业务逻辑
    **源码：**
    ![](https://raw.githubusercontent.com/deninising/onlinepicture/master/blog/20200224163909.png)

## 守护线程
- **JVM何时退出：** The Java Virtual Machine exits when the only threads running are all daemon threads.虚拟机只有当虚拟机中所有的线程为非守护线程的时候，才会正常自动退出。**当希望关闭某些线程，或是JVM进程结束的时候，一些线程会相应的自动退出，则可以考虑将该线程设计为守护线程**
- **setDaemon()**
    - **案例：**
        ```java
        package com.dennis.conccurency.chapter04;

        /**
        * 描述：守护线程
        *
        * @author Dennis
        * @version 1.0
        * @date 2020/2/24 21:25
        */
        public class DaemonThread {
            public static void main(String[] args) {

                Thread t = new Thread("MyThread") {
                    @Override
                    public void run() {
                        new Thread("child thread of my thread") {
                            @Override
                            public void run() {
                                if (this.isDaemon()) {
                                    System.out.println(this.getName() + " is a daemon thread");
                                } else {
                                    System.out.println(this.getName() + " is not a daemon thread");
                                }
                            }
                        }.start();

                        for (; ; ) {
                            System.out.println(Thread.currentThread().getName() + "is running");
                            try {
                                Thread.sleep(200);
                            } catch (InterruptedException e) {
                                e.printStackTrace();
                            }
                        }

                    }
                };

                t.setDaemon(true); // 将线程t设置为守护线程 ,注释掉该操作，会发现main线程结束后JVM不会自动退出,同时发现其子线程isDaemon属性与其相同
                t.start();

                try {
                    Thread.sleep(800);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("main thread has finished its lifecycle");
            }
        }
        ```
## Thread简单API
### sleep() (建议使用TimeUnit代替Thread.sleep()方法)
- **注意：** sleep()方法不会是当前线程放弃MONITOR锁对象

### yield（）
- **案例：**
    ```java
    package com.dennis.conccurency.chapter04;

    import java.util.stream.IntStream;

    /**
    * 描述：线程礼让,可将CPU的执行权让给其他急需被调度的线程
    *
    * @author Dennis
    * @version 1.0
    * @date 2020/2/24 22:03
    */
    public class ThreadYield {
        public static void main(String[] args) {
            IntStream.range(0, 3).mapToObj(ThreadYield::create).forEach(Thread::start);
        }

        private static Thread create(int index) {
            return new Thread() {
                @Override
                public void run() {
                    System.out.println(this.getName() + " is running");
    //                if (index == 0) {
    //                    yield();
    //                }// 注释起来后输出顺序基本按0 1 2 ,当取消时发现顺序基本时1 2 0,因为thread-0线程执行了CPU执行权礼让
                    System.out.println(index);
                }
            };
        }
    }
    ```
- **sleep()和yield()的区别：**
![](https://raw.githubusercontent.com/deninising/onlinepicture/master/blog/20200224222931.png)
### setPriority(int nun)设置线程优先级
- **注意：**
    - 不建议业务的实现强依赖线程的优先级，因为的优先级效果受到当前CPU空闲状态影响，有的时候达不到业务想要的效果
    - 一个线程的优先级最大大不过所在线程组的最大优先级
    - 线程默认的优先级和其父线程的优先级相同
    ![](https://raw.githubusercontent.com/deninising/onlinepicture/master/blog/20200224225328.png)
### public void interrupt()
- 以下方法为可中断方法，调用时会使得线程进入阻塞状态，调用interrupt()方法后(底层调用interrupt0()本地方法)可将线程的interrupt flag置为true，可中断方法被打断并检测到中断的signal随之抛出InterruptedException异常。
![](https://raw.githubusercontent.com/deninising/onlinepicture/master/blog/20200226204556.png)
- **案例：(代码注释部分是重点)**
    ```java
    package com.dennis.conccurency.chapter04;

    import java.util.concurrent.TimeUnit;

    /**
    * 描述：线程中断方法
    *
    * @author Dennis
    * @version 1.0
    * @date 2020/2/26 20:47
    */
    public class InterruptMethod {
        public static void main(String[] args) throws InterruptedException {

            Thread thread = new Thread(() -> {
                // 【1】线程中执行可中断方法，且当线程被中断时,虽然其interrupt flag被设置成过true,
                // 但当interruptedException的signal被检测到时,线程的interrupt flag又会被清除，
                // 即设置为false
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                    System.out.println("oh I am interrupted once!");
                }

                try {
                    TimeUnit.SECONDS.sleep(5);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                    System.out.println("oh I am interrupted twice!");
                }

                // 【2】调用线程的interrupt()线程的interrupt flag被设置成为true,线程的生命周期并不会结束
                while (true) {
                    System.out.println("I am always running");
                }
            });

            thread.start();

            TimeUnit.MILLISECONDS.sleep(2);
            System.out.println(thread.isInterrupted() + "====================================");
            // 【3】会将线程的interrupt flag 设置成true,线程的生命周期并不会结束
            thread.interrupt();
            TimeUnit.SECONDS.sleep(2);
            thread.interrupt();

            TimeUnit.SECONDS.sleep(1);
            // 查看当前线程interrupt flag状态,【1】执行时结果为false;【2】执行时结果为true
            System.out.println(thread.isInterrupted() + "====================================");
            System.out.println(thread.isInterrupted() + "====================================");
        }
    }
    ```

### public boolean isInterrupted()
- 只是用来获取对应线程的interrupt flag是否处于中断状态
### public static boolean interrupt()
- 获取到当前线程的interrupt flag状态值，并擦除标记
![](https://raw.githubusercontent.com/deninising/onlinepicture/master/blog/20200226212607.png)
- **案例：**
    ```java
    package com.dennis.conccurency.chapter04;

    import java.util.concurrent.TimeUnit;

    /**
    * 描述：获取到当前线程的interrupt flag状态值，并擦除标记
    *
    * @author Dennis
    * @version 1.0
    * @date 2020/2/26 21:30
    */
    public class StaticInterruptMethod {
        public static void main(String[] args) throws InterruptedException {

            Thread thread = new Thread() {
                String msg1 = "=====================";
                String msg2 = "+++++++++++++++++++++";
                String msg3 = null;


                @Override
                public void run() {

                    while (true) {
                        boolean b = Thread.interrupted();
                        // 但凡b能有可能两次为true,输出就会先输出====,最后变为输出+++++
                        if (b) {
                            msg3 = msg1;
                            msg1 = msg2;
                        }
                        System.out.println(msg3);
                    }
                }
            };

            thread.start();
        TimeUnit.SECONDS.sleep(2);
        // 【1】让b获得一次变为true的机会
        thread.interrupt();

    //        TimeUnit.SECONDS.sleep(1);
    //        // 【2】让b有一次获得变为true的机会
    //        thread.interrupt();
        }
    }
    ```
### join()
- 在A线程的执行过程中，调用B线程的join()插队方法，会使B线程获得执行权，且使当前的A线程进入到阻塞带，当B线程的生命周期结束或是达到指定的插队时间，A线程才会解除阻塞状态。
- **案例：**
    ```java
    package com.dennis.conccurency.chapter04;

    import java.util.List;
    import java.util.concurrent.TimeUnit;
    import java.util.stream.Collectors;
    import java.util.stream.IntStream;

    /**
    * 描述：
    *
    * @author Dennis
    * @version 1.0
    * @date 2020/2/26 22:28
    */
    public class JoinMethod {
        public static void main(String[] args) throws InterruptedException {

            List<Thread> threads = IntStream.range(1, 3).mapToObj(JoinMethod::createThread).collect(Collectors.toList());

            threads.forEach(Thread::start);

            System.out.println("===========================");
            for (Thread thread : threads) {
                // 当thread-0执行的join()之后,只能等到thread-0的线程生命周期结束后,main线程才能够调用thread-1的join()方法
                System.out.println("thread " + thread.getName() + " is going to join in a minute");
                thread.join();
            }
            System.out.println("===========================");

            IntStream.range(0, 10).forEach(i -> {
                System.out.println(Thread.currentThread().getName() + "#" + i);
            });
        }

        private static void shortSleep() {
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        private static Thread createThread(int i) {
            return new Thread(() -> {
                IntStream.range(0, 10).forEach(t -> {
                    System.out.println(Thread.currentThread().getName() + "#" + t);
                    shortSleep();
                });
            });
        }
    }
    ```
### 如何关闭一个线程(不建议使用stop()方法)
- **正常关闭**
    - 线程结束生命周期正常结束
    - 捕获中断信号关闭线程 interrupt flag
    - 定义valotile变量，封装开关方法
- **异常退出**
    - 由于Runnable和Thread的逻辑执行单元run()方法不接受受查异常，所以通过封装成非受查异常抛出并调用生命周期结束方法
- **进程假死**
    - 一般有内部线程阻塞导致，通常借用工具（jstack、jconsole、jvisualvm）排除
    - **通过封装执行线程+任务线程（守护线程）+join()的方法避免进程假死问题**
        **案例：**
        - 服务端代码
            ```java
            package com.dennis.conccurency.chapter05;

            import sun.font.TrueTypeFont;

            import java.util.TreeMap;
            import java.util.concurrent.TimeUnit;

            /**
            * 描述：通过封装执行线程+任务线程（守护线程）+join()的方法避免进程假死问题
            *
            * @author Dennis
            * @version 1.0
            * @date 2020/2/27 21:09
            */
            public class ThreadService {
                /**
                * 任务线程是否执行完成标识
                */
                private boolean finished = false;

                /**
                * 维持着一个执行线程成员变量,任务线程的启动通过执行线程来调用,任务线程作为执行线程的子线程
                */
                private Thread executeThread;

                /**
                * 封装执行方法
                */
                public void execute(Runnable task) {
                    executeThread = new Thread() {
                        @Override
                        public void run() {
                            Thread taskThread = new Thread(task);
                            // 设置为守护线程，当执行线程的生命周期结束时,其跟着结束
                            taskThread.setDaemon(true);

                            taskThread.start();
                            // 使任务线程插入到执行线程的执行中去,阻塞执行线程
                            try {
                                System.out.println("【1】执行线程(" + this.getName() + ")的执行权将被子线程(" + taskThread.getName() + ")剥夺!");
                                taskThread.join();
                                // 任务线程在规定的时间内执行完成,改变标志符号
                                System.out.println("【2】任务线(" + taskThread.getName() + ")程执行完毕！");
                                finished = true;
                            } catch (InterruptedException e) {
                                System.out.println("【2】任务线程(" + taskThread.getName() + ")执行时间超时,已被强制关闭!");
                            }
                            System.out.println("【3】执行线程(" + this.getName() + ")重新获取到执行权!");
                            System.out.println("【4】执行线程(" + this.getName() + ")的生命周期结束......");
                        }
                    };

                    // 启动执行线程
                    executeThread.start();
                }

                /**
                * 封装执行时间可控的关闭方法:多长时间任务线程还没执行完,就强制关闭
                */
                public void shutdownForceExtend(long millis) {
                    long currentTime = System.currentTimeMillis();
                    // 在给定时间内反复检测任务线程的执行情况
                    while (!finished) {
                        if (System.currentTimeMillis() - currentTime >= millis) {
                            // 任务线程执行时间超时,将被强制关闭！
                            executeThread.interrupt();
                            break;
                        }

                        try {
                            // 【1】防止线程while循环大量占用CPU资源
                            Thread.sleep(1);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                }
            }
            ```
        - 客户端代码
            ```java
            package com.dennis.conccurency.chapter05;

            import java.util.concurrent.TimeUnit;

            /**
            * 描述：
            *
            * @author Dennis
            * @version 1.0
            * @date 2020/2/27 21:51
            */
            public class DemoToShow {
                public static void main(String[] args) throws InterruptedException {
                    ThreadService threadService = new ThreadService();
                    long startTime = System.currentTimeMillis();
                    threadService.execute(() -> {
            //            while (true) {
            //                // 模拟一个很重的任务永远都执行不完
            //            }

                        try {
                            // 模拟一个1秒钟就能执行完成的任务
                            Thread.sleep(1_000);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    });
                    // 五秒钟执行不完就强制关闭
                    threadService.shutdownForceExtend(2_000);
                    TimeUnit.MILLISECONDS.sleep(20);
                    long endTime = System.currentTimeMillis();
                    System.out.println("【5】客户端线程("+Thread.currentThread().getName()+")总共耗时：" + (endTime - startTime) + "millis");

                }
            }
            ```
        - 结果：
            ```
            成功
            【1】执行线程(Thread-0)的执行权将被子线程(Thread-1)剥夺!
            【2】任务线程(Thread-1)执行时间超时,已被强制关闭!
            【3】执行线程(Thread-0)重新获取到执行权!
            【4】执行线程(Thread-0)的生命周期结束......
            【5】客户端线程(main)总共耗时：2056millis

            失败
            【1】执行线程(Thread-0)的执行权将被子线程(Thread-1)剥夺!
            【2】任务线(Thread-1)程执行完毕！
            【3】执行线程(Thread-0)重新获取到执行权!
            【4】执行线程(Thread-0)的生命周期结束......
            【5】客户端线程(main)总共耗时：1067millis
            ```
## synchronized关键字的本质作用
- **作用：** synchronized总是和互斥资源同时使用的，被synchronized作用的互斥资源将成为一个minitor对象，当线程成功获取到互斥资源时，其对应的monitor计数器将加1，释放掉互斥资源时计数器将减1，计数器为0时，其他在阻塞队列中等待获取互斥资源的线程才有权争夺互斥资源
- **案例：**
    ```java
    package com.dennis.conccurency.chapter06;

    import java.util.concurrent.TimeUnit;
    import java.util.stream.IntStream;

    /**
    * 描述：synchronised关键字和互斥资源
    *
    * @author Dennis
    * @version 1.0
    * @date 2020/2/29 13:44
    */
    public class Mutex {
        private static final Object MUTEX = new Object();

        public void accessResource() {
            synchronized (MUTEX) {
                try {
                    TimeUnit.MINUTES.sleep(10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }

        public static void main(String[] args) {
            final Mutex mutex = new Mutex();
            IntStream.range(0, 5).forEach(
                    i -> new Thread(mutex::accessResource).start()
            );
        }
    }
    ```
- **jconsole查看：**
    ![](https://raw.githubusercontent.com/deninising/onlinepicture/master/blog/20200229141443.png)
    ![](https://raw.githubusercontent.com/deninising/onlinepicture/master/blog/20200229141539.png)
- **jstack工具查看：**
    ![](https://raw.githubusercontent.com/deninising/onlinepicture/master/blog/20200229142938.png)
- **同步代码块的执行机制：**
    - 获取互斥资源
        ![](https://raw.githubusercontent.com/deninising/onlinepicture/master/blog/20200229164158.png)![](https://raw.githubusercontent.com/deninising/onlinepicture/master/blog/20200229164235.png)
    - 释放互斥资源
        ![](https://raw.githubusercontent.com/deninising/onlinepicture/master/blog/20200229164315.png)
- **同步方法使用的要点：**
    1. 与monitor关联的对象（互斥资源）不能为空
    2. synchronized的作用域不能太大，做好做到精确控制使用共享资源的代码块
    3. 避免不同的monitor企图锁相同的方法
    4. 多个锁的交叉导致死锁
## This Monitor 和Class Monitor
- **非静态 synchronized 修饰的同步方法，其monitor lock 为对象本身 this，非静态方法归实例对象所有**
- **静态 synchronized 修饰的同步方法，其monitor lock 为类对象本身，静态方法归类所有**

## 程序死锁及诊断
###  死锁：
![](https://raw.githubusercontent.com/deninising/onlinepicture/master/blog/20200229175306735.png)
- **交叉使用互斥资源引起的死锁案例：**
    ```java
    package com.dennis.conccurency.chapter06;

    import java.util.concurrent.TimeUnit;

    /**
    * 描述：交叉获取互斥资源引起的死锁
    *
    * @author Dennis
    * @version 1.0
    * @date 2020/2/29 17:25
    */
    public class DeadMutex {
        private static final Object READLOCK = new Object();
        private static final Object WRITELOCK = new Object();

        public void read() {
            synchronized (READLOCK) {
                System.out.println(Thread.currentThread().getName() + " get READLOCK");
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (WRITELOCK) {
                    System.out.println(Thread.currentThread().getName() + " get WRITELOCK");
                }
                System.out.println(Thread.currentThread().getName() + " release WRITELOCK");
            }
            System.out.println(Thread.currentThread().getName() + " release READLOCK");
        }

        public void write() {
            synchronized (WRITELOCK) {
                System.out.println(Thread.currentThread().getName() + " get WRITELOCK");
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (READLOCK) {
                    System.out.println(Thread.currentThread().getName() + " get READLOCK");
                }
                System.out.println(Thread.currentThread().getName() + " release READLOCK");
            }
            System.out.println(Thread.currentThread().getName() + " release WRITELOCK");
        }

        public static void main(String[] args) {
            DeadMutex deadMutex = new DeadMutex();
            new Thread(deadMutex::read).start();
            new Thread(deadMutex::write).start();
        }
    }
    ```
### 死锁的诊断
 - **交叉引起的死锁借助jstack可以很好的捕捉到：**
![](https://raw.githubusercontent.com/deninising/onlinepicture/master/blog/20200229182015.png)
可以看到个线程对锁资源的持有情况
![](https://raw.githubusercontent.com/deninising/onlinepicture/master/blog/20200229182314.png)
底部也会显示地给出死锁
- **其他情况引起的死锁是具体原因采取相应的诊断方法**
## 线程间的通信（进程内通信）
### 同步阻塞与异步阻塞
- **同步阻塞：**
    ![](https://raw.githubusercontent.com/deninising/onlinepicture/master/blog/20200229224446.png)
- **异步阻塞：**
    ![](https://raw.githubusercontent.com/deninising/onlinepicture/master/blog/20200229224525.png)
## 消费者和生产者模式
- **单线程版：**
    ```java
    package com.dennis.conccurency.chapter07;

    import java.util.LinkedList;

    /**
    * 描述： 自定义消息队列
    *
    * @author Dennis
    * @version 1.0
    * @date 2020/2/29 23:02
    */
    public class EventQueueForSingleThread {
        static class Event {
        }

        private final int max;

        private static final int DEFAULT_MAX_EVENT = 10;

        private final LinkedList<Event> eventQueue = new LinkedList<>();

        public EventQueueForSingleThread() {
            this(DEFAULT_MAX_EVENT);
        }

        public EventQueueForSingleThread(int max) {
            this.max = max;
        }

        //添加事件
        public void addEvent(Event event) {
            synchronized (eventQueue) {
                if (eventQueue.size() >= max) {
                    console("event queue is already full");
                    try {
                        eventQueue.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                eventQueue.addLast(event);
                console("a new event has be submitted");
                eventQueue.notify();
            }
        }

        //处理事件
        public void handleEvent() {
            synchronized (eventQueue) {
                if (eventQueue.isEmpty()) {
                    console("there is no even left need to be handled");
                    try {
                        eventQueue.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                Event event = eventQueue.removeFirst();
                console("the event:" + event + " has be handled");
                eventQueue.notify();
            }
        }

        private void console(String message) {
            System.out.println(Thread.currentThread().getName() + ":" + message);
        }
    }
    ```
- **多线程版本：**
    ```java
    package com.dennis.conccurency.chapter07;

    import java.util.LinkedList;

    /**
    * 描述： 支持多线程通信版本
    *
    * @author Dennis
    * @version 1.0
    * @date 2020/2/29 23:35
    */
    public class EventQueueForMultiThreads {

        // 生产满了才能消费，消费空了才能生产
        private boolean isProduced = false;

        static class Event {
        }

        private final int max;

        private static final int DEFAULT_MAX_EVENT = 10;

        private final LinkedList<EventQueueForSingleThread.Event> eventQueue = new LinkedList<>();

        public EventQueueForMultiThreads() {
            this(DEFAULT_MAX_EVENT);
        }

        public EventQueueForMultiThreads(int max) {
            this.max = max;
        }

        //添加事件
        public void addEvent(EventQueueForSingleThread.Event event) {
            synchronized (eventQueue) {
    //          while (eventQueue.size() >= max) {
                while (isProduced) {
                    console("event queue is already full");
                    try {
                        eventQueue.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                eventQueue.addLast(event);
                console(Thread.currentThread().getName() + "->" + "a new event has be submitted");
                if (eventQueue.size() == max) {
                    isProduced = true;
                    eventQueue.notifyAll();
                }
    //            eventQueue.notifyAll();
            }
        }

        //处理事件
        public void handleEvent() {
            synchronized (eventQueue) {
    //            while (eventQueue.isEmpty()) {
                while (!isProduced) {
                    console("there is no even left need to be handled");
                    try {
                        eventQueue.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                EventQueueForSingleThread.Event event = eventQueue.removeFirst();
                console(Thread.currentThread().getName() + "->" + "the event:" + event + " has be handled");
                if (eventQueue.isEmpty()) {
                    isProduced = false;
                    eventQueue.notifyAll();
                }
    //            eventQueue.notifyAll();
            }
        }

        private void console(String message) {
            System.out.println(Thread.currentThread().getName() + ":" + message);
        }
    }
    ```
- **客户端：**
    ```java
    package com.dennis.conccurency.chapter07;

    import java.util.ArrayList;
    import java.util.Arrays;
    import java.util.concurrent.TimeUnit;
    import java.util.stream.Stream;

    /**
    * 描述： 客户端程通信
    *
    * @author Dennis
    * @version 1.0
    * @date 2020/2/29 23:24
    */
    public class EventClient {
        public static void main(String[] args) throws InterruptedException {
            // 单线程版本
        /* EventQueueForSingleThread eventQueue = new EventQueueForSingleThread();

            new Thread(() -> {
                while (true) {
                    eventQueue.addEvent(new EventQueueForSingleThread.Event());
                }
            }, "producer").start();
            new Thread(() -> {
                while (true) {
                    eventQueue.handleEvent();
                }
            }, "consumer").start();*/

            // 多线程版本
            EventQueueForMultiThreads queueForMultiThreads = new EventQueueForMultiThreads();
            Stream.of("p1", "p2", "p3").map(name -> new Thread(() -> {
                while (true) {
                    queueForMultiThreads.addEvent(new EventQueueForSingleThread.Event());
                    try {
                        TimeUnit.MILLISECONDS.sleep(400);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }, name)).forEach(Thread::start);

            Stream.of("c1", "c2", "c3").map(name -> new Thread(() -> {
                while (true) {
                    queueForMultiThreads.handleEvent();
                    try {
                        TimeUnit.MILLISECONDS.sleep(400);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }, name)).forEach(Thread::start);
        }
    }
    ```
- **wait、notify、notifyAll、monitor、thread之间的关系**
![](https://raw.githubusercontent.com/deninising/onlinepicture/master/blog/20200301111712.png)
![](https://raw.githubusercontent.com/deninising/onlinepicture/master/blog/20200301111757.png)
## 自定义显示锁
- **synchronised关键字的缺陷：是一种排他式的数据同步机制，且当线程多个线程在竞争锁资源的时候会存在以下缺陷**
    - 无法控制阻塞时常
    - 阻塞不可以被中断
- **顶层接口：**
    ```java
    package com.dennis.conccurency.chapter08;

    import java.util.List;
    import java.util.concurrent.TimeoutException;

    /**
    * 描述：锁顶层接口
    *
    * @author Dennis
    * @version 1.0
    * @date 2020/3/1 20:27
    */
    public interface Lock {
        /**
        * 支持可中断
        */
        void lock() throws InterruptedException;

        /**
        * 支持中断和超时时间设定
        */
        void lock(long millis) throws InterruptedException, TimeoutException;

        void unLock();

        List<Thread> getBlockedThreads();

    }
    ```
- **实现类：**
    ```java
    package com.dennis.conccurency.chapter08;

    import java.util.ArrayList;
    import java.util.List;
    import java.util.concurrent.TimeoutException;

    /**
    * 描述：自定义显示锁
    *
    * @author Dennis
    * @version 1.0
    * @date 2020/3/1 20:26
    */
    public class BooleanLock implements Lock {
        /**
        * 存放获取到锁资源的当前线程
        */
        private Thread currentThread;
        /**
        * 标识锁的状态
        */
        private boolean locked;
        /**
        * 存放阻塞状态的线程
        */
        private final List<Thread> blockedThreadList = new ArrayList<>();


        @Override
        public void lock() throws InterruptedException {
            synchronized (this) {
                while (locked) {
                    if (!blockedThreadList.contains(Thread.currentThread()))
                        blockedThreadList.add(Thread.currentThread());
                    this.wait();
                }

                this.locked = true;
                blockedThreadList.remove(Thread.currentThread());
                this.currentThread = Thread.currentThread();
            }

        }

        @Override
        public void lock(long millis) throws InterruptedException, TimeoutException {
            synchronized (this) {
                if (millis <= 0)
                    lock();
                // 获取到允许阻塞的最终时间
                long endTime = millis + System.currentTimeMillis();
                long timeAllowedToBlock = endTime - System.currentTimeMillis();
                while (locked) {
                    // 判断是否超时,再决定是否有必要继续等待
                    if (timeAllowedToBlock <= 0)
                        throw new TimeoutException("the time of getting lock has been timeout");
                    if (!blockedThreadList.contains(Thread.currentThread()))
                        blockedThreadList.add(Thread.currentThread());
                    wait(millis);
                    timeAllowedToBlock = endTime - System.currentTimeMillis();
                }
                this.locked = true;
                currentThread = Thread.currentThread();
                blockedThreadList.remove(Thread.currentThread());
            }
        }

        @Override
        public void unLock() {
            synchronized (this) {
                if (this.currentThread == Thread.currentThread()) {
                    this.locked = false;
                    this.notifyAll();
                }
            }
        }

        @Override
        public List<Thread> getBlockedThreads() {
            return blockedThreadList;
        }
    }
    ```
- **测试类：**
    ```java
    package com.dennis.conccurency.chapter08;

    import java.util.concurrent.TimeUnit;
    import java.util.concurrent.TimeoutException;
    import java.util.stream.Collectors;
    import java.util.stream.IntStream;

    /**
    * 描述：多线程抢锁测试
    *
    * @author Dennis
    * @version 1.0
    * @date 2020/3/1 21:02
    */
    public class BooleanLockTest {
        public static void main(String[] args) throws InterruptedException {
            BooleanLock lock = new BooleanLock();

            // 常规抢锁
        /* IntStream.range(0, 4).forEach(i -> {
                new Thread(() -> {
                    try {
                        lock.lock();
                        System.out.println(Thread.currentThread().getName() + " is doing work");
                        System.out.println("thread blocked:" + lock.getBlockedThreads().stream().map(Thread::getName).collect(Collectors.toList()));
                        // 模拟3秒的执行任务的时间
                        TimeUnit.SECONDS.sleep(3);
                        lock.unLock();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }, "T-" + i).start();
            });*/

            // 测试timeout特性
            /*IntStream.range(0, 4).forEach(i -> {
                new Thread(() -> {
                    try {
                        try {
                            // 两秒后抢不到锁就抛超时异常
                            lock.lock(2_000);
                            System.out.println(Thread.currentThread().getName() + " is doing work");
                            System.out.println("thread blocked:" + lock.getBlockedThreads().stream().map(Thread::getName).collect(Collectors.toList()));
                        } catch (TimeoutException e) {
                            e.printStackTrace();
                        }
                        // 模拟3秒的执行任务的时间
                        TimeUnit.SECONDS.sleep(3);
                        lock.unLock();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }, "T-" + i).start();
            });*/

            // 测试显示锁抢锁时可自定中断的特性
            Thread Ta = new Thread(() -> {
                try {
                    lock.lock();
                } catch (InterruptedException e) {

                    e.printStackTrace();
                }
                try {
                    TimeUnit.SECONDS.sleep(5);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                lock.unLock();
            }, "Thread_A");

            Thread Tb = new Thread(() -> {
                try {
                    lock.lock();
                    try {
                        TimeUnit.SECONDS.sleep(5);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                } catch (InterruptedException e) {
                    System.out.println("Tb线程被主动中断！");
                }
                lock.unLock();
            }, "Thread_B");

            Ta.start();
            Tb.start();

            TimeUnit.MILLISECONDS.sleep(2_000);
            Tb.interrupt();
        }
    }
    ```
- **结果：**
    ```
    【1常规抢锁】
    T-0 is doing work
    thread blocked:[T-2, T-1, T-3]
    T-3 is doing work
    thread blocked:[T-2, T-1]
    T-2 is doing work
    thread blocked:[T-1]
    T-1 is doing work
    thread blocked:[]
    【2测试timeout特性】
    T-0 is doing work
    thread blocked:[T-1, T-2, T-3]
    java.util.concurrent.TimeoutException: the time of getting lock has been timeout
        at com.dennis.conccurency.chapter08.BooleanLock.lock(BooleanLock.java:56)
        at com.dennis.conccurency.chapter08.BooleanLockTest.lambda$null$0(BooleanLockTest.java:41)
        at java.lang.Thread.run(Thread.java:748)
    java.util.concurrent.TimeoutException: the time of getting lock has been timeout
        at com.dennis.conccurency.chapter08.BooleanLock.lock(BooleanLock.java:56)
        at com.dennis.conccurency.chapter08.BooleanLockTest.lambda$null$0(BooleanLockTest.java:41)
        at java.lang.Thread.run(Thread.java:748)
    java.util.concurrent.TimeoutException: the time of getting lock has been timeout
        at com.dennis.conccurency.chapter08.BooleanLock.lock(BooleanLock.java:56)
        at com.dennis.conccurency.chapter08.BooleanLockTest.lambda$null$0(BooleanLockTest.java:41)
        at java.lang.Thread.run(Thread.java:748)
    【3测试显示锁抢锁时可自定中断的特性】
    Tb线程被主动中断
    ```
# 阶段二：多线程设计模式详解
# 阶段三：JDK并发包详解
# 阶段四：并发编程深入讨论