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

![image-20220807230659165](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807230659165.png)

---



# 二、学习JVM有什么用

JVM是一种规范

各种的虚拟机才是才是具体的实现

如：

oracle    的`hotspot`

eclipse    的     `Open J9`

---

![image-20220807230745506](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807230745506.png)

---



# **三、学习路线**



---

![image-20220807230805350](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807230805350.png)

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

![image-20220807230827013](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807230827013.png)

---









PC的作用：

---

![image-20220807230859677](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807230859677.png)

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

![image-20220807230926551](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807230926551.png)

---







**栈帧与栈的关系：**

---

![image-20220807231005986](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807231005986.png)



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

![image-20220807231033103](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807231033103.png)

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

![image-20220807231052698](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807231052698.png)

---



栈帧过大导致栈内存溢出（这种情况出现的可能性较小）

---

![image-20220807231118982](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807231118982.png)

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

![image-20220807231252882](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807231252882.png)

---



---

![image-20220807231313227](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807231313227.png)

---



重新运行该方法

---

![image-20220807231332112](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807231332112.png)

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

![image-20220807231359023](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807231359023.png)

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

![image-20220807231425559](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807231425559.png)

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

![image-20220807231448187](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807231448187.png)

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

![image-20220807231509013](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807231509013.png)

---



运行17次后**OOM**`（内存溢出错误）`  【之前是1g】

---

![image-20220807231538365](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807231538365.png)

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

![image-20220807231559110](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807231559110.png)

---



---

![image-20220807231613467](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807231613467.png)

---



---

![image-20220807231628336](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807231628336.png)

---





3、等待控制台输出

~~~
1....
2....
3....
~~~



---

![image-20220807231659765](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807231659765.png)

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

![image-20220807231720443](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807231720443.png)

---





3、使用jconsole工具，执行一次gc；

---

![image-20220807231735587](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807231735587.png)

---



4、使用jconsole工具，查看新生代和老年代的内容

---

![image-20220807231801585](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807231801585.png)

---



发现：只是释放了新生代的的堆内存，老年代的没有被释放；



使用 `jvisualvm` 命令进行进一步分析

步骤：

1、输入命令 `jvisualvm`



---

![image-20220807231820879](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807231820879.png)

---









---

![image-20220807231834399](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807231834399.png)

---





---

![image-20220807231852456](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807231852456.png)

---







---

![image-20220807231909826](C:/Users/fgcy/AppData/Roaming/Typora/typora-user-images/image-20220807231909826.png)

---







---

![image-20220807231934732](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807231934732.png)

---



---

![image-20220807231949437](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807231949437.png)

---







---

![image-20220807232007594](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807232007594.png)

---



**小结：**

这里的Student引用一直被ArrayList持有，所以有被引用的对象不会被GC释放，内存也没有被释放





## 5.0 方法区



---

![image-20220807232028513](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807232028513.png)

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

![image-20220807232047685](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807232047685.png)

---

注意：

永久代的内存回收效率很低；当发生FullGC时，永久代才会进行垃圾回收；

FullGC需要等待到，老年代的空间不足时才会触发；触发时机晚了；



JDK1.8及其后续版本，方法区位于物理内存中；而不是位于堆中，但此时`字符串常量池`位于堆中；

![image-20220807232106759](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807232106759.png)

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

![image-20220807232129808](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807232129808.png)

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

![image-20220807232149242](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807232149242.png)

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

![image-20220807232224773](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807232224773.png)

---



---

![image-20220807232254313](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807232254313.png)

---



---

![image-20220807232318905](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807232318905.png)

---



---

![image-20220807232501850](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807232501850.png)

---



---

![image-20220807232518441](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807232518441.png)

---

![image-20220807232540846](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807232540846.png)

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

![image-20220731011028661](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220731011028661.png)

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

![image-20220807232615442](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807232615442.png)

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

![image-20220807232632245](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807232632245.png)

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

![image-20220807232646414](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807232646414.png)

---



例子3：

设置JVM参数

> -XX:StringTableSize=20 0000

---

![image-20220807232658253](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807232658253.png)

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

![image-20220807232711018](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807232711018.png)

---



3、一次回车（读取十次文件）

---

![image-20220731142800069](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220731142800069.png)

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

![image-20220731143040775](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220731143040775.png)

---



3、一次回车（读取十次文件）

---

![image-20220731143254742](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220731143254742.png)

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

![image-20220731151152900](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220731151152900.png)

---

java代码无法直接操作底层 文件IO，只能通过调用本地方法间接实现；

CPU分为两种状态：用户态，和内核态

内存也分为两种：JVM中的堆内存和系统内存

java无法直接获取到系统内存中的数据，只能先将系统内存中的数据复制一份到堆内存中，再操作堆内存中的数据

此时，读取一份文件需要复制两次









**使用了DirectBuffer**

---

![image-20220731151853223](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220731151853223.png)

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

![image-20220731160632622](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220731160632622.png)

---





演示垃圾回收ByteBuffer对象

1、还没运行程序

---

![image-20220731161433975](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220731161433975.png)

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

![image-20220731161757573](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220731161757573.png)

---



3、控制台敲回车

---

![image-20220731161837777](http://fgcy-pic.zhamao.ml/image-20220731161837777.png)

---



---

![image-20220731161938357](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220731161938357.png)

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

![image-20220731163440228](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220731163440228.png)

---



运行程序

---

![image-20220731163541095](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220731163541095.png)

---



敲回车，释放内存

---

![image-20220731163641246](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220731163641246.png)

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

![image-20220731172951500](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220731172951500.png)

---





---

![image-20220731172924325](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220731172924325.png)

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

![image-20220731174331705](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220731174331705.png)

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

![image-20220731224705621](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220731224705621.png)

---



---

![image-20220731224813682](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220731224813682.png)

---







GC Root有哪些

1 虚拟机运行过程中一些核心的类：

---

![image-20220731224952505](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220731224952505.png)

---



2 运行本地方法时，所引用的一些java对象

---

![image-20220731225233890](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220731225233890.png)

---



3 活动线程中，局部变量所引用的对象

---

![image-20220731230238168](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220731230238168.png)

---

![image-20220731230324179](http://fgcy-pic.zhamao.ml/image-20220731230324179.png)

---







4 被加了锁的对象

---

![image-20220731225509137](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220731225509137.png)

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

![image-20220801232427927](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220801232427927.png)

---

分两步，标记和清除

标记是将没有被根对象做出标记

清除是将被标记的对象所占用的内存的起始地址记录到空闲区，而不是将该内存空间置零（这是一个误区）



缺点就是：会造成内存碎片







### 2.2 标记整理



定义：Mark Compact

速度慢 没有内存碎片



---

![image-20220801233836932](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220801233836932.png)

---

速度慢的原因：涉及内存区块的拷贝与移动  以及  所有关联此地址的引用的改变





### 2.3 复制

定义：Copy 

不会有内存碎片 需要占用双倍内存空间



---

![image-20220801233458168](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220801233458168.png)

---

进行垃圾回收，找到有GCroot直接或间接引用的对象，将其从FROM区迁移到TO区；最后逻辑交换FROM区和TO区



## 3.0 分代垃圾回收



---

![image-20220801235201849](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220801235201849.png)

---



1、对象首先分配在伊甸园区域 

2、新生代空间不足时，触发 minor gc，伊甸园和 from 存活的对象使用 copy 复制到 to 中，存活的 对象年龄加 1并且交换 from to

3、minor gc 会引发 stop the world，暂停其它用户的线程，等垃圾回收结束（原因：需要将幸存区FROM中的对象 复制到 幸存区TO中，多线程运行时用以出现问题）用户线程才恢复运行 当对象寿命超过阈值时，会晋升至老年代，最大寿命是15（4bit），当空间不足时，即使幸存区中的对象没有超过15，也有可能晋升到老年代

4、当老年代空间不足，会先尝试触发 minor gc，如果之后空间仍不足，那么触发 full gc，STW的时 间更长（老年代一般使用标记清除或者是标记整理）

当发现触发了一次FullGC后还是空间不足，报错：OutOfMemory



### 3.1 相关VM参数

---

![image-20220807233333410](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807233333410.png)

---





### 3.2 使用参数



例子一：

~~~java
package cn.itcast.jvm.t2;

import java.util.ArrayList;

/**
 *  演示内存的分配策略
 */
public class Demo2_1 {
    private static final int _512KB = 512 * 1024;
    private static final int _1MB = 1024 * 1024;
    private static final int _6MB = 6 * 1024 * 1024;
    private static final int _7MB = 7 * 1024 * 1024;
    private static final int _8MB = 8 * 1024 * 1024;

    // -Xms20M -Xmx20M -Xmn10M -XX:+UseSerialGC -XX:+PrintGCDetails -verbose:gc -XX:-ScavengeBeforeFullGC
    public static void main(String[] args) throws InterruptedException {

    }
}

~~~

~~~
Heap【堆内存】
 def new generation   total 9216K, used 2322K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)   【新生代】
  eden space 8192K,  28% used [0x00000000fec00000, 0x00000000fee44af0, 0x00000000ff400000)   【伊甸园】
  from space 1024K,   0% used [0x00000000ff400000, 0x00000000ff400000, 0x00000000ff500000) 	  【from区】
  to   space 1024K,   0% used [0x00000000ff500000, 0x00000000ff500000, 0x00000000ff600000)		【to区】
tenured generation   total 10240K, used 0K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)  【老年代】
the space 10240K,   0% used [0x00000000ff600000, 0x00000000ff600000, 0x00000000ff600200, 0x0000000100000000) 
Metaspace       used 3244K, capacity 4496K, committed 4864K, reserved 1056768K 【元空间，这货不属于堆内存，只是为了方便查看信息】
  class space    used 352K, capacity 388K, committed 512K, reserved 1048576K
~~~

`新生代的最大值与最小值设置为10M`，所以  伊甸园 + form区 + to区=10M

新生代total（总共9M），原因：幸存区to必须为空，所以空出1M的内存







例子二：

~~~java
package cn.itcast.jvm.t2;

import java.sql.Array;
import java.util.ArrayList;

/**
 * 演示内存的分配策略
 */
public class Demo2_1 {
    private static final int _512KB = 512 * 1024;
    private static final int _1MB = 1024 * 1024;
    private static final int _6MB = 6 * 1024 * 1024;
    private static final int _7MB = 7 * 1024 * 1024;
    private static final int _8MB = 8 * 1024 * 1024;

    // -Xms20M -Xmx20M -Xmn10M -XX:+UseSerialGC -XX:+PrintGCDetails -verbose:gc -XX:-ScavengeBeforeFullGC
    public static void main(String[] args) throws InterruptedException {
        ArrayList<Object> list = new ArrayList<>();
        list.add(new byte[_7MB]);
    }
}

~~~

---

![image-20220807233410647](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807233410647.png)

---

发生一次MinorGC将伊甸园中的没用的对象回收掉，将伊甸园中一些还有GCRoot引用的对象放到from区中；为伊甸园区腾出空间；







例子三：

~~~java
package cn.itcast.jvm.t2;

import java.sql.Array;
import java.util.ArrayList;

/**
 * 演示内存的分配策略
 */
public class Demo2_1 {
    private static final int _512KB = 512 * 1024;
    private static final int _1MB = 1024 * 1024;
    private static final int _6MB = 6 * 1024 * 1024;
    private static final int _7MB = 7 * 1024 * 1024;
    private static final int _8MB = 8 * 1024 * 1024;

    // -Xms20M -Xmx20M -Xmn10M -XX:+UseSerialGC -XX:+PrintGCDetails -verbose:gc -XX:-ScavengeBeforeFullGC
    public static void main(String[] args) throws InterruptedException {
        ArrayList<Object> list = new ArrayList<>();
        list.add(new byte[_7MB]);//还剩：8192k *0.1 =819.2k
        list.add(new byte[_512KB]);//几乎占满伊甸园区 （也只是触发一次GC）
    }
}
~~~

---

