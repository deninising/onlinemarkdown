<center>
    <h1>
        jvm深入学习
    </h1>
</center>

# 类与类加载器
## 类的加载、连接与初始化
![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200422105244.png)
![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200422105900.png)
![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200422110446.png)
![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200422112058.png)
- **类的生命周期：**
    ![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200423111936.png)
    ![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200422114203.png)
- **类的加载时机：**
    ![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200422113625.png)
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
        * 描述：运行期常量与编译期常量的区别：
        * 【1】运行期常量不会进入常量池，但是编译期常量会存储到常量池中
        * 【2】运行期常量要求定义该常量的类或者接口执行初始化，而编译期只需要执行类或者接口的加载
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
    * 描述：接口加载的规则同类的加载规则相同,
    * 但是接口的初始化规则与类大不相同：
    * 【1】一个类的初始化并不要求必须初始化它实现的接口
    * 【2】一个接口的初始化也不要求必须对它的父接口进行初始化
    *
    * @author Dennis
    * @version 1.0
    * @date 2020/4/22 19:12
    */
    public class ClassLoadTest05 {
        public static void main(String[] args) {
            // 子接口通过多态的方式调用父接口的运行期常量，即使父接口进行了初始化但其子接口也不执行初始化===>对谁进行了主动使用，就对谁进行初始化
            System.out.println(Child05.P_STR);
            // 子接口执行初始化,其父接口不执行初始化
            System.out.println(Child05.C_STR);
            // 对于包含编译期常量的类或接口而言，其他类对该常量的调用，只要求编译期常量所在的类或者接口执行加载，不要求执行初始化，且常量会存储在调用该常量的方法所在的类的常量池当中
            // 编译期常量与运行期常量的区别：运行期常量需要类或者接口的运行期环境，因此需要对应的类或者接口执行初始化，而编译期常量只需要对应的类或者接口执行类加载即可
            System.out.println(Child05.C_STRING) ;
            System.out.println(Child05.P_STRING) ;
            System.out.println(Parent05.P_STRING) ;
        }
    }

    interface Parent05 {
        // 接口中无法定义static{}代码块，通过一下方式模拟类的初始化代码块，初始化阶段一下输出必执行
        public static final Thread thread = new Thread() {
            {
                System.out.println("interface Parent05 has been initialized");
            }
        };
        public static String P_STRING = "hello parent interface";
        public static final String P_STR = UUID.randomUUID().toString();
    }

    interface Child05 extends Parent05 {
        // 接口中无法定义static{}代码块，通过一下方式模拟类的初始化代码块，初始化阶段一下输出必执行
        public static final Thread thread = new Thread() {
            {
                System.out.println("interface Child05 has been initialized");
            }
        };
        public static String C_STRING = "hello child interface";
        public static String C_STR = UUID.randomUUID().toString();
    }



     ````
- **类加载过程综合练习案例：对于理解类加载顺序和类初始化顺序非常重要**
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
- **反射是对类的一种主动使用：**
    ````java
        package com.dennis.classloading;
        /**
        * 描述： 反射是对类的一种主动使用，而ClassLoader.load("xxx.xxx.xxx")只是对类的.class文件的一种加载而非主动使用，所以不会初始化
        *
        * @author Dennis
        * @version 1.0
        * @date 2020/4/23 17:51
        */
        public class ClassLoadTest07 {
            public static void main(String[] args) throws ClassNotFoundException {
                ClassLoader systemClassLoader = ClassLoader.getSystemClassLoader();
                // 只是显示的加载类的.class文件到虚拟,并不是对类的主动使用,因此类不会初始化
                Class<?> myClazz = systemClassLoader.loadClass("com.dennis.classloading.MyClass");
                System.out.println(myClazz);
                System.out.println("===================");
                // 反射获取类的class object,是对类的主动使用,会使类进行初始化
                Class<?> aClass = Class.forName("com.dennis.classloading.MyClass");
                System.out.println(aClass);
            }
        }

        class MyClass {
            static {
                System.out.println("MyClass has been initialized");
            }
        }
    ````
- **总结架构图：**
![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200423112622.png)

## 类加载器及加载机制
![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200423163341.png)
![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200423163718.png)
![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200423162529.png)
![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200423164100.png)

- **查看一个类被加载时，所用到的具体类加载器：**
    ````java
        package com.dennis.classloader;
    /**
    * 描述：查看一个类被加载时，所用到的具体类加载器
    *
    * @author Dennis
    * @version 1.0
    * @date 2020/4/23 16:47
    */
    public class ClassLoaderTest01 {
        public static void main(String[] args) throws ClassNotFoundException {
            Class<?> stringClazz = Class.forName("java.lang.String");
            System.out.println(stringClazz.getClassLoader());       // 采用根加载器加载String类

            Class<?> cClazz = Class.forName("com.dennis.classloader.C");
            System.out.println(cClazz.getClassLoader());            //采用AppClassLoader加载C类
        }
    }
    class C {

    }
    ````
- **通过当前类加载器应获取所有父加载器：**
    ````java
        package com.dennis.classloader;
        /**
        * 描述： 通过当前类加载器应获取所有父加载器
        * sun.misc.Launcher$AppClassLoader@18b4aac2 =====> 系统加载器
        * sun.misc.Launcher$ExtClassLoader@4554617c =====> 扩展加载器
        * null                                      =====> 根加载器
        * @author Dennis
        * @version 1.0
        * @date 2020/4/23 20:13
        */
        public class ClassLoaderTest02 {
            /**
            * 通过结果可以看出，当前类的类加载器为AppClassLoader,即：加载该类的类加载器为AppClassLoader
            * 虽然有父加载器ExtClassLoader,甚至祖父加载器BootClassLoader,但是当前类位于classPath下,而
            * ExtClassLoader(LoadJRE\lib\ext\*.jar或者Djava.ext.dirs指定目录下的jar包),
            * BootClassLoader(LoadJRE\lib\rt.jar或则Xbootclasspath选项所指定目录下的的jar包)
            * 均加载不了classPath下的.class文件,所以当前类又交回给AppClassLoader进行类的加载
            */
            public static void main(String[] args) {
                Class<?> clazz = ClassLoaderTest02.class;
                ClassLoader classLoader = clazz.getClassLoader();
                System.out.println(classLoader);
                while (classLoader != null ) {
                    classLoader = classLoader.getParent();
                    System.out.println(classLoader);
                }
            }
        }
    ````
    - **该图可很好总结上例中的文字描述**
    ![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200423223258.png)

