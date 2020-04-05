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
### 消费者和生产者模式
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
    - 无法控制阻塞时长
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

        /**
        *获取阻塞中的线程
        */
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
                    // 可能是唤醒的线程也可能是一个新的线程
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
                    // 可能是唤醒的线程也可能是一个新的线程
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
            // 利用锁的可重入性（只有能够加锁的线程才能解锁）
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
## 自定义线程池
- **框架示意图**
    ![](https://raw.githubusercontent.com/deninising/onlinepicture/master/blog/20200313192234.png)
- **代码：详见com.dennis.conccurency.chapter10**

## 类加载器
- **JVM内置三大类加载器：**
    - 根加载器：Bootstrap ClassLoader
    - 扩展加载器：Ext ClassLoader
    - 系统加载器：Application ClassLoader
- **三者关系图：**
![](https://raw.githubusercontent.com/deninising/onlinepicture/master/blog/20200316120856.png)
- **实例代码：**
    - 根加载器:最顶层加载器，没有任何父类加载器，有C++编写，负责加载虚拟机核心类库
        ````java
        package com.dennis.conccurency.chapter11;

        import java.util.Arrays;

        /**
        * 描述：根类加载器
        *
        * @author Dennis
        * @version 1.0
        * @date 2020/3/15 11:28
        */
        public class BootStrapClassLoader {
            public static void main(String[] args) {
                // 根类加载器没有指向它的引用
                ClassLoader classLoader = String.class.getClassLoader();
                System.out.println("bootStrap:" + classLoader);

                // 根类加载器所在的加载路径
                String bootPath = System.getProperty("sun.boot.class.path");
                Arrays.asList(bootPath.split(";")).forEach(System.out::println);
            }
        }
        ````
    - 结果
        ````text
        bootStrap:null
        D:\develop\java\jdk8\jre\lib\resources.jar
        D:\develop\java\jdk8\jre\lib\rt.jar
        D:\develop\java\jdk8\jre\lib\sunrsasign.jar
        D:\develop\java\jdk8\jre\lib\jsse.jar
        D:\develop\java\jdk8\jre\lib\jce.jar
        D:\develop\java\jdk8\jre\lib\charsets.jar
        D:\develop\java\jdk8\jre\lib\jfr.jar
        D:\develop\java\jdk8\jre\classes
        ````
    - 扩展加载器：由Java编写，父加载器为根加载器，主要加载jre/lib/ext下的类
        ````java
        package com.dennis.conccurency.chapter11;

        import java.util.Arrays;
        /**
        * 描述：扩展类加载器,加载JAVA_HOME下的jre/lb/ext子目录中的类库
        *
        * @author Dennis
        * @version 1.0
        * @date 2020/3/15 11:38
        */
        public class ExtClassLoader {
            public static void main(String[] args) {
                // 获取加载资源的路径
                String[] splits = System.getProperty("java.ext.dirs").split(";");
                Arrays.asList(splits).forEach(System.out::println);
            }
        }
        ````
    - 结果
        ````text
        D:\develop\java\jdk8\jre\lib\ext
        C:\WINDOWS\Sun\Java\lib\ext
        ````
    - 系统加载器：最常见的加载器，加载classPath下的类库资源，多有的第三方jar包也有它负责加载
        ````java
        package com.dennis.conccurency.chapter11;

        import java.util.Arrays;

        /**
        * 描述： 系统类加载器,加载classPath类路径下的所有第三方jar中的类资源
        *
        * @author Dennis
        * @version 1.0
        * @date 2020/3/15 11:42
        */
        public class ApplicationClassLoader {
            public static void main(String[] args) {
                System.out.println();
                String[] paths = System.getProperty("java.class.path").split(";");
                Arrays.asList(paths).forEach(System.out::println);

                System.out.println(ApplicationClassLoader.class.getClassLoader());
            }
        }
        ````
    - 结果
        ````text
        D:\develop\java\jdk8\jre\lib\charsets.jar
        D:\develop\java\jdk8\jre\lib\deploy.jar
        D:\develop\java\jdk8\jre\lib\ext\access-bridge-64.jar
        D:\develop\java\jdk8\jre\lib\ext\cldrdata.jar
        D:\develop\java\jdk8\jre\lib\ext\dnsns.jar
        D:\develop\java\jdk8\jre\lib\ext\jaccess.jar
        D:\develop\java\jdk8\jre\lib\ext\jfxrt.jar
        D:\develop\java\jdk8\jre\lib\ext\localedata.jar
        D:\develop\java\jdk8\jre\lib\ext\nashorn.jar
        D:\develop\java\jdk8\jre\lib\ext\sunec.jar
        D:\develop\java\jdk8\jre\lib\ext\sunjce_provider.jar
        D:\develop\java\jdk8\jre\lib\ext\sunmscapi.jar
        D:\develop\java\jdk8\jre\lib\ext\sunpkcs11.jar
        D:\develop\java\jdk8\jre\lib\ext\zipfs.jar
        D:\develop\java\jdk8\jre\lib\javaws.jar
        D:\develop\java\jdk8\jre\lib\jce.jar
        D:\develop\java\jdk8\jre\lib\jfr.jar
        D:\develop\java\jdk8\jre\lib\jfxswt.jar
        D:\develop\java\jdk8\jre\lib\jsse.jar
        D:\develop\java\jdk8\jre\lib\management-agent.jar
        D:\develop\java\jdk8\jre\lib\plugin.jar
        D:\develop\java\jdk8\jre\lib\resources.jar
        D:\develop\java\jdk8\jre\lib\rt.jar
        D:\Code\github\thread_learn\target\classes
        C:\IntelliJ IDEA\lib\idea_rt.jar
        sun.misc.Launcher$AppClassLoader@18b4aac2
        ````
- 自定义加载器：为了打破双亲委派机制通常会采用自定义加载器
    - 实例代码
        ````java
        package com.dennis.conccurency.chapter11;

        import java.io.ByteArrayOutputStream;
        import java.nio.file.Files;
        import java.nio.file.Path;
        import java.nio.file.Paths;

        /**
        * 描述：自定义类加载器
        *
        * @author Dennis
        * @version 1.0
        * @date 2020/3/15 12:24
        */
        public class MyClassLoader extends ClassLoader {
            // 定义默认的class存放路径
            private final static Path DEFAULT_CLASS_DIR = Paths.get("E:", "classLoader1");

            private final Path classDir;

            // 使用默认的class路径
            public MyClassLoader() {
                super();
                this.classDir = DEFAULT_CLASS_DIR;
            }

            // 允许传入指定路径
            public MyClassLoader(Path classDir) {
                super();
                this.classDir = classDir;
            }

            // 指定class路径的同时，指定父类加载器
            public MyClassLoader(Path classDir, ClassLoader parent) {
                super(parent);
                this.classDir = classDir;
            }

            // 重写父类的findClass方法
            @Override
            protected Class<?> findClass(String name) throws ClassNotFoundException {
                // 读取class的二进制数据
                byte[] bytes = readClassByte(name);
                // 若为空则抛出classNotFoundException
                if (bytes == null) {
                    throw new ClassNotFoundException("can not find the class :" + name);
                }
                // 调用defineClass方法定义class
                Class<?> aClass = this.defineClass(name, bytes, 0, bytes.length);
                return aClass;
            }

            // 将class文件读入内存
            private byte[] readClassByte(String name) throws ClassNotFoundException {
                // 将包名分隔符转换为文件分隔符
                String classPath = name.replace(".", "/");
                Path classFullPath = classDir.resolve(Paths.get(classPath + ".class"));
                if (!classFullPath.toFile().exists()) {
                    throw new ClassNotFoundException("the class name:" + name + " not found");
                }

                try (ByteArrayOutputStream outputStream = new ByteArrayOutputStream()) {
                    Files.copy(classFullPath, outputStream);
                    return outputStream.toByteArray();
                } catch (Exception e) {
                    throw new ClassNotFoundException("load the class " + name + "occur error:", e);
                }
            }

            @Override
            public String toString() {
                return "MyClassLoader{" +
                        "classDir=" + classDir +
                        '}';
            }
        }
        ````
    - 测试代码
        ````java
        package com.dennis.conccurency.chapter11;

        import java.lang.reflect.InvocationTargetException;
        import java.lang.reflect.Method;

        /**
        * 描述：自定义类加载器的定义(没有屏蔽双亲委派机制)
        *
        * @author Dennis
        * @version 1.0
        * @date 2020/3/15 13:01
        */
        public class MyClassLoaderTest {
            public static void main(String[] args)
                    throws ClassNotFoundException,
                    IllegalAccessException,
                    InstantiationException,
                    NoSuchMethodException,
                    InvocationTargetException {
                // 申明一个类加载器
                MyClassLoader myClassLoader = new MyClassLoader();
                // 使用myClassLoader加载class文件(class文件已编译到对用的文件夹中)
                Class<?> aClass = myClassLoader.loadClass("com.dennis.conccurency.chapter11.HelloWorld");
                System.out.println(aClass.getClassLoader());

                // 到该步骤以前，虽然类被加载,且输出了类的加载器信息,但是HelloWorld.java中的静态代码块并没有被输出,
                // 因为类的loadClass方法并不会导致类的主动初始化,其仅仅完成了加载过程中的加载阶段而已
                Object helloWorld = aClass.newInstance();// 主动加载,则静态代码块中的内容将初始化输出
                System.out.println(helloWorld);

                Method welcome = aClass.getMethod("welcome");
                String result = (String) welcome.invoke(helloWorld);
                System.out.println(result);
            }
        }
        ````
    - 结果
        ````text
        MyClassLoader{classDir=E:\classLoader1}
        hello world class is initialized!
        com.dennis.conccurency.chapter11.HelloWorld@1540e19d
        Hello world!
        ````
## 双亲委托机制
- **框架示意图**
![](https://raw.githubusercontent.com/deninising/onlinepicture/master/blog/20200316123223.png)
**说明：部分类容详见第九章和第十章（建议多次阅读）**

# 阶段二：多线程设计模式详解
## 重要的两种单例模式
- **holder模式下的单例**
    ````java
    package com.dennis.conccurency.chapter12;

    /**
    * 描述：Holder版单例模式
    *
    * @author Dennis
    * @version 1.0
    * @date 2020/3/16 12:01
    */
    public class Singleton4Holder {
        // 实例变量
        private byte[] data = new byte[1024];

        // 构造器私有化
        private Singleton4Holder() {
        }

        // 在静态内部类中持有单例对象,并且可被直接初始化
        private static class Holder {
            private static Singleton4Holder INSTANCE = new Singleton4Holder();
        }

        // 对提供getInstance方法,获取单例对象
        public static Singleton4Holder getInstance() {
            return Holder.INSTANCE;
        }
    }
    ````
    - 解释说明：在Singleton类中并没有instance的静态成员，而是将其放到了静态内部类Holder之中，因此在Singleton类的初始化过程中并不会创建Singleton的实例，Holder类中定义了Singleton的静态变量,并且直接进行了实例化，当Holder被主动引用的时候则会创建Singleton的实例，Singleton实例的创建过程在Java程序编译时期收集至<clinit>()方法中，该方法又是同步方法，同步方法可以保证内存的可见性、JVM指令的顺序性和原子性、Holder方式的单例设计是最好的设计之一，也是目前使用比较广的设计之一。
- **利用枚举实现单例**
    ````java
    package com.dennis.conccurency.chapter12;

    /**
    * 描述： 【特点一】枚举类版本的单例模式,来源《Effective Java》,
    *       【特点二】利用枚举类本身就是final且只会被实例化一次的特性,
    *       【特点三】当且仅当枚举只有一个枚举属性时,该枚举属性就是该类型的一个实例 
    *       
    * @author Dennis
    * @version 1.0
    * @date 2020/3/16 13:15
    */
    public enum Singleton4Enum {
        INSTANCE;

        // 实例变量
        byte[] data = new byte[1024];

        public static Singleton4Enum getInstance() {
            return INSTANCE;
        }

        public void calculate(){
            // handle the data......
            System.out.println("handle the data......");
        }
    }
    ````
- 测试代码
    ````java
    package com.dennis.conccurency.chapter12;
    /**
    * 描述：测试类
    * @author   Dennis
    * @date     2020/3/16 13:21
    * @version  1.0
    */
    public class Test4Enum {
        public static void main(String[] args) {
            Singleton4Enum singleton = Singleton4Enum.getInstance();
            singleton.calculate();
        }
    }
    ````
## 观察者模式
- **场景描述：虽然Thread为我们提供了可获取状态，以及判断是否alive的方法，但是这些方法均是针对线程本身的，而我们提交的任务Runnable在运行过程中所处的状态如何是无法直接获得的，比如它什么时候开始，什么时候结束，最不好的一种体验是无法获得Runnable任务执行后的结果。一般情况下想要获得最终结果，我们不得不为Thread或者Runnable传人共享变量，但是在多线程的情况下，共享变量将导致资源的竞争从而增加了数据不一致性的安全隐患。**
- **当某个对象发生状态改变需要通知第三方的时候，观察者模式就特别适合胜任这样的工作**
- **类图框架：**
![](https://raw.githubusercontent.com/deninising/onlinepicture/master/blog/20200317162845.png)
- **观察者接口：** 该接口主要是暴露给调用者使用的，其中四个枚举类型分别代表了当前任务执行生命周期的各个阶段，具体如下。

    ````java
    package com.dennis.conccurency.chapter13;

    /**
    * 描述：观察者模式
    *
    * @author Dennis
    * @version 1.0
    * @date 2020/3/17 12:01
    */
    public interface Observable {
        // 任务生命周期的枚举类
        enum Cycle {
            STARTED, RUNNING, DONE, ERROR;
        }

        // 获取当前任务的生命周期
        Cycle getCycle();

        // 定义任务线程的启动方法，主要是为了屏蔽Thread的其他方法
        void start();

        // 定义任务的中断方法，主要是为了屏蔽Thread的其他方法
        void interrupt();
    }
    ````
- **任务的生命周期接口：** TaskLifecycle接口定义了在任务执行的生命周期中会被触发的接口，其中EmptyLifycycle是-一个空的实现，主要是为了让使用者保持对Thread类的使用习惯。

    ````java
    package com.dennis.conccurency.chapter13;

    /**
    * 描述：主要定义任务执行的生命周期中将会被触发的接口
    *
    * @author Dennis
    * @version 1.0
    * @date 2020/3/17 12:10
    */
    public interface TaskLifecycle<T> {
        // 任务启动时将会触发onStart方法
        void onStart(Thread thread);

        // 运行时触发
        void onRunning(Thread thread);

        // 结束时触发
        void onFinished(Thread thread, T result);

        // 异常时触发
        void onError(Thread thread, Exception e);

        // 生命周期接口的空实现
        class EmptyTaskLifecycle<T> implements TaskLifecycle<T> {

            @Override
            public void onStart(Thread thread) {
                // do nothing
            }

            @Override
            public void onRunning(Thread thread) {
                // do nothing
            }

            @Override
            public void onFinished(Thread thread, T result) {
                // do nothing
            }

            @Override
            public void onError(Thread thread, Exception e) {
                // do nothing
            }
        }
    }
    ````
- **任务接口：**
    ````java
    package com.dennis.conccurency.chapter13;

    /**
    * 描述： 任务函数式接口
    * 由于我们需要对线程中的任务执行增加可观察的能力，并且需要获得最后的计算结果，因此Runnable接口
    * 在可观察的线程中将不再使用，取而代之的是Task接口，其作用与Runnable类似，主要用于承载任务的
    * 逻辑执行单元。
    *
    * @author Dennis
    * @version 1.0
    * @date 2020/3/17 12:16
    */
    @FunctionalInterface
    public interface Task<T> {
        // 唯一调用方法，不接受参数，有一返回值
        T call();
    }
    ````
- **观察者线程：** ObservableThread是任务监控的关键，它继承自Thread类和Observable接口，并且在构造期间需要传人Task的具体实现。
    ````java
    package com.dennis.conccurency.chapter13;

    import com.sun.crypto.provider.PBEWithMD5AndDESCipher;

    /**
    * 描述：观察者线程实现类：ObservableThread是任务监控的关键，它继承自Thread类和Observable接口，并且在构造期间需要传人Task的具体实现。
    *
    * @author Dennis
    * @version 1.0
    * @date 2020/3/17 12:22
    */
    public class ObservableThread<T> extends Thread implements Observable {
        private final TaskLifecycle<T> lifecycle;
        private final Task<T> task;

        private Cycle cycle;


        // 指定Task的实现，但是TaskLifecycle默认为空实现
        public ObservableThread(Task<T> task) {
            this(new TaskLifecycle.EmptyTaskLifecycle<>(), task);
        }

        public ObservableThread(TaskLifecycle<T> taskLifecycle, Task<T> task) {
            super();
            // task不能为null
            if (task == null)
                throw new IllegalArgumentException("the task is required");
            this.lifecycle = taskLifecycle;
            this.task = task;
        }

        /**
        * 重写父类的run方法，并且将其修饰为final类型，不允许子类再次对其进行重写，run
        * 方法在线程的运行期间，可监控任务在执行过程中的各个生命周期阶段，任务每经过一个
        * 阶段相当于发生了-次事件。
        */
        @Override
        public final void run() {
            // 在执行任务逻辑单元的时候，分别触发相应的事件
            this.update(Cycle.STARTED, null, null);
            try {
                this.update(Cycle.RUNNING, null, null);
                // 执行任务，没有输入参数，有一返回结果
                T result = this.task.call();
                this.update(Cycle.DONE, result, null);
            } catch (Exception e) {
                e.printStackTrace();
                this.update(Cycle.ERROR, null, e);
            }
        }

        /**
        * update方法用于通知时间的监听者，此时任务在执行过程中发生了什么，最主要的通
        * 知是异常的处理。如果监听者也就是TaskLifecycle, 在响应某个事件的过程中出现了意外，
        * 则会导致任务的正常执行受到影响，因此需要进行异常捕获，并忽略这些异常信息以保证
        * TaskLifecycle的实现不影响任务的正确执行，但是如果任务执行过程中出现错误并且抛出
        * 了异常，那么update方法就不能忽略该异常，需要继续抛出异常，保持与call方法同样的
        * 意图。
        */
        private void update(Cycle cycle, T result, Exception e) {
            this.cycle = cycle;

            if (lifecycle == null)
                return;
            try {
                switch (cycle) {
                    case STARTED:
                        this.lifecycle.onStart(currentThread());
                        break;
                    case RUNNING:
                        this.lifecycle.onRunning(currentThread());
                        break;
                    case DONE:
                        this.lifecycle.onFinished(currentThread(), result);
                        break;
                    case ERROR:
                        this.lifecycle.onError(currentThread(), e);
                        break;
                }
            } catch (Exception ex) {
                if (cycle == Cycle.ERROR) {
                    throw ex;
                }
            }
        }

        @Override
        public Cycle getCycle() {
            return this.cycle;
        }
    }
    ````
- **测试一：**
    ````java
    package com.dennis.conccurency.chapter13;

    import java.util.concurrent.TimeUnit;

    /**
    * 描述：最简单的测试类
    *
    * @author Dennis
    * @version 1.0
    * @date 2020/3/17 13:39
    */
    public class TestV1 {
        /**
        * 这段程序与你平时使用Thread并没有太大的区别，只不过ObservableThread是一个泛
        * 型类，我们将其定义为Void类型，表示不关心返回值，默认的EmptyLifecycle同样表示不
        * 关心生命周期的每一个阶段
        */
        public static void main(String[] args) {
            ObservableThread observableThread = new ObservableThread<>(() -> {
                try {
                    TimeUnit.SECONDS.sleep(4);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("finished done");
                return null;
            });
            observableThread.start();
        }
    }
    ````
- **测试二：**
    ````java
    package com.dennis.conccurency.chapter13;

    import java.util.concurrent.TimeUnit;

    /**
    * 描述： 一个能返回任务执行结果的测试
    *
    * @author Dennis
    * @version 1.0
    * @date 2020/3/17 13:43
    */
    public class TestV2 {
        /**
        * 下面这段程序代码定义了一个需要返回值的ObservableThread,并且通过重写
        * EmptyLifecycle的onFinish方法监听（观察）输出最终的返回结果。
        */
        public static void main(String[] args) {
            final TaskLifecycle<String> taskLifecycle = new TaskLifecycle.EmptyTaskLifecycle<String>() {
                @Override
                public void onFinished(Thread thread, String result) {
                    System.out.println("the result of the task executed by the thread," + thread.getName() + ", is:" + result);
                }
            };

            ObservableThread<String> stringObservableThread = new ObservableThread<>(taskLifecycle, () -> {
                try {
                    TimeUnit.SECONDS.sleep(3);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                // call（）方法的返回结果如下
                return "hello observer!";
            });
            stringObservableThread.start();
        }
    }
    ````
- **关键点总结：**
![](https://raw.githubusercontent.com/deninising/onlinepicture/master/blog/20200317164454.png)
## Single Thread Execution设计模式
- **场景：Single Thread Execution模式是指在同一时刻只能有一个线程去访问共享资源，就像独木桥一样每次只允许一人通行，简单来说，Single Thread Execution 就是采用排他式的操作保证在同一时刻只能有一个线程访问共享资源。**
- **登机案件案例：**
- **安检机类：** 充当共享（临界）资源的角色
    ````java
    package com.dennis.conccurency.chapter14;

    import java.util.concurrent.TimeUnit;

    /**
    * 描述：安检类充当共享资源
    *
    * @author Dennis
    * @version 1.0
    * @date 2020/3/17 19:02
    */
    public class FlightSecurity {
        /**
        * 乘客名
        */
        private String passengerName;
        /**
        * 登机牌Id
        */
        private String passId;
        /**
        * 身份Id
        */
        private String idCard;

        /**
        * 当前检测人编号
        */
        int count;

        /**
        * 检测方法
        */
        public synchronized void  pass(String passengerName, String idCard, String passId) {

            //=======线程不安全=========
            this.idCard = idCard;
            this.passId = passId;
            //=======线程不安全=========
            
            this.passengerName = passengerName;
            count++;
            check(passId, idCard);
        }

        private void check(String passId, String idCard) {
            if (passId.charAt(0) != idCard.charAt(0)) {
                throw new RuntimeException("====Exception====" + toString());
            } else {
                System.out.println("passenger:" + count +" " +passengerName + " has passed");
            }
        }

        @Override
        public String toString() {
            return "the count " + count + "passenger:" + passengerName + ",idCard" + idCard + ",passId" + passId + "pass failure";
        }


    }

    ````
- **资源操作类（乘客）：**
    ````java
    package com.dennis.conccurency.chapter14;

    public class Passengers extends Thread {
        private String passengerName;
        private String passId;
        private String idCard;
        // 共享资源
        private final FlightSecurity flightSecurity;

        public Passengers(String passengerName, String passId, String idCard, FlightSecurity flightSecurity) {
            this.passengerName = passengerName;
            this.passId = passId;
            this.idCard = idCard;
            this.flightSecurity = flightSecurity;
        }

        @Override
        public void run() {
            while (true) {
                // 不断过安检
                this.flightSecurity.pass(passengerName, idCard, passId);
            }
        }

        public static void main(String[] args) {
            FlightSecurity securityMachine = new FlightSecurity();
            Passengers passengersThread01 = new Passengers("zhangsan01", "A_passId_1101", "A_idCard_1102", securityMachine);
            Passengers passengersThread02 = new Passengers("zhangsan02", "B_passId_1101", "B_idCard_1102", securityMachine);
            Passengers passengersThread03 = new Passengers("zhangsan03", "C_passId_1101", "C_idCard_1102", securityMachine);
            Passengers passengersThread04 = new Passengers("zhangsan04", "D_passId_1101", "D_idCard_1102", securityMachine);
            passengersThread01.start();
            passengersThread02.start();
            passengersThread03.start();
            passengersThread04.start();
        }
    }
    ````
    - 说明：本章类容详见chapter16中哲学家吃面问题
## 读写分离设计模式
- **场景：**
    ![](https://raw.githubusercontent.com/deninising/onlinepicture/master/blog/20200318173119.png)
- **类图设计：**
    ![](https://raw.githubusercontent.com/deninising/onlinepicture/master/blog/20200318173634.png)
    - **说明：** 以上各类代码详见D:\Code\github\thread_learn\src\main\java\com\dennis\conccurency\chapter15
- **ShareData类：**
    ````java
    package com.dennis.conccurency.chapter15;

    import java.util.ArrayList;
    import java.util.List;
    import java.util.concurrent.TimeUnit;

    /**
    * 描述： 读写锁的使用
    *
    * @author Dennis
    * @version 1.0
    * @date 2020/3/18 14:36
    */
    public class ShareData {
        // 定义共享数据（资源）
        private final List<Character> container = new ArrayList<>();
        // 读写锁工厂
        private final ReadWriteLock readWriteLock = ReadWriteLock.getReadWriteLockImpl();
        // 获取读取锁
        private ReadLock readLock = readWriteLock.readLock();
        // 获取写入锁
        private WriteLock writeLock = readWriteLock.writeLock();

        private final int length;

        // 初始化共享数据
        public ShareData(int length) {
            this.length = length;
            for (int i = 0; i < length; i++) {
                container.add(i, 'C');
            }
        }

        // 读方法采用读取锁进行控制
        public char[] read() throws InterruptedException {
            try {
                readLock.lock();
                char[] chars;
                int size = container.size();
                chars = new char[size];
                for (int i = 0; i < length; i++) {
                    chars[i] = container.get(i);
                }
                slowly();
                return chars;
            } finally {
                // 操作关闭将锁释放
                readLock.unlock();
            }
        }

        // 写操作采用写入锁进行控制
        public void write(char c) throws InterruptedException {
            try {
                writeLock.lock();
                for (int i = 0; i < length; i++) {
                    container.add(i, c);
                }
                slowly();
            } finally {
                // 释放锁
                writeLock.unlock();
            }
        }

        private void slowly() {
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
    ````
- **测试类：**
    ````java
    package com.dennis.conccurency.chapter15;

    import java.util.stream.IntStream;

    /**
    * 描述：读写锁测试
    *
    * @author Dennis
    * @version 1.0
    * @date 2020/3/18 16:25
    */
    public class ReadWriteLockTest {
        // 定义异常待写入的字符串
        private static final String textToWrite = "this string is for writing into the container";

        public static void main(String[] args) {
            final ShareData data = new ShareData(50);
            // 2个线程用于写操作
            IntStream.range(0, 2).forEach(i -> {
                new Thread(() -> {
                    for (int n = 0; n < textToWrite.length(); n++) {
                        try {
                            data.write(textToWrite.charAt(n));
                            System.out.println(Thread.currentThread().getName() + "write:" + textToWrite.charAt(n));
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                }, "write" + i).start();
            });
            // 10个线程用于读操作
            IntStream.range(0, 10).forEach(i -> {
                new Thread(() -> {
                    while (true) {
                        try {
                            System.out.println(Thread.currentThread().getName() + "read:" + new String(data.read()));
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                }, "read" + i).start();
            });
        }
    }
    ````
- **总结：**
    ![](https://raw.githubusercontent.com/deninising/onlinepicture/master/blog/20200318174351.png)
    ![](https://raw.githubusercontent.com/deninising/onlinepicture/master/blog/20200318174447.png)
## 不可变对象设计模式（对该类下一个对象的每次操作都会返回一个新的对象）
**该节类容详见第十八章**
- **线程不安全累加器及synchronized方式解决不安全方式**
    ````java
    package com.dennis.conccurency.chapter16;

    import java.util.concurrent.TimeUnit;
    import java.util.stream.IntStream;

    /**
    * 描述：非线程安全的累加器
    *
    * @author Dennis
    * @version 1.0
    * @date 2020/3/19 11:21
    */
    public class IntegerAccumulator {
        private int init;

        public IntegerAccumulator(int init) {
            this.init = init;
        }

        // 加i
        public int add(int i) {
            this.init += i;
            return this.init;
        }

        // 获取值
        public int getValue() {
            return this.init;
        }

        public static void main(String[] args) {
            IntegerAccumulator accumulator = new IntegerAccumulator(0);

        /* IntStream.range(0, 3).forEach(item -> {
                new Thread(() -> {
                    int inc = 0;
                    while (true) {
                        int oldValue = accumulator.getValue();
                        int result = accumulator.add(inc);
                        if (oldValue + inc != result) {
                            System.out.println("error:" + oldValue + "+" + inc + " does not equal " + result);
                        } else {
                            System.out.println("calculate correctly!!!");
                        }
                        try {
                            TimeUnit.MILLISECONDS.sleep(100);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        inc++;
                    }
                }, "thread-" + item).start();
            });*/

            // 采用synchronized关键字解决同步解决线程安全问题
            IntStream.range(0, 3).forEach(item -> {
                new Thread(() -> {
                    int inc = 0;
                    while (true) {
                        int oldValue;
                        int result;
                        synchronized (IntegerAccumulator.class) {
                            oldValue = accumulator.getValue();
                            result = accumulator.add(inc);
                        }
                        if (oldValue + inc != result) {
                            System.out.println("error:" + oldValue + "+" + inc + " does not equal " + result);
                        } else {
                            System.out.println("calculate correctly!!!");
                        }
                        try {
                            TimeUnit.MILLISECONDS.sleep(100);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        inc++;
                    }
                }, "thread-" + item).start();
            });
        }
    }

    ````
- **不可变对象解决累加器‘‘共享数据’’不安全问题**
    ````java
    package com.dennis.conccurency.chapter16;

    import java.util.concurrent.TimeUnit;
    import java.util.stream.IntStream;

    /**
    * 描述： 不可变类，用final修饰类防止被继承重写
    *
    * @author Dennis
    * @version 1.0
    * @date 2020/3/19 11:38
    */
    public final class ImmutableIntAccumulator {
        private int init;

        public ImmutableIntAccumulator(int init) {
            this.init = init;
        }
        
        // 构造新的累加器，需要用到另外一个immutableIntAccumulator和初始值
        private ImmutableIntAccumulator(ImmutableIntAccumulator intAccumulator, int init) {
            this.init = intAccumulator.getValue() + init;
        }

        // 每次相加都会产生一个新的ImmutableIntAccumulator
        public final ImmutableIntAccumulator add(int inc) {
            return new ImmutableIntAccumulator(this, inc);
        }

        public int getValue() {
            return this.init;
        }

        public static void main(String[] args) {
            final ImmutableIntAccumulator[] accumulator = {new ImmutableIntAccumulator(0)};
            // 采用synchronized关键字解决同步解决线程安全问题
            IntStream.range(0, 3).forEach(item -> {
                new Thread(() -> {
                    int inc = 0;
                    while (true) {
                        int oldValue = accumulator[0].getValue();
                        ImmutableIntAccumulator newCalculator = accumulator[0].add(inc);
                        int result = newCalculator.getValue();
                        accumulator[0] = newCalculator;
                        if (oldValue + inc != result) {
                            System.out.println("error:" + oldValue + " + " + inc + " != " + result);
                        } else {
                            System.out.println("correct:" + oldValue + " + " + inc + " = " + result);
                        }
                        try {
                            TimeUnit.MILLISECONDS.sleep(50);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        inc++;
                    }
                }, "thread-" + item).start();
            });
        }
    }
    ````
## Future模式
- **场景：**
![](https://raw.githubusercontent.com/deninising/onlinepicture/master/blog/20200320104436.png)
- **类图设计：**
![](https://raw.githubusercontent.com/deninising/onlinepicture/master/blog/20200320105033.png)
- **Future接口：**
    ````java
    package com.dennis.conccurency.chapter17;

    /**
    * 描述：Future提供了获取计算结果和判断任务是否完成的两个接口，其中获取计算结果将会
    * 导致调用阻塞(在任务还未完成的情况下)。
    *
    * @author Dennis
    * @version 1.0
    * @date 2020/3/19 15:46
    */
    public interface Future<R> {
        // 用于获取结果
        R get() throws InterruptedException;

        //  用于判断内部任务是否执行结束
        boolean done();
    }
    ````
- **FutureService接口：**
    ````java
    package com.dennis.conccurency.chapter17;

    /**
    * 描述：   FutureService主要用于提交任务,提交的任务主要有两种，第-一种不需要返回值，第
    * 二种则需要获得最终的计算结果。FutureService 接口中提供了对FutureServiceImpl构建的
    * 工厂方法，JDK8中不仅支持default方法还支持静态方法,JDK9甚至还支持接口私有方法。
    *
    * @author Dennis
    * @version 1.0
    * @date 2020/3/19 15:51
    */
    public interface FutureService<IN, OUT> {
        // 无需返回结果
        Future<?> submit(Runnable runnable);

        // 提交任务且有返回结果，其中Task接口代替了Runnable接口
        Future<OUT> submit(Task<IN, OUT> task, IN input);

        // 提交任务且有返回结果，其中Task接口代替了Runnable接口
        Future<OUT> submit(Task<IN, OUT> task, IN input, Callback<OUT> callback);

        // 使用静态工厂方法创建了一个FutureService的实现类
        static <T, R> FutureService<T, R> newFutureService() {
            return new FutureServiceImpl<>();
        }
    }
    ````
- **Task接口：**
    ````java
   package com.dennis.conccurency.chapter17;

    /**
    * 描述：相当与Runnable接口，只是其接口方法改为有一传入参数且有返回值
    * Task接口主要是提供给调用者实现计算逻辑之用的，可以接受-一个参数并且返回最终
    * 的计算结果，这一点非常类似于JDK1.5中的Callable接口.
    *
    * @author Dennis
    * @version 1.0
    * @date 2020/3/19 16:01
    */
    @FunctionalInterface
    public interface Task<IN, OUT> {
        // 相当于Runnable的run方法,给定一个参数,经计算得到返回结果
        OUT get(IN input);
    }
    ````
- **FutureTask实现类：**
    ````java
    package com.dennis.conccurency.chapter17;

    /**
    * 描述：FutureTask是Future的- -个实现，除了实现Future中定义的get()以及done() 方法，还
    * 额外增加了protected 方法finish， 该方法主要用于接收任务被完成的通知.
    *
    * FutureTask中充分利用了线程间的通信:wait和notifyAll，当任务没有被完成之前通过
    * get方法获取结果，调用者会进入阻塞，直到任务完成并接收到其他线程的唤醒信号，finish
    * 方法接收到了任务完成通知，唤醒了因调用get而进人阻塞的线程。
    *
    * @author Dennis
    * @version 1.0
    * @date 2020/3/19 16:06
    */
    public class FutureTask<R> implements Future<R> {
        // 计算结果
        private R result;
        // 任务呢是否完成
        private boolean isDone;
        // 对象锁
        private final Object LOCK = new Object();

        @Override
        public R get() throws InterruptedException {
            synchronized (LOCK) {
                // 未计算完成,调用该方法的接口阻塞
                while (!isDone) {
                    LOCK.wait();
                }
                // 计算完成返回结果
                return this.result;
            }
        }

        //finish方法主要用于为FutureTask设置计算结果
        protected void finish(R result) {
            synchronized (LOCK) {
                if (isDone)
                    return;
                this.result = result;
                this.isDone = true;
                LOCK.notifyAll();
            }
        }

        @Override
        public boolean done() {
            return this.isDone;
        }
    }
    ````
- **FutureServiceImpl实现类：**
    ````java
    package com.dennis.conccurency.chapter17;

    import java.util.concurrent.atomic.AtomicInteger;

    /**
    * 描述：FutureServiceImpl的主要作用在于当提交任务时创建一个新的线程来受理该任务,进而达到任务异步执
    * 行的效果
    *
    * 在FutureServiceImpl的submit方法中，分别启动了新的线程运行任务，起到了异步的
    * 作用，在任务最终运行成功之后，会通知FutureTask任务已完成。
    *
    * @author Dennis
    * @version 1.0
    * @date 2020/3/19 15:59
    */
    public class FutureServiceImpl<IN, OUT> implements FutureService<IN, OUT> {
        // 为执行线程指定名字前缀
        private final static String FUTURE_THREAD_PREFIX = "FUTURE-";

        private final AtomicInteger nextCounter = new AtomicInteger(0);

        // 获取名称
        protected String getNextName() {
            return FUTURE_THREAD_PREFIX + nextCounter.getAndIncrement();
        }

        @Override
        public Future<?> submit(Runnable runnable) {
            final FutureTask<Void> future = new FutureTask<>();
            new Thread(() -> {
                runnable.run();
                // 任务执行结束将null作为结果传给future;
                future.finish(null);
            }, getNextName()).start();
            return future;
        }

        @Override
        public Future<OUT> submit(Task<IN, OUT> task, IN input) {
            final FutureTask<OUT> future = new FutureTask<>();
            new Thread(() -> {
                OUT out = task.get(input);
                // 任务执行结束之后，将真实的结果通过finish方法传递给future
                future.finish(out);
            }, getNextName()).start();
            return future;
        }

        public Future<OUT> submit(Task<IN, OUT> task, IN input, Callback<OUT> callback) {
            final FutureTask<OUT> future = new FutureTask<>();
            new Thread(() -> {
                OUT result = task.get(input);
                future.finish(result);
                /**
                * 修改后的submit方法，增加了一个Callback参数，主要用来接受并处理任务的计算结
                * 果，当提交的任务执行完成之后，会将结果传递给Callback接口进行进一步的执行， 这样
                * 在提交任务之后,无需调用get方法获取执行的结果，因此避开在主线中因为调用get方法获得结果而陷入阻塞。
                */
                if (callback != null)
                    callback.call(result);
            }, getNextName()).start();
            return future;
        }
    }
    ````
- **Callback接口：**
    ````java
    package com.dennis.conccurency.chapter17;

    /**
    * 描述：回调接口
    *
    * @author Dennis
    * @version 1.0
    * @date 2020/3/19 17:20
    */
    @FunctionalInterface
    public interface Callback<R> {
        // 任务完成后会调用该方法，其中R为任务执行后的结果
        void call(R result);
    }
    ````
- **测试类：**
    - **不含回调接口：**
        ````java
        package com.dennis.conccurency.chapter17;

        import java.util.concurrent.TimeUnit;

        /**
        * 描述： 测试类
        *
        * @author Dennis
        * @version 1.0
        * @date 2020/3/19 17:01
        */
        public class FutureTest01 {
            public static void main(String[] args) throws InterruptedException {
                /*FutureService<Void, Void> futureService = FutureService.newFutureService();
                Future<?> future = futureService.submit(() -> {
                    try {
                        TimeUnit.SECONDS.sleep(30);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println("异步任务执行结束！");
                });
                System.out.println("异步调用开始，异步任务正在执行......");
                // future的get方法将造成线程阻塞
                future.get();*/

                FutureService<String, Integer> futureService = FutureService.newFutureService();
                Future<Integer> future = futureService.submit(input -> {
                    try {
                        TimeUnit.SECONDS.sleep(10);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println("异步任务执行结束！");
                    return input.length();
                }, "hello world");
                System.out.println("异步调用开始，异步任务正在执行......");
                // future的get方法将造成线程阻塞
                System.out.println(future.get());
            }
        }
        ````
    - **含有回调接口：**
        ````java
        package com.dennis.conccurency.chapter17;

        import java.util.concurrent.TimeUnit;

        /**
        * 描述：
        *
        * @author Dennis
        * @version 1.0
        * @date 2020/3/19 17:34
        */
        public class FutureTest02 {
            public static void main(String[] args) {
                FutureService<String, Integer> futureService = FutureService.newFutureService();
                futureService.submit(input -> {
                    try {
                        // 模拟任务执行耗时
                        TimeUnit.SECONDS.sleep(10);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println("异步任务执行结束！结果如下：");
                    return input.length();
                }, "hello world", System.out::println);
                System.out.println("异步调用开始，异步任务正在执行......");
            }
        }
        ````
## Guarded Suspension 设计模式
- **场景：**
![](https://raw.githubusercontent.com/deninising/onlinepicture/master/blog/20200320114255.png)
- **实例代码：**
    ````java
    package com.dennis.conccurency.chapter18;

    import java.util.LinkedList;

    /**
    * 描述：确保挂起模式，条件不满足则挂起，反之开启
    *
    * @author Dennis
    * @version 1.0
    * @date 2020/3/20 11:31
    */
    public class GuardedSuspensionQueue {
        // 资源共享队列
        private final LinkedList<Integer> queue = new LinkedList<>();
        // 条件
        private final int LIMIT = 100;

        public void offer(Integer data) {
            synchronized (this) {
                while (this.queue.size() >= LIMIT) {
                    try {
                        this.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                this.queue.addLast(data);
                this.notifyAll();
            }
        }

        public Integer take() {
            synchronized (this) {
                while (this.queue.isEmpty()) {
                    try {
                        this.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                Integer integer = this.queue.removeFirst();
                this.notifyAll();
                return integer;
            }
        }
    }
    ````
- **总结：Guarded Suspension模式是一个非常基础的设计模式，它主要关注的是当某个条件(临界值)不满足时将操作的线程正确地挂起，以防止出现数据不一致或者操作超过临界值的控制范围。Guarded Suspension设计模式并不复杂，但是它是很多其他线程设计模式的基础，比如生产者消费者模式，后文中的Thread Worker设计模式，Balking 设计模式等，都可以看到Guarded Suspension模式的影子，Guarded Suspension的关注点在于临界值的条件是否满足，当达到设置的临界值时相关线程则会被挂起。** 

# 阶段三：JDK并发包详解
## Atomic包
### CAS算法
- CAS算法是一种对比设置算法，当其需要对某个volatile共享变量进行操作的时候，1、先获取该值，2、再比较获取到的值与期望值是否相等，相等则成功设置否则回到步骤1（自旋操作）
 **说明** 步骤2看似分为两步，实则底层调用cpu级别指令>>>为一原子操作。保证享变量的**可见性、有序性、原子性**
### 数字原子类型
- AtomicInteger
- AtomicBoolean：对boolean值采取原子操作控制，0>>>false , 1>>>true
- AtomicLong: 由于Long类型为8个字节，所以与AtomicInteger相比，它只是在cpu级别对总线进行的加锁操作，cpu中最小单元数据大小为4个字节（目前普遍这样）
- **案例：** 利用AtomicInteger事项非阻塞显示锁,利用compareAndSet(int expect,int update)方法的性质即：atomicInteger的原有值和传入的期望值不相等的时候该方法会返回false,且不会更新值
````java
    package com.dennis.conccurency.atomic.chapter01;

    import java.util.concurrent.atomic.AtomicInteger;

    /**
    * 描述： 利用AtomicInteger.compareAndSet(Int expect,Int value)的快速失败特性,来创建一个非阻塞锁
    *
    * @author Dennis
    * @version 1.0
    * @date 2020/3/31 16:36
    */
    public class UnBlockedLock {
        private final int UNLOCKED = 0;
        private final int LOCKED = 1;

        private final AtomicInteger LOCK_STATUS = new AtomicInteger(this.UNLOCKED);

        // 存放正持有锁资源的线程
        private Thread localThread;

        // 加锁,若获取锁资源失败,则直接抛异常-->fail-fast,放弃竞争锁资源
        public void tryLock() throws GetLockException {
            boolean success = LOCK_STATUS.compareAndSet(UNLOCKED, LOCKED);
            if (!success) {
                throw new GetLockException("it is failed to get the lock");
            }
            localThread = Thread.currentThread();
        }

        // 解锁
        public void unLock() {
            if (Thread.currentThread() != localThread || LOCK_STATUS.get() == UNLOCKED) {
                return;
            }
            LOCK_STATUS.compareAndSet(LOCKED, UNLOCKED);
        }
    }
````
- **测试：**
````java
    package com.dennis.conccurency.atomic.chapter01;

    import java.util.concurrent.TimeUnit;
    import java.util.stream.IntStream;

    public class UnBlockedLockTest {
        public static void main(String[] args) {
            UnBlockedLock lock = new UnBlockedLock();
            // 起5个工作线程
            IntStream.rangeClosed(1, 5).forEach(workThreadNum -> {
                new Thread(() -> {
                    try {
                        lock.tryLock();
                        // do working;
                        for (int i = 0; i <= 30; i++) {
                            System.out.println(Thread.currentThread().getName() + " is working......");
                            TimeUnit.MILLISECONDS.sleep(500);
                        }
                    } catch (GetLockException | InterruptedException e) {
                        e.printStackTrace();
                    } finally {
                        lock.unLock();
                    }
                }, "thread_" + workThreadNum).start();
            });
        }
    }
````
### 引用原子类型
- AtomicReference：用于自定义类型具备原子类型特点的设定
- AtomicStampedReference：解决自定义原子类型（链表的原子操作或栈的原子操作......）中A-B-A问题
- **自定义原子类案例**
    ````java
        package com.dennis.conccurency.atomic.chapter02;

        import java.util.concurrent.TimeUnit;
        import java.util.concurrent.atomic.AtomicReference;
        import java.util.concurrent.atomic.AtomicStampedReference;

        /**
        * 描述：解决A->B->A问题案例
        *
        * @author Dennis
        * @version 1.0
        * @date 2020/4/1 15:38
        */
        public class AtomicStampedReferenceTest {

            public static void main(String[] args) {
                Node<Student> studentNode01 = new Node<>(new Student("z01", 11));
                studentNode01.setPre(null);
                studentNode01.setNext(null);
                AtomicStampedReference<Node<Student>> atomicStudentNode = new AtomicStampedReference<>(studentNode01, 1);

                // a = "abc"; // 匿名内部类中无法访问该对象，需要做包装
                // 可以用AtomicReference<T>来包装匿名内部类将要访问并进行修改的对象T,不用数组进行包装

                    /*局部内部类和匿名内部类只能访问局部final变量"
                    从JDK 1.8开始，会默认给这两种内部类访问(读操作)的field 加上final(隐式地)，
                    所以你可能会在编译器中看到可以访问没有加final的变量，只有你去修改(写操作)它时，
                    编译器才会报错。总结：内部类访问外部类的变量本质上是对外部变量的一份拷贝的访问操
                    作,从而会引发数据不一致问题,因此需要对外部变量进行final修饰*/

                    AtomicReference<String> a = new AtomicReference<>("abc");
        //          String a = "abc";
                new Thread(() -> {
                    a.set("dfd");
        //            a = "dfd";

                // 可以用AtomicReference<T>来包装匿名内部类将要访问并进行修改的对象T,不用数组进行包装

                    int stamp = atomicStudentNode.getStamp();
                    try {
                        TimeUnit.SECONDS.sleep(3);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    Node<Student> reference = atomicStudentNode.getReference();
                    Node<Student> studentNode02 = new Node<>(new Student("z02", 12));
                    int realityStamp = atomicStudentNode.getStamp();
                    boolean success = atomicStudentNode.compareAndSet(reference, Node.appendAndNew(reference, studentNode02), stamp, stamp + 1);
                    if (success) {
                        System.out.println("1-成功设值！！！");
                        System.out.println(atomicStudentNode.getReference());
                        System.out.println("expect stamp ->"+stamp);
                        System.out.println("reality stamp ->"+realityStamp);
                    } else {
                        System.out.println("1-设置失败！！！");
                        System.out.println("expect stamp ->"+stamp);
                        System.out.println("reality stamp ->"+realityStamp);
                    }
                }).start();
                new Thread(() -> {
                    int stamp = atomicStudentNode.getStamp();
                    Node<Student> reference = atomicStudentNode.getReference();
                    Node<Student> studentNode02 = new Node<>(new Student("z02", 12));
                    int realityStamp = atomicStudentNode.getStamp();
                    boolean success = atomicStudentNode.compareAndSet(reference, Node.appendAndNew(reference, studentNode02), stamp, stamp + 1);
                    if (success) {
                        System.out.println("2-成功设值！！！");
                        System.out.println(atomicStudentNode.getReference());
                        System.out.println("expect stamp ->"+stamp);
                        System.out.println("reality stamp ->"+realityStamp);
                    } else {
                        System.out.println("2-设置失败！！！");
                        System.out.println("expect stamp ->"+stamp);
                        System.out.println("reality stamp ->"+realityStamp);
                    }
                }).start();
            }
            static class Node<T> {
                private Node<T> pre;
                private Node<T> next;
                private T value;

                public Node(T value) {
                    this.value = value;
                }

                public T getValue() {
                    return value;
                }

                public void setValue(T value) {
                    this.value = value;
                }

                public Node<T> getPre() {
                    return pre;
                }

                public void setPre(Node<T> pre) {
                    this.pre = pre;
                }

                public Node<T> getNext() {
                    return next;
                }

                public Node<T> setNext(Node<T> next) {
                    this.next = next;
                    return this;
                }

                static <T> Node<T> appendAndNew(Node<T> oldNode, Node<T> tail) {
                    oldNode.recAppend(oldNode, tail);
                    return new Node<>(oldNode.getValue()).setNext(oldNode.getNext());
                }

                // 递归添加设置尾部节点,一般还是推荐使用循环
                private void recAppend(Node<T> chainNode, Node<T> tail) {
                    Node<T> next;
                    if ((next = chainNode.getNext()) != null) {
                        recAppend(next, tail);
                    } else {
                        chainNode.setNext(tail);
                    }
                }

                @Override
                public String toString() {
                    return "Node{" +
                            "pre=" + pre +
                            ", next=" + next +
                            ", value=" + value +
                            '}';
                }
            }

            static class Student {
                private String name;
                private Integer age;

                public Student() {
                }

                public Student(String name, Integer age) {
                    this.name = name;
                    this.age = age;
                }

                public String getName() {
                    return name;
                }

                public void setName(String name) {
                    this.name = name;
                }

                public Integer getAge() {
                    return age;
                }

                public void setAge(Integer age) {
                    this.age = age;
                }
            }
        }

    ````

### 数组元素原子类型(不做重点)
- AtomicIntegerArray
- AtomicLongArray
- AtomicReferenceArray
### 字段属性原子类型（不做重点）
- AtomicIntegerFieldUpdater
- AtomicLongFieldUpdater
- AtomicReferenceFieldUpdater
## utils包

### CountDownLatch
- 计数器递减锁：调用latch.wait()方法的线程将被阻塞，直到其他线程使latch的值递减到0时，被阻塞住的线程被唤醒
- **案例：** 异步执行场景中,采用传统方式控制主线程
    ````java
        package com.dennis.conccurency.atomic.chapter03;

        import java.util.concurrent.ExecutorService;
        import java.util.concurrent.Executors;
        import java.util.concurrent.TimeUnit;

        /**
        * 描述：异步执行场景中,采用传统方式控制主线程
        *
        * @author Dennis
        * @version 1.0
        * @date 2020/4/1 22:28
        */
        public class TraditionAsyncThreadControl {
            // 线程池:不建议利用Executors工具类进行线程池的创建
            private static ExecutorService pool = Executors.newFixedThreadPool(2);

            public static void main(String[] args) throws InterruptedException {
                // [1]查询到数据
                int[] data = queryData();
                // [2]对查询到的数据进行处理:通过线程池中的线程进行异步处理
                for (int i = 0; i < data.length; i++) {
                    pool.execute(new DataHandleTask(data, i));
                }
                // [3]后续操作
                // 1.主线程虽然完成但是线程池中的线程任然为active状态
        //        System.out.println("the master thread has finished!!!");

                // 2.线程池执行完成,则池中线程也将关闭,该方法仍然是非阻塞方法,后续代码可被执行
        //        pool.shutdown();
        //        System.out.println("the master thread has finished!!!");

                // 3.阻塞当前线程1个小时的时间
        //        pool.awaitTermination(1, TimeUnit.HOURS);
        //        System.out.println("the master thread has finished!!!");

                // 4.预期阻塞当前线程1个小时的时间,在池期间线程池中任务若是全部成功执行完成,则阻塞结束,所有线程结束（推荐）
                pool.shutdown();
                pool.awaitTermination(1, TimeUnit.HOURS);
                System.out.println("the master thread has finished!!!");
            }

            static int[] queryData() {
                int[] data = new int[]{1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
                return data;
            }

            private static class DataHandleTask implements Runnable {
                private final int[] data;
                private final int index;

                private DataHandleTask(int[] data, int index) {
                    this.data = data;
                    this.index = index;
                }

                @Override
                public void run() {
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    if (data[index] % 2 == 0) {
                        data[index] = data[index] * 2;
                    }
                    System.out.println(Thread.currentThread().getThreadGroup());
                    System.out.println("the data of index of " + index + " has been precessed already");
                }
            }

        }
    ````
- **案例：** 异步执行场景中,采用CountDownLatch进行线程间的控制
    ````java
        package com.dennis.conccurency.countdownlatch.chapter01;

        import java.util.concurrent.CountDownLatch;
        import java.util.concurrent.ExecutorService;
        import java.util.concurrent.Executors;
        import java.util.concurrent.TimeUnit;

        /**
        * 描述：异步执行场景中,采用CountDownLatch进行线程间的控制
        *
        * @author Dennis
        * @version 1.0
        * @date 2020/4/2 10:30
        */
        public class CountDLatchAsyncThreadControl {
            // 线程池:不建议利用Executors工具类进行线程池的创建
            private static ExecutorService pool = Executors.newFixedThreadPool(2);

            public static void main(String[] args) throws InterruptedException {
                // [1]查询到数据
                int[] data = queryData();
                // 创建CountDownLatch变量
                CountDownLatch latch = new CountDownLatch(data.length);

                // [2]对查询到的数据进行处理:通过线程池中的线程进行异步处理
                for (int i = 0; i < data.length; i++) {
                    pool.execute(new CountDLatchAsyncThreadControl.DataHandleTask(data, i, latch));
                }
                // [3]后续操作
                // 1.主线程虽然完成但是线程池中的线程任然为active状态
        //        System.out.println("the master thread has finished!!!");

                // 2.线程池执行完成,则池中线程也将关闭,该方法仍然是非阻塞方法,后续代码可被执行
        //        pool.shutdown();
        //        System.out.println("the master thread has finished!!!");

                // 3.阻塞当前线程1个小时的时间
        //        pool.awaitTermination(1, TimeUnit.HOURS);
        //        System.out.println("the master thread has finished!!!");

                // 4.预期阻塞当前线程1个小时的时间,在池期间线程池中任务若是全部成功执行完成,则阻塞结束,所有线程结束（推荐）
        //        pool.shutdown();
        //        pool.awaitTermination(1, TimeUnit.HOURS);

                // 5.采用countDownLatch来阻塞住当前线程(优点,逻辑清晰,控制较传统方式根加灵活)
                // 预期阻塞当前线程1个小时的时间,在池期间线程池中任务若是全部成功执行完成,则阻塞结束,所有线程结束（推荐）
                pool.shutdown();
                latch.await(1, TimeUnit.HOURS);
                System.out.println("the master thread has finished!!!");
            }

            static int[] queryData() {
                int[] data = new int[]{1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
                return data;
            }

            private static class DataHandleTask implements Runnable {
                private final int[] data;
                private final int index;
                private final CountDownLatch latch;

                private DataHandleTask(int[] data, int index, CountDownLatch latch) {
                    this.data = data;
                    this.index = index;
                    this.latch = latch;
                }

                @Override
                public void run() {
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    if (data[index] % 2 == 0) {
                        data[index] = data[index] * 2;
                    }
                    latch.countDown();
                    System.out.println("the data of index of " + index + " has been precessed already");
                }
            }
        }

    ````
- **应用场景**
     ![](https://raw.githubusercontent.com/deninising/onlinepicture/master/blog/20200402121629.png)
### CyclicBarrier
- **一个可循环使用的屏障阻塞器**
- **与CountDownLatch的对比：** 其工作发放时类似于CountDownLatch,但他工作线程之间相互阻塞,而CountDownLatch是调用latch.await（）方法所在的线程线程将被阻塞，且CyclicBarrier的一轮阻塞功能可被循环使用（没搞懂，深解析reset()方法）
- **案例**
    ````java
        package com.dennis.conccurency.utils.cyclicbarrier;

        import java.util.concurrent.BrokenBarrierException;
        import java.util.concurrent.CyclicBarrier;
        import java.util.concurrent.TimeUnit;
        import java.util.stream.IntStream;

        /**
        * 描述：其工作发放时类似于CountDownLatch,但他工作线程之间相互阻塞,
        * 而CountDownLatch是调用latch.await（）方法所在的线程线程将被阻塞
        *
        * 整体执行循序：【2】-->【1】-->【3】
        *
        * @author Dennis
        * @version 1.0
        * @date 2020/4/2 14:15
        */
        public class CyclicBarrierExample01 {

            public static void main(String[] args) throws InterruptedException {
                CyclicBarrier barrier = new CyclicBarrier(3, () -> {
                    // 【1】
                    System.out.println("they three worker-thread has all finished and their blocked status will be canceled !!!");
                    // 【1】
                });
                IntStream.rangeClosed(1, 3).forEach(num -> {
                    new Thread(() -> {
                        // 【2】
                        try {
                            TimeUnit.SECONDS.sleep(num-1);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        int limit = 1;
                        for (; limit <= 10; ) {
                            try {
                                TimeUnit.MILLISECONDS.sleep(200);
                            } catch (InterruptedException e) {
                                e.printStackTrace();
                            }
                            System.out.println(Thread.currentThread().getName() + "has finished the lift the stone " + limit + " times");
                            limit++;
                        }
                        System.out.println(Thread.currentThread().getName() + "has finished all the task and waiting others");
                        try {
                            barrier.await();
                        } catch (InterruptedException | BrokenBarrierException e) {
                            e.printStackTrace();
                        }
                        // 【2】

                        // 【3】
                        System.out.println("ok let·s go:says " + Thread.currentThread().getName());
                        // 【3】
                    }, "thread-" + num).start();
                });

                TimeUnit.MILLISECONDS.sleep(2);
                barrier.reset();
            }
        }
    ````
### Exchanger:线程间数据交互工具
- **案例**
    ````java
        package com.dennis.conccurency.utils.exchanger;

        import java.util.concurrent.Exchanger;
        import java.util.concurrent.TimeUnit;
        import java.util.concurrent.TimeoutException;

        /**
        * 描述：线程间（pair）数据交换工具类：线程间交互数据,另一个匹配线程若是没有交换点,则当前线程将处于阻塞状态
        *
        * @author Dennis
        * @version 1.0
        * @date 2020/4/2 19:06
        */
        public class ExchangerExample01 {
            public static void main(String[] args) {
                Exchanger<Student> exchanger = new Exchanger<>();

                new Thread(() -> {
                    String threadName = Thread.currentThread().getName();
                    Student studentA = new Student(threadName, 20);

                    for (; ; ) {
                        System.out.println(threadName + " is go along to change the data...");
                        try {
                            Student studentB = exchanger.exchange(studentA,1,TimeUnit.SECONDS);
                            System.out.println(threadName + " has get the data from the partner thread:" + studentB);
                        } catch (InterruptedException | TimeoutException e) {
                            e.printStackTrace();
                        }
                    }

                }, "thread-A").start();


                new Thread(() -> {
                    String threadName = Thread.currentThread().getName();
                    Student studentB = new Student(threadName, 30);

                    for (; ; ) {
                        try {
                            TimeUnit.SECONDS.sleep(3);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        System.out.println(threadName + " is go along to change the data...");
                        try {
                            Student studentA = exchanger.exchange(studentB);
                            System.out.println(threadName + " has get the data from the partner thread:" + studentA);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }

                }, "thread-B").start();
            }

            static class Student {
                private String name;
                private int age;

                public Student(String name, int age) {
                    this.name = name;
                    this.age = age;
                }

                public String getName() {
                    return name;
                }

                public void setName(String name) {
                    this.name = name;
                }

                public int getAge() {
                    return age;
                }

                public void setAge(int age) {
                    this.age = age;
                }

                @Override
                public String toString() {
                    return "Student{" +
                            "name='" + name + '\'' +
                            ", age=" + age +
                            '}';
                }
            }
        }
    ````
### 信号量控制对象
- **案例：使用semaphoer实现显示锁(支持阻塞和非阻塞)**
    ````java
        package com.dennis.conccurency.utils.semapphore;

        import java.util.concurrent.Semaphore;
        import java.util.concurrent.TimeUnit;

        /**
        * 描述：采用Semaphore实现显示锁
        *
        * @author Dennis
        * @version 1.0
        * @date 2020/4/2 21:02
        */
        public class SemaphoreExample01 {

            public static void main(String[] args) {
                SemaphoreLock lock = new SemaphoreLock();
                new Thread(() -> {
                    try {
                        lock.lock();
                        String name = Thread.currentThread().getName();
                        for (int i = 0; i <= 10; i++) {
                            TimeUnit.MILLISECONDS.sleep(1000);
                            System.out.println(name + " is deal with shared data!!!");
                        }
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    } finally {
                        lock.unLock();
                    }
                }, "thread-A").start();

                Thread tB = new Thread(() -> {

                    try {
                        // 阻塞锁
                        lock.lock();
                        // 非阻塞锁
        //                lock.tryLock();
                        // 自定义阻塞时长锁
        //                lock.tryLock(1, TimeUnit.SECONDS);
                        String name = Thread.currentThread().getName();
                        for (int i = 0; i <= 10; i++) {
                            TimeUnit.MILLISECONDS.sleep(1000);
                            System.out.println(name + " is deal with shared data!!!");
                        }
                    } catch (Exception e) {
                        e.printStackTrace();
                    } finally {
                        lock.unLock();
                    }
                }, "thread-B");
                // jvm停止时间->jvm中所有非守护线程退出了,则jvm退出
        //        tB.setDaemon(true);
                tB.start();
            }

            static class SemaphoreLock extends Semaphore {

                private Thread currentThread;

                public SemaphoreLock() {
                    // 持有一张许可证
                    super(1);
                }

                // 阻塞锁
                private void lock() {
                    try {
                        // 获取许可证，若没有许可证可取,则当前线程将处于阻塞状态
                        this.acquire();
                        currentThread = Thread.currentThread();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }

                // 非阻塞锁
                private void tryLock() throws Exception {
                    // 获取许可证，若没有许可证可取,则返回false否则返回true
                    if (!this.tryAcquire())
                        throw new Exception("failed to get the lock source");
                    currentThread = Thread.currentThread();
                }

                // 自定义阻塞时长锁
                private void tryLock(long time, TimeUnit timeUnit) throws Exception {
                    // 获取许可证，若没有许可证可取,则返回false否则返回true
                    if (!this.tryAcquire(time, timeUnit))
                        throw new Exception("failed to get the lock source");
                    currentThread = Thread.currentThread();
                }

                private void unLock() {
                    if (Thread.currentThread() != currentThread) {
                        return;
                    }
                    this.release();
                }
            }
        }
    ````

# 阶段四：并发编程深入讨论