[![image-20220807233446770](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807233446770.png)

---

此时伊甸园区还有：8192*0.04=327K，会发生垃圾回收；







例子四：

~~~java
package cn.itcast.jvm.t2;

import java.sql.Array;
import java.util.ArrayList;

/**
 * 演示内存的分配策略
 */
public class Demo2_1 {
    private static final int _512KB = 512 * 1024;
    private static final int _1MB = 1024 * 1024;
    private static final int _6MB = 6 * 1024 * 1024;
    private static final int _7MB = 7 * 1024 * 1024;
    private static final int _8MB = 8 * 1024 * 1024;

    // -Xms20M -Xmx20M -Xmn10M -XX:+UseSerialGC -XX:+PrintGCDetails -verbose:gc -XX:-ScavengeBeforeFullGC
    public static void main(String[] args) throws InterruptedException {
        ArrayList<Object> list = new ArrayList<>();
        list.add(new byte[_7MB]);//还剩：8192k *0.1 =819.2k
        list.add(new byte[_512KB]);//8192*0.04=327K
        list.add(new byte[_512KB]);//尝试添加这个对象，失败 因为伊甸园内存不足，触发一次gc
    }
}
~~~

---

![image-20220807233508410](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807233508410.png)

---









例子五：

大对象，在新生代内存不足以容纳，而老年代可以容纳时，直接晋升到老年代（不经过GC）

伊甸园总共就8M左右，还占用了28%，肯定放不下一个8M的对象

~~~java
package cn.itcast.jvm.t2;

import java.sql.Array;
import java.util.ArrayList;

/**
 * 演示内存的分配策略
 */
public class Demo2_1 {
    private static final int _512KB = 512 * 1024;
    private static final int _1MB = 1024 * 1024;
    private static final int _6MB = 6 * 1024 * 1024;
    private static final int _7MB = 7 * 1024 * 1024;
    private static final int _8MB = 8 * 1024 * 1024;

    // -Xms20M -Xmx20M -Xmn10M -XX:+UseSerialGC -XX:+PrintGCDetails -verbose:gc -XX:-ScavengeBeforeFullGC
    public static void main(String[] args) throws InterruptedException {
        ArrayList<Object> list = new ArrayList<>();
        list.add(new byte[_8MB]);//还剩：8192k *0.1 =819.2k
    }
}
~~~

---

![image-20220807233532177](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807233532177.png)

---





例子六：当新生代和老年代的空间不足时，报错OutOfMemory

~~~java
package cn.itcast.jvm.t2;

import java.sql.Array;
import java.util.ArrayList;

/**
 * 演示内存的分配策略
 */
public class Demo2_1 {
    private static final int _512KB = 512 * 1024;
    private static final int _1MB = 1024 * 1024;
    private static final int _6MB = 6 * 1024 * 1024;
    private static final int _7MB = 7 * 1024 * 1024;
    private static final int _8MB = 8 * 1024 * 1024;

    // -Xms20M -Xmx20M -Xmn10M -XX:+UseSerialGC -XX:+PrintGCDetails -verbose:gc -XX:-ScavengeBeforeFullGC
    public static void main(String[] args) throws InterruptedException {
        ArrayList<Object> list = new ArrayList<>();
        list.add(new byte[_8MB]);//还剩：8192k *0.1 =819.2k
        list.add(new byte[_8MB]);//新生代和老年代都容纳不了
    }
}
~~~

---

![image-20220807233557478](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807233557478.png)

---

做了两次GC自救，还是没能救活；外抛异常；









例子七：

因为线程导致的OOM，不会使得整个进程结束；

~~~java
package cn.itcast.jvm.t2;

import java.sql.Array;
import java.util.ArrayList;

/**
 * 演示内存的分配策略
 */
public class Demo2_1 {
    private static final int _512KB = 512 * 1024;
    private static final int _1MB = 1024 * 1024;
    private static final int _6MB = 6 * 1024 * 1024;
    private static final int _7MB = 7 * 1024 * 1024;
    private static final int _8MB = 8 * 1024 * 1024;

    // -Xms20M -Xmx20M -Xmn10M -XX:+UseSerialGC -XX:+PrintGCDetails -verbose:gc -XX:-ScavengeBeforeFullGC
    public static void main(String[] args) throws InterruptedException {
    //开一个子线程 进行oom
        new Thread(() -> {
            ArrayList<byte[]> list = new ArrayList<>();
            list.add(new byte[_8MB]);
            list.add(new byte[_8MB]);
        }).start();


        //看是否会使得整个JVM挂掉
        System.out.println("sleep....");
        Thread.sleep(1800L);
    }
}
~~~

---

![image-20220807142117396](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807142117396.png)

---

主线程还能正常结束

## 4.0 垃圾回收器

1、`串行 `    **单线程**    **堆内存较小**，适合个人电脑

2、`吞吐量优先` 多线程 堆内存较大，多核 cpu 让**单位时间内**，**STW 的时间最短**  每次垃圾回收的时间不一定是最短的，但单位时间内，STW的时间一定是最短的；	   这样就称吞吐量高 

3、` 响应时间优先` 多线程 堆内存较大，多核 cpu 尽可能让 `单次` STW 的时间最短 



### 4.1 串行

`-XX:+UseSerialGC = Serial + SerialOld`

`Serial` 是工作在新生代  【使用复制算法】

`SerialOld` 是工作在老年代 【使用标记整理算法】



---

![image-20220807141612251](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807141612251.png)

---

多个线程在安全点停下，因为发生GC是有可能会改变对象地址；这样会出现问题； 	

这个串行的垃圾回收器是单线程执行的；等这个垃圾回收线程完成后，才开始其他用户线程的执行；





### 4.2 吞吐量优先

`-XX:+UseParallelGC` ~ `-XX:+UseParallelOldGC `  **jdk1.8默认开启这两个参数**

`-XX:+UseParallelGC:` **并行的**  垃圾回收器，工作在新生代，采用复制算法

`-XX:+UseParallelOldGC:`  **并行的**  垃圾回收器，工作在老年代，采用标记整理算法

上面两个参数只要有其中一个开启，另一个也会开启；



` -XX:ParallelGCThreads=n` :用于控制parallelGC的线程数量 【默认开启的线程数量与CPU的核数相同】

`-XX:+UseAdaptiveSizePolicy`   : 开启自适应的大小调整策略 【调整伊甸园和幸存区的比例，调整堆得大小，调整晋升阈值】



`-XX:GCTimeRatio=ratio` : 用于设置吞吐率目标 1 / (1+radio)，当达不到这个目标，就会开始调整值，一般是增大堆的大小；【一般radio=19 】

` -XX:MaxGCPauseMillis=ms` : 最大暂停毫秒数，默认200毫秒 

堆空间变大了，单位时间内进行GC的次数会少；吞吐率就可以上去

堆空间变大了，单次GC的时间就会变小

上面两个目标是冲突的，我们要注意平衡；

---

![image-20220807141512078](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807141512078.png)

---





### 4.3 响应时间优先

`-XX : +UseConcMarkSweepGC` ~ `-XX : +UseParNewGC` ~      `Se ri al Ol d`

`-XX : +UseConcMarkSweepGC`  【CMS】 这个是老年代的垃圾回收器，与之配合的是  `-XX : +UseParNewGC` 工作在新生代的垃圾回收器，但是当发生并发失败时，CMS老年代的垃圾回收器就会退化为串行的垃圾回收器

并发的垃圾回收器：在垃圾回收期间，用户线程还可以工作，双方抢占CPU资源；【只有某些时刻可以并发执行，大部分不行，但还是可以减少Stop The World的时间】

上一个介绍的，并行的垃圾回收器：在垃圾回收期间，不允许任何用户线程运行，但有多个线程同时执行垃圾回收

注意：

并发是一个CPU某个时刻只干一件事；并行是多个CPU某一个时刻同时在干一件事；



`-XX : Paral l el GCTh reads=n  ` ~  ` -XX : ConcGCTh reads=th reads `

`-XX : Paral l el GCTh reads=n ` : 并行的垃圾回收线程数【一般与CPU的核数相同】 

` -XX : ConcGCTh reads=th reads  `: 并发的垃圾回收线程数【一般设置为CPU核数的四分之一】



`-XX : CMSIni ti ati ngOccupancyFracti on=pe rcent ` 执行CMS垃圾回收的内存占比

​	因为CMS回收器，有浮动垃圾；如果像其他的垃圾回收器一样等到放不下新的对象时才去清理，就会浪费空间





`-XX : +CMSScavengeBefo reRemark`  该参数的作用是，在重新标记的时候，先清理一次新生代



CMS垃圾回收器的工作示意图：

----

![](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/Snipaste_2022-08-06_22-55-06.png)

---

1、等待多个线程进入安全点，STW开启垃圾回收线程，进行初始标记【这个过程很快，因为只是标记】

2、初始标记完成后，用户线程就可以恢复运行，同时垃圾回收线程可以并发运行【并发标记】；

3、并发标记后，开始STW，进行重新标记；因为用户线程运行期间，可能产生一些新的对象，会对标记结果产生结果；

4、重新标记后，用户线程又可以继续运行，此时垃圾回收线程可以并发清理； 

​		只有在初始标记和重新标记时STW，响应时间较短



给出一个概念：浮动垃圾,在并发清理时，由用户线程产生的垃圾；这些垃圾只能等待下一次垃圾回收时清理





注意：

CMS垃圾回收器，采用标记清除算法；会产生大量内存碎片；因为内存碎片过多导致无法为用户线程分配内存，造成并发失败；此时回退化为SerialOld垃圾回收器；此时，机会耗费很多时间





### 4.4 G1

定义：Garbage First 

2004 论文发布 

2009 JDK 6u14 体验 

2012 JDK 7u4 官方支持 

2017 JDK 9 默认 【废除CMS垃圾回收器】（本身是一款并发的垃圾回收器）



适用场景 ：

同时注重吞吐量（Throughput）和低延迟（Low latency），默认的暂停目标是 200 ms 

超大堆内存，会将堆划分为多个**大小相等**的` Region `

整体上是 标记+整理 算法，两个区域之间是 复制 算法 



相关 JVM 参数

 -XX:+UseG1GC 

-XX:G1HeapRegionSize=size 

-XX:MaxGCPauseMillis=time



**（1）垃圾回收阶段**

---

![image-20220807112008878](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807112008878.png)

---









**（2) Young Collection** 

​	

---

![image-20220807112216078](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807112216078.png)

---

G1会将都内存划分为多个大小相等的区域，其中一些区域为伊甸园区【E】，有大小限制

当伊甸园区的内存被占满时，会触发一次垃圾回收；

进行新生代的垃圾回收时会STW







---

![image-20220807112849366](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807112849366.png)

---

将新生代幸存下来的对象，通过复制算法，放到幸存区；











---

![image-20220807113203408](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807113203408.png)

---

再次触发一次垃圾回收，将幸存区中符合晋升条件的对象，复制到老年代

将幸存区中不满足晋升条件的对象，复制到 另一个幸存区中；

将伊甸园中，有GCRoot对象引用的对象复制到幸存区中；











**（3) Young Collection + CM  【新生代的垃圾回收 + 并发标记】**

​		在` Young GC `时会进行 GC Root 的初始标记 【上一步，新生代的垃圾回收，STW时进行，并不会占用并发标记的时间】

​		**老年代占 用**  堆空间比例达到阈值时，进行并发标记（不会 STW），由下面的 JVM 参数决定

​		` -XX:InitiatingHeapOccupancyPercent=percent `（默认45%）

----

![image-20220807114608726](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807114608726.png)

---

当老年代【O】占用达到整个堆空间的45%时，就会进行并发标记





**（4) Mixed Collection** 

​	会对 E、S、O 进行全面垃圾回收 

​	最终标记（Remark）会 STW ，原因：并发标记时，用户线程也在工作，此时可能会产生新的垃圾【没有GCRoot对象应用的对象】

​	拷贝存活（Evacuation）会 STW 



`-XX:MaxGCPauseMillis=ms`  之前提过的设置G1垃圾回收器最大暂停时间，决定响应时间的参数，与吞吐量互斥



----

![image-20220807115458981](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807115458981.png)

---

对 E、S、O 进行全面垃圾回收 ，将伊甸园的幸存对象复制到幸存区，将幸存区的未能晋升的对象复制到幸存区【非自己】；

老年代的垃圾回收有点特别：因为要兼顾最大暂停时间，所以G1不会对所有的老年代区域进行垃圾回收；仅会挑选一些回收价值较高的区域进行垃圾回收；

将有GCRoot对象引用的对象复制到老年代【非自己】



G1：被称为Garbage first，原因：在混合收集阶段，对老年代的垃圾回收，会首先挑选回收价值最高的区域进行垃圾回收；

注意：在老年代的区域还不是特别多时，就算对全部的老年代进行垃圾回收，也不会影响最大暂停时间时，会对所有的老年代区域进行垃圾回收；



（5) Full GC 

SerialGC 

新生代内存不足发生的垃圾收集 - minor gc

 老年代内存不足发生的垃圾收集 - full gc 



ParallelGC 

新生代内存不足发生的垃圾收集 - minor gc 

老年代内存不足发生的垃圾收集 - full gc 



CMS 

新生代内存不足发生的垃圾收集 - minor gc 

老年代内存不足

​	并发收集失败后，会退化为串行收集，此时为FullGC



G1 

新生代内存不足发生的垃圾收集 - minor gc 

老年代内存不足

​	当老年代的内存使用占整个堆内存的45%以上时，就会进行并发标记，然后进行混合收集；此时当垃圾收集速度 大于 并发时垃圾产生的速度

  【这样不会发生 FullGC】,当垃圾收集速度 小于 并发时垃圾产生的速度，此时并发收集失败，会退化为FullGC，STW地时间会特别长







**（6) Young Collection 跨代引用** 

​		新生代回收的跨代引用（老年代引用新生代）问题



---

![image-20220807133853983](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807133853983.png)

---

回顾垃圾回收过程：

首先是寻找根对象，然后使用可达性分析算法找到存活对象；然后使用三种算法之中的一种或多种进行垃圾回收；



问题：对新生代进行可达性分析时，要先找根对象；而老年代中有大量的根对象，那么是否是遍历老年代；

​		G1中是不需要的；它提出了一个叫卡表的概念（将作为老年代的region进行细分，分为一个个的卡，一个512K），在新生代中有一个` Remembered Set`的概		念，它用于记录有哪些脏卡；

​		G1使用 `post-write barrier`（写屏障） ,来更新脏卡；当老年代中的某个卡中的对象，引用了新生代中的对象时，写屏障会将该卡标记为脏卡；并记录到` Remembered Set` 中

​		

卡表 与 Remembered Set 

更新脏卡的过程是一个异步的过程：

 	在引用变更时通过  ` post-write barrier`（写屏障） + `dirty card queue` （脏卡队列）
 	
 	`  concurrent refinement threads` 更新 `Remembered Set`

---

![image-20220807133918471](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807133918471.png)

---





**（7) Remark** 

​		`pre-write barrier` +` satb_mark_queue`



`pre-write barrier` : 写屏障，当对象引用发生变化时，会将该对象，放入一个队列（**satb_mark_queue**），并标记为灰色，等待以后进行重新标记

---

![image-20220807140534030](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807140534030.png)

---

黑色代表：已经处理完成

灰色代表：正在处理

白色代表：尚未处理





（8) JDK 8u20 字符串去重 

​	优点：节省大量内存 

​	缺点：略微多占用了 cpu 时间，新生代回收时间略微增加



开启参数： ` -XX:+UseStringDeduplication`

​	JDK1.8中，字符串的底层其实是char数组

~~~java
String s1 = new String("hello"); // char[]{'h','e','l','l','o'}
String s2 = new String("hello"); // char[]{'h','e','l','l','o'}
~~~

将所有新分配的字符串放入一个队列 

当新生代回收时，G1并发检查是否有字符串重复 

如果它们值一样，让它们  **引用同一个 char[]** 



注意，与 String.intern() 不一样

​	String.intern() 关注的是字符串对象

​	而字符串去重关注的是 char[] 

​	在 JVM 内部，使用了不同的字符串表





**（9) JDK 8u40 并发标记类卸载** 

​	所有对象都经过并发标记后，就能知道哪些类不再被使用

​	当 一个类加载器 的 **所有类都不再使用**，则 **卸 载它所加载的所有类** 

​	注意：

​		JDK的三个类加载器，是不会进行类卸载的；只有自定义的类加载器才会进行类卸载；



`	XX:+ClassUnloadingWithConcurrentMark` 默认启用







（10) JDK 8u60 回收巨型对象 

​		一个对象大于 region 的一半时，称之为`巨型对象 `



​		G1 不会对巨型对象进行拷贝 

​		回收时被优先考虑 

​		G1 会跟踪老年代所有 incoming 引用，这样老年代 incoming 引用为0 的巨型对象就可以在新生 代垃圾回收时处理掉【即：当老年代中的所有卡不再对某个		新生代中的巨型对象进行引用时，就可以回收掉】



---

![image-20220807151352836](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807151352836.png)

----





---

![image-20220807151250734](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807151250734.png)

---





**（11) JDK 9 并发标记起始时间的调整** 

​		并发标记必须在堆空间占满前完成，否则退化为 FullGC 【不是串行】



​		JDK 9 之前需要使用	` -XX:InitiatingHeapOccupancyPercent `  45%

​		JDK 9 可以动态调整 

​		`	-XX:InitiatingHeapOccupancyPercent `用来设置初始值 

​			进行数据采样并动态调整 

​			总会添加一个安全的空档空间



## 5.0 垃圾回收调优



预备知识 

掌握 GC 相关的 VM 参数，会基本的空间调整 掌握相关工具 

明白一点：调优跟应用、环境有关，没有放之四海而皆准的法则



​	参考Oracle的JVM调优指南：