- **通过资源名称resourceName获取到.class文件所在位置url：**
    ````java
        package com.dennis.classloader;
        import java.io.IOException;
        import java.net.URL;
        import java.util.Enumeration;
        /**
        * 描述：通过资源名称resourceName获取到.class文件所在位置url
        * @author   Dennis
        * @date     2020/4/23 21:04
        * @version  1.0
        */
        public class ClassLoaderTest03 {
            public static void main(String[] args) throws IOException {
                // 通过调用类加载器加载.class文件和资源文件的线程来获取上下文类加载器
                ClassLoader contextClassLoader = Thread.currentThread().getContextClassLoader();
                // 资源名称
                String resourceName = "com/dennis/classloader/ClassLoaderTest03.class";
                // 获取定位资源文件的URLs
                Enumeration<URL> urls = contextClassLoader.getResources(resourceName);
                while (urls.hasMoreElements()){
                    System.out.println(urls.nextElement()); //file:/D:/Code/github/jvm_learn/target/classes/com/dennis/classloader/ClassLoaderTest03.class
                }
            }
        }
    ````
- 自定义类加载器：
    ````java
        package com.dennis.classloader;
        import java.io.ByteArrayOutputStream;
        import java.io.File;
        import java.io.FileInputStream;
        import java.io.IOException;

        /**
        * 描述： 自定义类加载器
        * <p>
        * 重要概念：定义类加载器：是指真真正正上将当前类加载到了当前jvm虚拟机中的那个加载器（定义类加载器只能有一个）
        * 初始类加载器：是指原理上能够将当前类加载到jvm虚拟机中的所有加载器（可以有多个）
        *
        * @author Dennis
        * @version 1.0
        * @date 2020/4/24 17:27
        */
        public class ClassLoaderTest04 extends ClassLoader {
            private String classLoaderName;
            private final String fileExtension = ".class";

            public ClassLoaderTest04(String classLoaderName) {
                super(); // 默认parentClassLoader为systemClassLoader
                this.classLoaderName = classLoaderName;
            }

            public ClassLoaderTest04(ClassLoader parent, String classLoaderName) {
                super(parent); //显示指定父类加载器
                this.classLoaderName = classLoaderName;
            }

            @Override
            public String toString() {
                return "[" + this.classLoaderName + "]";
            }

            @Override
            protected Class<?> findClass(String name) throws ClassNotFoundException {
                byte[] bytes = this.loadClassData(name);
                return this.defineClass(name, bytes, 0, bytes.length);
            }

            // 将.class文件加载成byte[]资源
            private byte[] loadClassData(String resourceName) {
                System.out.println("method loadClassData defined in ClassLoaderTest04 has been invoked");
                FileInputStream fis = null;
                ByteArrayOutputStream baos = null;
                byte[] resultData = null;
                int data;
                resourceName = resourceName.replace(".", "/");
                try {
                    fis = new FileInputStream(new File(resourceName + fileExtension));
                    baos = new ByteArrayOutputStream();
                    while ((data = fis.read()) != -1) {
                        baos.write(data);
                    }
                    resultData = baos.toByteArray();
                } catch (Exception e) {
                    e.printStackTrace();
                } finally {
                    if (fis != null && baos != null) {
                        try {
                            fis.close();
                            baos.close();
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                    }
                }
                return resultData;
            }
            public static void main(String[] args) throws ClassNotFoundException, IllegalAccessException, InstantiationException {
                ClassLoaderTest04 myClassLoader = new ClassLoaderTest04("ClassLoaderTest04");
                Class<?> clazz = myClassLoader.loadClass("com.dennis.classloader.ClassLoaderTest01");
                ClassLoader classLoader = clazz.getClassLoader(); // 由定义类加载加载该类，本例中用的时appClassLoader(双亲委托机制决定的)
                System.out.println(classLoader); // AppClassLoader
                System.out.println(clazz);
                Object instance = clazz.newInstance();
                if (instance instanceof ClassLoaderTest01) {
                    System.out.println(instance);
                } else {
                    System.out.println("类型不匹配");
                }
            }

        }
    ````
    ![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200425144323.png)
    
- 自定义类加载器改进版及命名空间对类加载的影响
    - **案例一**
        ````java
            package com.dennis.classloader;

            import java.io.ByteArrayOutputStream;
            import java.io.File;
            import java.io.FileInputStream;
            import java.io.IOException;

            /**
            * 描述：对ClassLoaderTest04的改进,新定义一个类加载器加载指定目录
            * （classPath:folder/classes/com/dennis/classloader/）下
            * 的.class文件(当父加载器无法通过binaryName成功加载指定类文件时)
            *
            * @author Dennis
            * @version 1.0
            * @date 2020/4/25 11:06
            */
            public class ClassLoaderTest05 extends ClassLoader {
                private static final String CLAZZ_PATH = "folder/classes/";
                private static final String FILE_EXTENSION = ".class";
                private String classLoaderName = null;

                public ClassLoaderTest05(String classLoaderName) {
                    super();
                    this.classLoaderName = classLoaderName;
                }

                public ClassLoaderTest05(ClassLoader parent, String classLoaderName) {
                    super(parent);
                    this.classLoaderName = classLoaderName;
                }

                // 从规定好了目录中加载指定的.class文件
                private byte[] loadClassData(String binaryName) {
                    FileInputStream fis = null;
                    ByteArrayOutputStream baos = null;
                    byte[] resultData = null;
                    int data;

                    binaryName = binaryName.replace(".", "/");
                    File file = new File(CLAZZ_PATH + binaryName + FILE_EXTENSION);

                    try {
                        baos = new ByteArrayOutputStream();
                        fis = new FileInputStream(file);
                        while ((data = fis.read()) != -1) {
                            baos.write(data);
                        }
                        resultData = baos.toByteArray();
                    } catch (Exception e) {
                        e.printStackTrace();
                    } finally {
                        if (fis != null) {
                            try {
                                fis.close();
                                baos.close();
                            } catch (IOException e) {
                                e.printStackTrace();
                            }
                        }
                    }
                    return resultData;
                }

                //重写父类的findClass方法，当父类加载器在原有的类文件加载路径中无法加载到.class文件时，便会调用自定义的类加载器中的findClass方法加载类文件
                @Override
                protected Class<?> findClass(String name) throws ClassNotFoundException {
                    System.out.println("method findClass() override in ClassLoadTest05 is invoked !!!");
                    byte[] data = loadClassData(name);
                    return defineClass(name, data, 0, data.length);
                }

                // 测试：【1】将classpath下的.class文件剪切到自定义类指定的.class存放目录下
                public static void main(String[] args) throws ClassNotFoundException, IllegalAccessException, InstantiationException {

                    System.out.println("==========================用自定义类加载器实例加载器指定类文件===========================");
                    ClassLoaderTest05 loader01 = new ClassLoaderTest05("loader01");
                    Class<?> clazz01 = loader01.loadClass("com.dennis.classloader.ClassLoaderTest01");
                    System.out.println(clazz01 + ":" + clazz01.hashCode()); // 被加载类的Class 对象：class com.dennis.classloader.ClassLoaderTest01:356573597
                    System.out.println(clazz01.getClassLoader()); // 获取定义类加载器ClassLoader 对象：com.dennis.classloader.ClassLoaderTest05@4554617c

                    System.out.println("==========================用同一个类加载器实例对同一个类进行第二次加载===========================");
                    Class<?> clazz02 = loader01.loadClass("com.dennis.classloader.ClassLoaderTest01");
                    System.out.println(clazz02 + ":" + clazz02.hashCode()); // 被加载类的Class 对象：class com.dennis.classloader.ClassLoaderTest01:356573597
                    System.out.println(clazz02.getClassLoader());// 获取定义类加载器ClassLoader 对象：com.dennis.classloader.ClassLoaderTest05@4554617c

                    System.out.println("==========================用另一个类加载器实例对同一个类进行第二次加载===========================");
                    ClassLoaderTest05 loader02 = new ClassLoaderTest05("loader02");
                    Class<?> clazz03 = loader02.loadClass("com.dennis.classloader.ClassLoaderTest01");
                    System.out.println(clazz03 + ":" + clazz03.hashCode()); // 被加载类的Class 对象：class com.dennis.classloader.ClassLoaderTest01:2133927002
                    System.out.println(clazz03.getClassLoader());// 获取定义类加载器ClassLoader 对象：com.dennis.classloader.ClassLoaderTest05@677327b6

                    System.out.println("==========================将load01做为loader03的父加载器且同一个类进行第二次加载===========================");
                    ClassLoaderTest05 loader03 = new ClassLoaderTest05(loader01, "loader03");
                    Class<?> loader04 = loader03.loadClass("com.dennis.classloader.ClassLoaderTest01");
                    System.out.println(loader04 + ":" + loader04.hashCode()); // 被加载类的Class 对象：class com.dennis.classloader.ClassLoaderTest01:356573597
                    System.out.println(loader04.getClassLoader());// 获取定义类加载器ClassLoader 对象：com.dennis.classloader.ClassLoaderTest05@4554617c

                    //结论：
                    // 在同一个命名空间中：同一个类文件只能加载一次
                    // 在不同的命名空间中：同一个类文件可被加载多次
                    // 类加载器的命名空间由所有父类加载器实例和当前类加载实例所确定
                }

            }
        ````
        - **输出结果：**
        ````text
            ==========================用自定义类加载器实例加载器指定类文件===========================
            method findClass() override in ClassLoadTest05 is invoked !!!
            class com.dennis.classloader.ClassLoaderTest01:356573597
            com.dennis.classloader.ClassLoaderTest05@4554617c
            ==========================用同一个类加载器实例对同一个类进行第二次加载===========================
            class com.dennis.classloader.ClassLoaderTest01:356573597
            com.dennis.classloader.ClassLoaderTest05@4554617c
            ==========================用另一个类加载器实例对同一个类进行第二次加载===========================
            method findClass() override in ClassLoadTest05 is invoked !!!
            class com.dennis.classloader.ClassLoaderTest01:2133927002
            com.dennis.classloader.ClassLoaderTest05@677327b6
            ==========================将load01做为loader03的父加载器且同一个类进行第二次加载===========================
            class com.dennis.classloader.ClassLoaderTest01:356573597
            com.dennis.classloader.ClassLoaderTest05@4554617c
        ````
        - **结论：**
            - 在同一个命名空间中：同一个类文件只能加载一次
            - 在不同的命名空间中：同一个类文件可被加载多次
            - **类加载器的命名空间由所有父类加载器实例和当前类加载实例所唯一确定。某个类在jvm中只能被加载一次，大前题是在同一个类加载的命名空间中**

    - **案例二**
        ````java
            package com.dennis.classloader;
            // Cat类中访问Dog类的class对象
            public class Cat {
                public Cat() {
                    System.out.println("Cat is loaded by:" + this.getClass().getClassLoader());
                    System.out.println("from class Cat:" + Dog.class + "==>classLoader:" + Dog.class.getClassLoader());
                }
            }

            package com.dennis.classloader;
            // Dog类中访问Cat类的class对象
            public class Dog {
                public Dog() {
                    System.out.println("Dog is loaded by:" + this.getClass().getClassLoader());
                    System.out.println("from class Dog:" + Cat.class + "==>classLoader:" + Cat.class.getClassLoader());
                }
            }

            package com.dennis.classloader;
            /**
            * 描述：
            * 测试：将Cat.class置于子加载器的指定的.class文件目录下，使用自定义类加载器加载Cat类,将Dog.class置于classpath下将有appClassLoader加载
            *
            * @author Dennis
            * @version 1.0
            * @date 2020/4/25 21:26
            */
            public class ClassLoaderTest07 {
                public static void main(String[] args) throws Exception {
                    ClassLoaderTest05 loader = new ClassLoaderTest05("loader");
                    Class<?> clazzCat = loader.loadClass("com.dennis.classloader.Cat");
                    clazzCat.newInstance();// 主动使用cat类,其构造器中会去访问Dog类的Class对象

                    System.out.println("========================");
                    Class<?> clazzDog = loader.loadClass("com.dennis.classloader.Dog");
                    clazzDog.newInstance(); // 主动使用Dog类,其构造器中会去访问Cat类的Class对象
                }
            }
        ````
        - **输出结果：**
        ````text
            method findClass() override in ClassLoadTest05 is invoked !!!
            Cat is loaded by:com.dennis.classloader.ClassLoaderTest05@74a14482
            from class Cat:class com.dennis.classloader.Dog==>classLoader:sun.misc.Launcher$AppClassLoader@18b4aac2
            ========================
            Dog is loaded by:sun.misc.Launcher$AppClassLoader@18b4aac2
            Exception in thread "main" java.lang.NoClassDefFoundError: com/dennis/classloader/Cat
            at com.dennis.classloader.Dog.<init>(Dog.java:7)
            at sun.reflect.NativeConstructorAccessorImpl.newInstance0(
            Native Method)
            at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
            at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
            at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
            at java.lang.Class.newInstance(Class.java:442)
            at com.dennis.classloader.ClassLoaderTest07.main(ClassLoaderTest07.java:25)
            Caused by: java.lang.ClassNotFoundException: com.dennis.classloader.Cat
            at java.net.URLClassLoader.findClass(URLClassLoader.java:382)
            at java.lang.ClassLoader.loadClass(ClassLoader.java:418)
            at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:355)
            at java.lang.ClassLoader.loadClass(ClassLoader.java:351)
                    ... 7 more
        ````
        - **结论：**
            -  子加载器所加载的类能够访问父加载器所加载的类
            -  父加载器所加载的类访问不了子加载器所加载的类
    ![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200428211256.png)
    - **案例三**
        ````java
            package com.dennis.classloader;

            public class Pig {
                private Pig pig;

                public void setPig(Object object) {
                    this.pig = (Pig) object;
                    System.out.println("method in Class pig is invoked successfully");
                }
            }



            package com.dennis.classloader;

            import java.lang.reflect.Method;

            /**
            * 描述：命名空间和双亲委托机制的作用及意义
            * 【1】确保java核心库的类型安全：所有java应用都至少会引用java.lang.Object类，运行期间，这个类将被加载到虚拟机的内存中去。如果这个类被自定义的
            * 类加载器所加载了，那么很可能在java虚拟机中存在多个版本的java.lang.Object类，且这些类是互相不兼容的，互相不可见的(类加载器的命名空间使然)
            * 因此，加载类加载器的双亲委托机制的作用下，java核心类库中的类的加载都是由启动类加载器同一完成的，从而确保了java应用程序所使用的都是同一个版本
            * 的Java核心类库，他们之间相互兼容，从而保证java核心库的类型安全
            * 【2】可以确保java核心类库的类不被自定义的类所代替
            * 【3】互为平级的类加载器或是不存在包含等级的类加载器，是可以为相同binaryName的类创建各自的命名空间，如此以来在jvm中这些类之间互相不可访问的（即使
            * 他们的binaryName相同看似为同一个类）互不兼容的。到这类技术可在多框架中得到很好的实际应用
            *
            * @author Dennis
            * @version 1.0
            * @date 2020/4/26 15:25
            */
            public class ClassLoaderTest08 {
                public static void main(String[] args) throws Exception {
                    ClassLoaderTest05 myLoader01 = new ClassLoaderTest05("myLoader01");
                    ClassLoaderTest05 myLoader02 = new ClassLoaderTest05("myLoader02");

                    // 当使用自定义类加载加载时,以下两个类对象将位于两个不同的命名空间中,
                    // 若通过委托机制被appClassLoader加载时,则位于同一个命名空间中,
                    // 若为于同一命名空间中,相同binaryName的类只会被加载一次,即pigClazz01 = pigClazz02
                    Class<?> pigClazz01 = myLoader01.loadClass("com.dennis.classloader.Pig");
                    Class<?> pigClazz02 = myLoader02.loadClass("com.dennis.classloader.Pig");

                    System.out.println(pigClazz01 == pigClazz02);

                    Object pig01 = pigClazz01.newInstance(); // 用于调用方法
                    Object pig02 = pigClazz02.newInstance(); // 用于传参

                    Method setPig = pigClazz01.getMethod("setPig", Object.class);
                    setPig.invoke(pig01, pig02);// 调用pig01的setPig方法,传递的参数为pig02
                }
            }
        ````
        - **输出结果：**
        ````text
            method findClass() override in ClassLoadTest05 is invoked !!!
            method findClass() override in ClassLoadTest05 is invoked !!!
            false
            Exception in thread "main" java.lang.reflect.InvocationTargetException
                at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
                at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
                at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
                at java.lang.reflect.Method.invoke(Method.java:498)
                at com.dennis.classloader.ClassLoaderTest08.main(ClassLoaderTest08.java:29)
            Caused by: java.lang.ClassCastException: com.dennis.classloader.Pig cannot be cast to com.dennis.classloader.Pig
                at com.dennis.classloader.Pig.setPig(Pig.java:7)
                ... 5 more
        ````
        - **结论：**
        - 不同命名空间中可以存在binaryName相同的类，但他们之间互不兼容，互不可见
