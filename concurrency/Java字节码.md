**字节码详情**

为了理解字节码的详细信息，我们需要清楚Java虚拟机(JVM)是如何处理执行过程中的字节码。JVM是基于栈的机器。每一个线程都有一个用来存储帧集(frames)的JVM栈.每次方法调用都会产生一个帧，这个帧包括一个操作栈，一个本地变量的数组和一个运行时常量池的引用。

从概念上来说，帧如下图所示

![字节码详情](/Users/li/Documents/note/noteImgPool/Java方法运行帧.gif)

本地变量的数组也称为本地变量表，包括方法的参数，它也可以用来存储本地变量的值。首先存放的参数，从地址0开始编码。如果帧是一个构造器或者实例方法，this引用讲存储在地址0中，地址1存放第一个参数，地址2存放第二个参数，依次类推。如果帧是一个静态方法，第一个方法参数会存在地址0中，地址1存放第二个参数，依次类推。

本地变量数组的大小是在编译期决定的，它取决于本地变量和正常方法参数的数量和大小。操作栈是一个用于push和pop值的后进先出的栈，它的大小也是在编译器决定的。一些操作码指令将值push到操作栈，其它的操作码从栈上获取操作数，操作它们，并将值push回去。操作栈常用来接收方法的返回值。

```java
Classfile /Users/li/Downloads/Test.class
  Last modified Mar 15, 2018; size 309 bytes
  MD5 checksum 4553b41dee731dd099687198deb75de7
  Compiled from "Test.java"
public class Test
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #4.#15         // java/lang/Object."<init>":()V
   #2 = Fieldref           #3.#16         // Test.name:Ljava/lang/String;
   #3 = Class              #17            // Test
   #4 = Class              #18            // java/lang/Object
   #5 = Utf8               name
   #6 = Utf8               Ljava/lang/String;
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               employeeName
  #12 = Utf8               ()Ljava/lang/String;
  #13 = Utf8               SourceFile
  #14 = Utf8               Test.java
  #15 = NameAndType        #7:#8          // "<init>":()V
  #16 = NameAndType        #5:#6          // name:Ljava/lang/String;
  #17 = Utf8               Test
  #18 = Utf8               java/lang/Object
{
  java.lang.String name;
    descriptor: Ljava/lang/String;
    flags:

  public Test();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 1: 0

  public java.lang.String employeeName();
    descriptor: ()Ljava/lang/String;
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: getfield      #2                  // Field name:Ljava/lang/String;
         4: areturn
      LineNumberTable:
        line 4: 0
}
SourceFile: "Test.java"
```

这个方法(line 44)的字节码有三个操作码组成。

1. 第一个操作码aload_0，用于将本地变量表中索引为0的变量的值推送到操作栈上。前面提到过，本地变量表可以用来为方法传递参数的，构造器和实例方法的this引用总是存放在本地变量表的地址0处，this引用必须入栈，因为方法需要访问实例的数据，名称和类。
2. 第二个操作码getfield，用于从对象中提取字段。当该操作码执行的时候，操作栈顶的值就会弹出(pop)，然后＃2就用来在类运行时常量池中构建一个用于存放字段name引用地址的索引，当这个引用被提取时，它会被推送到操作栈上。
3. 第三个操作码areturn，返回一个来自方法的引用。比较特殊的是，areturn的执行会导致操作栈顶的值，name字段的 引用都会被弹出，然后推送到调用方法的操作栈。

employeeName的方法相当简单，在考虑一个更复杂的例子之前，我们需要检查每一个操作码左边的值。在employeeName的方法的字节码中，这些值是0，1，4。每个方法都有一个对应的字节码数组。这些值对应每个操作码和它们在参数数组的索引，一些操作码含有参数，这些参数会占据字节数组的空间。aload_0没有参数，自然而然就占据一个字节，因此，下一个操作码getfield，就在位置1上，然而areturn在位置4上，因为getfield操作码和他的参数占据了位置1，2，3。位置1被操作码getfield使用，位置2，3被用于存放参数，这些参数用于构成在类运行时常量池中存放值的地方的一个索引。下面的图，展示了employeeName的方法的字节码数数组看起来的样子

