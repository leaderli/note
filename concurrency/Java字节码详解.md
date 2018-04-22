# 基础概念

## 变量

### 局部变量

JVM是一个基于栈的架构，每一个线程都有一个用来存储帧集(frames)的JVM栈，每次方法被调用的时候(包含main方法)，在栈上会分配一个新的帧。这个帧包含一个本地变量的数组，一个操作数栈，一个运行时常量的引用。这个数组包含方法运行过程中用到的所有变量，包括this引用，方法参数，以及局部变量。对于类方法来说，方法参数是从数组第0个位置开始的，对于实例方法来说，第0个位置上的变量时this指针。从概念来说，帧的情况如下图所示

![字节码详情](/Users/li/Documents/note/noteImgPool/Java方法运行帧.gif)



本地变量可以是以下几种类型：

```
char
long
short
int
double
float
引用
返回地址
```

```shell
常用字节码指令
invokespecial #调用实例初始化方法，私有方法，父类的方法
invokevirtual #调用实例类(子类)的方法
dup #复制一份实例引用 一份作为构造函数 一份作为实例返回
```



除了long和double类型，每个变量都只占用本地变量区中的一个变量槽(slot)，而long和double只所以会占用两个变量槽(slot)，因为long和double是64位的。

当一个新的变量创建时，操作数栈(Operand stack)会存储这个新变量的值。然后这个变量会被存储到本地变量表中对应的位置上。如果这个变量不是基础类型的话，本地变量槽存的就是一个引用，这个引用指向堆中的一个对象。

比如

```java
int i = 5;
```

编译后就变成了

```java
0: bipush      5 //用来将一个字节作为整型数字压入操作数栈中，在这里5就会被压入操作数栈上。
2: istore_0 //这是istore_这组指令集（译注：严格来说，这个应该叫做操作码，opcode ,指令是指操作码加上对应的操作数，oprand。不过操作码一般作为指令的助记符，这里统称为指令）中的一条，这组指令是将一个整型数字存储到本地变量中。n代表的是局部变量区中的位置，并且只能是0,1,2,3。再多的话只能用另一条指令istore了，这条指令会接受一个操作数，对应的是局部变量区中的位置信息。
```

这条指令执行的时候，内存布局是这样的：

![Java字节码详解图1](/Users/li/Documents/note/noteImgPool/Java字节码详解图1.jpg)

每个方法都包含一个本地变量表，如果这段代码在一个方法里面的话，你会在方法的本地变量表中发现如下信息

![Java字节码详解图2](/Users/li/Documents/note/noteImgPool/Java字节码详解图2.jpg)

### 字段

****java类里面的字段作为类对象实例的一部分，存储在堆里(类变量存储在对应类对象里)。关于字段的信息会添加到类的field_info数组里，像下面这样

![Java字节码详解图3](/Users/li/Documents/note/noteImgPool/Java字节码详解图3.jpg)

另外如果变量被初始化了，那么初始化的字节码会添加到构造方法里。

下面这段代码编译之后：

```java
public class SimpleClass {
    public int simpleField = 100;
}
```

如果你用javap进行反编译，这个被添加到了field_info数组里的字段就会多了一段描述：

```java
public int simpleField;
    Signature: I
    flags: ACC_PUBLIC
```

初始化变量的字节码会被加到构造方法里，像下面这样：

```java
public SimpleClass();
  Signature: ()V
  flags: ACC_PUBLIC
  Code:
    stack=2, locals=1, args_size=1
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: aload_0
       5: bipush        100
       7: putfield      #2                  // Field simpleField:I
      10: return
```

上述代码执行的时候内存里面是这样的：

![Java字节码详解图4](/Users/li/Documents/note/noteImgPool/Java字节码详解图4.jpg)

这里的putfield指令的操作数引用的常量池里的第二个位置，JVM会为每个类型维护一个常量池，运行时的数据结构有点类似一个符号表，尽管它包含的信息更多。java的字节码操作需要对应的数据，但是这些数据太大了，存储在字节码里不合适，它们都被存储在常量池里，而字节码包含一个常量池的引用，当类文件生成的时候，其中一块就是常量池

```java
Constant pool:
   #1 = Methodref          #4.#16         //  java/lang/Object."<init>":()V
   #2 = Fieldref           #3.#17         //  SimpleClass.simpleField:I
   #3 = Class              #13            //  SimpleClass
   #4 = Class              #19            //  java/lang/Object
   #5 = Utf8               simpleField
   #6 = Utf8               I
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               LocalVariableTable
  #12 = Utf8               this
  #13 = Utf8               SimpleClass
  #14 = Utf8               SourceFile
  #15 = Utf8               SimpleClass.java
  #16 = NameAndType        #7:#8          //  "<init>":()V
  #17 = NameAndType        #5:#6          //  simpleField:I
  #18 = Utf8               LSimpleClass;
  #19 = Utf8               java/lang/Object
```

### 常量字段（类常量）

带有final标记的常量字段在class文件里会被标记成ACC_FINAL.

```java
public class SimpleClass {
    public int simpleField = 100;
}
```

字段的描述信息会标记成ACC_FINAL:

```java
public static final int simpleField = 100;
    Signature: I
    flags: ACC_PUBLIC, ACC_FINAL
    ConstantValue: int 100
```

对应的初始化代码并不变：

```java
4: aload_0
5: bipush        100
7: putfield      #2                  // Field simpleField:I
```

##### 静态变量

带有static修饰符的静态变量则会被标记成ACC_STATIC：

```java
public static int simpleField;
    Signature: I
    flags: ACC_PUBLIC, ACC_STATIC
```

不过在实例的构造方法中却再也找不到对应的初始化代码了。因为static变量会在类的构造方法中进行初始化，并且它用的是putstatic指令而不是putfiled。

```java
static {};
  Signature: ()V
  flags: ACC_STATIC
  Code:
    stack=1, locals=0, args_size=0
       0: bipush         100
       2: putstatic      #2                  // Field simpleField:I
       5: return
```

![Java字节码详解图4](/Users/li/Documents/note/noteImgPool/Java字节码详解图4.jpg)