## 启动类加载器与其他类加载器的区别
- **案例**
    ````java
        package com.dennis.classloader;
        /**
        * 描述：跟加载器与其它类加载器的区别
        *
        * @author Dennis
        * @version 1.0
        * @date 2020/4/26 20:45
        */
        public class ClassLoaderTest09 {
            public static void main(String[] args) throws Exception {
                ClassLoaderTest05 myClassLoader = new ClassLoaderTest05("myClassLoader");
                Class<?> test01 = myClassLoader.loadClass("com.dennis.classloader.ClassLoaderTest01");

                ClassLoader loader01 = test01.getClassLoader();
                System.out.println(loader01); // com.dennis.classloader.ClassLoaderTest05@74a14482
                ClassLoader loader02 = loader01.getClass().getClassLoader();
                System.out.println(loader02); //sun.misc.Launcher$AppClassLoader@18b4aac2
                ClassLoader loader03 = loader02.getClass().getClassLoader();
                System.out.println(loader03); //null ==> 跟加载器
            }
        }
    ````
    - **结论**
        - AppClassLoader.class是由根加载器所加载的
        - 內建于JVM中的启动类加载器会加载java.lang.ClassLoader以及其他的Java平台类，当JVM启动时，一块特殊的机器码会运行，它会加载扩展类加载器与系统类加载器， 这块特殊的机器码叫做启动类加载器(Bootstrap)。**启动加载器内嵌于jvm中，由C语言编写，与虚拟机平台相关**
        - 启动类加载器并不是Java类，而其他的加载器则都是Java类，启动类加载器是特定于平台的机器指令（特殊的机器码），它负责开启整个加载过程。
        - 所有类加载器(除了启动类加载器)都被实现为Java类。不过，总归要有一个组件来加载第一 个Java类加载器,从而让整个加载过程能够顺利执行进行下去，加载第一个纯Java类加载器就是启动类加载器的职责。启动类加载器还会负责加载供JRE正常运行所需要的基本组件，这包括java . util与java. lang包中的类等等。

