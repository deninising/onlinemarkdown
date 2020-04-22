<center>
    <h1>
        jvm深入学习
    </h1>
</center>

## 类的加载机制
![](https://raw.githubusercontent.com/deninising/onlinepicture/master/blog/20200422105244.png)
![](https://raw.githubusercontent.com/deninising/onlinepicture/master/blog/20200422105900.png)
![](https://raw.githubusercontent.com/deninising/onlinepicture/master/blog/20200422110446.png)
![](https://raw.githubusercontent.com/deninising/onlinepicture/master/blog/20200422112058.png)
- **类的生命周期：**
    ![](https://raw.githubusercontent.com/deninising/onlinepicture/master/blog/20200422114203.png)
- **类的加载时机：**
    ![](https://raw.githubusercontent.com/deninising/onlinepicture/master/blog/20200422113625.png)
    - 虚拟机规范中没有强行的规定类加载时机，这点交予虚拟机实现者自由控制，唯一能确定的是类的加载一定会在类的初始阶段之前
- **类的初始化时机：被Java程序首次主动使用时**
    - 遇到new **（创建类的实例）**、getstatic **（访问类的静态变量）**、putstatic **（对类的静态变量赋值）**、invokestatic **（调用类的静态方法）** ,这4条字节码指令
    - 对类进行反射调用时
    - 初始化一个类时，会先初始化它的父类（存在父类且父类还未被初始化）
    - 当虚拟机启动时，会自动初始化带main()函数的主类

- **测试案例：**
    ````java
        package com.dennis.classloading;
        /**
        * 描述：类的加载，连接（验证二进制文件正确性、准备静态属性设置默认值、解析符号引用改为直接地址引用）、初始化（静态变量赋初始值）
        *      -XX：+TraceClassLoading,用于追踪类的加载信息并打印
        *      -XX：-<options> 关闭jvm中options选项
        *      -XX：+<options> 开启jvm中options选项
        *      -XX：<options>=<value> 开启jvm中options选项赋值
        *
        * @author Dennis
        * @version 1.0
        * @date 2020/4/22 9:53
        */
        public class ClassLoadTest01 {
            public static void main(String[] args) {
                /*
                *  1、对静态属性的调用，只有直接定义了它的类才会被初始化
                *  2、当一个类被初始化时，其所父类也将被初始化
                *  3、父类static块中的代码优先执行
                */
        //        System.out.println(Child.str); //问题：Child类没有被初始化但是其类信息有没有被加载呢？-XX：+TraceClassLoading,用于追踪类的加载信息并打印
                System.out.println(Child.str1);
            }
        }

        class Child extends Parent {

            public static String str1 = "hello child!";

            // 类的静态代码块在类的初始化阶段执行
            static {
                System.out.println("child has been initialized!");
            }
        }

        class Parent {
            public static String str = "hello parent!";

            static {
                System.out.println("parent has been initialized!");
            }
        }
    ````

- **final关键字的本质作用，以及相关助记符**
    ````java
        package com.dennis.classloading;

        /**
        * 描述：final关键字的本质作用，以及相关助记符
        *
        * @author Dennis
        * @version 1.0
        * @date 2020/4/22 14:52
        */
        public class ClassLoadTest02 {

            public static void main(String[] args) {
                /*
                * 编译期常量在编译阶段会存入到调用这个常量的方法所在的类的常量池中去
                * 本质上，在程序运行阶段，调用的类并没有直接引用定义常量的类，
                * 因此不会去触发定义常量的类的初始化
                * 【注意】本例中，在编译阶段，在MyParent类中定义的常量STR会被
                *  存储到ClassLoadTest02类中的常量池中去，之后ClassLoadTest02与
                *  Parent02之间就没有关系了，甚至可以将MyParent.class文件删除
                */
        //        System.out.println(Parent02.str);
        //        System.out.println(Parent02.STR);

                // 【助记符】
                //iconst_m1:表示将常量值-1从常量池推送至栈顶                                    =====>-1
                //iconst_n:表示将范围在[0,5]内的常量值从常量池推送至栈顶                          =====>0,1,2,3,4,5
                //bipush:表示将范围在[-128,-2]U[6,127]内的常量值从常量池推至栈顶                  =====>byte
                //sipush:表示将范围在[-32768,-129]U[128,32767]内的常量值从常量池推至栈顶          ====>char,short
                //ldc:表示将范围在[-2147483648,-32769]U[32768,2147483647],float或String类型   =====>int,float,string
                // 的常量值从常量池中推至栈顶
                //ldc2_w：表示将long或者double类型常量值推送至栈顶                              =====>long,double
                System.out.println(Parent02.NUM);
        //        System.out.println(Parent02.L_NUM);
            }
        }

        class Parent02 {
            public static String str = "hello MyParent02";

            public static final String STR = "hello MyParent02";

            public static final  int NUM = 32767;

            public static final  long L_NUM = Long.MAX_VALUE;

            public static final  double D_NUM = 0.878;

            static {
                System.out.println("MyParent is initialized!");
            }
        }
    ````

- **运行期常量与编译期常量(前一案例)的区别：能否入常量池**
    ````java
        package com.dennis.classloading;

        /**
        * 描述：运行期常量与编译期常量的区别：运行期常量不会进去常量池，但是编译期常量会存储到常量池中
        *
        * @author Dennis
        * @version 1.0
        * @date 2020/4/22 17:03
        */
        public class ClassLoadTest03 {
            public static void main(String[] args) {
                /**
                * 运行期常量不会像编译期常量那样，在编译期间就会将常量值存储到调用该常量的方法的类的常量池中去，
                * 该常量值在编译期间是未知的，只有在程序的运行期间去主动调用定义该常量的类并获取常量值
                * 因此，定义运行期常量的类会被初始化且该常量也不会存储到常量池中
                */
                System.out.println(Parent03.UUID);
            }
        }

        class Parent03 {
            // 此为一运行期常量，在编译阶段无法确定
            public static final String UUID = java.util.UUID.randomUUID().toString();
            static {
                System.out.println("class Parent has been initialized!");
            }
        }
    ````
- **数组创建的本质**
    ````java
        package com.dennis.classloading;

        /**
        * 描述：数组创建的本质
        * 创建数组不会主动调用component元素类,jvm会自动创建一个将自定的元素作为数组的component的新的类型
        * 【对于数组来说，其类型是由JVM动态生成的】
        *
        * @author Dennis
        * @version 1.0
        * @date 2020/4/22 17:24
        */
        public class ClassLoadTest04 {

            public static void main(String[] args) {
                // 助记符：anewarray ===>component为引用类型
                // class [Lcom.dennis.classloading.Parent04;
                Parent04[] pArray = new Parent04[1];
                System.out.println(pArray.getClass());

                // 助记符：multianewarray
                // class [[Lcom.dennis.classloading.Parent04;
                Parent04[][] ppArray = new Parent04[1][1];
                System.out.println(ppArray.getClass());

                // 助记符：newarray ===>component为基本类型
                // class [I
                int[] intArray = new int[2];
                System.out.println(intArray.getClass());
            }
        }
        class Parent04 {
            public static String str = "hello parent";
            static {
                System.out.println("class Parent04 has been initialized!");
            }
        }
    ````
- **接口的加载规则：同类的加载规则相同**
    ````java
        package com.dennis.classloading;

        import java.util.UUID;

        /**
        * 描述：接口加载的规则同类的加载规则相同
        *
        * @author Dennis
        * @version 1.0
        * @date 2020/4/22 19:12
        */
        public class ClassLoadTest05 {
            public static void main(String[] args) {
                System.out.println(Child05.C_STR);
            }
        }

        interface Parent05 {
            public static final String P_STR = "hello parent interface";
        }

        interface Child05 extends Parent05 {
            public static  String C_STRING = "hello child interface";
            public static  String C_STR = UUID.randomUUID().toString();
        }
    ````
- **类加载过程综合练习案例：非常重要**
    ````java
        package com.dennis.classloading;

        /**
        * 描述：类加载过程中，准备阶段的重要意义
        * 【知识要点】
        * 1、类初始化前：必定完成准备阶段，所有属性值JVM自动设置为默认值
        * 2、类初始时机：首次对类进行主动调用
        * 3、类初始次数：初始化只执行一次
        * 4、类初始化顺序：由上至下顺序执行
        *
        * @author Dennis
        * @version 1.0
        * @date 2020/4/22 20:06
        */
        public class ClassLoadTest06 {
            public static void main(String[] args) {
                Singleton singleton = Singleton.getInstance();
                System.out.println("counter1:" + singleton.getCounter1());
                System.out.println("counter2:" + singleton.getCounter2());
            }
        }

        class Singleton {
            /**
            * 准被阶段所有的属性值均为默认值：counter1 = 0,counter2 = 0,singleton = null;
            * 准备阶段完成后对象才有可能被初始化，且初始化只能执行一次
            */
        //    private static int counter1;
        //    private static int counter2 = 0;

            /**
            * 类初始化时被赋值为 1
            */
            private static int counter1 = 1;


            /**
            * 对该类的主动调用，将首次也是唯一一次对类进行初始，构造器的初始化在static{}之前执行
            */
        //    private static Singleton singleton = new Singleton();

            static {
                System.out.println("class Singleton is been initialized");
            }

            // 构造器初始化阶段执行
            private Singleton() {
                System.out.println("constructor is invoking");
                counter1++;
                counter2++;
                System.out.println(counter1);
                System.out.println(counter2);
            }

            /**
            * 对该类的主动调用，将首次也是唯一一次对类进行初始,构造器的初始化在static{}之后执行
            */
            private static Singleton singleton = new Singleton();

            /**
            * 类初始化时被赋值为 0
            */
            private static int counter2 = 0;

            public static Singleton getInstance() {
                System.out.println("getInstance is been invoking");
                return singleton;
            }

            public int getCounter1() { 
                return counter1;
            }

            public int getCounter2() {
                return counter2;
            }
        }

    ````