![employeeName方法的字节码数组](/Users/li/Documents/note/noteImgPool/employeeName方法的字节码数组.gif)

实际上，字节码数组包含代表指令的字节。使用一个16进制的编辑器查看class文件，可能看到字节码数组中有下面的值：

![字节码数组中的值](/Users/li/Documents/note/noteImgPool/字节码数组中的值.gif)

```java
public synchronized int top1()  
{  
  return intArr[0];  
}  
public int top2()  
{  
 synchronized (this) {  
  return intArr[0];  
 }  
}
```

```java
Method int top1()  
   0 aload_0           //将本地变量表中索引为0的对象引用this入栈。  
                        
   1 getfield #6 <Field int intArr[]>  
                       //弹出对象引用this，将访问常量池的intArr对象引用入栈。  
                        
   4 iconst_0          //将0入栈。  
   5 iaload            //弹出栈顶的两个值，将intArr中索引为0的值入栈。  
                         
   6 ireturn           //弹出栈顶的值，将其压入调用方法的操作栈，并退出。  
                      
  
Method int top2()  
   0 aload_0           //将本地变量表中索引为0的对象引用this入栈。  
   1 astore_2          //弹出this引用，存放到本地变量表中索引为2的地方。  
   2 aload_2           //将this引用入栈。  
   3 monitorenter      //弹出this引用，获取对象的监视器。  
                        
   4 aload_0           //开始进入同步块。将this引用压入本地变量表索引为0的地方。  
                         
   5 getfield #6 <Field int intArr[]>  
                       //弹出this引用，压入访问常量池的intArr引用。  
                       
   8 iconst_0          //压入0。  
   9 iaload            //弹出顶部的两个值，压入intArr索引为0的值。  
               
  10 istore_1          //弹出值，将它存放到本地变量表索引为1的地方。  
                         
  11 jsr 19            //压入下一个操作码(14)的地址，并跳转到位置19。  
  14 iload_1           //压入本地变量表中索引为1的值。  
                        
  15 ireturn           //弹出顶部的值，并将其压入到调用方法的操作栈中，退出。  
                        
  16 aload_2           //同步块结束。将this引用压入到本地变量表索引为2的地方。   
                        
  17 monitorexit       //弹出this引用，退出监视器。  
                       
  18 athrow            //弹出this引用，抛出异常。  
                         
  19 astore_3          //弹出返回地址(14)，并将其存放到本地变量表索引为3的地方。  
                        
  20 aload_2           //将this引用压入到本地变量索引为2的地方。  
                         
  21 monitorexit       //弹出this引用，并退出监视器。  
                         
  22 ret 3             //从本地变量表索引为3的值(14)指示的地方返回。  
Exception table:       //如果在位置4(包括4)和位置16(排除16)中出现异常，则跳转到位置16.  
from to target type      
 4   16   16   any 
```

top2比top1大，执行还慢，是因为top2采取的同步和异常处理方式。注意到top1采用synchronized方法修饰符，这不会产生额外的字节码。相反top2在方法体内使用synchronized同步代码块，会产生monitorenter和monitorexit操作码的字节码，还有额外的用于处理异常的字节码。如果执行到一个同步锁的块(一个监视器)内部时，抛出了一个异常，这个锁要保证在退出同步代码块前被释放。top1的实现要比top2的效率高一些，这能获取一丁点的性能提升。

当synchronized方法修饰符出现的时候，锁的获取和释放不是通过monitorenter和monitorexit操作码来实现的，而是在JVM调用一个方法时，它检查ACC_SYNCHRONIZED属性的标识。如果有这个标识，正在执行的线程将会先获取一个锁，调用方法然后在方法返回时释放锁。如果同步方法抛出异常，在异常离开方法前会自动释放锁。