## 扩展类加载器
-  **扩展类加载器只能加载对应路径下的jar包中的二进制资源，不能直接加载.class文件资源**

## 系统类加载器可被替换（java.system.class.loader）
- 可通过启动类参数中java.system.class.loader属性来指定自定义系统类加载器
## 阶段性重点源码阅读推荐（直接见源码）
- **重要类**
    - ClassLoader
    - Launcher
    - Class
    - SystemClassLoaderAction
- **重要方法,按阅读顺序**
    - ClassLoader.getSystemClassLoader();
    - Launcher.getLauncher();
    - Class.forName();
    - Thread.getContextClassLoader();
    - SystemClassLoaderAction.run()
## 线程上下文加载器
- 问题：由jdbc标准引出
    - jdbc使用时的伪代码如下：
    ````java
        Class.forName("com.mysql.jdbc.Driver");
        Connection conn = DriverManager.getConnection();
        Statement st = conn.getStatament();
    ````
    - 问题分析：
    ````text
    1、JDBC是java中一个为了与数据库进行连接而设计的一套标准，其并没有具体实现，所有的类都位于rt.jar中，因此这些未实现的顶层接口将被启动类加载器所加载
    2、JDBC的具体实现是由数据库第三方厂商提供的，具体实现的类的jar包在使用时会置于classpath下，以此这些具体实现类将会被appClassLoader进行加载
    3、由于jvm双亲委托机制的存在，使得子加载器所加载的类不能被父加载器所加载的类访问
    4、问题：JDBC标准库中类如何去访问厂商所提供的具体实现类
    ````
