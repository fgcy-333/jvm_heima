# 一、**什么是** **JVM ?**

定义：

Java Virtual Machine  （java的运行）java二进制字节码的运行环境



好处：

一次编写，到处运行的基石（jvm屏蔽了操作系统的差异）

自动内存管理，垃圾回收功能，对比于c、c++需要手动释放内存

数组下标越界检查，C语言也没有这种检查

多态，jvm使用虚方法表实现了多态



比较：jvm jre jdk

---

[![vp6p1U.png](https://s1.ax1x.com/2022/07/27/vp6p1U.png)](https://imgtu.com/i/vp6p1U)

---



# 二、学习JVM有什么用

JVM是一种规范

各种的虚拟机才是才是具体的实现

如：

oracle    的`hotspot`

eclipse    的     `Open J9`

---

[![vp6t9f.png](https://s1.ax1x.com/2022/07/27/vp6t9f.png)](https://imgtu.com/i/vp6t9f)

---



# **三、学习路线**



---

[![vp6qgO.png](https://s1.ax1x.com/2022/07/27/vp6qgO.png)](https://imgtu.com/i/vp6qgO)

---



**分三大块：一、类加载器快 二、JVM内存结构  三、执行引擎**



字节码文件通过类加载器加载进JVM中

类是放在方法区中的

类的实例与就是对象，是放在堆中

对象调用方法时，又会使用到虚拟机栈，程序计数器，本地方法栈



每行代码使用解释器逐行进行执行的

即时编译器对热点代码（频繁执行的代码）进行优化

垃圾回收，会对堆中没有被引用的对象进行回收释放内存

通过本地方法接口，来调用操作系统的一些方法







内存结构-----------------》垃圾回收--------------------------》编译优化-------------》类加载器（加载优化）-----------》即时编译器













# **四、内存结构**

##  1.0 程序计数器

Program Counter Register 程序计数器（寄存器）

---

[![vpgAW6.png](https://s1.ax1x.com/2022/07/27/vpgAW6.png)](https://imgtu.com/i/vpgAW6)

---









PC的作用：

---

[![vpg76O.png](https://s1.ax1x.com/2022/07/27/vpg76O.png)](https://imgtu.com/i/vpg76O)

---

java代码会通过jdk中的javac工具转化为二进制的字节码文件

然后通过类加载器将字节码文件加载进jvm中；

程序计数器先将第一条指令的地址装载到pc中

然后解释器将一行代码解释为机器码

交由cpu执行（此时pc会装载下一条指令的地址）；



注意：

每个线程都有自己的程序计数器

作用，是  **记住**   `下一条`   jvm指令的执行地址





程序计数器特点

1、`线程私有`的

2、JVM规范中规定了：PC是唯一 一个不会存在内存溢出的区（堆、栈、方法区都有内存溢出的问题）





##  2.0 虚拟机栈

Java Virtual Machine Stacks （Java 虚拟机栈）

---

[![vp4LB8.png](https://s1.ax1x.com/2022/07/27/vp4LB8.png)](https://imgtu.com/i/vp4LB8)

---







**栈帧与栈的关系：**

---

[![vp5Qu6.png](https://s1.ax1x.com/2022/07/27/vp5Qu6.png)](https://imgtu.com/i/vp5Qu6)



每个  **线程运行时所需要的内存**，称为  `虚拟机栈`

栈帧：**方法运行时**  `所需要的内存`（入参，局部参数，返回地址【记录一个方法调用完一个方法后，结束的位置（这个方法结束后，下一条指令是什么）】）

​			一个栈帧对应一个方法的调用



每个栈由多个栈帧（Frame）组成，对应着每次方法调用时所占用的内存

**每个线程**只能  **有一个活动栈帧**，  对应   着  **当前正在执行**的**那个方法**（**在栈顶**）

当前正在执行的方法对应的栈帧，叫做**当前栈帧**；



问题辨析

1. 垃圾回收是否涉及栈内存？

   垃圾回收针对堆内存中没有引用的对象，栈帧随着方法结束而出栈；

   没有涉及垃圾回收；

   

2. 栈内存分配越大越好吗？

   `Linux`、`MacOs`、`Solaris`默认是`1024k`

   Widow由   `默认的虚拟内存`  决定

   栈内存不是分配得越大越好，栈内存是线程运行所需要的内存空间，以为内存空间数量一定，栈内存空间越大，线程数量越少；

   可以通过一个虚拟机参数指定 栈内存大小：`-Xss 1024k`



3. 方法内的局部变量是否线程安全？

   看一个线程安不安全，就是看  `多个线程`  对 ` 一个变量`  是否是共享的；

   还是私有的；

   局部变量存在在于栈帧中，线程运行在虚拟机方法栈中，栈帧会压入方法栈中；

   每个线程都有自己的栈，所以局部变量是线程私有的；

   所以局部变量是线程安全的；







当是一个静态变量，所有该类的对象都可以访问的变量：

那么就存在线程安全问题，因为这个变量不是线程私有的，而是多个线程共享的；

---

[![v9LiIP.png](https://s1.ax1x.com/2022/07/28/v9LiIP.png)](https://imgtu.com/i/v9LiIP)

---





结论：

​	如果 ` 方法内局部变量`(入参，局部变量)  没有逃离方法的作用范围，它是线程安全的

​	如果是`局部变量  引用了  对象`，并`逃离方法的作用范围`，需要考虑线程安全



注意：

不能等主线程的核心逻辑都跑完才开始，启动一个新的线程；否则相当于单线程；



### 2.1 **栈内存溢出**

栈帧过多导致栈内存溢出（一般是这种情况比较多）

一般是方法递归导致的

---

[![v9Xf8f.png](https://s1.ax1x.com/2022/07/28/v9Xf8f.png)](https://imgtu.com/i/v9Xf8f)

---



栈帧过大导致栈内存溢出（这种情况出现的可能性较小）

---

[![v9XvxU.png](https://s1.ax1x.com/2022/07/28/v9XvxU.png)](https://imgtu.com/i/v9XvxU)

---





~~~java
/**
 * 演示栈内存溢出 java.lang.StackOverflowError
 * -Xss256k
 */
public class Demo1_2 {
    private static int count;

    public static void main(String[] args) {
        try {
            method1();
        } catch (Throwable e) {
            e.printStackTrace();
            System.out.println(count);
        }
    }

    private static void method1() {
        count++;
        method1();
    }
}
~~~

~~~
java.lang.StackOverflowError
	at cn.itcast.jvm.t1.stack.Demo1_2.method1(Demo1_2.java:21)


31601
~~~





设置栈大小，改变虚拟机参数 `-Xss256k`

步骤：

---

![image-20220728211445100](http://fgcy-pic.zhamao.ml/image-20220728211445100.png)

---



---

![image-20220728211734466](http://fgcy-pic.zhamao.ml/image-20220728211734466.png)

---



重新运行该方法

---

![image-20220728211935684](http://fgcy-pic.zhamao.ml/image-20220728211935684.png)

---





不是   `自己写的递归无终点方法`  导致的栈内存溢出

两个类**循环引用**导致出现 无限递归

例子：

~~~java
package cn.itcast.jvm.t1.stack;

import com.fasterxml.jackson.annotation.JsonIgnore;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;

import java.util.Arrays;
import java.util.List;

/**
 * json 数据转换
 */
public class Demo1_19 {
	//部门中有员工列表，员工中有部门
    public static void main(String[] args) throws JsonProcessingException {
        Dept d = new Dept();
        d.setName("Market");

        Emp e1 = new Emp();
        e1.setName("zhang");
        e1.setDept(d);

        Emp e2 = new Emp();
        e2.setName("li");
        e2.setDept(d);

        d.setEmps(Arrays.asList(e1, e2));

        // { name: 'Market', emps: [{ name:'zhang', dept:{ name:'', emps: [ {}]} },] }
        ObjectMapper mapper = new ObjectMapper();
        System.out.println(mapper.writeValueAsString(d));
    }
}


class Emp {
    private String name;

    private Dept dept;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Dept getDept() {
        return dept;
    }

    public void setDept(Dept dept) {
        this.dept = dept;
    }
}


class Dept {
    private String name;
    private List<Emp> emps;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public List<Emp> getEmps() {
        return emps;
    }

    public void setEmps(List<Emp> emps) {
        this.emps = emps;
    }
}

~~~







json映射异常，栈内存溢出

~~~
Exception in thread "main" com.fasterxml.jackson.databind.JsonMappingException: Infinite recursion (StackOverflowError) (through reference chain: cn.itcast.jvm.t1.stack.Dept["emps"]->java.util.ArrayList[0]->cn.itcast.jvm.t1.stack.Emp["dept"]->cn.itcast.jvm.t1.stack.Dept["emps"]->java.util.ArrayList[0]->cn.itcast.jvm.t1.stack.Emp["dept"]->cn.itcast.jvm.t1.stack.Dept["emps"]->java.util.ArrayList[0]->cn.itcast.jvm.t1.stack.Emp["dept"]->cn.itcast.jvm.t1.stack.Dept["emps"]->java.util.ArrayList[0]->cn.itcast.jvm.t1.stack.Emp["dept"]->cn.itcast.jvm.t1.stack.Dept["emps"]->java.util.ArrayList[0]->cn.itcast.jvm.t1.stack.Emp["dept"]-
......................................................
	at com.fasterxml.jackson.databind.ser.std.BeanSerializerBase.serializeFields(BeanSerializerBase.java:658)
~~~



解决：

---

![image-20220728213824984](http://fgcy-pic.zhamao.ml/image-20220728213824984.png)

---



~~~
{"name":"Market","emps":[{"name":"zhang"},{"name":"li"}]}
~~~



### 2.2 线程运行诊断



Linux下跑一个后台的java程序

> nohub java 名字 &



监控某个进程对cpu、内存的占用情况

> top





用ps命令进一步定位是  哪个线程`(十进制)`   引起的cpu占用过高

> ps H -eo pid,tid,%cpu | grep 进程id 



> jstack 进程id

可以**根据  线程id** 找到有问题的线程`(十六进制)`，进一步定位到问题代码的源码行号





案例2：程序运行很长时间没有结果`(用上面的线程诊断步骤，来判断该程序发生死锁)`

~~~java
package cn.itcast.jvm.t1.stack;

/**
 * 演示线程死锁
 */
class A{};
class B{};
public class Demo1_3 {
    static A a = new A();
    static B b = new B();


    public static void main(String[] args) throws InterruptedException {
        new Thread(()->{
            synchronized (a) {
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (b) {
                    System.out.println("我获得了 a 和 b");
                }
            }
        }).start();
        //在这先等一秒，再开启下面的线程
        //保证锁a一定被他人占有
        Thread.sleep(1000);
        new Thread(()->{
            synchronized (b) {
                //在这停下来，尝试获取锁a
                synchronized (a) {
                    System.out.println("我获得了 a 和 b");
                }
            }
        }).start();
    }
}
~~~





 ## 3.0 本地方法栈



---

![image-20220728223924580](http://fgcy-pic.zhamao.ml/image-20220728223924580.png)

---

**本地方法栈**，给   **本地的方法**   运行提供一块`内存空间`

本地方法：用`   C或C++语言  `编写的方法；用于  直接  **调用操作系统底层的API**



例如：Object类中 	

~~~java
private static native void registerNatives();
 
public final native Class<?> getClass();
 
public native int hashCode();

protected native Object clone() throws CloneNotSupportedException;

public final native void notify();

public final native void notifyAll();

public final native void wait(long timeout) throws InterruptedException;
~~~

**java代码**  通过   **本地方法接口 （JNI）** 调用  `使用C或C++语言写的`  可以直接调用`底层操作系统的api`的方法



 	

 ## 4.0 堆



---

![image-20220728225327246](http://fgcy-pic.zhamao.ml/image-20220728225327246.png)

---



**程序计数器、本地方法栈、虚拟机栈**   这些JVM内存结构都是线程私有的



Heap 堆 通过 new 关键字，创建对象都会使用堆内存



特点 ：

它是**线程共享**的，**堆中对象**  `都  `需要考虑   **线程安全**  的问题

虚拟机栈中的局部变量(`对象` && `逃离方法作用范围`)

堆中有垃圾回收机制，当堆中的对象没有被引用时，就会被回收掉（以释放掉空闲的内存）





### 4.1 堆内存溢出 



举例：

~~~java
import java.util.ArrayList;
import java.util.List;

/**
 * 演示堆内存溢出 java.lang.OutOfMemoryError: Java heap space
 * -Xmx8m
 */
public class Demo1_5 {

    public static void main(String[] args) {
        int i = 0;
        try {
            List<String> list = new ArrayList<>();
            String a = "hello";
            while (true) {
                list.add(a); // hello, hellohello, hellohellohellohello ...
                a = a + a;  // hellohellohellohello
                i++;
            }
        } catch (Throwable e) {
            e.printStackTrace();
            System.out.println(i);
        }
    }
}
~~~

~~~
java.lang.OutOfMemoryError: Java heap space   堆空间不足导致的内存溢出现象
24
~~~



修改堆空间最大值：

----

![image-20220728230908843](http://fgcy-pic.zhamao.ml/image-20220728230908843.png)

---



运行17次后**OOM**`（内存溢出错误）`  【之前是1g】

---

![image-20220728230934123](http://fgcy-pic.zhamao.ml/image-20220728230934123.png)

---



### 4.2 堆 内 存 诊 断

1 . `j p s  `工 具查看 当 前 系 统 中 有 哪 些 `j a v a  进 程`  显示进程编号



2. j m a p  工 具 查 看    `某个java进程`   的  `堆 内 存 占 用 情 况 `

> ​	 j m a p  -  h e a p  进 程id 

注意：

这个监控的是 `某一个 时刻的`



3. j c o n s ole  工 具 `图 形 界 面 的 `，` 多 功 能 `  (线程，cpu)的 监 测 工 具 ， 可 以 `连 续 监 测`







例子1：（使用 JDK 中的工具 `jmap`）

步骤：



1、启动代码

~~~java
package cn.itcast.jvm.t1.heap;

/**
 * 演示堆内存
 */
public class Demo1_4 {

    public static void main(String[] args) throws InterruptedException {
        System.out.println("1...");
        Thread.sleep(30000);



        byte[] array = new byte[1024 * 1024 * 10]; // 10 Mb
        System.out.println("2...");
        Thread.sleep(20000);



        array = null;
        System.gc();
        System.out.println("3...");
        Thread.sleep(1000000L);
    }
}

~~~

2、使用jdk中的`jps命令` 显示有哪些java进程

该程序一共有三个阶段，此时处于第一个阶段

3、在控制台窗口输入命令  `jmap -heap 进程号`

4、等控制台输出2时，在控制台窗口输入命令  `jmap -heap 进程号`

5、等控制台输出3时，在控制台窗口输入命令  `jmap -heap 进程号`



~~~
D:\learn\黑马程序员\资料-解密JVM\代码\jvm>jps         -- 显示有哪些java进程
24260 Demo1_4
12312
1704 Jps
24232 Launcher
9500 RemoteMavenServer36



D:\learn\黑马程序员\资料-解密JVM\代码\jvm>jmap -heap 24260
Attaching to process ID 24260, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.201-b09

using thread-local object allocation.
Parallel GC with 13 thread(s)

Heap Configuration:
   MinHeapFreeRatio         = 0
   MaxHeapFreeRatio         = 100
   MaxHeapSize              = 4257218560 (4060.0MB)     -- 最大堆内存空间
   NewSize                  = 88604672 (84.5MB)         -- 新生代 
   MaxNewSize               = 1418723328 (1353.0MB)
   OldSize                  = 177733632 (169.5MB)       --老年代
   NewRatio                 = 2
   SurvivorRatio            = 8                         --幸存区
   MetaspaceSize            = 21807104 (20.796875MB)	--元空间
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 0 (0.0MB)

Heap Usage:
PS Young Generation
Eden Space:             --伊甸园区空间
   capacity = 66584576 (63.5MB)
   used     = 7990264 (7.620109558105469MB)  -- 占用7M
   free     = 58594312 (55.87989044189453MB)
   12.000172532449557% used
From Space:
   capacity = 11010048 (10.5MB)
   used     = 0 (0.0MB)
   free     = 11010048 (10.5MB)
   0.0% used
To Space:
   capacity = 11010048 (10.5MB)
   used     = 0 (0.0MB)
   free     = 11010048 (10.5MB)
   0.0% used
PS Old Generation
   capacity = 177733632 (169.5MB)
   used     = 0 (0.0MB)
   free     = 177733632 (169.5MB)
   0.0% used

3157 interned Strings occupying 280136 bytes.






-- 此时创建一个大小为10M的字节数组
D:\learn\黑马程序员\资料-解密JVM\代码\jvm>jmap -heap 24260
Attaching to process ID 24260, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.201-b09

using thread-local object allocation.
Parallel GC with 13 thread(s)

Heap Configuration:
   MinHeapFreeRatio         = 0
   MaxHeapFreeRatio         = 100
   MaxHeapSize              = 4257218560 (4060.0MB)
   NewSize                  = 88604672 (84.5MB)
   MaxNewSize               = 1418723328 (1353.0MB)
   OldSize                  = 177733632 (169.5MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 0 (0.0MB)

Heap Usage:
PS Young Generation
Eden Space:
   capacity = 66584576 (63.5MB)
   used     = 18476040 (17.62012481689453MB)  -- 大概占用10M
   free     = 48108536 (45.87987518310547MB)
   27.748228058101624% used
From Space:
   capacity = 11010048 (10.5MB)
   used     = 0 (0.0MB)
   free     = 11010048 (10.5MB)
   0.0% used
To Space:
   capacity = 11010048 (10.5MB)
   used     = 0 (0.0MB)
   free     = 11010048 (10.5MB)
   0.0% used
PS Old Generation
   capacity = 177733632 (169.5MB)
   used     = 0 (0.0MB)
   free     = 177733632 (169.5MB)
   0.0% used

3158 interned Strings occupying 280184 bytes.





-- 启用GC  释放内存
D:\learn\黑马程序员\资料-解密JVM\代码\jvm>jmap -heap 24260
Attaching to process ID 24260, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.201-b09

using thread-local object allocation.
Parallel GC with 13 thread(s)

Heap Configuration:
   MinHeapFreeRatio         = 0
   MaxHeapFreeRatio         = 100
   MaxHeapSize              = 4257218560 (4060.0MB)
   NewSize                  = 88604672 (84.5MB)
   MaxNewSize               = 1418723328 (1353.0MB)
   OldSize                  = 177733632 (169.5MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 0 (0.0MB)

Heap Usage:
PS Young Generation
Eden Space:
   capacity = 66584576 (63.5MB)
   used     = 1331712 (1.27001953125MB)   -- 占用1M
   free     = 65252864 (62.22998046875MB)
   2.0000307578740157% used
From Space:
   capacity = 11010048 (10.5MB)
   used     = 0 (0.0MB)
   free     = 11010048 (10.5MB)
   0.0% used
To Space:
   capacity = 11010048 (10.5MB)
   used     = 0 (0.0MB)
   free     = 11010048 (10.5MB)
   0.0% used
PS Old Generation
   capacity = 177733632 (169.5MB)
   used     = 1047456 (0.998931884765625MB)
   free     = 176686176 (168.50106811523438MB)
   0.5893403449944691% used

3144 interned Strings occupying 279192 bytes.
~~~



例子2：（使用 JDK 中的工具 `jconsole`）

使用步骤：

1：启动代码

~~~java
package cn.itcast.jvm.t1.heap;

/**
 * 演示堆内存
 */
public class Demo1_4 {

    public static void main(String[] args) throws InterruptedException {
        System.out.println("1...");
        Thread.sleep(30000);



        byte[] array = new byte[1024 * 1024 * 10]; // 10 Mb
        System.out.println("2...");
        Thread.sleep(20000);



        array = null;
        System.gc();
        System.out.println("3...");
        Thread.sleep(1000000L);
    }
}

~~~

2、在控制台输入命令`jconsole`

---

![image-20220730101601389](http://fgcy-pic.zhamao.ml/image-20220730101601389.png)

---



---

![image-20220730101631727](http://fgcy-pic.zhamao.ml/image-20220730101631727.png)

---



---

![image-20220730101644886](http://fgcy-pic.zhamao.ml/image-20220730101644886.png)

---





3、等待控制台输出

~~~
1....
2....
3....
~~~



---

![image-20220730100933253](http://fgcy-pic.zhamao.ml/image-20220730100933253.png)

---









案例 

垃圾回收后，内存占用仍然很高

1、运行代码

~~~java
package cn.itcast.jvm.t1.heap;

import java.util.ArrayList;
import java.util.List;

/**
 * 演示查看对象个数 堆转储 dump
 */
public class Demo1_13 {

    public static void main(String[] args) throws InterruptedException {
        List<Student> students = new ArrayList<>();
        for (int i = 0; i < 200; i++) {
            students.add(new Student());
//            Student student = new Student();
        }
        Thread.sleep(1000000000L);
    }
}
class Student {
    private byte[] big = new byte[1024*1024];
}
~~~



2、使用 `jmap -heap 进程id` **（通过jps查看进程id）**

---

![image-20220730103623810](http://fgcy-pic.zhamao.ml/image-20220730103623810.png)

---





3、使用jconsole工具，执行一次gc；

---

![image-20220730103951361](http://fgcy-pic.zhamao.ml/image-20220730103951361.png)

---



4、使用jconsole工具，查看新生代和老年代的内容

---

![image-20220730103832890](http://fgcy-pic.zhamao.ml/image-20220730103832890.png)

---



发现：只是释放了新生代的的堆内存，老年代的没有被释放；



使用 `jvisualvm` 命令进行进一步分析

步骤：

1、输入命令 `jvisualvm`



---

![image-20220730104807659](http://fgcy-pic.zhamao.ml/image-20220730104807659.png)

---









---

![image-20220730104830054](http://fgcy-pic.zhamao.ml/image-20220730104830054.png)

---





---

![image-20220730104858915](http://fgcy-pic.zhamao.ml/image-20220730104858915.png)

---







---

![image-20220730104935009](http://fgcy-pic.zhamao.ml/image-20220730104935009.png)

---







---

![image-20220730104953556](http://fgcy-pic.zhamao.ml/image-20220730104953556.png)

---



---

![image-20220730105038175](http://fgcy-pic.zhamao.ml/image-20220730105038175.png)

---







---

![image-20220730105357683](http://fgcy-pic.zhamao.ml/image-20220730105357683.png)

---



**小结：**

这里的Student引用一直被ArrayList持有，所以有被引用的对象不会被GC释放，内存也没有被释放





## 5.0 方法区



---

![image-20220730105948752](http://fgcy-pic.zhamao.ml/image-20220730105948752.png)

---





### 5.1 定义

JVM虚拟机规范：

> https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html



1、方法区于堆一样都是线程共享的区域；

2、存储了更类的结构相关的信息：**成员变量**，方法数据，**方法与构造器的`代码部分`**；

3、方法区在虚拟机启动时被创建，逻辑上是堆的组成部分（但具体的JVM厂商，可以放到其他位置），JVM规范并不强制规定方法区的位置

Oracle的Hotspot在jdk8以前，方法区的实现叫永久代，这个永久代就是堆内存的一部分；1.8的时候又换了一种实现，此时方法区叫做元空间，此时元空间（规范叫做方法区）使用的是系统的内存

4、方法区内存不足时，也会外抛一个 `OutOfMemoryError`的异常



### 5.2 组成

以下两图基于Oracle的Hotspot虚拟机：

JDK1.7以前，方法区（永久代）存在于堆中，而字符串常量池存在于永久代中

---

![image-20220730111421869](http://fgcy-pic.zhamao.ml/image-20220730111421869.png)

---

注意：

永久代的内存回收效率很低；当发生FullGC时，永久代才会进行垃圾回收；

FullGC需要等待到，老年代的空间不足时才会触发；触发时机晚了；



JDK1.8及其后续版本，方法区位于物理内存中；而不是位于堆中，但此时`字符串常量池`位于堆中；

![image-20220730111438844](http://fgcy-pic.zhamao.ml/image-20220730111438844.png)

---



小结：

1、JVM是一个规范，它规定了要有方法区这个东西；建议放在堆中；

2、Hotspot虚拟机中，字符串常量池无论那个版本都位于堆中；



### 5.3 方法区内存溢出



1.8 以前会导致永久代内存溢出

~~~java
package cn.itcast.jvm;


import com.sun.xml.internal.ws.org.objectweb.asm.ClassWriter;
import com.sun.xml.internal.ws.org.objectweb.asm.Opcodes;

/**
 * 演示永久代内存溢出  java.lang.OutOfMemoryError: PermGen space
 * -XX:MaxPermSize=1k
 */
public class Demo1_8 extends ClassLoader {
    public static void main(String[] args) {
        int j = 0;
        try {
            Demo1_8 test = new Demo1_8();
            for (int i = 0; i < 200000; i++, j++) {
                ClassWriter cw = new ClassWriter(0);
                cw.visit(Opcodes.V1_6, Opcodes.ACC_PUBLIC, "Class" + i, null, "java/lang/Object", null);
                byte[] code = cw.toByteArray();
                test.defineClass("Class" + i, code, 0, code.length);
            }
        } finally {
            System.out.println(j);
        }
    }
}

~~~



---

![image-20220731004653878](http://fgcy-pic.zhamao.ml/image-20220731004653878.png)

---







1.8 之后会导致元空间内存溢出



~~~java
package cn.itcast.jvm.t1.metaspace;

import jdk.internal.org.objectweb.asm.ClassWriter;
import jdk.internal.org.objectweb.asm.Opcodes;

/**
 * 演示元空间内存溢出 java.lang.OutOfMemoryError: Metaspace
 * -XX:MaxMetaspaceSize=10m
 */
public class Demo1_8 extends ClassLoader { // 可以用来加载类的二进制字节码
    public static void main(String[] args) {
        int j = 0;
        try {
            Demo1_8 test = new Demo1_8();
            for (int i = 0; i < 10000; i++, j++) {
                // ClassWriter 作用是生成类的二进制字节码
                ClassWriter cw = new ClassWriter(0);
                // 版本号， public， 类名, 包名, 父类， 接口
                cw.visit(Opcodes.V1_8, Opcodes.ACC_PUBLIC, "Class" + i, null, "java/lang/Object", null);
                // 返回 byte[]
                byte[] code = cw.toByteArray();
                // 执行了类的加载
                test.defineClass("Class" + i, code, 0, code.length); // Class 对象
            }
        } finally {
            System.out.println(j);
       }
    }
}
~~~



元空间使用的是系统内存，默认没有设置上限;

---

![image-20220730115526725](http://fgcy-pic.zhamao.ml/image-20220730115526725.png)

---

~~~
3331
Exception in thread "main" java.lang.OutOfMemoryError: Compressed class space
	at java.lang.ClassLoader.defineClass1(Native Method)
	at java.lang.ClassLoader.defineClass(ClassLoader.java:763)
	at java.lang.ClassLoader.defineClass(ClassLoader.java:642)
	at cn.itcast.jvm.t1.metaspace.Demo1_8.main(Demo1_8.java:24)

进程已结束，退出代码为 1
~~~



场景：

在使用Spring、Mybatis时会使用到一些动态生成字节码，完成动态的类加载；

使用CGlib技术完成动态代理



`ClassWriter`类作用，在JVM运行期间动态生成类的字节码（二进制），完成动态的类加载； 



这样会导致在在运行过程中产生大量的类，造成OOM（OutOfMemory）





### 5.4 运行时常量池

常量池，就是一张表， **虚拟机指令**  根据这张常量表找到要执行的类名、方法名、参数类型、字面量 等信息 ；

运行时常量池，**常量池**  是 `*.class 文件`中的，当该类被加载，它的  ` 常量池信息`  就会放入   `运行时常量池`，并把里面的  **符号地址**  变为  **真实地址**



一个基本的HelloWorld.java文件  

~~~java
package cn.itcast.jvm.t5;

// 二进制字节码（类基本信息，常量池，类方法定义->包含了虚拟机指令）
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("hello world");
    }
}
~~~



进入 HelloWorld.java文件所在目录，通过 `javac HelloWorld.java` 编译程序

进入HelloWorld.class文件所在目录，通过`javap -v HelloWorld.class` 进行反编译字节码文件



反编译信息如下：



类的基本信息

~~~
Classfile /D:/learn/黑马程序员/资料-解密JVM/代码/jvm/out/production/jvm/cn/itcast/jvm/t5/HelloWorld.class   //文件所在的绝对路径
  Last modified 2022-7-28; size 567 bytes         //最后修改时间
  MD5 checksum 8efebdac91aa496515fa1c161184e354         //签名
  Compiled from "HelloWorld.java"				//源文件
public class cn.itcast.jvm.t5.HelloWorld		//权限修饰符 包名 类名
  minor version: 0
  major version: 52                         //52对应jdk8
  flags: ACC_PUBLIC, ACC_SUPER
~~~



类方法定义：

~~~
{
  public cn.itcast.jvm.t5.HelloWorld();               //默认构造
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 4: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lcn/itcast/jvm/t5/HelloWorld;

  public static void main(java.lang.String[]);            //main方法
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:  //一下是虚拟机指令  
      stack=2, locals=1, args_size=1
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;  //获取静态变量System.out
         3: ldc           #3                  // String hello world
         5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V       //执行一次虚方法调用
         8: return                                                                                           //整个main方法执行结束
      LineNumberTable:
        line 6: 0
        line 7: 8
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       9     0  args   [Ljava/lang/String;
}
~~~



常量池中的信息：

一直往下找

其中当常量池中的常量信息被加载进运行时常量池中时， `#数字` 会转为一个  **内存地址**

~~~
Constant pool:
   #1 = Methodref          #6.#20         // java/lang/Object."<init>":()V
   #2 = Fieldref           #21.#22        // java/lang/System.out:Ljava/io/PrintStream;
   #3 = String             #23            // hello world
   #4 = Methodref          #24.#25        // java/io/PrintStream.println:(Ljava/lang/String;)V
   #5 = Class              #26            // cn/itcast/jvm/t5/HelloWorld
   #6 = Class              #27            // java/lang/Object
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               LocalVariableTable
  #12 = Utf8               this
  #13 = Utf8               Lcn/itcast/jvm/t5/HelloWorld;
  #14 = Utf8               main
  #15 = Utf8               ([Ljava/lang/String;)V
  #16 = Utf8               args
  #17 = Utf8               [Ljava/lang/String;
  #18 = Utf8               SourceFile
  #19 = Utf8               HelloWorld.java
  #20 = NameAndType        #7:#8          // "<init>":()V
  #21 = Class              #28            // java/lang/System
  #22 = NameAndType        #29:#30        // out:Ljava/io/PrintStream;
  #23 = Utf8               hello world
  #24 = Class              #31            // java/io/PrintStream
  #25 = NameAndType        #32:#33        // println:(Ljava/lang/String;)V
  #26 = Utf8               cn/itcast/jvm/t5/HelloWorld
  #27 = Utf8               java/lang/Object
  #28 = Utf8               java/lang/System
  #29 = Utf8               out
  #30 = Utf8               Ljava/io/PrintStream;
  #31 = Utf8               java/io/PrintStream
  #32 = Utf8               println
  #33 = Utf8               (Ljava/lang/String;)V
~~~



### 5.5 StringTable

理解字符串常量池：

源码：

~~~java
// StringTable [ "a", "b" ,"ab" ]  hashtable （哈希表）结构，不能扩容
public class Demo1_22 {
    // 常量池中的信息，都会被加载到运行时常量池中， 这时 a b ab 都是常量池中的 符号 ，还没有变为  java 字符串对象
    // ldc #2       从#2位置中加载符号 然后会把 a 符号变为 "a" 字符串对象 找一下串池中有没有这个对象（值匹配），没有就加入到串池中；
    // ldc #3       从#3位置中加载符号 会把 b 符号变为 "b" 字符串对象 找一下串池中有没有这个对象（值匹配），没有就加入到串池中；
    // ldc #4 会把 ab 从#4位置中加载符号  符号变为 "ab" 字符串对象 找一下串池中有没有这个对象（值匹配），没有就加入到串池中；

    public static void main(String[] args) {
        String s1 = "a"; //将字符串对象放到StringTable中的行为是懒惰的
        String s2 = "b";
        String s3 = "ab";
    }
}
~~~



反编译后的信息：

main方法：

~~~
 public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=1, locals=4, args_size=1
         0: ldc           #2             // String a //到常量池中的#2位置中获取 信息 然后这里是将信息 变为一个字符串对象(并将这个对象放到串池中)
         2: astore_1                                        //将上一步获取的字符串对象，存放到mian方法栈帧中的局部变量中
         3: ldc           #3                  // String b
         5: astore_2
         6: ldc           #4                  // String ab
         8: astore_3
         9: return
      LineNumberTable:
        line 11: 0
        line 12: 3
        line 13: 6
        line 21: 9
      LocalVariableTable:     //局部变量表
        Start  Length  Slot  Name   Signature
            0      10     0  args   [Ljava/lang/String;
            3       7     1    s1   Ljava/lang/String;
            6       4     2    s2   Ljava/lang/String;
            9       1     3    s3   Ljava/lang/String;

~~~



例子二：

源码：

~~~java
// StringTable [ "a", "b" ,"ab" ]  hashtable 结构，不能扩容
public class Demo1_22 {
    // 常量池中的信息，都会被加载到运行时常量池中， 这时 a b ab 都是常量池中的符号，还没有变为 java 字符串对象
    // ldc #2 会把 a 符号变为 "a" 字符串对象
    // ldc #3 会把 b 符号变为 "b" 字符串对象
    // ldc #4 会把 ab 符号变为 "ab" 字符串对象

    public static void main(String[] args) {
        String s1 = "a"; // 懒惰的
        String s2 = "b";
        String s3 = "ab";
        String s4 = s1 + s2; // new StringBuilder().append("a").append("b").toString()  new String("ab")
        String s5 = "a" + "b";  // javac 在编译期间的优化（因为这个在编译器就可以确定下来 ），结果已经在编译期确定为ab

        System.out.println(s3 == s5);
    }
}
~~~



反编译后信息：

~~~
 public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=3, locals=6, args_size=1
         0: ldc           #2                  // String a
         2: astore_1
         3: ldc           #3                  // String b
         5: astore_2
         6: ldc           #4                  // String ab
         8: astore_3
         
   --------------------------------------之前分析过--------------------------------------------------------------      
         
         9: new           #5                  // class java/lang/StringBuilder  为创建StringBuilder对象 开辟一块内存空间
        12: dup
        13: invokespecial #6                  // Method java/lang/StringBuilder."<init>":()V  调用StringBuilder的无参构造初始化变量
        16: aload_1                                                                           拿到局部变量表中1号位置的变量
        17: invokevirtual #7                  // Method java/lang/StringBuilder.append:f(Ljava/lang/String;)Ljava/lang/StringBuilder; 调用StringBuilder的append方法入参为上一步变量 
        
        20: aload_2
        21: invokevirtual #7                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
        24: invokevirtual #8  //Method java/lang/StringBuilder.toString:()Ljava/lang/String; 拿串池中的“ab”对象，去堆中新建字符串对象
        27: astore        4
        29: ldc           #4                  // String ab
        31: astore        5
        33: getstatic     #9                  // Field java/lang/System.out:Ljava/io/PrintStream;
        36: aload_3
        37: aload         5
        39: if_acmpne     46
        42: iconst_1
        43: goto          47
        46: iconst_0
        47: invokevirtual #10                 // Method java/io/PrintStream.println:(Z)V
        50: return
      LineNumberTable:
        line 11: 0
        line 12: 3
        line 13: 6
        line 14: 9
        line 15: 29
        line 17: 33
        line 21: 50
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      51     0  args   [Ljava/lang/String;
            3      48     1    s1   Ljava/lang/String;
            6      45     2    s2   Ljava/lang/String;
            9      42     3    s3   Ljava/lang/String;
           29      22     4    s4   Ljava/lang/String;
           33      18     5    s5   Ljava/lang/String;
      StackMapTable: number_of_entries = 2
        frame_type = 255 /* full_frame */
          offset_delta = 46
          locals = [ class "[Ljava/lang/String;", class java/lang/String, class java/lang/String, class java/lang/String, class java/lang/String, class java/lang/String ]
          stack = [ class java/io/PrintStream ]
        frame_type = 255 /* full_frame */
          offset_delta = 0
          locals = [ class "[Ljava/lang/String;", class java/lang/String, class java/lang/String, class java/lang/String, class java/lang/String, class java/lang/String ]
          stack = [ class java/io/PrintStream, int ]

~~~



串池：

~~~
Constant pool:
   #1 = Methodref          #12.#36        // java/lang/Object."<init>":()V
   #2 = String             #37            // a
   #3 = String             #38            // b
   #4 = String             #39            // ab
   #5 = Class              #40            // java/lang/StringBuilder
   #6 = Methodref          #5.#36         // java/lang/StringBuilder."<init>":()V
   #7 = Methodref          #5.#41         // java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
   #8 = Methodref          #5.#42         // java/lang/StringBuilder.toString:()Ljava/lang/String;
   #9 = Fieldref           #43.#44        // java/lang/System.out:Ljava/io/PrintStream;
  #10 = Methodref          #45.#46        // java/io/PrintStream.println:(Z)V
  #11 = Class              #47            // cn/itcast/jvm/t1/stringtable/Demo1_22
  #12 = Class              #48            // java/lang/Object
  #13 = Utf8               <init>
  #14 = Utf8               ()V
  #15 = Utf8               Code
  #16 = Utf8               LineNumberTable
  #17 = Utf8               LocalVariableTable
  #18 = Utf8               this
  #19 = Utf8               Lcn/itcast/jvm/t1/stringtable/Demo1_22;
  #20 = Utf8               main
  #21 = Utf8               ([Ljava/lang/String;)V
  #22 = Utf8               args
  #23 = Utf8               [Ljava/lang/String;
  #24 = Utf8               s1
  #25 = Utf8               Ljava/lang/String;
  #26 = Utf8               s2
  #27 = Utf8               s3
  #28 = Utf8               s4
  #29 = Utf8               s5
  #30 = Utf8               StackMapTable
  #31 = Class              #23            // "[Ljava/lang/String;"
  #32 = Class              #49            // java/lang/String
  #33 = Class              #50            // java/io/PrintStream
  #34 = Utf8               SourceFile
  #35 = Utf8               Demo1_22.java
  #36 = NameAndType        #13:#14        // "<init>":()V
  #37 = Utf8               a
  #38 = Utf8               b
  #39 = Utf8               ab
  #40 = Utf8               java/lang/StringBuilder
  #41 = NameAndType        #51:#52        // append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
  #42 = NameAndType        #53:#54        // toString:()Ljava/lang/String;
  #43 = Class              #55            // java/lang/System
  #44 = NameAndType        #56:#57        // out:Ljava/io/PrintStream;
  #45 = Class              #50            // java/io/PrintStream
  #46 = NameAndType        #58:#59        // println:(Z)V
  #47 = Utf8               cn/itcast/jvm/t1/stringtable/Demo1_22
  #48 = Utf8               java/lang/Object
  #49 = Utf8               java/lang/String
  #50 = Utf8               java/io/PrintStream
  #51 = Utf8               append
  #52 = Utf8               (Ljava/lang/String;)Ljava/lang/StringBuilder;
  #53 = Utf8               toString
  #54 = Utf8               ()Ljava/lang/String;
  #55 = Utf8               java/lang/System
  #56 = Utf8               out
  #57 = Utf8               Ljava/io/PrintStream;
  #58 = Utf8               println
  #59 = Utf8               (Z)V
~~~



证明字符串延迟实例化：

~~~java
package cn.itcast.jvm.t1.stringtable;

/**
 * 演示字符串字面量也是【延迟】成为对象的
 */
public class TestString {
    public static void main(String[] args) {
        int x = args.length;
        System.out.println(); // 字符串个数 2149 

        System.out.print("1");
        System.out.print("2");
        System.out.print("3");
        System.out.print("4");
        System.out.print("5");
        System.out.print("6");
        System.out.print("7");
        System.out.print("8");
        System.out.print("9");
        System.out.print("0");
        System.out.print("1"); // 字符串个数 2159
        System.out.print("2");
        System.out.print("3");
        System.out.print("4");
        System.out.print("5");
        System.out.print("6");
        System.out.print("7");
        System.out.print("8");
        System.out.print("9");
        System.out.print("0");
        System.out.print(x); // 字符串个数
    }
}
~~~



---

![image-20220730171415309](http://fgcy-pic.zhamao.ml/image-20220730171415309.png)

---



---

![image-20220730171503918](http://fgcy-pic.zhamao.ml/image-20220730171503918.png)

---



---

![image-20220730171651580](http://fgcy-pic.zhamao.ml/image-20220730171651580.png)

---



---

![image-20220730171807689](http://fgcy-pic.zhamao.ml/image-20220730171807689.png)

---



---

![image-20220730172105262](http://fgcy-pic.zhamao.ml/image-20220730172105262.png)

---



---

![image-20220730172458930](http://fgcy-pic.zhamao.ml/image-20220730172458930.png)

---







### 5.6 StringTable 特性

常量池中的字符串仅是符号，第一次用到时才变为对象 利用串池的机制，来避免重复创建字符串对象 

字符串变量拼接的原理是 StringBuilder （1.8） 

字符串常量拼接的原理是编译期优化 

可以使用 intern 方法，主动将串池中还没有的字符串对象放入串池 

​	1.7、1.8 以后的 ：将这个字符串对象尝试放入串池，如果  有  则**并不会放入**，如果没有则放入串池， 会把串 池中的对象返回 

​	1.6 将这个字符串对象尝试放入串池，如果  有  则**并不会放入**，如果没有会把此对象**复制一份**， 放入串池， 会把串池中的对象返回

举例1（1.8）：

~~~java
public class Demo1_23 {

    //  [ "a", "b","ab"]
    public static void main(String[] args) {

        //字符串变量拼接，底层调用StringBuilder方法，最后调用toString方法转为String对象 本质是在堆中创建一个String对象
        String s = new String("a") + new String("b");

        String s2 = s.intern(); // 将这个字符串对象尝试放入串池，如果有则并不会放入，如果没有则放入串池， 会把串池中的对象返回

        System.out.println( s2 == "ab");
        System.out.println( s == "ab");
    }
}
~~~

~~~
true
true
~~~



举例2（1.8）：

~~~java
public class Demo1_23 {
    
        //  ["ab", "a", "b"]
        public static void main(String[] args) {
    
            String x = "ab";
            String s = new String("a") + new String("b");
    
            // 堆  new String("a")   new String("b") new String("ab")
            String s2 = s.intern(); // 将这个字符串对象尝试放入串池，如果有则并不会放入，如果没有则放入串池， 会把串池中的对象返回
    
            System.out.println( s2 == x);
            System.out.println( s == x );
        }
    
    }
~~~

~~~
true
false
~~~





举例3（1.6）：

~~~java
public class Demo1_23 {

    // ["ab","a", "b"]
    public static void main(String[] args) {

        String x = "ab";
        String s = new String("a") + new String("b");

        // 堆  new String("a")   new String("b")  new String("ab")
        String s2 = s.intern(); // 将这个字符串对象尝试放入串池，如果有则并不会放入，如果没有则放入串池， 会把串池中的对象返回

        System.out.println( s2 == x);
        System.out.println( s == x );
    }

}
~~~

~~~
true
false
~~~



举例4（1.6）：

~~~java
package cn.itcast.jvm;

public class Demo1_23 {

    // ["a", "b", "ab"]
    public static void main(String[] args) {


        String s = new String("a") + new String("b");

        // 堆  new String("a")   new String("b")  new String("ab")
        String s2 = s.intern(); // 将这个字符串对象尝试放入串池，如果有则并不会放入，如果没有则放入串池， 会把串池中的对象返回
        // s 拷贝一份，放入串池

        String x = "ab";
        System.out.println( s2 == x);
        System.out.println( s == x );
    }
}
~~~

~~~
true
false
~~~









### 5.7 StringTable面试题

1.8：

~~~java
String s1 = "a";
String s2 = "b";
String s3 = "a" + "b"; //编译器优化 "ab"
String s4 = s1 + s2;   //new StringBuilder().append("a").append("b").toString()  toString即new String("ab")
String s5 = "ab";
String s6 = s4.intern(); //s4是堆中的字符串对象ab，尝试加入到串池中，发现已经有了，不加入；返回串池中“ab”d的对象

// 问
System.out.println(s3 == s4); //堆 跟 字符串常量池
System.out.println(s3 == s5);//字符串常量池 跟 字符串常量池
System.out.println(s3 == s6);//字符串常量池 跟 字符串常量池

String x2 = new String("c") + new String("d");//堆中对象ab
String x1 = "cd";//串池中对象cd
x2.intern();//尝试加入，但发现已经有了；不加入

System.out.println(x1 == x2);////字符串常量池 跟 堆
~~~

~~~
false
true
true
false
~~~



~~~java
package cn.itcast.jvm.t1.stringtable;

/**
 * 演示字符串相关面试题
 */
public class Demo1_21 {

    public static void main(String[] args) {
        String s1 = "a";
        String s2 = "b";
        String s3 = "a" + "b"; //编译器优化 "ab"
        String s4 = s1 + s2;   //new StringBuilder().append("a").append("b").toString()  toString即new String("ab")
        String s5 = "ab";
        String s6 = s4.intern(); //s4是堆中的字符串对象ab，尝试加入到串池中，发现已经有了，不加入；返回串池中“ab”d的对象

// 问
        System.out.println(s3 == s4); //堆 跟 字符串常量池
        System.out.println(s3 == s5);//字符串常量池 跟 字符串常量池
        System.out.println(s3 == s6);//字符串常量池 跟 字符串常量池

        String x2 = new String("c") + new String("d");//堆中对象ab
        x2.intern();//尝试加入，但发现没有，加入
        String x1 = "cd";//串池中对象cd

        System.out.println(x1 == x2);////字符串常量池 跟 字符串常量池
    }
}
~~~

~~~
false
true
true
true
~~~



小结：

所有对象都在堆中，在堆中的常量池（HashTable）只是维护了引用，通过重写equal方法来改变对象的比较方式





### 5.8 StringTable位置



---

![image-20220731011028661](http://fgcy-pic.zhamao.ml/image-20220731011028661.png)

---



证明：

1.6

~~~java
package cn.itcast.jvm;

import java.util.ArrayList;
import java.util.List;

/**
 * 演示 StringTable 位置
 * 在jdk8下设置 -Xmx10m -XX:-UseGCOverheadLimit
 * 在jdk6下设置 -XX:MaxPermSize=10m
 */
public class Demo1_6 {

    public static void main(String[] args) throws InterruptedException {
        List<String> list = new ArrayList<String>();
        int i = 0;
        try {
            for (int j = 0; j < 260000; j++) {
                list.add(String.valueOf(j).intern());//将堆中的对象加入到串池中，堆中的字符串对象有被ArrayList引用，且ArrayList对象长时间存活
                i++;
            }
        } catch (Throwable e) {
            e.printStackTrace();
        } finally {
            System.out.println(i);
        }
    }
}
~~~



---

![image-20220731011429788](http://fgcy-pic.zhamao.ml/image-20220731011429788.png)

---



1.8不是很严谨就不详细写了；对象存在于堆中，串池也存在于堆中，无法证明是不是因为串池中对象过多导致的OOM







### 5.9 StringTable 垃圾回收

**证明**

例子1：

~~~java
package cn.itcast.jvm.t1.stringtable;

import java.util.ArrayList;
import java.util.List;

/**
 * 演示 StringTable 垃圾回收
 * -Xmx10m 最大堆内存
 * -XX:+PrintStringTableStatistics 串池的统计信息
 * -XX:+PrintGCDetails -verbose:gc 打印垃圾回收的详细信息
 */
public class Demo1_7 {
    public static void main(String[] args) throws InterruptedException {
        int i = 0;
        try {
            for (int j = 0; j < 0; j++) { // j=100, j=10000
                String.valueOf(j).intern();
                i++;
           }
        } catch (Throwable e) {
            e.printStackTrace();
        } finally {
            System.out.println(i);
        }

    }
}
~~~

---

~~~
0
内存占用情况：
Heap
 PSYoungGen      total 2560K, used 1918K [0x00000000ffd00000, 0x0000000100000000, 0x0000000100000000)
  eden space 2048K, 93% used [0x00000000ffd00000,0x00000000ffedf868,0x00000000fff00000)
  from space 512K, 0% used [0x00000000fff80000,0x00000000fff80000,0x0000000100000000)
  to   space 512K, 0% used [0x00000000fff00000,0x00000000fff00000,0x00000000fff80000)
 ParOldGen       total 7168K, used 0K [0x00000000ff600000, 0x00000000ffd00000, 0x00000000ffd00000)
  object space 7168K, 0% used [0x00000000ff600000,0x00000000ff600000,0x00000000ffd00000)
 Metaspace       used 3245K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 352K, capacity 388K, committed 512K, reserved 1048576K
  
  
符号表的统计信息：
SymbolTable statistics:
Number of buckets       :     20011 =    160088 bytes, avg   8.000
Number of entries       :     13247 =    317928 bytes, avg  24.000
Number of literals      :     13247 =    589032 bytes, avg  44.465
Total footprint         :           =   1067048 bytes
Average bucket size     :     0.662
Variance of bucket size :     0.663
Std. dev. of bucket size:     0.814
Maximum bucket size     :         6

字符串常量池的统计信息：
StringTable statistics:
Number of buckets       :     60013 =    480104 bytes, avg   8.000 //hash表中数组元素个数
Number of entries       :      1761 =     42264 bytes, avg  24.000 //键值对个数
Number of literals      :      1761 =    178736 bytes, avg 101.497 //字符串个数
Total footprint         :           =    701104 bytes
Average bucket size     :     0.029
Variance of bucket size :     0.029
Std. dev. of bucket size:     0.172
Maximum bucket size     :         2
~~~



改变后，例子2：

没有进行垃圾回收

~~~java
package cn.itcast.jvm.t1.stringtable;

import java.util.ArrayList;
import java.util.List;

/**
 * 演示 StringTable 垃圾回收
 * -Xmx10m 最大堆内存
 * -XX:+PrintStringTableStatistics 串池的统计信息
 * -XX:+PrintGCDetails -verbose:gc 打印垃圾回收的详细信息
 */
public class Demo1_7 {
    public static void main(String[] args) throws InterruptedException {
        int i = 0;
        try {
            for (int j = 0; j < 100; j++) { // j=100, j=10000
                String.valueOf(j).intern();
                i++;
            }
        } catch (Throwable e) {
            e.printStackTrace();
        } finally {
            System.out.println(i);
        }

    }
}

~~~

~~~
100
Heap
 PSYoungGen      total 2560K, used 1834K [0x00000000ffd00000, 0x0000000100000000, 0x0000000100000000)
  eden space 2048K, 89% used [0x00000000ffd00000,0x00000000ffeca9b8,0x00000000fff00000)
  from space 512K, 0% used [0x00000000fff80000,0x00000000fff80000,0x0000000100000000)
  to   space 512K, 0% used [0x00000000fff00000,0x00000000fff00000,0x00000000fff80000)
 ParOldGen       total 7168K, used 0K [0x00000000ff600000, 0x00000000ffd00000, 0x00000000ffd00000)
  object space 7168K, 0% used [0x00000000ff600000,0x00000000ff600000,0x00000000ffd00000)
 Metaspace       used 3157K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 344K, capacity 388K, committed 512K, reserved 1048576K
SymbolTable statistics:
Number of buckets       :     20011 =    160088 bytes, avg   8.000
Number of entries       :     13060 =    313440 bytes, avg  24.000
Number of literals      :     13060 =    583496 bytes, avg  44.678
Total footprint         :           =   1057024 bytes
Average bucket size     :     0.653
Variance of bucket size :     0.653
Std. dev. of bucket size:     0.808
Maximum bucket size     :         6
StringTable statistics:
Number of buckets       :     60013 =    480104 bytes, avg   8.000
Number of entries       :      1842 =     44208 bytes, avg  24.000
Number of literals      :      1842 =    182384 bytes, avg  99.014
Total footprint         :           =    706696 bytes
Average bucket size     :     0.031
Variance of bucket size :     0.031
Std. dev. of bucket size:     0.175
Maximum bucket size     :         2
~~~





改变后，例子3：

有发生垃圾回收；

~~~java
package cn.itcast.jvm.t1.stringtable;

import java.util.ArrayList;
import java.util.List;

/**
 * 演示 StringTable 垃圾回收
 * -Xmx10m 最大堆内存
 * -XX:+PrintStringTableStatistics 串池的统计信息
 * -XX:+PrintGCDetails -verbose:gc 打印垃圾回收的详细信息
 */
public class Demo1_7 {
    public static void main(String[] args) throws InterruptedException {
        int i = 0;
        try {
            for (int j = 0; j < 10000; j++) { // j=100, j=10000
                String.valueOf(j).intern();
                i++;
            }
        } catch (Throwable e) {
            e.printStackTrace();
        } finally {
            System.out.println(i);
        }

    }
}
~~~

~~~
[GC (Allocation Failure) [PSYoungGen: 2048K->488K(2560K)] 2048K->763K(9728K), 0.0005872 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]  //发生了一次新生代的垃圾回收
10000
Heap
 PSYoungGen      total 2560K, used 775K [0x00000000ffd00000, 0x0000000100000000, 0x0000000100000000)
  eden space 2048K, 14% used [0x00000000ffd00000,0x00000000ffd47d70,0x00000000fff00000)
  from space 512K, 95% used [0x00000000fff00000,0x00000000fff7a020,0x00000000fff80000)
  to   space 512K, 0% used [0x00000000fff80000,0x00000000fff80000,0x0000000100000000)
 ParOldGen       total 7168K, used 275K [0x00000000ff600000, 0x00000000ffd00000, 0x00000000ffd00000)
  object space 7168K, 3% used [0x00000000ff600000,0x00000000ff644c70,0x00000000ffd00000)
 Metaspace       used 3247K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 352K, capacity 388K, committed 512K, reserved 1048576K
SymbolTable statistics:
Number of buckets       :     20011 =    160088 bytes, avg   8.000
Number of entries       :     13247 =    317928 bytes, avg  24.000
Number of literals      :     13247 =    589032 bytes, avg  44.465
Total footprint         :           =   1067048 bytes
Average bucket size     :     0.662
Variance of bucket size :     0.663
Std. dev. of bucket size:     0.814
Maximum bucket size     :         6
StringTable statistics:
Number of buckets       :     60013 =    480104 bytes, avg   8.000
Number of entries       :      7074 =    169776 bytes, avg  24.000
Number of literals      :      7074 =    433712 bytes, avg  61.311
Total footprint         :           =   1083592 bytes
Average bucket size     :     0.118
Variance of bucket size :     0.121
Std. dev. of bucket size:     0.347
Maximum bucket size     :         3
~~~







### 5.10 StringTable 性能调优

- **调整 -XX:StringTableSize=桶个数**

例子1：

~~~java
package cn.itcast.jvm.t1.stringtable;

import java.io.BufferedReader;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStreamReader;

/**
 * 演示串池大小对性能的影响
 * -Xms500m 堆的最小容量
 * -Xmx500m 堆的最大容量
 * -XX:+PrintStringTableStatistics
 */
public class Demo1_24 {

    public static void main(String[] args) throws IOException {
        try (BufferedReader reader = new BufferedReader(new InputStreamReader(new FileInputStream("linux.words"), "utf-8"))) {
            String line = null;
            long start = System.nanoTime();
            //1秒等于1000毫秒、1毫秒等于1000微秒、1微秒等于1000纳秒、1纳秒等于1000皮秒
            while (true) {
                line = reader.readLine();
                if (line == null) {
                    break;
                }
                line.intern();
            }
            System.out.println("cost:" + (System.nanoTime() - start) / 1000000);
        }
    }
}

~~~



linux.words

---

![image-20220731140434954](http://fgcy-pic.zhamao.ml/image-20220731140434954.png)

---



~~~
cost:248   //0.2秒
SymbolTable statistics:
Number of buckets       :     20011 =    160088 bytes, avg   8.000
Number of entries       :     13250 =    318000 bytes, avg  24.000
Number of literals      :     13250 =    589096 bytes, avg  44.460
Total footprint         :           =   1067184 bytes
Average bucket size     :     0.662
Variance of bucket size :     0.663
Std. dev. of bucket size:     0.814
Maximum bucket size     :         6
StringTable statistics:
Number of buckets       :     60013 =    480104 bytes, avg   8.000 //默认六万个桶，即数组容量为60013个
Number of entries       :    481502 =  11556048 bytes, avg  24.000
Number of literals      :    481502 =  29751672 bytes, avg  61.789
Total footprint         :           =  41787824 bytes
Average bucket size     :     8.023
Variance of bucket size :     8.085
Std. dev. of bucket size:     2.843
Maximum bucket size     :        23
~~~



例子2：

设置JVM参数：

> ```
> -XX:StringTableSize=1009
> ```

---

![image-20220731141225209](http://fgcy-pic.zhamao.ml/image-20220731141225209.png)

---



例子3：

设置JVM参数

> -XX:StringTableSize=20 0000

---

![image-20220731141456465](http://fgcy-pic.zhamao.ml/image-20220731141456465.png)

---



小结：

通过设置JVM参数，使得串池桶的个数变大，这样可以减少hash碰撞概率，提升效率





- **考虑将字符串对象是否入池**



不入

1、运行程序

~~~java
public class Demo1_25 {

    public static void main(String[] args) throws IOException {

        List<String> address = new ArrayList<>();
        System.in.read();
        for (int i = 0; i < 10; i++) {
            try (BufferedReader reader = new BufferedReader(new InputStreamReader(new FileInputStream("linux.words"), "utf-8"))) {
                String line = null;
                long start = System.nanoTime();
                while (true) {
                    line = reader.readLine();
                    if(line == null) {
                        break;
                    }
                    address.add(line);
                }
                System.out.println("cost:" +(System.nanoTime()-start)/1000000);
            }
        }
        System.in.read();
    }
}
~~~



2、打开jvisualvm

---

![image-20220731142610723](http://fgcy-pic.zhamao.ml/image-20220731142610723.png)

---



3、一次回车（读取十次文件）

---

![image-20220731142800069](http://fgcy-pic.zhamao.ml/image-20220731142800069.png)

---





进入串池

1、运行代码

~~~java
public class Demo1_25 {

    public static void main(String[] args) throws IOException {

        List<String> address = new ArrayList<>();
        System.in.read();
        for (int i = 0; i < 10; i++) {
            try (BufferedReader reader = new BufferedReader(new InputStreamReader(new FileInputStream("linux.words"), "utf-8"))) {
                String line = null;
                long start = System.nanoTime();
                while (true) {
                    line = reader.readLine();
                    if(line == null) {
                        break;
                    }
                    address.add(line.intern());
                }
                System.out.println("cost:" +(System.nanoTime()-start)/1000000);
            }
        }
        System.in.read();
    }
}
~~~



2、打开jvisualvm

---

![image-20220731143040775](http://fgcy-pic.zhamao.ml/image-20220731143040775.png)

---



3、一次回车（读取十次文件）

---

![image-20220731143254742](http://fgcy-pic.zhamao.ml/image-20220731143254742.png)

---





##  6.0 直接内存

### 6.1 定义

Direct Memory 常见于 NIO 操作时，用于数据缓冲区 

这个直接内存，是属于操作系统的内存，java想要分配和释放都比较缓慢

分配回收成本较高，但读写性能高 







传统阻塞IO与NIO的比较

~~~java
package cn.itcast.jvm.t1.direct;

import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;

/**
 * 演示 ByteBuffer 作用
 */
public class Demo1_9 {
    static final String FROM = "D:\\BaiduNetdiskDownload\\146.消息中间件-RocketMQ06.mp4";
    static final String TO = "D:\\146.消息中间件-RocketMQ06.mp4";
    static final int _1Mb = 1024 * 1024;

    public static void main(String[] args) {
        io(); // io 用时：1535.586957 1766.963399 1359.240226
        directBuffer(); // directBuffer 用时：479.295165 702.291454 562.56592
    }


    //使用直接内存的NIO
    private static void directBuffer() {
        long start = System.nanoTime();
        try (
             FileChannel from = new FileInputStream(FROM).getChannel();
             FileChannel to = new FileOutputStream(TO).getChannel();
        ) {
            //由操作系统分配一块直接内存 （该内存操作系统和java代码可以直接访问 ）
            ByteBuffer bb = ByteBuffer.allocateDirect(_1Mb);
            while (true) {
                int len = from.read(bb);
                if (len == -1) {
                    break;
                }
                bb.flip();
                to.write(bb);
                bb.clear();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        long end = System.nanoTime();
        System.out.println("directBuffer 用时：" + (end - start) / 1000_000.0);
    }

    //使用传统的IO
    private static void io() {
        long start = System.nanoTime();
        try (
             FileInputStream from = new FileInputStream(FROM);
             FileOutputStream to = new FileOutputStream(TO);
        ) {
            byte[] buf = new byte[_1Mb];
            while (true) {
                int len = from.read(buf);
                if (len == -1) {
                    break;
                }
                to.write(buf, 0, len);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        long end = System.nanoTime();
        System.out.println("io 用时：" + (end - start) / 1000_000.0);
    }
}

~~~

~~~
io 用时：936.9851   //0.9s
directBuffer 用时：504.2338  //0.5s
~~~

解释读取大文件时，直接内存会更快



**使用传统IO**

---

![image-20220731151152900](http://fgcy-pic.zhamao.ml/image-20220731151152900.png)

---

java代码无法直接操作底层 文件IO，只能通过调用本地方法间接实现；

CPU分为两种状态：用户态，和内核态

内存也分为两种：JVM中的堆内存和系统内存

java无法直接获取到系统内存中的数据，只能先将系统内存中的数据复制一份到堆内存中，再操作堆内存中的数据

此时，读取一份文件需要复制两次









**使用了DirectBuffer**

---

![image-20220731151853223](http://fgcy-pic.zhamao.ml/image-20220731151853223.png)

---

由操作系统分配一块直接内存 （该内存操作系统和java代码可以直接访问 ）

比传统IO少了一次复制操作，效率更高；





**不受 JVM 内存回收管理**

例子1：（演示直接内存溢出）

~~~java
package cn.itcast.jvm.t1.direct;

import java.nio.ByteBuffer;
import java.util.ArrayList;
import java.util.List;


/**
 * 演示直接内存溢出
 */
public class Demo1_10 {
    static int _100Mb = 1024 * 1024 * 100;

    public static void main(String[] args) {
        List<ByteBuffer> list = new ArrayList<>();
        int i = 0;
        try {
            while (true) {
                ByteBuffer byteBuffer = ByteBuffer.allocateDirect(_100Mb);
                list.add(byteBuffer);
                i++;
            }
        } finally {
            System.out.println(i);
        }
        // 方法区是jvm规范， jdk6 中对方法区的实现称为永久代
        //                  jdk8 对方法区的实现称为元空间
    }
}

~~~

---

![image-20220731160632622](http://fgcy-pic.zhamao.ml/image-20220731160632622.png)

---





演示垃圾回收ByteBuffer对象

1、还没运行程序

---

![image-20220731161433975](http://fgcy-pic.zhamao.ml/image-20220731161433975.png)

---



2、运行程序

~~~java
package cn.itcast.jvm.t1.direct;

import java.io.IOException;
import java.nio.ByteBuffer;


public class Demo1_26 {
    static int _1Gb = 1024 * 1024 * 1024;

    public static void main(String[] args) throws IOException {
        ByteBuffer byteBuffer = ByteBuffer.allocateDirect(_1Gb);
        System.out.println("分配完毕...");
        System.in.read();
        System.out.println("开始释放...");
        byteBuffer = null;
        System.gc(); // 显式的垃圾回收，Full GC
        System.in.read();
    }
}
~~~

---

![image-20220731161525406](http://fgcy-pic.zhamao.ml/image-20220731161525406.png)

---



---

![image-20220731161757573](http://fgcy-pic.zhamao.ml/image-20220731161757573.png)

---



3、控制台敲回车

---

![image-20220731161837777](http://fgcy-pic.zhamao.ml/image-20220731161837777.png)

---



---

![image-20220731161938357](http://fgcy-pic.zhamao.ml/image-20220731161938357.png)

---



**不是说不受JVM内存回收管理嘛？**

解释如下：

### 6.2 分配和回收原理 

使用了 Unsafe 对象完成直接内存的分配回收，并且回收需要主动调用 freeMemory 方法 

~~~java
package cn.itcast.jvm.t1.direct;

import sun.misc.Unsafe;

import java.io.IOException;
import java.lang.reflect.Field;

/**
 * 直接内存分配的底层原理：Unsafe
 */
public class Demo1_27 {
    static int _1Gb = 1024 * 1024 * 1024;

    public static void main(String[] args) throws IOException {
        Unsafe unsafe = getUnsafe();
        // 分配内存
        long base = unsafe.allocateMemory(_1Gb);
        unsafe.setMemory(base, _1Gb, (byte) 0);
        System.in.read();

        // 释放内存
        unsafe.freeMemory(base);
        System.in.read();
    }

    public static Unsafe getUnsafe() {
        try {
            Field f = Unsafe.class.getDeclaredField("theUnsafe");
            f.setAccessible(true);
            Unsafe unsafe = (Unsafe) f.get(null);
            return unsafe;
        } catch (NoSuchFieldException | IllegalAccessException e) {
            throw new RuntimeException(e);
        }
    }
}

~~~



一开始

---

![image-20220731163440228](http://fgcy-pic.zhamao.ml/image-20220731163440228.png)

---



运行程序

---

![image-20220731163541095](http://fgcy-pic.zhamao.ml/image-20220731163541095.png)

---



敲回车，释放内存

---

![image-20220731163641246](http://fgcy-pic.zhamao.ml/image-20220731163641246.png)

---







ByteBuffer源码分析

通过ByteBuffer类的静态allocateDirect方法，分配1个G的直接内存

~~~java
    ByteBuffer byteBuffer = ByteBuffer.allocateDirect(_1Gb);
~~~



查看`allocateDirect方法`，该方法调用了DirectByteBuffer类的构造方法

~~~java
   public static ByteBuffer allocateDirect(int capacity) {
        return new DirectByteBuffer(capacity);
    }
~~~



查看DirectByteBuffer的构造方法，该构造方法中通过unsafe对象创建了直接内存

~~~java
    // Primary constructor
    //
    DirectByteBuffer(int cap) {                   // package-private

        super(-1, 0, cap, cap);
        boolean pa = VM.isDirectMemoryPageAligned();
        int ps = Bits.pageSize();
        long size = Math.max(1L, (long)cap + (pa ? ps : 0));
        Bits.reserveMemory(size, cap);

        long base = 0;
        try {
            base = unsafe.allocateMemory(size);   //使用unsafe对象实现直接内存的创建1
        } catch (OutOfMemoryError x) {
            Bits.unreserveMemory(size, cap);
            throw x;
        }
        unsafe.setMemory(base, size, (byte) 0);   //使用unsafe对象实现直接内存的创建2
        if (pa && (base % ps != 0)) {
            // Round up to page boundary
            address = base + ps - (base & (ps - 1));
        } else {
            address = base;
        }
        cleaner = Cleaner.create(this, new Deallocator(base, size, cap)); //第二个参数是一个回调任务对象
        att = null;
    }
~~~



但是，前面说了：释放直接内存，必须调用unsafe对象的  `freeMemory(直接内存的地址)`方法



此时我们查看一下这行代码

~~~java
  cleaner = Cleaner.create(this, new Deallocator(base, size, cap)); //第二个参数是一个回调任务对象
~~~



查看一下Deallocator类

~~~java
  private static class Deallocator implements Runnable//该类是一个任务类
~~~



在该类的run方法中，使用的freeMemory方法

~~~java
      public void run() {
            if (address == 0) {
                // Paranoia
                return;
            }
            unsafe.freeMemory(address);
            address = 0;
            Bits.unreserveMemory(size, capacity);
        }
~~~



Cleaner类是一种特殊的类型，虚引用类型

~~~java
public class Cleaner extends PhantomReference<Object>
~~~

当虚引用对象所关联的实际对象被回收掉时，就会调用虚引用对象的clean方法 



clean方法 如下：

~~~java
    public void clean() {
        if (remove(this)) {
            try {
                this.thunk.run();//该clean方法又会调用回调任务对象的run方法
                //那个任务对象的run方法中由unsafe.freeMemory(java.Lang.Long)方法
            } catch (final Throwable var2) {
                AccessController.doPrivileged(new PrivilegedAction<Void>() {
                    public Void run() {
                        if (System.err != null) {
                            (new Error("Cleaner terminated abnormally", var2)).printStackTrace();
                        }

                        System.exit(1);
                        return null;
                    }
                });
            }

        }
    }
~~~



小结：

ByteBuffer 的实现类内部，使用了 Cleaner （虚引用）来监测 ByteBuffer 对象，一旦 ByteBuffer 对象被垃圾回收

那么就会由 `ReferenceHandler 线程` (守护线程) 调用Cleaner（虚引用类型） 的 clean 方法 （当虚引用对象所关联的实际对象被垃圾回收时，会被RefferenceHandler线程 调用clean方法）

clean方法又会调用回调任务对象的run方法 该方法中会 调 用 freeMemory 来释放直接内存



注意：

> System.gc();  //该方法触发的是一次FullGC，这种方式触发GC，也称为显式垃圾回收



当使用JVM参数  `-XX:+DisableExplicitGC` 禁用显式回收对直接内存的影响

也即是说，System.gc();失效



例子：（禁用显式回收对直接内存的影响）

~~~java
package cn.itcast.jvm.t1.direct;

import java.io.IOException;
import java.nio.ByteBuffer;

/**
 * 禁用显式回收对直接内存的影响
 */
public class Demo1_26 {
    static int _1Gb = 1024 * 1024 * 1024;

    /*
     * -XX:+DisableExplicitGC 显式的
     */
    public static void main(String[] args) throws IOException {
        ByteBuffer byteBuffer = ByteBuffer.allocateDirect(_1Gb);
        System.out.println("分配完毕...");
        System.in.read();
        System.out.println("开始释放...");
        byteBuffer = null;
        System.gc(); // 显式的垃圾回收，Full GC
        System.in.read();
    }
}
~~~



---

![image-20220731172951500](http://fgcy-pic.zhamao.ml/image-20220731172951500.png)

---





---

![image-20220731172924325](http://fgcy-pic.zhamao.ml/image-20220731172924325.png)

---



小结：

当发生这种禁用显式垃圾回收时，可以使用unsafe对象的freeMomory方法，手动释放直接内存





# 五、垃圾回收



## 1.0 如何判断对象可以回收

### 1.1 引用计数法

当一个对象有被引用时，计数器加1

当一个对象被断开引用时，计数器减1



问题：循环引用

----

![image-20220731174331705](http://fgcy-pic.zhamao.ml/image-20220731174331705.png)

---

A对象中引用了B对象；B对象中引用了A对象；即使A、B对象没有被其他对象所引用时，也不会触发垃圾回收







### 1.2 可达性分析算法（jvm采用的算法）

根对象：可以先理解为，肯定不能被垃圾回收的对象



在进行垃圾回收之前，会对堆中的所有对象进行一遍扫描，查看每一个对象是否被  `根对象`   **直接或间接引用**的对象，如是，则不能被回收掉



`Java虚拟机中的垃圾回收器`  采用  `可达性分析` 来  `探索所有存活的对象`

扫描堆中的对象,看是否能够沿着 `GC Root对象` 为  **起点**  的  引用链`找到该对象`,找不到,表示可以回收



哪些对象可以作为GC Root?

~~~java
package cn.itcast.jvm.t2;

import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

/**
 * 演示GC Roots
 */
public class Demo2_2 {

    public static void main(String[] args) throws InterruptedException, IOException {
        List<Object> list1 = new ArrayList<>();
        list1.add("a");
        list1.add("b");
        System.out.println(1);
        System.in.read();

        list1 = null;
        System.out.println(2);
        System.in.read();
        System.out.println("end...");
    }
}
~~~



命令行窗口运行：

> jmap -dump:format=b,live,file=1.bin 21384

`-dump: `当前堆中运行时的一些状态，转储为一个文件；

`b`: 二进制的形式

`live`: 先进行一次垃圾回收，然后获取哪些还存活的对象

`file=` :文件名字以及存放位置



> 下载一个   可视化的堆分析工具 
>
>  https://www.eclipse.org/downloads/download.php?file=/mat/1.13.0/rcp/MemoryAnalyzer-1.13.0.20220615-win32.win32.x86_64.zip



注意：

该MAT工具，仅支持jdk11及以上







使用：

---

![image-20220731224705621](http://fgcy-pic.zhamao.ml/image-20220731224705621.png)

---



---

![image-20220731224813682](http://fgcy-pic.zhamao.ml/image-20220731224813682.png)

---







GC Root有哪些

1 虚拟机运行过程中一些核心的类：

---

![image-20220731224952505](http://fgcy-pic.zhamao.ml/image-20220731224952505.png)

---



2 运行本地方法时，所引用的一些java对象

---

![image-20220731225233890](http://fgcy-pic.zhamao.ml/image-20220731225233890.png)

---



3 活动线程中，局部变量所引用的对象

---

![image-20220731230238168](http://fgcy-pic.zhamao.ml/image-20220731230238168.png)

---

![image-20220731230324179](http://fgcy-pic.zhamao.ml/image-20220731230324179.png)

---







4 被加了锁的对象

---

![image-20220731225509137](http://fgcy-pic.zhamao.ml/image-20220731225509137.png)

---





### 1.3 四种引用

1. 强引用 只有所有 GC Roots 对象都不通过【强引用】引用该对象，该对象才能被垃圾回收

2. 软引用（SoftReference） 仅有软引用  引用  该对象时，**在垃圾回收后**，`内存仍不足时`会再次出发垃圾回收，回收软引用 对象   **可以**   配合引用队列来释放软引用自身 

3. 弱引用（WeakReference） 仅有弱引用引用该对象时，**在垃圾回收时**，无论内存`是否充足`，**都会回收弱引用对象**    **可以**   配合引用队列来释放弱引用自身 

4. 虚引用（PhantomReference） **必须**配合引用队列使用，主要配合 ByteBuffer 使用，被引用对象回收时，会将虚引用入队， 由 Reference Handler 线程调用虚引用相关方法释放直接内存 

5. 终结器引用（FinalReference） 无需手动编码，但其内部配合引用队列使用，在垃圾回收时，终结器引用入队**（被引用对象 暂时没有被回收）**，再由 Finalizer 线程 (优先级很低，被执行的概率低)  通过   终结器引用   找到   被引用对象  并   调用它的 finalize 方法  ，**第二次 GC 时**才能回收被引用对象

   所以不推荐使用Object的finalize()方法，回收对象



---

![image-20220731230625601](http://fgcy-pic.zhamao.ml/image-20220731230625601.png)

---





演示  强  弱比较

~~~java
package cn.itcast.jvm.t2;

import java.io.IOException;
import java.lang.ref.SoftReference;
import java.util.ArrayList;
import java.util.List;

/**
 * 演示强引用
 * -Xmx20m -XX:+PrintGCDetails -verbose:gc
 */
public class Demo2_3 {

    private static final int _4MB = 4 * 1024 * 1024;


    public static void main(String[] args) throws IOException {
        hard();
    }

    public static void hard() throws IOException {
        List<byte[]> list = new ArrayList<>();
        for (int i = 0; i < 5; i++) {
            list.add(new byte[_4MB]);
        }

        System.in.read();
    }

}
~~~

~~~
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at cn.itcast.jvm.t2.Demo2_3.hard(Demo2_3.java:28)
	at cn.itcast.jvm.t2.Demo2_3.main(Demo2_3.java:19)
~~~



~~~java
package cn.itcast.jvm.t2;

import java.io.IOException;
import java.lang.ref.SoftReference;
import java.util.ArrayList;
import java.util.List;

/**
 * 演示软引用
 * -Xmx20m -XX:+PrintGCDetails -verbose:gc
 */
public class Demo2_3 {

    private static final int _4MB = 4 * 1024 * 1024;


    public static void main(String[] args) throws IOException {
        soft();
    }



    public static void soft() {
        // list --> SoftReference --> byte[]

        List<SoftReference<byte[]>> list = new ArrayList<>();
        for (int i = 0; i < 5; i++) {
            SoftReference<byte[]> ref = new SoftReference<>(new byte[_4MB]);
            System.out.println(ref.get());
            list.add(ref);
            System.out.println(list.size());

        }
        System.out.println("循环结束：" + list.size());
        for (SoftReference<byte[]> ref : list) {
            System.out.println(ref.get());
        }
    }
}

~~~

~~~
[B@7f31245a
1
[B@6d6f6e28
2
[B@135fbaa4
3
[GC (Allocation Failure) [PSYoungGen: 2138K->512K(6144K)] 14426K->13063K(19968K), 0.0006212 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[B@45ee12a7
4
[GC (Allocation Failure) --[PSYoungGen: 4720K->4720K(6144K)] 17271K->17335K(19968K), 0.0004683 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Ergonomics) [PSYoungGen: 4720K->4533K(6144K)] [ParOldGen: 12615K->12542K(13824K)] 17335K->17075K(19968K), [Metaspace: 3242K->3242K(1056768K)], 0.0034264 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) --[PSYoungGen: 4533K->4533K(6144K)] 17075K->17130K(19968K), 0.0003634 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Allocation Failure) [PSYoungGen: 4533K->0K(6144K)] [ParOldGen: 12597K->673K(8704K)] 17130K->673K(14848K), [Metaspace: 3242K->3242K(1056768K)], 0.0034938 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[B@330bedb4
5
循环结束：5
null   //空的软引用对象
null
null
null
[B@330bedb4
Heap
 PSYoungGen      total 6144K, used 4264K [0x00000000ff980000, 0x0000000100000000, 0x0000000100000000)
  eden space 5632K, 75% used [0x00000000ff980000,0x00000000ffdaa2b8,0x00000000fff00000)
  from space 512K, 0% used [0x00000000fff00000,0x00000000fff00000,0x00000000fff80000)
  to   space 512K, 0% used [0x00000000fff80000,0x00000000fff80000,0x0000000100000000)
 ParOldGen       total 8704K, used 673K [0x00000000fec00000, 0x00000000ff480000, 0x00000000ff980000)
  object space 8704K, 7% used [0x00000000fec00000,0x00000000feca8500,0x00000000ff480000)
 Metaspace       used 3250K, capacity 4500K, committed 4864K, reserved 1056768K
  class space    used 352K, capacity 388K, committed 512K, reserved 1048576K
~~~





配合引用队列完成为空的软引用的清理

~~~java
package cn.itcast.jvm.t2;

import java.lang.ref.Reference;
import java.lang.ref.ReferenceQueue;
import java.lang.ref.SoftReference;
import java.util.ArrayList;
import java.util.List;

/**
 * 演示软引用, 配合引用队列
 * -Xmx20m -XX:+PrintGCDetails -verbose:gc
 */
public class Demo2_4 {
    private static final int _4MB = 4 * 1024 * 1024;

    public static void main(String[] args) {
        List<SoftReference<byte[]>> list = new ArrayList<>();

        // 引用队列
        ReferenceQueue<byte[]> queue = new ReferenceQueue<>();

        for (int i = 0; i < 5; i++) {
            // 关联了引用队列， 当软引用所关联的 byte[]被回收时，软引用自己会加入到 queue 中去
            SoftReference<byte[]> ref = new SoftReference<>(new byte[_4MB], queue);
            System.out.println(ref.get());
            list.add(ref);
            System.out.println(list.size());
        }

        // 从队列中获取无用的 软引用对象，并移除
        Reference<? extends byte[]> poll = queue.poll();
        while (poll != null) {
            list.remove(poll);
            poll = queue.poll();
        }

        System.out.println("===========================");
        for (SoftReference<byte[]> reference : list) {
            System.out.println(reference.get());
        }

    }
}

~~~

~~~
[B@7f31245a
1
[B@6d6f6e28
2
[B@135fbaa4
3
[GC (Allocation Failure) [PSYoungGen: 2138K->504K(6144K)] 14426K->13055K(19968K), 0.0010074 secs] [Times: user=0.05 sys=0.02, real=0.01 secs] 
[B@45ee12a7
4
[GC (Allocation Failure) --[PSYoungGen: 4712K->4712K(6144K)] 17263K->17370K(19968K), 0.0011186 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Ergonomics) [PSYoungGen: 4712K->4519K(6144K)] [ParOldGen: 12658K->12554K(13824K)] 17370K->17074K(19968K), [Metaspace: 3241K->3241K(1056768K)], 0.0038776 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) --[PSYoungGen: 4519K->4519K(6144K)] 17074K->17170K(19968K), 0.0005025 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Allocation Failure) [PSYoungGen: 4519K->0K(6144K)] [ParOldGen: 12650K->671K(8704K)] 17170K->671K(14848K), [Metaspace: 3241K->3241K(1056768K)], 0.0056360 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
[B@330bedb4
5
===========================
[B@330bedb4
Heap
 PSYoungGen      total 6144K, used 4377K [0x00000000ff980000, 0x0000000100000000, 0x0000000100000000)
  eden space 5632K, 77% used [0x00000000ff980000,0x00000000ffdc6548,0x00000000fff00000)
  from space 512K, 0% used [0x00000000fff00000,0x00000000fff00000,0x00000000fff80000)
  to   space 512K, 0% used [0x00000000fff80000,0x00000000fff80000,0x0000000100000000)
 ParOldGen       total 8704K, used 671K [0x00000000fec00000, 0x00000000ff480000, 0x00000000ff980000)
  object space 8704K, 7% used [0x00000000fec00000,0x00000000feca7f98,0x00000000ff480000)
 Metaspace       used 3248K, capacity 4500K, committed 4864K, reserved 1056768K
  class space    used 352K, capacity 388K, committed 512K, reserved 1048576K
~~~







弱引用示例：

~~~java
package cn.itcast.jvm.t2;

import java.lang.ref.Reference;
import java.lang.ref.ReferenceQueue;
import java.lang.ref.SoftReference;
import java.lang.ref.WeakReference;
import java.util.ArrayList;
import java.util.List;

/**
 * 演示弱引用
 * -Xmx20m -XX:+PrintGCDetails -verbose:gc
 */
public class Demo2_5 {
    private static final int _4MB = 4 * 1024 * 1024;

    public static void main(String[] args) {
        //  list --> WeakReference --> byte[]
        List<WeakReference<byte[]>> list = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            WeakReference<byte[]> ref = new WeakReference<>(new byte[_4MB]);
            list.add(ref);
            for (WeakReference<byte[]> w : list) {
                System.out.print(w.get()+" ");
            }
            System.out.println();
        }
        System.out.println("循环结束：" + list.size());
    }
}
~~~

~~~
[B@7f31245a 
[B@7f31245a [B@6d6f6e28 
[B@7f31245a [B@6d6f6e28 [B@135fbaa4 
[GC (Allocation Failure) [PSYoungGen: 2138K->488K(6144K)] 14426K->13082K(19968K), 0.0008480 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[B@7f31245a [B@6d6f6e28 [B@135fbaa4 [B@45ee12a7 
[GC (Allocation Failure) [PSYoungGen: 4696K->488K(6144K)] 17290K->13202K(19968K), 0.0004814 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[B@7f31245a [B@6d6f6e28 [B@135fbaa4 null [B@330bedb4 
[GC (Allocation Failure) [PSYoungGen: 4696K->504K(6144K)] 17410K->13282K(19968K), 0.0004436 secs] [Times: user=0.02 sys=0.00, real=0.01 secs] 
[B@7f31245a [B@6d6f6e28 [B@135fbaa4 null null [B@2503dbd3 
[GC (Allocation Failure) [PSYoungGen: 4711K->504K(6144K)] 17489K->13330K(19968K), 0.0003351 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[B@7f31245a [B@6d6f6e28 [B@135fbaa4 null null null [B@4b67cf4d 
[GC (Allocation Failure) [PSYoungGen: 4710K->504K(6144K)] 17537K->13370K(19968K), 0.0002823 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[B@7f31245a [B@6d6f6e28 [B@135fbaa4 null null null null [B@7ea987ac 
[GC (Allocation Failure) [PSYoungGen: 4710K->504K(5120K)] 17576K->13406K(18944K), 0.0002916 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[B@7f31245a [B@6d6f6e28 [B@135fbaa4 null null null null null [B@12a3a380 
[GC (Allocation Failure) [PSYoungGen: 4690K->64K(5632K)] 17592K->13398K(19456K), 0.0003781 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Ergonomics) [PSYoungGen: 64K->0K(5632K)] [ParOldGen: 13334K->681K(8704K)] 13398K->681K(14336K), [Metaspace: 3242K->3242K(1056768K)], 0.0051074 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
null null null null null null null null null [B@29453f44 
循环结束：10
Heap
 PSYoungGen      total 5632K, used 4370K [0x00000000ff980000, 0x0000000100000000, 0x0000000100000000)
  eden space 4608K, 94% used [0x00000000ff980000,0x00000000ffdc4ba0,0x00000000ffe00000)
  from space 1024K, 0% used [0x00000000ffe00000,0x00000000ffe00000,0x00000000fff00000)
  to   space 1024K, 0% used [0x00000000fff00000,0x00000000fff00000,0x0000000100000000)
 ParOldGen       total 8704K, used 681K [0x00000000fec00000, 0x00000000ff480000, 0x00000000ff980000)
  object space 8704K, 7% used [0x00000000fec00000,0x00000000fecaa4a8,0x00000000ff480000)
 Metaspace       used 3249K, capacity 4500K, committed 4864K, reserved 1056768K
  class space    used 352K, capacity 388K, committed 512K, reserved 1048576K
~~~

没有回收掉的原因是因为，对象进入了老年代；minorGC触发的是新生代的的回收；

所以与之前说的只要触发GC，弱引用所关联的对象就会被回收掉不冲突；

进行垃圾回收时虽然会将软引用，弱引用所关联的对象释放掉（软引用是发生gc且内存还是不够才干掉关联对象），但想要干掉软引用、弱引用则需要配合队列使用；





## 2.0 垃圾回收算法



### 2.1 标记清除

定义： Mark Sweep 

特点：

速度较快 会造成内存碎片



---

![image-20220801232427927](http://fgcy-pic.zhamao.ml/image-20220801232427927.png)

---

分两步，标记和清除

标记是将没有被根对象做出标记

清除是将被标记的对象所占用的内存的起始地址记录到空闲区，而不是将该内存空间置零（这是一个误区）



缺点就是：会造成内存碎片







### 2.2 标记整理



定义：Mark Compact

速度慢 没有内存碎片



---

![image-20220801233836932](http://fgcy-pic.zhamao.ml/image-20220801233836932.png)

---

速度慢的原因：涉及内存区块的拷贝与移动  以及  所有关联此地址的引用的改变





### 2.3 复制

定义：Copy 

不会有内存碎片 需要占用双倍内存空间



---

![image-20220801233458168](http://fgcy-pic.zhamao.ml/image-20220801233458168.png)

---

进行垃圾回收，找到有GCroot直接或间接引用的对象，将其从FROM区迁移到TO区；最后逻辑交换FROM区和TO区



## 3.0 分代垃圾回收



---

![image-20220801235201849](http://fgcy-pic.zhamao.ml/image-20220801235201849.png)

---



1、对象首先分配在伊甸园区域 

2、新生代空间不足时，触发 minor gc，伊甸园和 from 存活的对象使用 copy 复制到 to 中，存活的 对象年龄加 1并且交换 from to

3、minor gc 会引发 stop the world，暂停其它用户的线程，等垃圾回收结束（原因：需要将幸存区FROM中的对象 复制到 幸存区TO中，多线程运行时用以出现问题）用户线程才恢复运行 当对象寿命超过阈值时，会晋升至老年代，最大寿命是15（4bit），当空间不足时，即使幸存区中的对象没有超过15，也有可能晋升到老年代

4、当老年代空间不足，会先尝试触发 minor gc，如果之后空间仍不足，那么触发 full gc，STW的时 间更长（老年代一般使用标记清除或者是标记整理）

当发现触发了一次FullGC后还是空间不足，报错：OutOfMemory



## 4.0 垃圾回收器







## 5.0 垃圾回收调优























































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