> [Java Platform, Standard Edition HotSpot Virtual Machine Garbage Collection Tuning Guide, Release 12 (oracle.com)](https://docs.oracle.com/en/java/javase/12/gctuning/)





查看虚拟机运行参数：

```
"D:\sofeware\Idea\JDK1.8\bin\java" -XX:+PrintFlagsFinal -version | findstr "GC"
```



~~~
  uintx AdaptiveSizeMajorGCDecayTimeScale         = 10                                  {product}
    uintx AutoGCSelectPauseMillis                   = 5000                                {product}
     bool BindGCTaskThreadsToCPUs                   = false                               {product}
    uintx CMSFullGCsBeforeCompaction                = 0                                   {product}
    uintx ConcGCThreads                             = 0                                   {product}
     bool DisableExplicitGC                         = false                               {product}
     bool ExplicitGCInvokesConcurrent               = false                               {product}
     bool ExplicitGCInvokesConcurrentAndUnloadsClasses  = false                               {product}
    uintx G1MixedGCCountTarget                      = 8                                   {product}
    uintx GCDrainStackTargetSize                    = 64                                  {product}
    uintx GCHeapFreeLimit                           = 2                                   {product}
    uintx GCLockerEdenExpansionPercent              = 5                                   {product}
     bool GCLockerInvokesConcurrent                 = false                               {product}
    uintx GCLogFileSize                             = 8192                                {product}
    uintx GCPauseIntervalMillis                     = 0                                   {product}
    uintx GCTaskTimeStampEntries                    = 200                                 {product}
    uintx GCTimeLimit                               = 98                                  {product}
    uintx GCTimeRatio                               = 99                                  {product}
     bool HeapDumpAfterFullGC                       = false                               {manageable}
     bool HeapDumpBeforeFullGC                      = false                               {manageable}
    uintx HeapSizePerGCThread                       = 87241520                            {product}
    uintx MaxGCMinorPauseMillis                     = 4294967295                          {product}
    uintx MaxGCPauseMillis                          = 4294967295                          {product}
    uintx NumberOfGCLogFiles                        = 0                                   {product}
     intx ParGCArrayScanChunk                       = 50                                  {product}
    uintx ParGCDesiredObjsFromOverflowList          = 20                                  {product}
     bool ParGCTrimOverflow                         = true                                {product}
     bool ParGCUseLocalOverflow                     = false                               {product}
    uintx ParallelGCBufferWastePct                  = 10                                  {product}
    uintx ParallelGCThreads                         = 13                                  {product}
     bool ParallelGCVerbose                         = false                               {product}
     bool PrintClassHistogramAfterFullGC            = false                               {manageable}
     bool PrintClassHistogramBeforeFullGC           = false                               {manageable}
     bool PrintGC                                   = false                               {manageable}
     bool PrintGCApplicationConcurrentTime          = false                               {product}
     bool PrintGCApplicationStoppedTime             = false                               {product}
     bool PrintGCCause                              = true                                {product}
     bool PrintGCDateStamps                         = false                               {manageable}
     bool PrintGCDetails                            = false                               {manageable}
     bool PrintGCID                                 = false                               {manageable}
     bool PrintGCTaskTimeStamps                     = false                               {product}
     bool PrintGCTimeStamps                         = false                               {manageable}
     bool PrintHeapAtGC                             = false                               {product rw}
     bool PrintHeapAtGCExtended                     = false                               {product rw}
     bool PrintJNIGCStalls                          = false                               {product}
     bool PrintParallelOldGCPhaseTimes              = false                               {product}
     bool PrintReferenceGC                          = false                               {product}
     bool ScavengeBeforeFullGC                      = true                                {product}
     bool TraceDynamicGCThreads                     = false                               {product}
     bool TraceParallelOldGCTasks                   = false                               {product}
     bool UseAdaptiveGCBoundary                     = false                               {product}
     bool UseAdaptiveSizeDecayMajorGCCost           = true                                {product}
     bool UseAdaptiveSizePolicyWithSystemGC         = false                               {product}
     bool UseAutoGCSelectPolicy                     = false                               {product}
     bool UseConcMarkSweepGC                        = false                               {product}
     bool UseDynamicNumberOfGCThreads               = false                               {product}
     bool UseG1GC                                   = false                               {product}
     bool UseGCLogFileRotation                      = false                               {product}
     bool UseGCOverheadLimit                        = true                                {product}
     bool UseGCTaskAffinity                         = false                               {product}
     bool UseMaximumCompactionOnSystemGC            = true                                {product}
     bool UseParNewGC                               = false                               {product}
     bool UseParallelGC                            := true                                {product}
     bool UseParallelOldGC                          = true                                {product}
     bool UseSerialGC                               = false                               {product}
java version "1.8.0_201"
Java(TM) SE Runtime Environment (build 1.8.0_201-b09)
Java HotSpot(TM) 64-Bit Server VM (build 25.201-b09, mixed mode)
~~~





## 5.1 调优领域

内存 



锁竞争 



cpu 占用 



io









## 5.2 确定目标

【低延迟】还是【高吞吐量】，选择合适的回收器 

CMS，G1，ZGC 

ParallelGC

以上是HotSpot虚拟机的垃圾回收器



Zing 是一个虚拟机，它的垃圾回收器也是兼顾高吞吐量和高响应











## 5.3 最快的 GC 

答案是不发生



GC 查看 FullGC 前后的内存占用，考虑下面几个问题 ：



数据是不是太多？

​	resultSet = statement.executeQuery("select * from table") 



数据表示是否太臃肿？

​	对象图

​	对象大小 1最小的对象是16B； Integer一个对象头16B，int类型的数据要占用4B，再加上对齐就是24B 



是否存在内存泄漏？ 

​	static Map map = new HashMap<>();定义了一个静态的Map集合，不断地往里面放数据

​	软【发生一次GC，发现堆内存还是不足，将软引用关联的对象所占用的内存释放掉】

​    弱 【只要发生GC，就将堆内存中弱引用所关联的对象所占用的内存释放掉】

​	第三方缓存实现







## 5.4 新生代调优 

1、新生代的特点 所有的 new 操作的内存分配非常廉价 

​	这里提出一个： `thread-local allocation buffer`(TLAB) 本地线程缓冲区的概念；为了避免不同线程，想要使用同一块线程而提出的；

​	为不同线程在伊甸园区分配一块区域，它们会在各自的缓冲区中创建对象；避免并发问题



2、死亡对象的回收代价是零 

​	伊甸园和from区中的幸存对象，会被复制到to区中；然后直接将伊甸园区和之前的from区的内存地址标记为空闲；



3、大部分对象用过即死 



4、Minor GC 的时间远远低于 Full GC

​	原因：新生代中存活的对象本来就少，且使用的是复制算法，STW的时间很短；





新生代的调优一般可采用 调大堆内存空间；

越大越好吗？



GC is performed in this region more often than in other regions. If the size for the young generation is too small, then a lot of minor garbage collections are performed. If the size is too large, then only full garbage collections are performed, which can take a long time to complete. Oracle recommends that you keep the size for the young generation greater than 25% and less than 50% of the overall heap size.

~~~
GC在这个区域比在其他区域执行得更频繁。如果新生代的大小太小，那么就会执行许多小的垃圾收集
如果太大，则只执行完整的垃圾收集（FullGC）【因为新生代大了，所以老年代就会变小】，这可能需要很长时间才能完成
Oracle建议您将新生代的大小保持在总堆大小的25%以上、50%以下
~~~

但是，总的来说还是希望 新生代的内存尽可能大

JVM参数：` -Xmn  ` 为新生代代设置堆的初始和最大大小(以字节为单位)



**新生代配置原则：**

1、新生代能容纳所有【并发量 * (请求与响应所创建的对象所需的内存)】为合适；因为这样可以不触发，或者是较少第触发垃圾回收；

2、幸存区大到能保留【当前活跃对象+需要晋升对象】 

​		当前活跃对象：当前不能回收，但发生GC后一定会回收

​		需要晋升对象：当前不能回收，发生GC后也不能回收，只是没达到晋升阈值；

​		当幸存区较小，JVM会动态调整幸存区的晋升阈值；使得一些对象会提前晋升到老年代中；这样会使得当老年代的内存不足触发FullGC时，该对象才会被释放		掉，这样就会大大延长了垃圾的生存时间；



3、晋升阈值配置得当，让长时间存活对象尽快晋升



` -XX:MaxTenuringThreshold=threshold` 设置最大晋升阈值

` -XX:+PrintTenuringDistribution` 打印晋升的详细信息



~~~
Desired survivor size 48286924 bytes, new threshold 10 (max 10)
- age 1: 28992024 bytes, 28992024 total   【年龄为一的：刚逃过一次MinorGC】
- age 2: 1366864 bytes, 30358888 total
- age 3: 1425912 bytes, 31784800 total
...

~~~





## 5.5 老年代调优 

**以 CMS 为例** 

1、CMS 的老年代内存越大越好 

​		CMS是一个工作于老年代的垃圾回收器，是可以并发执行的，响应时间较短的垃圾回收器；用户线程与垃圾回收线程并发运行，由用户线程在次期间产生的垃		圾【浮动垃圾】，如果不能即时清理，就会并发失败；CMS会退化为 Se ri al Ol d（串行的垃圾回收器）；所以，老年代的空间越大越好，为浮动垃圾提供额		外的内存，避免并发失败；而发生FullGC

先尝试不做调优，如果没有 Full GC 那么已经...，否则先尝试调优新生代 

观察发生 Full GC 时老年代内存占用，将老年代内存预设调大 1/4 ~ 1/3 

​	`-XX:CMSInitiatingOccupancyFraction=percent` 空间占用达到老年代百分之多少时，使用CMS进行垃圾回收【一般预留20%的空间给浮动垃圾就可以了】



## 5.6 案例 

案例1 Full GC 和 Minor GC频繁 

​	做法：调大新生代的大小，新生代的变大使得幸存区也跟着变大，晋升阈值提升；把存活时间较短的对象留在新生代；将存活时间较长的对象放到老年代



案例2 请求高峰期发生 Full GC，单次暂停时间特别长 （CMS） 

​	做法：因为CMS最耗时间的是在重新标记阶段；所以选择在进行重新标记前先进行一次MinorGC，以减少新生代的对象；减少重新标记时，对新生代对象的处	理；

​	`-XX : +CMSScavengeBeforeRemark`



案例3 老年代充裕情况下，发生 Full GC （CMS jdk1.7）

​		老年代充裕情况下:说明没有因为内存不足【碎片】导致并发失败，考虑是不是是JDK1.7版本；

​		因为JDK1.7的永久代放在堆内存中，永久代的空间不足也会导致FullGC







# 六、类加载与字节码技术



---

![image-20220807175240103](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807175240103.png)

---







## 1. 类文件结构 

一个简单的 HelloWorld.java

~~~java
package cn.itcast.jvm.t5;
// HelloWorld 示例
public class HelloWorld {
public static void main(String[] args) {
System.out.println("hello world");
  }
}
~~~

编译：执行 `javac -parameters -d . HellowWorld.java`



编译为 HelloWorld.class 后是这个样子的：

~~~
0000000 ca fe ba be 00 00 00 34 00 23 0a 00 06 00 15 09
0000020 00 16 00 17 08 00 18 0a 00 19 00 1a 07 00 1b 07
0000040 00 1c 01 00 06 3c 69 6e 69 74 3e 01 00 03 28 29
0000060 56 01 00 04 43 6f 64 65 01 00 0f 4c 69 6e 65 4e
0000100 75 6d 62 65 72 54 61 62 6c 65 01 00 12 4c 6f 63
0000120 61 6c 56 61 72 69 61 62 6c 65 54 61 62 6c 65 01
0000140 00 04 74 68 69 73 01 00 1d 4c 63 6e 2f 69 74 63
0000160 61 73 74 2f 6a 76 6d 2f 74 35 2f 48 65 6c 6c 6f
0000200 57 6f 72 6c 64 3b 01 00 04 6d 61 69 6e 01 00 16
0000220 28 5b 4c 6a 61 76 61 2f 6c 61 6e 67 2f 53 74 72
0000240 69 6e 67 3b 29 56 01 00 04 61 72 67 73 01 00 13
0000260 5b 4c 6a 61 76 61 2f 6c 61 6e 67 2f 53 74 72 69
0000300 6e 67 3b 01 00 10 4d 65 74 68 6f 64 50 61 72 61
0000320 6d 65 74 65 72 73 01 00 0a 53 6f 75 72 63 65 46
0000340 69 6c 65 01 00 0f 48 65 6c 6c 6f 57 6f 72 6c 64
0000360 2e 6a 61 76 61 0c 00 07 00 08 07 00 1d 0c 00 1e
0000400 00 1f 01 00 0b 68 65 6c 6c 6f 20 77 6f 72 6c 64
0000420 07 00 20 0c 00 21 00 22 01 00 1b 63 6e 2f 69 74
0000440 63 61 73 74 2f 6a 76 6d 2f 74 35 2f 48 65 6c 6c
0000460 6f 57 6f 72 6c 64 01 00 10 6a 61 76 61 2f 6c 61
0000500 6e 67 2f 4f 62 6a 65 63 74 01 00 10 6a 61 76 61
0000520 2f 6c 61 6e 67 2f 53 79 73 74 65 6d 01 00 03 6f
0000540 75 74 01 00 15 4c 6a 61 76 61 2f 69 6f 2f 50 72
0000560 69 6e 74 53 74 72 65 61 6d 3b 01 00 13 6a 61 76
0000600 61 2f 69 6f 2f 50 72 69 6e 74 53 74 72 65 61 6d
0000620 01 00 07 70 72 69 6e 74 6c 6e 01 00 15 28 4c 6a
0000640 61 76 61 2f 6c 61 6e 67 2f 53 74 72 69 6e 67 3b
0000660 29 56 00 21 00 05 00 06 00 00 00 00 00 02 00 01
0000700 00 07 00 08 00 01 00 09 00 00 00 2f 00 01 00 01
0000720 00 00 00 05 2a b7 00 01 b1 00 00 00 02 00 0a 00
0000740 00 00 06 00 01 00 00 00 04 00 0b 00 00 00 0c 00
0000760 01 00 00 00 05 00 0c 00 0d 00 00 00 09 00 0e 00
0001000 0f 00 02 00 09 00 00 00 37 00 02 00 01 00 00 00
0001020 09 b2 00 02 12 03 b6 00 04 b1 00 00 00 02 00 0a
0001040 00 00 00 0a 00 02 00 00 00 06 00 08 00 07 00 0b
0001060 00 00 00 0c 00 01 00 00 00 09 00 10 00 11 00 00
0001100 00 12 00 00 00 05 01 00 10 00 00 00 01 00 13 00
0001120 00 00 02 00 14
~~~



根据 JVM 规范，类文件结构如下:

~~~
ClassFile {
        u4 magic;         【u4代表的是字节数，此处是前四个字节 魔数】
        u2 minor_version;	【小版本号】
        u2 major_version;	【主版本号】
        u2 constant_pool_count; 	【常量池相关】
        cp_info constant_pool[constant_pool_count-1];	【常量池相关】
        u2 access_flags;	【访问修饰】
        u2 this_class;		【自己的包名类名信息】
        u2 super_class;		【父类信息】
        u2 interfaces_count;【接口信息】
        u2 interfaces[interfaces_count];  【接口信息】
        u2 fields_count;	【成员变量信息】
        field_info fields[fields_count]; 【成员变量信息】
        u2 methods_count;   【方法信息】
        method_info methods[methods_count]; 【方法信息】
        u2 attributes_count;  【类的附加属性信息】
        attribute_info attributes[attributes_count]; 【类的附加属性信息】
}
~~~



### 1.1 魔数

0~3 字节【0 1 2 3】，用这四个字节表示它是一个 class 	类型的文件 

不同类型的文件由不同的魔数信息

前四个字节 咖啡卑鄙 标识该文件是一个class类型的文件

0000000 <u>**ca fe ba be**</u> 00 00 00 34 00 23 0a 00 06 00 15 09





### 1.2 版本 

第4~7 字节，表示类的版本 00 34（52） 表示是 Java 8 【内部有对应关系】

0000000 ca fe ba be **00 00 00 34** 00 23 0a 00 06 00 15 09



### 1.3 常量池

---

![image-20220807210200551](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807210200551.png)

---



8~9 字节两个字节，表示常量池长度，00 23 （35） 表示常量池有 #1~#34项，注意 #0 项不计入，也没有值 

0000000 ca fe ba be 00 00 00 34 **00 23** 0a 00 06 00 15 09 



第#1项 0a （十进制：10）表示一个 Method 信息【方法引用信息】，00 06（十进制：6） 和 00 15（十进制：21） 表示它引用了常量池中 #6 和 #21 项来获得 这个方法的【所属类】和【方法名】 

0000000 ca fe ba be 00 00 00 34 00 23 **0a 00 06 00 15** 09

~~~
找#6 找#28 java/lang/Object

找#21 找#7和#8 <init>()V
~~~



第#2项 09 表示一个 Field 信息，00 16（22）和 00 17（23） 表示它引用了常量池中 #22 和 # 23 项 来获得这个成员变量的【所属类】和【成员变量名】 0000000 ca fe ba be 00 00 00 34 00 23 0a 00 06 00 15 **09** 

0000020 **00 16 00 17** 08 00 18 0a 00 19 00 1a 07 00 1b 07 



第#3项 08 表示一个字符串常量名称，00 18（24）表示它引用了常量池中 #24 项 

0000020 00 16 00 17 **08 00 18** 0a 00 19 00 1a 07 00 1b 07 



第#4项 0a 表示一个 Method 信息，00 19（25） 和 00 1a（26） 表示它引用了常量池中 #25 和 #26 项来获得这个方法的【所属类】和【方法名】 

0000020 00 16 00 17 08 00 18 **0a 00 19 00 1a** 07 00 1b 07 



第#5项 07 表示一个 Class 信息

00 1b（27） 表示它引用了常量池中 #27 项 0000020 00 16 00 17 08 00 18 0a 00 19 00 1a **07 00 1b** 07 



第#6项 07 表示一个 Class 信息，00 1c（28） 表示它引用了常量池中 #28 项 

0000020 00 16 00 17 08 00 18 0a 00 19 00 1a 07 00 1b **07** 

0000040 **00 1c** 01 00 06 3c 69 6e 69 74 3e 01 00 03 28 29 



第#7项 01 表示一个 utf8 串，00 06 表示长度，3c 69 6e 69 74 3e 是【  \<init>】 

0000040 00 1c **01 00 06 3c 69 6e 69 74 3e** 01 00 03 28 29



第#8项 01 表示一个 utf8 串，00 03 表示长度，28 29 56 是【()V】其实就是表示无参、无返回值 

0000040 00 1c 01 00 06 3c 69 6e 69 74 3e **01 00 03 28 29** 

0000060 **56** 01 00 04 43 6f 64 65 01 00 0f 4c 69 6e 65 4e 



第#9项 01 表示一个 utf8 串，00 04 表示长度，43 6f 64 65 是【Code】 

0000060 56 **01 00 04 43 6f 64 65** 01 00 0f 4c 69 6e 65 4e 



第#10项 01 表示一个 utf8 串，00 0f（15） 表示长度，4c 69 6e 65 4e 75 6d 62 65 72 54 61 62 6c 65 是【LineNumberTable】 

0000060 56 01 00 04 43 6f 64 65 **01 00 0f 4c 69 6e 65 4e** 

0000100 **75 6d 62 65 72 54 61 62 6c 65** 01 00 12 4c 6f 63 



第#11项 01 表示一个 utf8 串，00 12（18） 表示长度，4c 6f 63 61 6c 56 61 72 69 61 62 6c 65 54 61 62 6c 65是【LocalVariableTable】 

0000100 75 6d 62 65 72 54 61 62 6c 65 **01 00 12 4c 6f 63** 

0000120 **61 6c 56 61 72 69 61 62 6c 65 54 61 62 6c 65** 01



 第#12项 01 表示一个 utf8 串，00 04 表示长度，74 68 69 73 是【this】 

0000120 61 6c 56 61 72 69 61 62 6c 65 54 61 62 6c 65 **01** 

0000140 00 **04 74 68 69 73** 01 00 1d 4c 63 6e 2f 69 74 63 



第#13项 01 表示一个 utf8 串，00 1d（29） 表示长度，是【Lcn/itcast/jvm/t5/HelloWorld;】 

0000140 00 04 74 68 69 73 **01 00 1d 4c 63 6e 2f 69 74 63** 

0000160 **61 73 74 2f 6a 76 6d 2f 74 35 2f 48 65 6c 6c 6f** 

0000200 **57 6f 72 6c 64 3b** 01 00 04 6d 61 69 6e 01 00 16 



第#14项 01 表示一个 utf8 串，00 04 表示长度，74 68 69 73 是【main】 

0000200 57 6f 72 6c 64 3b **01 00 04 6d 61 69 6e** 01 00 16 



第#15项 01 表示一个 utf8 串，00 16（22） 表示长度，是【([Ljava/lang/String;)V】其实就是参数为 字符串数组，无返回值 

0000200 57 6f 72 6c 64 3b 01 00 04 6d 61 69 6e **01 00 16** 

0000220 **28 5b 4c 6a 61 76 61 2f 6c 61 6e 67 2f 53 74 72** 

0000240 **69 6e 67 3b 29 56** 01 00 04 61 72 67 73 01 00 13 



第#16项 01 表示一个 utf8 串，00 04 表示长度，是【args】 

0000240 69 6e 67 3b 29 56 **01 00 04 61 72 67 73** 01 00 13 



第#17项 01 表示一个 utf8 串，00 13（19） 表示长度，是【[Ljava/lang/String;】 

0000240 69 6e 67 3b 29 56 01 00 04 61 72 67 73 **01 00 13** 

0000260 **5b 4c 6a 61 76 61 2f 6c 61 6e 67 2f 53 74 72 69** 

0000300 **6e 67 3b** 01 00 10 4d 65 74 68 6f 64 50 61 72 61

 

第#18项 01 表示一个 utf8 串，00 10（16） 表示长度，是【MethodParameters】 

0000300 6e 67 3b **01 00 10 4d 65 74 68 6f 64 50 61 72 61** 

0000320 **6d 65 74 65 72 73** 01 00 0a 53 6f 75 72 63 65 46 



第#19项 01 表示一个 utf8 串，00 0a（10） 表示长度，是【SourceFile】 

0000320 6d 65 74 65 72 73 **01 00 0a 53 6f 75 72 63 65 46** 

0000340 **69 6c 65** 01 00 0f 48 65 6c 6c 6f 57 6f 72 6c 64 



第#20项 01 表示一个 utf8 串，00 0f（15） 表示长度，是【HelloWorld.java】 

0000340 69 6c 65 **01 00 0f 48 65 6c 6c 6f 57 6f 72 6c 64** 

0000360 **2e 6a 61 76 61** 0c 00 07 00 08 07 00 1d 0c 00 1e



第#21项 0c 表示一个 【名+类型】，00 07 00 08 引用了常量池中 #7 #8 两项 

0000360 2e 6a 61 76 61 **0c 00 07 00 08** 07 00 1d 0c 00 1e 



第#22项 07 表示一个 Class 信息，00 1d（29） 引用了常量池中 #29 项 

0000360 2e 6a 61 76 61 0c 00 07 00 08 **07 00 1d** 0c 00 1e



 第#23项 0c 表示一个 【名+类型】，00 1e（30） 00 1f （31）引用了常量池中 #30 #31 两项 

0000360 2e 6a 61 76 61 0c 00 07 00 08 07 00 1d **0c 00 1e** 

0000400 **00 1f** 01 00 0b 68 65 6c 6c 6f 20 77 6f 72 6c 64



第#24项 01 表示一个 utf8 串，00 0f（12） 表示长度，是【hello world】 

0000400 00 1f **01 00 0b 68 65 6c 6c 6f 20 77 6f 72 6c 64** 



第#25项 07 表示一个 Class 信息，00 20（32） 引用了常量池中 #32 项 

0000420 **07 00 20** 0c 00 21 00 22 01 00 1b 63 6e 2f 69 74 



第#26项 0c 表示一个 【名+类型】，00 21（33） 00 22（34）引用了常量池中 #33 #34 两项 

0000420 07 00 20 **0c 00 21 00 22** 01 00 1b 63 6e 2f 69 74 



第#27项 01 表示一个 utf8 串，00 1b（27） 表示长度，是【cn/itcast/jvm/t5/HelloWorld】 

0000420 07 00 20 0c 00 21 00 22 **01 00 1b 63 6e 2f 69 74** 

0000440 **63 61 73 74 2f 6a 76 6d 2f 74 35 2f 48 65 6c 6c** 

0000460 **6f 57 6f 72 6c 64** 01 00 10 6a 61 76 61 2f 6c 61 



第#28项 01 表示一个 utf8 串，00 10（16） 表示长度，是【java/lang/Object】 

0000460 6f 57 6f 72 6c 64 **01 00 10 6a 61 76 61 2f 6c 61**

0000500 **6e 67 2f 4f 62 6a 65 63 74** 01 00 10 6a 61 76 61 



第#29项 01 表示一个 utf8 串，00 10（16） 表示长度，是【java/lang/System】 

0000500 6e 67 2f 4f 62 6a 65 63 74 **01 00 10 6a 61 76 61** 

0000520 **2f 6c 61 6e 67 2f 53 79 73 74 65 6d** 01 00 03 6f 



第#30项 01 表示一个 utf8 串，00 03 表示长度，是【out】 

0000520 2f 6c 61 6e 67 2f 53 79 73 74 65 6d **01 00 03 6f** 

0000540 **75 74** 01 00 15 4c 6a 61 76 61 2f 69 6f 2f 50 72 



第#31项 01 表示一个 utf8 串，00 15（21） 表示长度，是【Ljava/io/PrintStream;】 

0000540 75 74 **01 00 15 4c 6a 61 76 61 2f 69 6f 2f 50 72** 

0000560 **69 6e 74 53 74 72 65 61 6d 3b** 01 00 13 6a 61 76 



第#32项 01 表示一个 utf8 串，00 13（19） 表示长度，是【java/io/PrintStream】 

0000560 69 6e 74 53 74 72 65 61 6d 3b **01 00 13 6a 61 76** 

0000600 **61 2f 69 6f 2f 50 72 69 6e 74 53 74 72 65 61 6d** 



第#33项 01 表示一个 utf8 串，00 07 表示长度，是【println】 

0000620 **01 00 07 70 72 69 6e 74 6c 6e** 01 00 15 28 4c 6a 



第#34项 01 表示一个 utf8 串，00 15（21） 表示长度，是【(Ljava/lang/String;)V】

 0000620 01 00 07 70 72 69 6e 74 6c 6e **01 00 15 28 4c 6a** 

0000640 **61 76 61 2f 6c 61 6e 67 2f 53 74 72 69 6e 67 3b** 

0000660 **29 56** 00 21 00 05 00 06 00 00 00 00 00 02 00 01



### 1.4 访问标识与继承信息

21 表示该 class 是一个类，公共的 

0000660 29 56 **00 21** 00 05 00 06 00 00 00 00 00 02 00 01



05 表示根据常量池中 #5 找到本类全限定名 

0000660 29 56 00 21 **00 05** 00 06 00 00 00 00 00 02 00 01 



06 表示根据常量池中 #6 找到父类全限定名 

0000660 29 56 00 21 00 05 **00 06** 00 00 00 00 00 02 00 01 



表示接口的数量，本类为 0 0000660 29 56 00 21 00 05 00 06 **00 00** 00 00 00 02 00 01

---

![image-20220807220952550](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807220952550.png)

---





### 1.5 Field 信息 

表示成员变量数量，本类为 0 

0000660 29 56 00 21 00 05 00 06 00 00 **00 00** 00 02 00 01



---

![image-20220807221351274](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220807221351274.png)

---

L：是引用类型

[：是一维数组







### 1.6 Method 信息 

表示方法数量，本类为 2 

0000660 29 56 00 21 00 05 00 06 00 00 00 00 **00 02** 00 01



一个`方法`由 `访问修饰符`，`名称`，`参数描述`，`方法属性数量`，`方法属性`组成 

红色代表访问修饰符（本类中是 public）

蓝色代表引用了常量池 #07 项作为方法名称 

绿色代表引用了常量池 #08 项作为方法参数描述

黄色代表方法属性数量，本方法是 1 

红色代表方法属性

00 09 表示引用了常量池 #09 项，发现是【Code】属性 

00 00 00 2f 表示此属性的长度是 47 

00 01 表示【操作数栈】最大深度 

00 01 表示【局部变量表】最大槽（slot）数



00 00 00 05 表示字节码长度，本例是 5 

2a b7 00 01 b1 是字节码指令 

00 00 00 02 表示方法细节属性数量，本例是 2 

00 0a 表示引用了常量池 #10 项，发现是【LineNumberTable】属性



00 00 00 06 表示此属性的总长度，本例是 6 

00 01 表示【LineNumberTable】长度 

00 00 表示【字节码】行号 00 04 表示【java 源码】行号 

00 0b 表示引用了常量池 #11 项，发现是【LocalVariableTable】属性



00 00 00 0c 表示此属性的总长度，本例是 12 00 01 表示【LocalVariableTable】长度 

00 00 表示局部变量生命周期开始，相对于字节码的偏移量 

00 05 表示局部变量覆盖的范围长度 

00 0c 表示局部变量名称，本例引用了常量池 #12 项，是【this】 

00 0d 表示局部变量的类型，本例引用了常量池 #13 项，是 【Lcn/itcast/jvm/t5/HelloWorld;】 

00 00 表示局部变量占有的槽位（slot）编号，本例是 0



----

![image-20220808221303085](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220808221303085.png)

---









红色代表访问修饰符（本类中是 public static） 

蓝色代表引用了常量池 #14 项作为方法名称 

绿色代表引用了常量池 #15 项作为方法参数描述 

黄色代表方法属性数量，本方法是 2 红色代表方法属性（属性1）

00 09 表示引用了常量池 #09 项，发现是【Code】属性

00 00 00 37 表示此属性的长度是 55 00 02 表示【操作数栈】最大深度 

00 01 表示【局部变量表】最大槽（slot）数 

00 00 00 05 表示字节码长度，本例是 9 b2 00 02 12 03 b6 00 04 b1 是字节码指令 

00 00 00 02 表示方法细节属性数量，本例是 2 

00 0a 表示引用了常量池 #10 项，发现是【LineNumberTable】属性



00 00 00 0c 表示此属性的总长度，本例是 12 

00 01 表示【LocalVariableTable】长度 

00 00 表示局部变量生命周期开始，相对于字节码的偏移量 

00 09 表示局部变量覆盖的范围长度

00 10 表示局部变量名称，本例引用了常量池 #16 项，是【args】 

00 11 表示局部变量的类型，本例引用了常量池 #17 项，是【[Ljava/lang/String;】 

00 00 表示局部变量占有的槽位（slot）编号，本例是 0



---

![image-20220808222238237](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220808222238237.png)

---

红色代表方法属性（属性2） 

00 12 表示引用了常量池 #18 项，发现是【MethodParameters】属性 

00 00 00 05 表示此属性的总长度，本例是 5 01 参数数量 

00 10 表示引用了常量池 #16 项，是【args】 00 00 访问修饰符



---

![image-20220808222321341](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220808222321341.png)

---





### 1.7 附加属性 

00 01 表示附加属性数量 

00 13 表示引用了常量池 #19 项，即【SourceFile】

 00 00 00 02 表示此属性的长度 

00 14 表示引用了常量池 #20 项，即【HelloWorld.java】 

---

![image-20220808222406790](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220808222406790.png)

---





类的格式

参考文献 ： https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html



## 2. 字节码指令 

### 2.1 入门

 接着上一节，研究一下两组字节码指令，一个是 

`public cn.itcast.jvm.t5.HelloWorld();` 构造方法的字节码指令

~~~
2a b7 00 01 b1
~~~

1、2a => aload_0 加载 slot 0 的局部变量，即 this，做为下面的 invokespecial 构造方法调用的参数 

2、b7 => invokespecial 预备调用构造方法，哪个方法呢？ 

3、00 01 引用常量池中 #1 项，即【 Method java/lang/Object."":()V 】 

4、 b1 表示返回

~~~
this .  init()v  return
~~~





另一个是 `public static void main(java.lang.String[]); `主方法的字节码指令

~~~
b2 00 02 12 03 b6 00 04 b1
~~~

1、b2 => getstatic 用来加载静态变量，哪个静态变量呢？ 

2、00 02 引用常量池中 #2 项，即【Field java/lang/System.out:Ljava/io/PrintStream;】 

3、12 => ldc 加载参数，哪个参数呢？ 

4、03 引用常量池中 #3 项，即 【String hello world】 

5、b6 => invokevirtual 预备调用成员方法，哪个方法呢？ 

6、00 04 引用常量池中 #4 项，即【Method java/io/PrintStream.println:(Ljava/lang/String;)V】 

7、b1 表示返回

~~~
System.out   "hello world"  .  println(String)v  return
~~~



请参考:

 https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html#jvms-6.5







### 2.2 javap 工具 

自己分析类文件结构太麻烦了，Oracle 提供了 javap 工具来反编译 class 文件

~~~
javap -v HelloWorld.class
Classfile /root/HelloWorld.class
Last modified Jul 7, 2019; size 597 bytes	   【大小】
MD5 checksum 361dca1c3f4ae38644a9cd5060ac6dbc 【MD5的校验签名】
Compiled from "HelloWorld.java"
public class cn.itcast.jvm.t5.HelloWorld
minor version: 0
major version: 52	【对应JDK8】
flags: ACC_PUBLIC, ACC_SUPER	【public修饰权限】
Constant pool:
#1 = Methodref #6.#21 // java/lang/Object."<init>":()V 【 方法引用  Object的无参构造】
#2 = Fieldref #22.#23 //
java/lang/System.out:Ljava/io/PrintStream;
#3 = String #24 // hello world
#4 = Methodref #25.#26 // java/io/PrintStream.println:
(Ljava/lang/String;)V
#5 = Class #27 // cn/itcast/jvm/t5/HelloWorld
#6 = Class #28 // java/lang/Object
#7 = Utf8 <init>
#8 = Utf8 ()V
#9 = Utf8 Code
#10 = Utf8 LineNumberTable
#11 = Utf8 LocalVariableTable
#12 = Utf8 this
#13 = Utf8 Lcn/itcast/jvm/t5/HelloWorld;
#14 = Utf8 main
#15 = Utf8 ([Ljava/lang/String;)V
#16 = Utf8 args
#17 = Utf8 [Ljava/lang/String;
#18 = Utf8 MethodParameters
#19 = Utf8 SourceFile
#20 = Utf8 HelloWorld.java
#21 = NameAndType #7:#8 // "<init>":()V
#22 = Class #29 // java/lang/System
#23 = NameAndType #30:#31 // out:Ljava/io/PrintStream;
#24 = Utf8 hello world
#25 = Class #32 // java/io/PrintStream
#26 = NameAndType #33:#34 // println:(Ljava/lang/String;)V
#27 = Utf8 cn/itcast/jvm/t5/HelloWorld
#28 = Utf8 java/lang/Object
#29 = Utf8 java/lang/System
#30 = Utf8 out
#31 = Utf8 Ljava/io/PrintStream;
#32 = Utf8 java/io/PrintStream
#33 = Utf8 println
#34 = Utf8 (Ljava/lang/String;)V
{
        public cn.itcast.jvm.t5.HelloWorld();
        descriptor: ()V
        flags: ACC_PUBLIC
        Code:
        stack=1, locals=1, args_size=1
        0: aload_0
        1: invokespecial #1 // Method java/lang/Object."
        <init>":()V
        4: return
        LineNumberTable:
        line 4: 0
        LocalVariableTable:
        Start Length Slot Name Signature
        0 5 0 this Lcn/itcast/jvm/t5/HelloWorld;
        public static void main(java.lang.String[]);
        descriptor: ([Ljava/lang/String;)V
        flags: ACC_PUBLIC, ACC_STATIC
        Code:
        stack=2, locals=1, args_size=1
        0: getstatic #2 // Field
        java/lang/System.out:Ljava/io/PrintStream;
        3: ldc #3 // String hello world
        5: invokevirtual #4 // Method
        java/io/PrintStream.println:(Ljava/lang/String;)V
        8: return
        LineNumberTable:
        line 6: 0
        line 7: 8
        LocalVariableTable:
        Start Length Slot Name Signature
        0 9 0 args [Ljava/lang/String;
        MethodParameters:
        Name Flags
        args
}
~~~



### 2.3 图解方法执行流程 

1）原始 java 代码

~~~java
package cn.itcast.jvm.t3.bytecode;
/**
* 演示 字节码指令 和 操作数栈、常量池的关系
*/
public class Demo3_1 {
    public static void main(String[] args) {
    int a = 10;
    int b = Short.MAX_VALUE + 1;
    int c = a + b;
    System.out.println(c);
    }
}
~~~



2）编译后的字节码文件

~~~
javap -v Demo3_1.class
Classfile /root/Demo3_1.class
Last modified Jul 7, 2019; size 665 bytes
MD5 checksum a2c29a22421e218d4924d31e6990cfc5
Compiled from "Demo3_1.java"
public class cn.itcast.jvm.t3.bytecode.Demo3_1
minor version: 0
major version: 52
flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
#1 = Methodref #7.#26 // java/lang/Object."<init>":()V
#2 = Class #27 // java/lang/Short
#3 = Integer 32768
#4 = Fieldref #28.#29 //
java/lang/System.out:Ljava/io/PrintStream;
#5 = Methodref #30.#31 // java/io/PrintStream.println:(I)V
#6 = Class #32 // cn/itcast/jvm/t3/bytecode/Demo3_1
#7 = Class #33 // java/lang/Object
#8 = Utf8 <init>
#9 = Utf8 ()V
#10 = Utf8 Code
#11 = Utf8 LineNumberTable
#12 = Utf8 LocalVariableTable
#13 = Utf8 this#20 = Utf8 I
#21 = Utf8 b
#22 = Utf8 c
#23 = Utf8 MethodParameters
#24 = Utf8 SourceFile
#25 = Utf8 Demo3_1.java
#26 = NameAndType #8:#9 // "<init>":()V
#27 = Utf8 java/lang/Short
#28 = Class #34 // java/lang/System
#29 = NameAndType #35:#36 // out:Ljava/io/PrintStream;
#30 = Class #37 // java/io/PrintStream
#31 = NameAndType #38:#39 // println:(I)V
#32 = Utf8 cn/itcast/jvm/t3/bytecode/Demo3_1
#33 = Utf8 java/lang/Object
#34 = Utf8 java/lang/System
#35 = Utf8 out
#36 = Utf8 Ljava/io/PrintStream;
#37 = Utf8 java/io/PrintStream
#38 = Utf8 println
#39 = Utf8 (I)V
{
public cn.itcast.jvm.t3.bytecode.Demo3_1();
descriptor: ()V
flags: ACC_PUBLIC
Code:
stack=1, locals=1, args_size=1
0: aload_0
1: invokespecial #1 // Method java/lang/Object."
<init>":()V
4: return
LineNumberTable:
line 6: 0
LocalVariableTable:
Start Length Slot Name Signature
0 5 0 this Lcn/itcast/jvm/t3/bytecode/Demo3_1;
public static void main(java.lang.String[]);
descriptor: ([Ljava/lang/String;)V
flags: ACC_PUBLIC, ACC_STATIC
Code:
stack=2, locals=4, args_size=1
0: bipush 10
2: istore_1
3: ldc #3 // int 32768
5: istore_2
6: iload_1
7: iload_2
8: iadd
9: istore_3
10: getstatic #4 // Field
java/lang/System.out:Ljava/io/PrintStream;
13: iload_3
14: invokevirtual #5 // Method
java/io/PrintStream.println:(I)V
17: return
LineNumberTable:
line 8: 0
line 9: 3
line 10: 6
line 11: 10
line 12: 17
LocalVariableTable:
Start Length Slot Name Signature
0 18 0 args [Ljava/lang/String;
3 15 1 a I
6 12 2 b I
10 8 3 c I
MethodParameters:
Name Flags
args
}

#14 = Utf8 Lcn/itcast/jvm/t3/bytecode/Demo3_1;
#15 = Utf8 main
#16 = Utf8 ([Ljava/lang/String;)V
#17 = Utf8 args
#18 = Utf8 [Ljava/lang/String;
#19 = Utf8 a
#20 = Utf8 I
#21 = Utf8 b
#22 = Utf8 c
#23 = Utf8 MethodParameters
#24 = Utf8 SourceFile
#25 = Utf8 Demo3_1.java
#26 = NameAndType #8:#9 // "<init>":()V
#27 = Utf8 java/lang/Short
#28 = Class #34 // java/lang/System
#29 = NameAndType #35:#36 // out:Ljava/io/PrintStream;
#30 = Class #37 // java/io/PrintStream
#31 = NameAndType #38:#39 // println:(I)V
#32 = Utf8 cn/itcast/jvm/t3/bytecode/Demo3_1
#33 = Utf8 java/lang/Object
#34 = Utf8 java/lang/System
#35 = Utf8 out
#36 = Utf8 Ljava/io/PrintStream;
#37 = Utf8 java/io/PrintStream
#38 = Utf8 println
#39 = Utf8 (I)V
{
public cn.itcast.jvm.t3.bytecode.Demo3_1();
descriptor: ()V
flags: ACC_PUBLIC
Code:
stack=1, locals=1, args_size=1
0: aload_0
1: invokespecial #1 // Method java/lang/Object."
<init>":()V
4: return
LineNumberTable:
line 6: 0
LocalVariableTable:
Start Length Slot Name Signature
0 5 0 this Lcn/itcast/jvm/t3/bytecode/Demo3_1;
public static void main(java.lang.String[]);
descriptor: ([Ljava/lang/String;)V
flags: ACC_PUBLIC, ACC_STATIC
Code:
stack=2, locals=4, args_size=1
0: bipush 10
2: istore_1
3: ldc #3 // int 32768
5: istore_2
6: iload_1
7: iload_2
8: iadd
9: istore_3
10: getstatic #4 // Field
java/lang/System.out:Ljava/io/PrintStream;
13: iload_3
14: invokevirtual #5 // Method
java/io/PrintStream.println:(I)V
17: return
LineNumberTable:
line 8: 0
line 9: 3
line 10: 6
line 11: 10
line 12: 17
LocalVariableTable:
Start Length Slot Name Signature
0 18 0 args [Ljava/lang/String;
3 15 1 a I
6 12 2 b I
10 8 3 c I
MethodParameters:
Name Flags
args
}

~~~





3）常量池载入运行时常量池

---

![image-20220808225817493](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220808225817493.png)

---





4）方法字节码载入方法区

---

![image-20220808225845252](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220808225845252.png)

---



5）main 线程开始运行，分配栈帧内存

（stack=2，locals=4）

----

![image-20220808230231526](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220808230231526.png)

---

操作数栈的宽度是四个字节



6）执行引擎开始执行字节码

bipush 10 

将一个 byte 压入操作数栈（其长度会补齐 4 个字节），类似的指令还有 

sipush 将一个 short 压入操作数栈（其长度会补齐 4 个字节） 

ldc 将一个 int 压入操作数栈 

ldc2_w 将一个 long 压入操作数栈（分两次压入，因为 long 是 8 个字节） 

这里小的数字都是和字节码指令存在一起，超过 short 范围的数字存入了常量池



istore_1

将操作数栈顶数据弹出，存入局部变量表的 slot 1

---

![image-20220808231259983](http://fgcy-pic.zhamao.ml/image-20220808231259983.png)

---





---

![image-20220808231322044](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220808231322044.png)

---



ldc #3

从常量池加载 #3 数据到操作数栈 

注意 Short.MAX_VALUE 是 32767，所以 32768 = Short.MAX_VALUE + 1 实际是在编译期间计算 好的【算是一种编译器的优化】

---

![image-20220808231612428](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220808231612428.png)

---





istore_2

----

![image-20220808231816628](http://fgcy-pic.zhamao.ml/image-20220808231816628.png)

----



---

![image-20220808231845387](http://fgcy-pic.zhamao.ml/image-20220808231845387.png)

---





iload_1

---

![image-20220808232202160](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220808232202160.png)

---



iload_2

---

![image-20220808232233002](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220808232233002.png)

---





iadd

---

![image-20220808232304007](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220808232304007.png)

---



---

![image-20220808232323679](http://fgcy-pic.zhamao.ml/image-20220808232323679.png)

---



istore_3

---

![image-20220808232359932](http://fgcy-pic.zhamao.ml/image-20220808232359932.png)

---



---

![image-20220808232419598](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220808232419598.png)

---





getstatic #4

---

![image-20220809194216749](http://fgcy-pic.zhamao.ml/image-20220809194216749.png)

---



---

![image-20220809194232084](http://fgcy-pic.zhamao.ml/image-20220809194232084.png)

---



iload_3

---

![image-20220809194258685](http://fgcy-pic.zhamao.ml/image-20220809194258685.png)

---



---

![image-20220809194312978](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220809194312978.png)

---



invokevirtual #5 

找到常量池 #5 项 定位到方法区 java/io/PrintStream.println:(I)V 方法 

生成新的栈帧（分配 locals、stack等） 

传递参数，执行新栈帧中的字节码



---

![image-20220809194453876](http://fgcy-pic.zhamao.ml/image-20220809194453876.png)

---

执行完毕，弹出栈帧 

清除 main 操作数栈内容

----

![image-20220809194520533](http://fgcy-pic.zhamao.ml/image-20220809194520533.png)

---

return 

完成 main 方法调用，弹出 main 栈帧 

程序结束



### 2.4 练习 - 分析 i++

目的：从字节码角度分析 a++ 相关题目 

源码：

~~~java
package cn.itcast.jvm.t3.bytecode;
/**
* 从字节码角度分析 a++ 相关题目
*/
public class Demo3_2 {
public static void main(String[] args) {
    int a = 10;
    int b = a++ + ++a + a--;
    System.out.println(a);
    System.out.println(b);
    }
}
~~~



字节码：

~~~
public static void main(java.lang.String[]);
descriptor: ([Ljava/lang/String;)V
flags: (0x0009) ACC_PUBLIC, ACC_STATIC
 Code:
  stack=2, locals=3, args_size=1
    0: bipush 10
    2: istore_1
    3: iload_1
    4: iinc 1, 1
    7: iinc 1, 1
    10: iload_1
    11: iadd
    12: iload_1
    13: iinc 1, -1
    16: iadd
    17: istore_2
    18: getstatic #2 // Field
    java/lang/System.out:Ljava/io/PrintStream;
    21: iload_1
    22: invokevirtual #3 // Method
    java/io/PrintStream.println:(I)V
    25: getstatic #2 // Field
    java/lang/System.out:Ljava/io/PrintStream;
    28: iload_2
    29: invokevirtual #3 // Method
    java/io/PrintStream.println:(I)V
    32: return
    
LineNumberTable:
    line 8: 0
    line 9: 3
    line 10: 18
    line 11: 25
    line 12: 32
    LocalVariableTable:
    Start Length Slot Name Signature
    0 33 0 args [Ljava/lang/String;
    3 30 1 a I
    18 15 2 b I
~~~

分析：

注意` iinc 指令`是直接在  **局部变量 `slot `**上进行运算 

a++ 和 ++a 的区别是先执行` iload` 还是 先执行 `iinc`



---

![image-20220809203918195](http://fgcy-pic.zhamao.ml/image-20220809203918195.png)

---

![image-20220809203945867](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220809203945867.png)

----

![image-20220809204002762](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220809204002762.png)

---



----

![image-20220809204024144](http://fgcy-pic.zhamao.ml/image-20220809204024144.png)

---



----

![image-20220809204041830](http://fgcy-pic.zhamao.ml/image-20220809204041830.png)

----

![image-20220809204056743](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220809204056743.png)

---

![image-20220809204109092](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220809204109092.png)

----

![image-20220809204120623](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220809204120623.png)

---

![image-20220809204135324](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220809204135324.png)

----

![image-20220809204221238](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220809204221238.png)

----

![image-20220809204232916](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220809204232916.png)

----



### 2.5 条件判断指令

---

![image-20220809205201784](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220809205201784.png)

![image-20220809205229204](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220809205229204.png)

---

几点说明： 

byte，short，char 都会按 int 比较，因为操作数栈都是 4 字节 

goto 用来进行跳转到指定行号的字节码



例子：

源码：

~~~java
public class Demo3_3 {
public static void main(String[] args) {
    int a = 0;
        if(a == 0) {
        	a = 10;
     	} else {
        	a = 20;
        }
    }
}

~~~





字节码：

~~~
0: iconst_0   【注意：-1到5之间的数 用iconst表示】这里表示将一个常量0放到操作数栈中
1: istore_1		将操作数栈中的数放到局部变量表一号槽位中
2: iload_1		将局部变量表一号槽位中的数加载到操作数栈中
3: ifne 12		判断该数是否等于零 不等于零跳到12行，等于执行下一跳指令
6: bipush 10    将10放到操作数栈中
8: istore_1		将操作数栈中的数放到局部变量表一号槽位中
9: goto 15		跳转到15行
12: bipush 20	将20放到操作数栈中
14: istore_1	将操作数栈中的数放到局部变量表一号槽位中
15: return		结束方法
~~~





以上比较指令中没有 long，float，double 的比较，那么它们要比较怎 么办？ 

参考 https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-6.html#jvms-6.5.lcmp







### 2.6 循环控制指令

其实循环控制还是前面介绍的那些指令，



例如 while 循环：

~~~java
public class Demo3_4 {
public static void main(String[] args) {
        int a = 0;
        while (a < 10) {
            a++;
        }
    }
}
~~~





字节码是：

~~~
0: iconst_0   【注意：-1到5之间的数 用iconst表示】这里表示将一个常量0放到操作数栈中
1: istore_1		将操作数栈中的数放到一号槽位中
2: iload_1		把一号槽位中的数放到操作数栈中
3: bipush 10	将10放到操作数栈中
5: if_icmpge 14	 零是否大于等于十 大于等于跳到14 小于继续下一条
8: iinc 1, 1	槽位为一的数自增一
11: goto 2		跳转到2
14: return		结束方法
~~~



再比如 do while 循环：

~~~java
public class Demo3_5 {
    public static void main(String[] args) {
        int a = 0;
        do {
       	  a++;
        } while (a < 10);
    }
}
~~~

~~~
 0: iconst_0
 1: istore_1
 2: iinc 1, 1
 5: iload_1
 6: bipush 10
 8: if_icmplt 2
 11: return
~~~



最后再看看 for 循环：

~~~java
public class Demo3_6 {
    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            
        }
    }
}
~~~



字节码是：

~~~
0: iconst_0
1: istore_1
2: iload_1
3: bipush 10
5: if_icmpge 14
8: iinc 1, 1
11: goto 2
14: return
~~~

注意 比较 while 和 for 的字节码，你发现它们是一模一样的，殊途也能同归



### 2.7 练习 - 判断结果

请从字节码角度分析，下列代码运行的结果：

~~~java
public class Demo3_6_1 {
    public static void main(String[] args) {
        int i = 0;
        int x = 0;
        while (i < 10) {
            x = x++;
            i++;
        }
        System.out.println(x); // 结果是 0
    }
}
~~~



分析 `x=x++`

我们知到 x++对应两条指令，1：从局部变量表中获取数据到操作数栈中，2:将局部变量表中的槽位中的数据自增一



分析赋值运算符`=` 将操作数中的数字再赋值给局部变量表中的槽位中的数





### 2.8 构造方法



cinit()V

~~~java
public class Demo3_8_1 {
    static int i = 10;
    static {
        i = 20;
    }
    static {
        i = 30;
    }
}
~~~

编译器会按从上至下的顺序，收集所有 static 静态代码块和静态成员赋值的代码，合并为一个特殊的方 法 \<cinit>()V ：

字节码如下：

~~~
0: bipush 10   					将数字10放到操作数栈中
2: putstatic #2 // Field i:I	将操作数中的数字放到常量池中的一个名为I的变量中
5: bipush 20					将数字20放到操作数栈中
7: putstatic #2 // Field i:I	将操作数中的数字放到常量池中的一个名为I的变量中
10: bipush 30					将数字30放到操作数栈中
12: putstatic #2 // Field i:I	将操作数中的数字放到常量池中的一个名为I的变量中
15: return						结束方法
~~~

`<cinit>()V 方法`会在类加载的初始化阶段被调用  【类的构造方法】



\<init>()V

~~~java
public class Demo3_8_2 {
    private String a = "s1";
     {
        b = 20;
    }
        private int b = 10;
     {
        a = "s2";
    }
    public Demo3_8_2(String a, int b) {
        this.a = a;
        this.b = b;
    }
    public static void main(String[] args) {
        Demo3_8_2 d = new Demo3_8_2("s3", 30);
        System.out.println(d.a);
        System.out.println(d.b);
    }
}
~~~

编译器会按从上至下的顺序，收集所有 {} 代码块和成员变量赋值的代码，形成新的构造方法

**但原始构 造方法内的代码总是在最后**



字节码：

~~~
public cn.itcast.jvm.t3.bytecode.Demo3_8_2(java.lang.String, int);
descriptor: (Ljava/lang/String;I)V
flags: ACC_PUBLIC
    Code:
    stack=2, locals=3, args_size=3
        0: aload_0        【加载this】
        1: invokespecial #1 // super.<init>()V   调用Object的init方法 
        4: aload_0			加载this
        5: ldc #2 // <- "s1"	将常量池中的“s1”加载到操作数栈
        7: putfield #3 // -> this.a		将"s1"赋值给this的a变量
        10: aload_0
        11: bipush 20 // <- 20
        13: putfield #4 // -> this.b
        16: aload_0
        17: bipush 10 // <- 10
        19: putfield #4 // -> this.b
        22: aload_0
        23: ldc #5 // <- "s2"
        25: putfield #3 // -> this.a
        28: aload_0 // ------------------------------
        29: aload_1 // <- slot 1(a) "s3" |
        30: putfield #3 // -> this.a |
        33: aload_0 |
        34: iload_2 // <- slot 2(b) 30 |
        35: putfield #4 // -> this.b --------------------
        38: return
    LineNumberTable: ...
    LocalVariableTable:
        Start Length Slot Name Signature
        0 39 0 this Lcn/itcast/jvm/t3/bytecode/Demo3_8_2;
        0 39 1 a Ljava/lang/String;
        0 39 2 b I
MethodParameters: ...

~~~





### 2.9 方法调用

看一下几种不同的方法调用对应的字节码指令：

~~~java
public class Demo3_9 {
    public Demo3_9() { }
    private void test1() { }
    private final void test2() { }
    public void test3() { }
    public static void test4() { }
    public static void main(String[] args) {
        Demo3_9 d = new Demo3_9();
        d.test1();
        d.test2();
        d.test3();
        d.test4();
        Demo3_9.test4();
    }
}
~~~

~~~
0: new #2 // class cn/itcast/jvm/t3/bytecode/Demo3_9   【分配该对象在堆中所需要的内存，将对象的引用放到操作数栈中】
3: dup   【将栈顶的对象引用进行一次复制，放在栈顶； 	】
4: invokespecial #3 // Method "<init>":()V   【根据栈顶的引用调用对象的构造方法】 调用结束后，将栈顶元素出栈
7: astore_1									【将操作数栈中的对象地址放到局部变量表的一号槽位中】
8: aload_1
9: invokespecial #4 // Method test1:()V
12: aload_1
13: invokespecial #5 // Method test2:()V
16: aload_1
17: invokevirtual #6 // Method test3:()V
20: aload_1   【加载对象的地址到操作数栈】
21: pop		【发现是静态方法的调用，将对象丢弃出操作数栈】
22: invokestatic #7 // Method test4:()V	【调用静态方法】
25: invokestatic #7 // Method test4:()V
28: return
~~~

`invokespecial   ` ：私有方法、构造方法

`invokestatic` ： 静态方法

上面两个方法属于静态绑定，在字节码生成后（编译期），就可以确定是哪个类的哪个方法；【因为静态的方法，和私有的方法不可以重写】

`invokevisual` :普通public的方法 【可以重写，即可以多态，编译期间不可以确认调用哪个类的哪个方法】 动态绑定 	 



字节码指令分析

1、new 是创建【对象】，给对象分配堆内存，执行成功会将【对象引用】压入操作数栈

2、dup 是复制操作数栈栈顶的内容，本例即为【对象引用】，为什么需要两份引用呢

​	一个是要配合 invokespecial 调用该对象的构造方法 "\<init>":()V （会消耗掉栈顶一个引用），另一个要配合 astore_1 赋值给局部变量

3、最终方法（final），私有方法（private），构造方法都是由 invokespecial 指令来调用，属于静态绑定

4、普通成员方法是由 invokevirtual 调用，属于动态绑定，即支持多态

5、成员方法与静态方法调用的另一个区别是，执行方法前是否需要【对象引用】

6、比较有意思的是 d.test4(); 是通过【对象引用】调用一个静态方法，可以看到在调用

7、invokestatic 之前执行了 pop 指令，把【对象引用】从操作数栈弹掉了😂

8、还有一个执行 invokespecial 的情况是通过 super 调用父类方法





### 2.10 多态的原理

源码：

~~~java
package cn.itcast.jvm.t3.bytecode;

import java.io.IOException;

/**
 * 演示多态原理，注意加上下面的 JVM 参数，禁用指针压缩
 * -XX:-UseCompressedOops -XX:-UseCompressedClassPointers
 */
public class Demo3_10 {

    public static void test(Animal animal) {
        animal.eat();
        System.out.println(animal.toString());
    }

    public static void main(String[] args) throws IOException {
        test(new Cat());
        test(new Dog());
        System.in.read();
    }
}

abstract class Animal {
    public abstract void eat();

    @Override
    public String toString() {
        return "我是" + this.getClass().getSimpleName();
    }
}

class Dog extends Animal {

    @Override
    public void eat() {
        System.out.println("啃骨头");
    }
}

class Cat extends Animal {

    @Override
    public void eat() {
        System.out.println("吃鱼");
    }
}
~~~



1）运行代码 

停在 System.in.read() 方法上，这时运行 jps 获取进程 id



2）运行 HSDB 工具 进入 JDK 安装目录，执行

~~~
java -cp ./lib/sa-jdi.jar sun.jvm.hotspot.HSDB
~~~

进入图形界面

----

![image-20220809233300483](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220809233300483.png)

----



3）查找某个对象

打开 `Tools -> Find Object By Query `

输入` select d from cn.itcast.jvm.t3.bytecode.Dog d `点击 Execute 执行

-----

![image-20220809235241134](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220809235241134.png)

----



4）查看对象内存结构 

点击超链接可以看到对象的内存结构，此对象没有任何属性，因此只有对象头的 16 字节，前 8 字节是 MarkWord，后 8 字节就是对象的 Class 指针 

但目前看不到它的实际地址

----

![image-20220809235352985](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220809235352985.png)

---

5）查看对象 Class 的内存地址 

可以通过 Windows -> Console 进入命令行模式，执行

~~~
mem 0x00000001299b4978 2
~~~

mem 有两个参数，参数 1 是对象地址，参数 2 是查看 2 行（即 16 字节） 

结果中第二行 0x000000001b7d4028 即为 Class 的内存地址

----

![image-20220810195514780](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220810195514780.png)

----



6）查看类的 vtable

方法1：Alt+R 进入 Inspector 工具，输入刚才的 Class 内存地址，看到如下界面

---

![image-20220813004142929](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220813004142929.png)

---



方法2：或者 Tools -> Class Browser 输入 Dog 查找，可以得到相同的结果

---

![image-20220810200332444](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220810200332444.png)

----

无论通过哪种方法，都可以找到 Dog` Class 的 vtable 长度`为 6，意思就是 Dog 类有 6 个虚方法【多态 相关的】（final，static 不会列入）

 那么这 6 个方法都是谁呢？

从 **Class 的起始地址开始算**，**偏移 0x1b8 就是 vtable 的起始地址**，进行计 算得到：

~~~
0x000000001b7d4028
               1b8 +
---------------------
0x000000001b7d41e0
~~~



通过 Windows -> Console 进入命令行模式，执行

~~~
mem 0x000000001b7d41e0 6
~~~

类地址 + 偏移量地址 = vtable地址

在Inspector中找到Vtable的的长度为6



~~~
0x000000001b7d41e0: 0x000000001b3d1b10
0x000000001b7d41e8: 0x000000001b3d15e8
0x000000001b7d41f0: 0x000000001b7d35e8
0x000000001b7d41f8: 0x000000001b3d1540
0x000000001b7d4200: 0x000000001b3d1678
0x000000001b7d4208: 0x000000001b7d3fa8
~~~

就得到了 6 个虚方法的入口地址





通过对象找到类对象，找到虚方法表，通过虚方法表找到方法的入口地址【连接阶段就生成虚方法表】





7）验证方法地址

通过 Tools -> Class Browser 查看每个类的方法定义，比较可知

----

![image-20220813004223713](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220813004223713.png)

---

![image-20220813004257145](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220813004257145.png)

-----

![image-20220813004327252](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220813004327252.png)

----



~~~
Dog - public void eat() @0x000000001b7d3fa8
Animal - public java.lang.String toString() @0x000000001b7d35e8;
Object - protected void finalize() @0x000000001b3d1b10;
Object - public boolean equals(java.lang.Object) @0x000000001b3d15e8;
Object - public native int hashCode() @0x000000001b3d1540;
Object - protected native java.lang.Object clone() @0x000000001b3d1678;
~~~

对号入座，发现 

eat() 方法是 Dog 类自己的 

toString() 方法是继承 String 类的

finalize() ，equals()，hashCode()，clone() 都是继承 Object 类的



8）小结

当执行 invokevirtual 指令时： 

1. 先通过栈帧中的**对象引用**找到对象 
2. 分析**对象头**，找到对象的实际 Class【对象头中包含有8个字节的关于该对象Class对象的地址】
3.  Class 结构中有` vtable`，它在**类加载的链接阶段**就已经根据方法的重写规则生成好了
4. 查表得到方法的具体地址 
5. 执行方法的字节码



注意：

静态方法，私有方法，最终方法不会在虚方法表





### 2.11 异常处理

try-catch

~~~java
public class Demo3_11_1 {
public static void main(String[] args) {
         int i = 0;
        try {
            i = 10;
        } catch (Exception e) {
            i = 20;
        }
    }
}
~~~

注意 为了抓住重点，下面的字节码省略了不重要的部分:\

~~~java
public static void main(java.lang.String[]);
descriptor: ([Ljava/lang/String;)V
flags: ACC_PUBLIC, ACC_STATIC
    Code:
    stack=1, locals=3, args_size=1
        0: iconst_0
        1: istore_1     i=0
        2: bipush 10
        4: istore_1		i=10
        5: goto 12		转去结束方法
        8: astore_2    将异常对象的引用地址存放到局部变量表中的二号槽位
        9: bipush 20
        11: istore_1	将操作数栈中的20 放到局部变量表中的1号槽位
        12: return		结束方法
    Exception table:   异常表
    from to target type
    2 5 8 Class java/lang/Exception   包含第二行到第四行的指令【包前不包后】出现异常后，判断是否是表中的异常类型或子类 是就进入第八行指令
    LineNumberTable: ...
    LocalVariableTable:
        Start Length Slot Name Signature
        9 3 2 e Ljava/lang/Exception;
        0 13 0 args [Ljava/lang/String;
        2 11 1 i I
StackMapTable: ...
MethodParameters: ...
}
~~~

重点小结：

可以看到多出来一个 Exception table 的结构，[from, to) 是前闭后开的检测范围，一旦这个范围 内的字节码执行出现异常，则通过 type 匹配异常类型，如果一致，进入 target 所指示行号 8 行的字节码指令 astore_2 是将异常对象引用存入局部变量表的 slot 2 位置





多个 single-catch 块的情况

~~~java
public class Demo3_11_2 {
    public static void main(String[] args) {
        int i = 0;
        try {
            i = 10;
        } catch (ArithmeticException e) {
            i = 30;
        } catch (NullPointerException e) {
            i = 40;
        } catch (Exception e) {
            i = 50;
        }
    }
}
~~~

~~~
public static void main(java.lang.String[]);
descriptor: ([Ljava/lang/String;)V
flags: ACC_PUBLIC, ACC_STATIC
    Code:
    stack=1, locals=3, args_size=1
        0: iconst_0
        1: istore_1
        2: bipush 10
        4: istore_1
        5: goto 26
        8: astore_2
        9: bipush 30
        11: istore_1
        12: goto 26
        15: astore_2
        16: bipush 40
        18: istore_1
        19: goto 26
        22: astore_2
        23: bipush 50
        25: istore_1
        26: return
    Exception table:
    from to target type
    2 5 8 Class java/lang/ArithmeticException
    2 5 15 Class java/lang/NullPointerException
    2 5 22 Class java/lang/Exception
LineNumberTable: ...
    LocalVariableTable:
    Start Length Slot Name Signature
    9 3 2 e Ljava/lang/ArithmeticException; 这里三个异常类型的对象都会放到一个槽位中，因为每次只会出现一个异常，所以可以共用
    16 3 2 e Ljava/lang/NullPointerException;
    23 3 2 e Ljava/lang/Exception;
    0 27 0 args [Ljava/lang/String;
    2 25 1 i I
StackMapTable: ...
MethodParameters: ...
~~~

注意：

因为异常出现时，只能进入 Exception table 中一个分支，所以局部变量表 slot 2 位置被共用





multi-catch 的情况：

~~~java
public class Demo3_11_3 {
    public static void main(String[] args) {
        try {
            Method test = Demo3_11_3.class.getMethod("test");
            test.invoke(null);
        } catch (NoSuchMethodException | IllegalAccessException |
            InvocationTargetException e) {
            e.printStackTrace();
        }
    }
    
    public static void test() {
        System.out.println("ok");
    }
}
~~~

~~~
public static void main(java.lang.String[]);
descriptor: ([Ljava/lang/String;)V
flags: ACC_PUBLIC, ACC_STATIC
    Code:
    stack=3, locals=2, args_size=1
        0: ldc #2
        2: ldc #3
        4: iconst_0
        5: anewarray #4
        8: invokevirtual #5
        11: astore_1
        12: aload_1
        13: aconst_null
        14: iconst_0
        15: anewarray #6
        18: invokevirtual #7
        21: pop
        22: goto 30
        25: astore_1
        26: aload_1
        27: invokevirtual #11 // e.printStackTrace:()V
        30: return
Exception table:
        from to target type
        0 22 25 Class java/lang/NoSuchMethodException
        0 22 25 Class java/lang/IllegalAccessException
        0 22 25 Class java/lang/reflect/InvocationTargetException  无论是哪个异常，都会去执行25行指令
LineNumberTable: ...
LocalVariableTable:
        Start Length Slot Name Signature
        12 10 1 test Ljava/lang/reflect/Method;
        26 4 1 e Ljava/lang/ReflectiveOperationException;
        0 31 0 args [Ljava/lang/String;
StackMapTable: ...
MethodParameters: ...
~~~



finally

~~~java
public class Demo3_11_4 {
    public static void main(String[] args) {
        int i = 0;
        try {
            i = 10;
        } catch (Exception e) {
            i = 20;
        } finally {
            i = 30;
        }
    }
}
~~~

~~~
public static void main(java.lang.String[]);
descriptor: ([Ljava/lang/String;)V
flags: ACC_PUBLIC, ACC_STATIC
    Code:
    stack=1, locals=4, args_size=1
        0: iconst_0
        1: istore_1 // 0 -> i
        2: bipush 10 //           try --------------------------------------
        4: istore_1 // 10 -> i    catch-------------------------
        5: bipush 30 //           finally 要执行的逻辑
        7: istore_1 // 30 -> i
        8: goto 27 // 			  跳转到return -----------------------------------
        11: astore_2 // 		  将抓到的异常对象放在二号槽位上
        12: bipush 20 // 		  抓到异常后的逻辑
        14: istore_1 // 20 -> i 
        15: bipush 30 // finally   finally 要执行的逻辑
        17: istore_1 // 30 -> i 
        18: goto 27 //			 跳转到return -----------------------------------
        21: astore_3 // 		 捕获任何异常，放到三号槽位中【try块中出现了error、throwable、或不在范围内的异常 以及 catch中出现的任何异常】
        22: bipush 30 // finally   finally 要执行的逻辑
        24: istore_1 // 30 -> i 
        25: aload_3 // <-        取槽位为三的异常对象
        26: athrow //   		抛出
        27: return				结束方法
Exception table:
        from to target type
        2 5 11 Class java/lang/Exception
        2 5 21 any // 剩余的异常类型，比如 Error
        11 15 21 any // 剩余的异常类型，比如 Error
LineNumberTable: ...
LocalVariableTable:
        Start Length Slot Name Signature
        12 3 2 e Ljava/lang/Exception;
        0 28 0 args [Ljava/lang/String;
        2 26 1 i I
StackMapTable: ...
MethodParameters: ...
~~~

可以看到 finally 中的代码被复制了 3 份，分别放入 **try 流程**，**catch 流程**以及 **catch 剩余的异常类型流 程**



### 2.12 练习 - finally 面试题

**finally 出现了 return** 

先问问自己，下面的题目输出什么？

~~~java
public class Demo3_12_2 {
    public static void main(String[] args) {
        int result = test();
        System.out.println(result);
    }
    
    public static int test() {
        try {
            return 10;
        } finally {
            return 20;
        }
    }
}
~~~

~~~
public static int test();
descriptor: ()I
flags: ACC_PUBLIC, ACC_STATIC
    Code:
    stack=1, locals=2, args_size=0
        0: bipush 10 // <- 10 放入栈顶
        2: istore_0 // 10 -> slot 0 (从栈顶移除了)
        3: bipush 20 // <- 20 放入栈顶
        5: ireturn // 返回栈顶 int(20)
        6: astore_1 // catch any -> slot 1
        7: bipush 20 // <- 20 放入栈顶
        9: ireturn // 返回栈顶 int(20)
Exception table:
        from to target type
        0 3 6 any
LineNumberTable: ...
StackMapTable: ...
~~~

由于 finally 中的 ireturn 被插入了所有可能的流程，因此返回结果肯定以 finally 的为准 

至于字节码中第 2 行，似乎没啥用，且留个伏笔，看下个例子 

跟上例中的 finally 相比，发现**没有 athrow** 了，

这告诉我们：**如果在 finally 中出现了 return，会 吞掉  异常**😱😱😱，



可以试一下下面的代码:

~~~java
public class Demo3_12_1 {
    public static void main(String[] args) {
        int result = test();
        System.out.println(result);
    }
    public static int test() {
        try {
            int i = 1/0;
            return 10;
        } finally {
            return 20;
        }
    }
}
~~~

注意：

这样是不会发生异常的，结果是20





finally 对返回值影响 

同样问问自己，下面的题目输出什么？

~~~java
public class Demo3_12_2 {
    public static void main(String[] args) {
        int result = test();
        System.out.println(result);
    }
    public static int test() {
        int i = 10;
        try {
            return i;
        } finally {
            i = 20;
        }
    }
}
~~~

~~~
public static int test();
descriptor: ()I
flags: ACC_PUBLIC, ACC_STATIC
    Code:
    stack=1, locals=3, args_size=0
        0: bipush 10 // <- 10 放入栈顶
        2: istore_0 // 10 -> i
        3: iload_0 // <- i(10)
        4: istore_1 // 10 -> slot 1，    【本来想直接返回的，发现有finally】暂存至 slot 1，目的是为了固定返回值
        5: bipush 20 // <- 20 放入栈顶
        7: istore_0 // 20 -> i
        8: iload_1 // <- slot 1(10) 载入 slot 1 暂存的值
        9: ireturn // 返回栈顶的 int(10)
        10: astore_2
        11: bipush 20
        13: istore_0
        14: aload_2
        15: athrow
Exception table:
        from to target type
        3     5   10    any
LineNumberTable: ...
LocalVariableTable:
        Start Length Slot Name Signature
         3       13     0    i      I
StackMapTable: ...
~~~



### 2.13 synchronized

~~~java
public class Demo3_13 {
    public static void main(String[] args) {
        Object lock = new Object();
            synchronized (lock) {
            System.out.println("ok");
        }
    }
}
~~~

~~~
public static void main(java.lang.String[]);
descriptor: ([Ljava/lang/String;)V
flags: ACC_PUBLIC, ACC_STATIC
    Code:
    stack=2, locals=4, args_size=1
        0: new #2 // new Object  创建一块内存空间用于存放对象【放到操作数栈中】
        3: dup						复制操作数栈中的引用，放到栈顶中
        4: invokespecial #1 // invokespecial <init>:()V	通过栈顶的对象引用调用构造器 用完弹栈
        7: astore_1 // lock引用 -> lock					将目前栈顶的对象引用放到1号槽位
        8: aload_1 // <- lock （synchronized开始）         从一号槽位中，取出lock对象
        9: dup											 将lock对象赋值一份放到栈顶
        10: astore_2 // lock引用 -> slot 2				将栈顶中的对象放到2号槽位
        11: monitorenter // monitorenter(lock引用)		将目前栈顶的对象加锁
        12: getstatic #3 // <- System.out				 加载静态变量到操作数栈中
        15: ldc #4 // <- "ok"							 加载字符串常量到操作数栈中
        17: invokevirtual #5 // invokevirtual println:  调用静态对象的println方法
        (Ljava/lang/String;)V
        20: aload_2 // <- slot 2(lock引用)				将2号槽位的lock对象，加载到栈中
        21: monitorexit // monitorexit(lock引用)			根据对象解锁
        22: goto 30										准备结束
        25: astore_3 // any -> slot 3                   将异常对象放到三号槽位中
        26: aload_2 // <- slot 2(lock引用)				加载lock对象的引用到二号槽位
        27: monitorexit // monitorexit(lock引用)			解锁
        28: aload_3										 加载异常对象到操作数栈中
        29: athrow										外抛异常
        30: return										结束
Exception table:
        from    to    target    type
        12      22       25     any
        25      28       25     any
LineNumberTable: ...
LocalVariableTable:
        Start Length Slot  Name    Signature
        0       31     0   args    [Ljava/lang/String;
        8       23     1   lock    Ljava/lang/Object;
StackMapTable: ...
MethodParameters: ...
~~~

注意：

方法级别的 `synchronized `不会在字节码指令中有所体现



## 3. 编译期处理

所谓的 语法糖 ，其实就是指 java 编译器把 *.java 源码编译为 *.class 字节码的过程中，自动生成 和转换的一些代码

主要是为了减轻程序员的负担，算是 java 编译器给我们的一个额外福利（给糖吃 嘛） 



注意，以下代码的分析，借助了 javap 工具，idea 的反编译功能，idea 插件 jclasslib 等工具

另外， **编译器转换的结果直接**就是 **class 字节码**，只是为了便于阅读，给出了 **几乎等价 的 java 源码方式**，并 **不是编译器还会转换出中间的 java 源码**，**切记**。





### 3.1 默认构造器

~~~java
public class Candy1 {
}
~~~



编译成class后的代码：

~~~java
public class Candy1 {
    // 这个无参构造是编译器帮助我们加上的
    public Candy1() {
        super(); // 即调用父类 Object 的无参构造方法，即调用 java/lang/Object."<init>":()V
    }
}
~~~



### 3.2 自动拆装箱 

这个特性是 JDK 5 开始加入的， 代码片段1 ：

~~~java
public class Candy2 {
    public static void main(String[] args) {
        Integer x = 1;  
        int y = x;
    }
}
~~~

这段代码在 JDK 5 之前是无法编译通过的，必须改写为 代码片段2 :

~~~java
public class Candy2 {
    public static void main(String[] args) {
        Integer x = Integer.valueOf(1);  //在-128~127之间会直接有一个Integer对象，超过这个范围就会new Integer对象【装箱】
        int y = x.intValue();   //拆箱	
    }
}
~~~

显然之前版本的代码太麻烦了，需要在基本类型和包装类型之间来回转换（尤其是集合类中操作的都是 包装类型）

因此这些转换的事情在 **JDK 5 以后**都由 编译器 在 **编译阶段完成**

即 代码片段1 都会在**编 译阶段 **被 转换为 代码片段2;







### 3.3 泛型集合取值

泛型也是在` JDK 5 `开始加入的特性，但 java 在编译泛型代码后会执行 `泛型擦除 `的动作，即`泛型信息 在编译为字节码之后`就 **丢失了**

实际的类型都当做了 Object 类型来处理：

~~~java
public class Candy3 {
    public static void main(String[] args) {
        List<Integer> list = new ArrayList<>();
        list.add(10); // 实际调用的是 List.add(Object e)
        Integer x = list.get(0); // 实际调用的是 Object obj = List.get(int index);
    }
}
~~~

所以在取值时，编译器真正生成的字节码中，还要额外做一个类型转换的操作：

~~~java
// 需要将 Object 转为 Integer
Integer x = (Integer)list.get(0);
~~~



如果前面的 x 变量类型修改为 int 基本类型那么最终生成的字节码是：

~~~java
// 需要将 Object 转为 Integer, 并执行拆箱操作
int x = ((Integer)list.get(0)).intValue();
~~~

还好这些麻烦事都不用自己做。





擦除的是字节码上的泛型信息，可以看到 LocalVariableTypeTable 仍然保留了方法参数泛型的信息

字节码如下：

~~~java
public cn.itcast.jvm.t3.candy.Candy3();
descriptor: ()V
flags: ACC_PUBLIC
    Code:
    stack=1, locals=1, args_size=1
        0: aload_0
        1: invokespecial #1 // Method java/lang/Object."<init>":()V
        4: return
LineNumberTable:
line 6: 0
LocalVariableTable:
        Start Length Slot Name    Signature
        0        5    0    this     Lcn/itcast/jvm/t3/candy/Candy3;
public static void main(java.lang.String[]);
descriptor: ([Ljava/lang/String;)V
flags: ACC_PUBLIC, ACC_STATIC
    Code:
    stack=2, locals=3, args_size=1
        0: new #2 // class java/util/ArrayList    创建空间 将对象引用放到栈中
        3: dup									   //复制引用 放到栈顶
        4: invokespecial #3 // Method java/util/ArrayList." <init>":()V  调用ArrayList的构造方法
        7: astore_1       // 将对象引用放到栈中
        8: aload_1			//将对象放到一号槽位中
        9: bipush 10			//将10放到栈中
        11: invokestatic #4 // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;  调用静态方法，转为包装Integer
        14: invokeinterface #5, 2 // InterfaceMethod java/util/List.add:(Ljava/lang/Object;)Z  调用接口方法 返回布尔
        //上面一行是泛型擦除原理，调用的方法不区分类型      
        19: pop
              
        20: aload_1   //从槽位一中获取list引用 到栈中
        21: iconst_0   //将0放到栈中
        22: invokeinterface #6, 2 // InterfaceMethod java/util/List.get:(I)Ljava/lang/Object; 调用接口方法，返回值类型Object
          //泛型擦除本质，返回的对象是Object类型    
        27: checkcast #7 // class java/lang/Integer   强制类型转换为Integer
        30: astore_2		//转换后的对象，放到2号槽位
        31: return   		//结束方法
LineNumberTable:
        line 8: 0
        line 9: 8
        line 10: 20
        line 11: 31
LocalVariableTable: 【局部变量表】
        Start    Length   Slot    Name    Signature
        0          32      0      args      [Ljava/lang/String;
        8          24      1      list       Ljava/util/List;
LocalVariableTypeTable: 【局部变量类型表】记录泛型信息
        Start    Length     Slot      Name     Signature
        8          24         1       list      Ljava/util/List<Ljava/lang/Integer;>;
~~~





使用反射，仍然能够获得这些信息：

~~~java
public Set<Integer> test(List<String> list, Map<Integer, Object> map) {
}
~~~

注意：有局限性===》只能获取方法入参和返回值的泛型类型



~~~java
Method test = Candy3.class.getMethod("test", List.class, Map.class);
Type[] types = test.getGenericParameterTypes();

for (Type type : types) {
    if (type instanceof ParameterizedType) {
        ParameterizedType parameterizedType = (ParameterizedType) type;
        System.out.println("原始类型 - " + parameterizedType.getRawType());
        Type[] arguments = parameterizedType.getActualTypeArguments();
        for (int i = 0; i < arguments.length; i++) {
            System.out.printf("泛型参数[%d] - %s\n", i, arguments[i]);
        }
    }
}
~~~

~~~
原始类型 - interface java.util.List
泛型参数[0] - class java.lang.String
原始类型 - interface java.util.Map
泛型参数[0] - class java.lang.Integer
泛型参数[1] - class java.lang.Object
~~~



### 3.4 可变参数

可变参数也是 JDK 5 开始加入的新特性： 

例如：

~~~java
public class Candy4 {
    public static void foo(String... args) {
        String[] array = args; // 直接赋值
        System.out.println(array);
    }
    public static void main(String[] args) {
        foo("hello", "world");
    }
}
~~~

可变参数 String... args 其实是一个 String[] args ，从代码中的赋值语句中就可以看出来



同样 java 编译器会在编译期间将上述代码变换为：

~~~java
public class Candy4 {
    public static void foo(String[] args) {
        String[] array = args; // 直接赋值
        System.out.println(array);
    }
    public static void main(String[] args) {
        foo(new String[]{"hello", "world"});
    }
}
~~~

注意 如果调用了 **foo()** 则等价代码为 **foo(new String[]{})** ，创建了一个  **空的数组**  ，而  **不会传递 null** 进去







### 3.5 foreach 循环

仍是 JDK 5 开始引入的语法糖，数组的循环：

~~~java
public class Candy5_1 {
    public static void main(String[] args) {
        int[] array = {1, 2, 3, 4, 5}; // 数组赋初值的简化写法也是语法糖哦
        for (int e : array) {
            System.out.println(e);
        }
    }
}
~~~



会被编译器转换为：

~~~java
public class Candy5_1 {
    public Candy5_1() {
    }
    public static void main(String[] args) {
        int[] array = new int[]{1, 2, 3, 4, 5};
        for(int i = 0; i < array.length; ++i) {
            int e = array[i];
            System.out.println(e);
        }
    }
}
~~~







而集合的循环：

~~~java
public class Candy5_2 {
    public static void main(String[] args) {
        List<Integer> list = Arrays.asList(1,2,3,4,5);
        for (Integer i : list) {
            System.out.println(i);
        }
    }
}
~~~

实际被编译器转换为对迭代器的调用：

~~~java
public class Candy5_2 {
    public Candy5_2() {
    }
    
    public static void main(String[] args) {
        List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);
        Iterator iter = list.iterator();
        while(iter.hasNext()) {
            Integer e = (Integer)iter.next();
            System.out.println(e);
        }
    }
}
~~~

注意 foreach 循环写法，能够配合数组，以及所有实现了 Iterable 接口的集合类一起使用，其中 Iterable 用来获取集合的迭代器（ Iterator ）







### 3.6 switch 字符串

`从 JDK 7 开始`，switch 可以作用于**字符串和枚举类**

这个功能其实也是语法糖【因为原生的字节码指令是不支持switch中使用字符串和枚举类】是通过一些转换实现的

例如：

~~~java
public class Candy6_1 {
    public static void choose(String str) {
        switch (str) {
             case "hello": {
                System.out.println("h");
                break;
            }
            case "world": {
                System.out.println("w");
                break;
            }
        }
    }
}
~~~

注意 switch 配合 String 和枚举使用时，变量不能为null，原因分析完语法糖转换后的代码应当自然清 楚



会被编译器转换为：【拆成两个switch】

~~~java
public class Candy6_1 {
        public Candy6_1() {
        }

        public static void choose(String str) {
            byte x = -1;
            switch (str.hashCode()) {
                case 99162322: // hello 的 hashCode
                    if (str.equals("hello")) {
                        x = 0;
                    }
                    break;
                case 113318802: // world 的 hashCode
                    if (str.equals("world")) {
                        x = 1;
                    }
            }

            switch (x) {
                case 0:
                    System.out.println("h");
                    break;
                case 1:
                    System.out.println("w");
            }
        }
}
~~~

可以看到，执行了两遍 switch，第一遍是根据字符串的 hashCode 和 equals 将字符串的转换为相应 byte 类型，第二遍才是利用 byte 执行进行比较

为什么第一遍时必须既比较 hashCode，又利用 equals 比较呢？

hashCode 是为了提高效率，减少可 能的比较；

而 equals 是为了防止 hashCode 冲突，



String重写Object类的hashCode方法 源码如下：【内容相同，hashcode必相同】

~~~java
 public int hashCode() {
    int h = hash;
    //首先判断本对象是否已经计算过hash值并且字符串的长度大于0
     //还没
    if (h == 0 && value.length > 0) {
      char val[] = value;
      //循环字符串中的字符，用以计算hash值
      for (int i = 0; i < value.length; i++) {
        h = 31 * h + val[i];
      }
      hash = h;
    }
    return h;
  }
~~~



String重写Object类的equal方法 源码如下：

~~~java
public boolean equals(Object anObject) {
    //判断两个对象的引用地址是否一致
    if (this == anObject) {
      return true;
    }
    //判断传入的对象是否是String类型
    if (anObject instanceof String) {
      String anotherString = (String)anObject;
      int n = value.length;
      //判断两个字符串是否长度相等
      if (n == anotherString.value.length) {
        char v1[] = value;
        char v2[] = anotherString.value;
        int i = 0;
        //逐个比较两个字符串中的字符
        while (n-- != 0) {
          if (v1[i] != v2[i])
            return false;
          i++;
        }
        return true;
      }
    }
    return false;
  }
~~~



**hashCode()的返回值大多数情况是将对象的内部地址转换成整数并返回，少数情况不是这样，也就代表着HashCode不是内存地址**







例如 BM 和 C. 这两个字符串的hashCode值都是 2123 ，如果有如下代码：

~~~java
public class Candy6_2 {
    public static void choose(String str) {
        switch (str) {
            case "BM": {
                System.out.println("h");
                break;
            }
            case "C.": {
                System.out.println("w");
                break;
            }
        }
    }
}
~~~



会被编译器转换为：

~~~java
public class Candy6_2 {
public Candy6_2() {
}
public static void choose(String str) {
    byte x = -1;
    switch(str.hashCode()) {
        case 2123: // hashCode 值可能相同，需要进一步用 equals 比较
            if (str.equals("C.")) {
                x = 1;
            } else if (str.equals("BM")) {
                x = 0;
            }
        default:
            switch(x) {
                case 0:
                    System.out.println("h");
                    break;
                case 1:
                    System.out.println("w");
            }
        }
    }
}
~~~





### 3.7 switch 枚举

 switch 枚举的例子，原始代码：

~~~java
enum Sex {
    MALE, FEMALE
}
~~~



~~~java
public class Candy7 {
    public static void foo(Sex sex) {
        switch (sex) {
        case MALE:
            System.out.println("男"); break;
        case FEMALE:
            System.out.println("女"); break;
        }
    }
}
~~~





转换后代码：

~~~java
public class Candy7 {
/**
* 定义一个合成类（仅 jvm 使用，对我们不可见）
* 用来映射枚举的 ordinal 与数组元素的关系
* 枚举的 ordinal 表示枚举对象的序号，从 0 开始
* 即 MALE 的 ordinal()=0，FEMALE 的 ordinal()=1
*/
    static class $MAP {
    // 数组大小即为枚举元素个数，里面存储case用来对比的数字
        static int[] map = new int[2];
        static {
            map[Sex.MALE.ordinal()] = 1;//数组第一个元素，值为1
            map[Sex.FEMALE.ordinal()] = 2;//数组第二个元素，值为2
        }
    }
    public static void foo(Sex sex) {
        int x = $MAP.map[sex.ordinal()];
        switch (x) {
            case 1:
                System.out.println("男");
                break;
            case 2:
                System.out.println("女");
                break;
        }
    }
}
~~~



### 3.8 枚举类 

JDK 7 新增了枚举类，以前面的性别枚举为例：【枚举类 也称为多例，可以用于实现单例】

~~~java
enum Sex {
    MALE , FEMALE
}
~~~



转换后代码：

~~~java
public final class Sex extends Enum<Sex> {
    public static final Sex MALE;
    public static final Sex FEMALE;
    private static final Sex[] $VALUES;
    
    static {
        MALE = new Sex("MALE", 0);
        FEMALE = new Sex("FEMALE", 1);
        $VALUES = new Sex[]{MALE, FEMALE};
    }
    /**
    * Sole constructor. Programmers cannot invoke this constructor.
    * It is for use by code emitted by the compiler in response to
    * enum type declarations.
    *
    * @param name - The name of this enum constant, which is the identifier
    * used to declare it.
    * @param ordinal - The ordinal of this enumeration constant (its position
    * in the enum declaration, where the initial constant is
    	assigned
    */
    //调用枚举类的父类构造，入参两个名称和标识号
    private Sex(String name, int ordinal) {
        super(name, ordinal);
    }
    
    //返回一个克隆的数组，里面放着Sex的两个实例
    public static Sex[] values() {
        return $VALUES.clone();
    }
    
    //类似于Map集合，通过key 找value
    public static Sex valueOf(String name) {
        return Enum.valueOf(Sex.class, name);
    }
}
~~~





### 3.9 try-with-resources

​	JDK 7 开始新增了对需要关闭的资源处理的特殊语法` try-with-resources`：

~~~java
try(资源变量 = 创建资源对象){

} catch( ) {
    
}
~~~

其中资源对象需要实现 AutoCloseable 接口，

例如 `InputStream `、 `OutputStream `、 `Connection `、 `Statement `、 `ResultSet `等接口都实现了 **`AutoCloseable `**，

使用 try-withresources 可以不用写 finally 语句块，编译器会帮助生成关闭资源代码，

例如：

源码：

~~~java
public class Candy9 {
    public static void main(String[] args) {
        try(InputStream is = new FileInputStream("d:\\1.txt")) {
            System.out.println(is);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
~~~



会被转换为：

~~~java
public class Candy9 {
    public Candy9() {
    }

    public static void main(String[] args) {
        try {
            InputStream is = new FileInputStream("d:\\1.txt");
            Throwable t = null;
            try {
                System.out.println(is);
            } catch (Throwable e1) {
                // t 是我们代码出现的异常
                t = e1;
                throw e1;
            } finally {
                // 判断了资源不为空
                if (is != null) {
                    // 如果我们代码有异常
                    if (t != null) {
                        try {
                            is.close();
                        } catch (Throwable e2) {
                            // 如果 close 出现异常，
                            //try代码中有问题，关闭资源时也出现问题 此时关闭异常作为被压制异常添加
                            //【也就是说此处会有两个异常被抛出】
                            t.addSuppressed(e2);
                        }
                    } else {
                        // 我们try块中没有异常
                        //如果出现异常 就是关闭资源时出异常
                        is.close();
                    }
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
~~~



为什么要设计一个` addSuppressed(Throwable e) `（添加被压制异常）的方法呢？

是为了防止异常信 息的丢失

 try-with-resources 生成的 fianlly 中如果抛出了异常：

~~~java
public class Test6 {
    public static void main(String[] args) {
        try (MyResource resource = new MyResource()) {
            int i = 1/0;
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

class MyResource implements AutoCloseable {
    public void close() throws Exception {
        throw new Exception("close 异常");
    }
}
~~~

输出：

~~~
java.lang.ArithmeticException: / by zero
    at test.Test6.main(Test6.java:7)
    Suppressed: java.lang.Exception: close 异常
        at test.MyResource.close(Test6.java:18)
        at test.Test6.main(Test6.java:6)
~~~

如以上代码所示，两个异常信息都不会丢



### 3.10 方法重写时的桥接方法 

我们都知道，方法重写时对返回值分两种情况： 

父子类的返回值完全一致 

子类返回值可以是父类返回值的子类（比较绕口，见下面的例子）

~~~java
class A {
    public Number m() {
        return 1;
    }
}
class B extends A {
    @Override
    // 子类 m 方法的返回值是 Integer 是父类 m 方法返回值 Number 的子类
    public Integer m() {
        return 2;
    }
}
~~~

注意：

Number 是一个抽象类，也是一个超类（即父类）

Number 类属于 java.lang 包，所有的包装类（如 `Double、Float、Byte、Short、Integer 以及 Long`）都是抽象类 Number 的子类

----

![image-20220813002021693](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220813002021693.png)

----



对于子类，java 编译器会做如下处理：

~~~java
class B extends A {
    public Integer m() {
        return 2;
    }
    // 此方法才是真正重写了父类 public Number m() 方法
    public synthetic bridge Number m() {
        // 调用 public Integer m()
        return m();
    }
}
~~~

其中桥接方法比较特殊，仅对 java 虚拟机可见，并且与原来的 public Integer m() 没有命名冲突，可以 用下面**反射代码**来验证：

~~~java
for (Method m : B.class.getDeclaredMethods()) {
    System.out.println(m);
}
~~~

会输出：

~~~java
public java.lang.Integer test.candy.B.m()
public java.lang.Number test.candy.B.m()
~~~





##  4. 类加载阶段 



## 5. 类加载器 



## 6. 运行期优化



















































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