- 解决方法：
    1、jvm中引入了线程上下文加载器这一概念:
        Thread.concurrentThread.getContextClassLoader()
        Thread.concurrentThread.setContextClassLoader(ClassLoader ccl)
    2、线程上下文加载器的作用就是使得当前线程中的类（假设由启动类加载器加载），能够访问到上下文加载器(假设被设置为AppClassLoader)所加载的类
    3、通过以上两点使得SPI的开发成为了可能，同时在必要的时候打破了jvm的双亲委托机制，使得java生态圈中许多优秀框架的开发得以进行
- 重要知识点：
    - 线程上下文类加载器是从JDK1.2开始引入的，类Thread中的getContextClassLoader()和setContextClassLoader(ClassLoader cl)分别用来获取和设置上下文类加载器。如果没有通过setContextClassLoader (ClassLoader cl )进行设置的话，线程将继承其父线程的上下文类加载器。
    - Java应用运行时的初始线程(main()方法所在的线程)的上下文类加载器是系统类加载器。在线程中运行的代码可以通过该类加载器来加载类与资源。
![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200427161120.png)

- 结合java.util.ServiceLoader 与 mysql.Driver 的实际案例，分析contextClassLoader在实际场景中的应用和重要性
    ````java
        package com.dennis.classloader;
        import java.sql.Driver;
        import java.util.Iterator;
        import java.util.ServiceLoader;

        /**
        * 描述： 结合java.util.ServiceLoader 与 mysql.Driver 的实际案,例分析contextClassLoader在实际场景中的应用和重要性
        *
        * @author Dennis
        * @version 1.0
        * @date 2020/4/28 9:37
        */
        public class ClassLoaderTest10 {
            public static void main(String[] args) {
                // 【1】Driver.class只是JDBC标准中的一个接口,那么ServiceLoader是怎么寻找并且加载到com.mysql.jdbc.Driver.class的？
                // 【2】加载ServiceLoader类的加载器是启动类加载器,因此它所依赖的类Driver.class也将由启动类加载器,而具体实现
                // 类com.mysql.jdbc.Driver.class是由AppClassLoader所加载的,那么标准中的Driver与具体实现的Driver是如何达到兼容的？
                ServiceLoader<Driver> loader = ServiceLoader.load(Driver.class);
                Iterator<Driver> iterator = loader.iterator();

                while (iterator.hasNext()) {
                    Driver driver = iterator.next();
                    System.out.println("driver:" + driver.getClass() + ",classLoader:" + driver.getClass().getClassLoader());
                }

                System.out.println("当前线程上下文类加载器：" + Thread.currentThread().getContextClassLoader());
                System.out.println("ServiceLoader的类加载器：" + ServiceLoader.class.getClassLoader());
            }
        }
    ````
    - **源码流程分析**
![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200428105008.png)
![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200428105039.png)
    - **总结：**
        - 加载方面：通过标准中的规定->将具体实现类的binaryName放于规定的文件中->便于通过JDBC中接口的binaryNamey和PREFIX去定位和读取->为类的加载提供条件
        - 访问方面：通过指定线程上下文加载器去加载具体实现类，打破了双亲委托机制，使得即便顶层接口是由启动类加载器加载，而具体实现是由AppClassLoader所加载，启动类也可以访问到具体实现类（只要将加载顶层接口时的线程的线程上下文加载器设置为AppClassLoader即可，或则去获取到加载顶层接口时的线程的线程上下文加载器并指定它来加载具体实现类（本案例中用的后一种方式）
- 案例分析：分析以下代码执行时的源码执行流程
    ````text
    Class.forName("com.mysql.com.mysql.jdbc.Driver")
    Connection connection = DriverManager.getConnection("jdbc:mysql://192.168.1.57:3306/test", "root", "1234");
    ````
    ````java
        package com.dennis.classloader;

        import com.mysql.jdbc.Driver;

        import java.sql.Connection;
        import java.sql.DriverManager;
        import java.sql.SQLException;

        /**
        * 描述：使用mysql驱动类时深度剖析
        *
        * @author Dennis
        * @version 1.0
        * @date 2020/4/28 11:48
        */
        public class ClassLoaderTest11 {
            public static void main(String[] args) throws ClassNotFoundException, SQLException {
                // 加载子类的时候会自动加载父类
                ClassLoaderTest05 loader = new ClassLoaderTest05("loader");
                loader.loadClass("com.dennis.classloader.Child");

                // 查看当前线程的上下文类加载器
                ClassLoader contextClassLoader = Thread.currentThread().getContextClassLoader();
                System.out.println("当前线程上下文类加载器：" + contextClassLoader);


                // 加载实现类时会去首先加载其接口（若之前没有被加载）,所以加载java.sql.Driver类的线程为当前线程,其线程上下文加载器为AppClassLoader
                Class.forName("com.mysql.jdbc.Driver");
                ClassLoader classLoader = Driver.class.getClassLoader();
                System.out.println("实现类的类加载器：" + classLoader);// 实现类的类加载也为AppClassLoader,因此JDBC与厂商实现类可兼容

                // 类DriverManager的初始化过程中又会采用标准SPI类加载的方式加载一次JDBC中驱动的具体实现类,
                // 因此Class.forName("com.mysql.jdbc.Driver")中重点在于类的初始化时驱动的注册操作 java.sql.DriverManager.registerDriver(new Driver());
                Connection connection = DriverManager.getConnection("jdbc:mysql://192.168.1.57:3306/test", "root", "1234");
            }
        }

        class Child extends Parent {
        }

        class Parent {
        }
    ````


## 类的卸载
![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200425153522.png)
![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200425153629.png)
![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200425153754.png)
- **手动调用gc()卸载一个已加载的类**
    ````java
        package com.dennis.classloader;
        import java.util.concurrent.TimeUnit;
        /**
        * 描述：手动调用gc()方法卸载一个类:-XX：+TraceClassUnloading
        * @author Dennis
        * @version 1.0
        * @date 2020/4/25 19:02
        */
        public class ClassLoaderTest06 {
            public static void main(String[] args) throws ClassNotFoundException, InterruptedException {
                ClassLoaderTest05 loader01 = new ClassLoaderTest05("loader01");
                loader01.loadClass("com.dennis.classloader.ClassLoaderTest01");//【注意】需将.class文件手动剪切到自定义类加载的类加载路径

                // 【1】栈空间变量“loader01"放弃对自定义加载器对象的引用,此时类加载器对象到根路径不可达，
                // 【2】类加载器对象对它所加载的Class对象也有一个引用，此时对于这些Class对象而言到根路径也不可达
                // 因此调用system.gc()时，已加载的Class对象： ClassLoaderTest01 将被卸载
                loader01 = null;
                System.gc();
                TimeUnit.SECONDS.sleep(50);
            }
        }
    ````
- **输出结果**
    ````text
        method findClass() override in ClassLoadTest05 is invoked !!!
        [Unloading class com.dennis.classloader.ClassLoaderTest01 0x00000007c0061028]   
    ````
- **jvisualvm:查看结果**
![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200425194315.png)
- **总结：**
    - 【1】栈空间变量“loader01"放弃对自定义加载器对象的引用,此时类加载器对象到根路径不可
      【2】类加载器对象对它所加载的Class对象也有一个引用，此时对于这些Class对象而言到根路径也不可达
     因此调用system.gc()时，已加载的Class对象： ClassLoaderTest01 将被卸载
    - **由于无法手动放弃对虚拟机自带的三个类加载器（跟加载器，扩展加载器，系统加载器）对象的引用，所以虚拟机自带的类加载所加载的类不会被卸载**
    - **自定义类所加载的类是可以被卸载的**
# JVM字节码
1. 使用javap -verbose命令 分析-个字节码文件时， 将会分析该字节码文件的魔数、版本号、常量池、类信息、类的构造方法、类中的方法信息、类变量与成员变量等信息。
2. 魔数:所有的.class字节码文件的前4个字节都是魔数，魔数值为固定值: 0xCAFEBABE 。
3. 魔数之后的4个字节为版本信息，前两个字节表示minor version (次版本号)，后两个字节表示major version (主版本号)。这里的版本号为00 00 00 34, 换算成十进制，表示次版本号为0，主版本号为52。所以，该文件的版本号为: 1.8.0。 可以通过java -vers ion命令来验证这一点。 
4. 常量池(constant pool) :紧接着主版本号之后的就是常池入口。 一个Java类中定义的很 多信息都是由常量池来维护和描述的，可以将常量池看作是Class文件的资源仓库，比如说Java类中定义的方法与变量信息，都是存储在常量池中。常量池中主要存储两类常量:字面量与符号引用。字面量如文本字符串，Java中声明为final的常量值等，而符号引用如类和接口的全局限定名，字段的名称和描述符，方法的名称和描述符等。
5. 常量池的总体结构: Java类所对应的常量池主要由常量池数量与常量池数组(常量表)这两部分共同构成。常量池数量紧跟在主版本号后面，占据2个字节;常量池数组则紧跟在常量池数量之后。常量池数组与-般的数组不同的是，常量池数组中不同的元素的类型、结构都是不同的，长度当然也就不同;但是，每种元素的第一 个数据都是一 个u1类型，该字节是个标志位，占据1个字节。JVM在解析常量池时，会根据这个u1类型来获取元素的具体类型。值得注意的是，常量池数组中元素的个数=常量池数- 1 (其中0暂时不使用)，目的是满足某些常量池索引值的数据在特定情况下需要表达「不引用任何一个常量池J的含义;根本原因在于，索引为0也是一个常量 (保留常量)，只不过它不位于常量表中，这个常量就对应null值;所以，常量池的索引从1而非0开始。
6. 在JVM规范中，每个变量/字段都有描述信息，描述信息主要的作用是描述字段的数据类型、方法的参数列表(包括数量、类型与顺序)与返回值。根据描述符规则，基本数据类型和代表无返回值的void类型都用一个大写字符来表示，对象类型则使用字符加对象的全限定名称来表示。为了压缩字节码文件的体积，对于基本数据类型，JVM都只使用个大写字母来表示，如下所示:B-byte,C-char,D-double,F-float,I-int,J-long,S-short,z-boolean,V-void,L-对象类型，如Ljava/lang/String;
7. 对于数组类型来说，每一个维度使用个前置的[来表示， 如int[ ]被记录为[I, String[ ][ ]被记录为[[Ijava/lang/String;
8. 用描述符描述方法时，按照先参数列表，后返回值的顺序来描述。参数列表按照参数的严格顺序放在一组()之内， 如方法: String  getRealnamebyIdAndNickname(int id, String name )的描述符为: (I， Ljava/lang/String;) Ljava/lang/String
## 常量池字节码
![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200428163637.png)
## 类的字节码整体结构
![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200429104530.png)
![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200429104641.png)
![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200429104712.png)
## 访问标识符字节码
![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200429140530.png)
## 成员变量字节码
![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200429140708.png)
![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200429140734.png)
## 成员方法字节码
![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200429140758.png)
![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200429140840.png)
![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200429140909.png)
## 方法属性Code结构
![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200429123711.png)
![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200429123742.png)
![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200429123321.png)
![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200430101209.png)
## 属性字节码
![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200429172137.png)
## 从字节码层面分析异常执行流程
- **Java字节码对于异常的处理方式:**
    1. 统一采用异常表的方式来对异常进行处理。
    2. 在jdk 1.4.2之前的版本中，并不是使用异常表的方式来对异常进行处理的，而是采用特定的指令方式。
    3. 当异常处理存在finally语句块时， 现代化的JVM采取的处理方式是将finally语句块的字节码拼接到每一个catch块后面和try块后面,换句话说，程序中存在多少个catch块和try块，就会在对应块后面重复多少个finally语句块的字节码。
- **异常表**
![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200430232110.png)
- **案例一**
![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200430120123.png)
- **案例二**
![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200430122239.png)
## 从静态分派和动态分派理解方法重写和重载的区别
- **栈帧(stack frame)**
    栈帧是一种用于帮助虚拟机执行方法调用与方法执行的数据结构。
    栈帧本身是一种数据结构，封装了方法的局部变表动态链接信息、方法的返回地址以及操作数栈等信息。

- **符号引用，直接引用**
    有些符号引用是在类加载阶段或是第一次使用时就会转换为直接引用， 这种转换叫做静态解析;另外-些符号引用则是在每次运行期
    转换为直接引用，这种转换叫做动态链接，这体现为Java的多态性。

- **java方法调用的特调**
    方法调用并不等于方法执行，方法调用阶段唯一的任务就是确定被调用方法的版本（即调用哪一个方法），暂时还不涉及方法内部的具体运行过程。

    在程序运行时，进行方法调用是最普遍、最频繁的操作，但是Class文件的编译过程不包括传统编译中的连接步骤，一切方法调用在Class文件里面存储的都只是符号引用，而不是方法在实际运行时内存布局中的入口地址（相对于之前说的直接引用）。这个特性给Java带来了更强大的动态扩展能力，但也使得Java方法调用过程变得相对复杂起来，需要在类加载期间，甚至到运行期间才能确定目标方法的直接引用。

- **解析**
    所有方法调用中的目标方法在Class文件里面都是一个常量池中的引用，在类加载的解析阶段，会将其中的一部分符号引用转化为直接引用。这种解析能成立的前提是：方法在程序真正执行之前就有一个可确定的调用版本，并且这个方法的调用版本在运行期是不可改变的。换句话说，调用目标在程序代码写好、编译器进行编译时就必须确定下来，这类方法的调用称为解析。

    在Java语言中符合“编译期可知，运行期不可变”这个要求的方法，主要包括静态方法和私有方法两大类，前者与类型直接关联，后者在外部不可被访问，这两种方法各自的特点决定了他们不可能通过继承或别的方式重写其他版本，因此他们适合在类加载阶段进行解析。

    静态方法、私有方法、实例构造器、父类方法。这些方法称为非虚方法，它们在类加载的时候就会把符号引用解析为该方法的直接引用。与之相反，其他方法称为虚方法（除去final方法）。


- **静态分派**
    ````java
        package com.dennis.bytecode;

        /**
        * 描述：静态分派（重载方法的调用，变量的声明，编译期即可确定）
        *
        * @author Dennis
        * @version 1.0
        * @date 2020/4/30 21:07
        */
        public class BycodeTest04 {
            public void getAnimalName(Animal animal) {
                System.out.println("please choose a specific animal ");
            }

            public void getAnimalName(Cat cat) {
                System.out.println("name:" + cat.name);
            }

            public void getAnimalName(Dog dog) {
                System.out.println("name:" + dog.name);
            }

            public void getAnimalName(Tiger tiger) {
                System.out.println("name:" + tiger.name);
            }

            public static void main(String[] args) {

                //【1】我们把“Animal”称为变量的静态类型，后面的“Cat”称为变量的实际类型，静态类型和实际类型在程序中都可以发生一些变化，
                // 区别是静态类型的变化仅仅在编码使用时发生，变量本身的静态类型不会被改变，并且最终的静态类型在编译器可知；而实际类型变化的
                // 结果在运行期才确定，编译器在编译期并不知道一个对象的实际类型是什么。
                //【2】变量的声明为一种静态分派，编译期就已确定，变量cat dog tiger均为Animal静态类型而非实际运行时期的实际类型
                Animal cat = new Cat();
                Animal dog = new Dog();
                Animal tiger = new Tiger();

                // 【3】方法的重载，编译器在重载时是通过参数的静态类型而不是实际类型作为判定的依据，本质上方法的重载就是更改参数列表声
                // 明时的类型或则个数，这些因素均在编译期即可确定，因此方法的重载也是静态分派行为；
                // 所有以下调用均调用的是同一个版本的方法：getAnimalName(Animal animal)
                BycodeTest04 bycodeTest04 = new BycodeTest04();
                bycodeTest04.getAnimalName(cat);
                bycodeTest04.getAnimalName(dog);
                bycodeTest04.getAnimalName(tiger);

                // 类型强转，以更改调用方法版本
                bycodeTest04.getAnimalName((Cat) cat);
            }
        }

        class Animal {
        }

        class Cat extends Animal {
            public String name = "cat";
        }

        class Dog extends Animal {
            public String name = "dog";
        }

        class Tiger extends Animal {
            public String name = "tiger";
        }
    ````
- **动态分派**
![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200430230529.png)
## 从字节码分析关键字synchronized
## 从字节码和操作数栈分析计算的执行过程
**现代JVM在执行Java代码的时候，通常都会将解释执行与编译执行二者结合起来进行。**

- 所谓解释执行，就是通过解释器来读取字节码，映射为对应的本地机器码并执行->边解释边执行。
- 所谓编译执行，就是通过即时编译器(Just In Time, JIT) 将字节码全部转换为本地机器码后再一同执行->现代JVM会根据代码热点来生成相应的本地机器码。

**基于栈的指令集与基于寄存器的指令集之间的关系:**

1. JVM执行指令时所采取的方式是基于栈的指令集。

2. 基于栈的指令集主要的操作有入栈与出栈两种。

3. 基于栈的指令集的优势在于它可以在不同平台之间移植，而基于寄存器的指令集是与硬件架构紧密关联的，无法做到可移植。

4. 基于栈的指令集的缺点在于完成相同的操作，指令数量通常要比基于寄存器的指令集数量要多;基于栈的指令集是在内存中完成操作的,而基于寄存器的指令集是直接由CPU来执行的，它是在高速缓冲区中进行执行的，速度要快很多。虽然虚拟机可以采用一些优化手段，但总体来说，基于栈的指令集的执行速度要慢一些。

![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200501223219.png)

## 通过字节码分析++i 与 i++的区别
- **i++**
![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200502103241.png)
- **++i**
![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200502103742.png)
## 字节码层面动态代理源码分析

# JMM
## 堆空间内存溢出
- -Xms5m -Xmx5m：堆内存大小设为5M且不可扩容
    ````java
        package com.dennis.memory;
        import java.util.ArrayList;

        /**
        * 描述：OutOfMemoryError
        * -Xms5m -Xmx5m  -XX:+HeapDumpOnOutOfMemoryError
        *
        * @author Dennis
        * @version 1.0
        * @date 2020/5/2 14:30
        */
        public class JmmTest01 {
            public static void main(String[] args) throws InterruptedException {
                ArrayList<JmmTest01> list = new ArrayList<>();
                while (true) {
                    list.add(new JmmTest01());
        //            TimeUnit.MILLISECONDS.sleep(1);
                }
            }
        }
    ````
- 结果
    ````text
        java.lang.OutOfMemoryError: GC overhead limit exceeded
        Dumping heap to java_pid21148.hprof ...
        Heap dump file created [9018416 bytes in 0.052 secs]
        Exception in thread "main" java.lang.OutOfMemoryError: GC overhead limit exceeded
            at com.dennis.memory.JmmTest01.main(JmmTest01.java:18)
    ````


## 元空间内存溢出
- -XX:MaxMetaspaceSize=30m
    ````java
        package com.dennis.memory;

        import net.sf.cglib.proxy.Enhancer;
        import net.sf.cglib.proxy.MethodInterceptor;

        import java.util.concurrent.TimeUnit;

        /**
        * 描述： 采用充满元空间的方法模拟内存溢出，借助ciglib不停地动态生成Class obj，使得元空间内存的使用耗尽，最终导致内存溢出
        * -XX:MaxMetaspaceSize=30m
        *
        * @author Dennis
        * @version 1.0
        * @date 2020/5/2 17:01
        */
        public class JmmTest03 {

            public static void main(String[] args) throws InterruptedException {
                int i = 0;
                for (; ; ) {
                    Enhancer enhancer = new Enhancer();
                    enhancer.setSuperclass(JmmTest03.class);
                    enhancer.setUseCache(false);
                    enhancer.setCallback((MethodInterceptor) (obj, method, args1, proxy) ->
                            proxy.invokeSuper(obj, args1)
                    );

                    TimeUnit.MILLISECONDS.sleep(10);
                    System.out.println("动态class obj:" + (++i));
                    enhancer.create();//Caused by: java.lang.OutOfMemoryError: Metaspace
                }
            }
        }
    ````
    - 结果：
        ````text
            Exception in thread "main" net.sf.cglib.core.CodeGenerationException: java.lang.reflect.InvocationTargetException-->null
                at net.sf.cglib.core.AbstractClassGenerator.generate(AbstractClassGenerator.java:345)
                at net.sf.cglib.proxy.Enhancer.generate(Enhancer.java:492)
                at net.sf.cglib.core.AbstractClassGenerator$ClassLoaderData.get(AbstractClassGenerator.java:114)
                at net.sf.cglib.core.AbstractClassGenerator.create(AbstractClassGenerator.java:291)
                at net.sf.cglib.proxy.Enhancer.createHelper(Enhancer.java:480)
                at net.sf.cglib.proxy.Enhancer.create(Enhancer.java:305)
                at com.dennis.memory.JmmTest03.main(JmmTest03.java:30)
            Caused by: java.lang.reflect.InvocationTargetException
                at sun.reflect.GeneratedMethodAccessor1.invoke(Unknown Source)
                at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
                at java.lang.reflect.Method.invoke(Method.java:498)
                at net.sf.cglib.core.ReflectUtils.defineClass(ReflectUtils.java:459)
                at net.sf.cglib.core.AbstractClassGenerator.generate(AbstractClassGenerator.java:336)
                ... 6 more
            Caused by: java.lang.OutOfMemoryError: Metaspace
                at java.lang.ClassLoader.defineClass1(Native Method)
                at java.lang.ClassLoader.defineClass(ClassLoader.java:756)
                ... 11 more
        ````
        ![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200502173320.png)
- 推荐文章：https://www.infoq.cn/article/Java-PERMGEN-Removed
    ![](https://gitee.com/liao_peng/cloudpic/raw/master/blog/20200502181641.png)
## 栈溢出
- -Xss200k
    ````java
        package com.dennis.memory;

        import java.util.concurrent.TimeUnit;
        /**
        * 描述：虚拟机栈溢出
        * -Xss200k
        *
        * @author Dennis
        * @version 1.0
        * @date 2020/5/2 15:06
        */
        public class JmmTest02 {
            private int depth;

            public int getDepth() {
                return this.depth;
            }

            public void recurseDo() {
                System.out.println("递归深度：" + this.depth);
                this.depth++;
                try {
                    TimeUnit.MILLISECONDS.sleep(20);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                recurseDo();
            }

            public static void main(String[] args) {
                JmmTest02 jmmTest02 = new JmmTest02();
                jmmTest02.recurseDo();//Exception in thread "main" java.lang.StackOverflowError
            }
        }
    ````
- 结果
    ````text
        ...
        递归深度：1250
        递归深度：1251
        递归深度：1252
        递归深度：1253
        Exception in thread "main" java.lang.StackOverflowError
            at sun.nio.cs.UTF_8.updatePositions(UTF_8.java:77)
            at sun.nio.cs.UTF_8.access$200(UTF_8.java:57)
            at sun.nio.cs.UTF_8$Encoder.encodeArrayLoop(UTF_8.java:636)
            at sun.nio.cs.UTF_8$Encoder.encodeLoop(UTF_8.java:691)
            at java.nio.charset.CharsetEncoder.encode(CharsetEncoder.java:579)
            at sun.nio.cs.StreamEncoder.implWrite(StreamEncoder.java:271)
            at sun.nio.cs.StreamEncoder.write(StreamEncoder.java:125)
            at java.io.OutputStreamWriter.write(OutputStreamWriter.java:207)
            at java.io.BufferedWriter.flushBuffer(BufferedWriter.java:129)
            at java.io.PrintStream.write(PrintStream.java:526)
            at java.io.PrintStream.print(PrintStream.java:669)
            at java.io.PrintStream.println(PrintStream.java:806)
            at com.dennis.memory.JmmTest02.recurseDo(JmmTest02.java:21)
            at com.dennis.memory.JmmTest02.recurseDo(JmmTest02.java:28)
            at com.dennis.memory.JmmTest02.recurseDo(JmmTest02.java:28)
            ...
    ````
