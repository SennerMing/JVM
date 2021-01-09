# Compiling for Java Virtual Machine





# The .class File Format





# Loading，Linking And Initializing（加载链接以及初始化）

## 1 Java Virtual Machine Startup（java虚拟机的启动）



## 2 Creation And Loading（创建与加载）

ClassLoader分为两类：启动类加载器和用户定义类加载器

### 2.1 Loading using the Bootstrap Class Loader（使用启动类加载器）

BootstrapClassLoader（启动类加载器）

### 2.2 Loading using a User-Defined Class Loader（使用用户定义类类加载器）

ExtClassLoader（扩展类加载器）

AppClassLoader（应用类加载器）

### 2.3 Creating Array Classes（创建数组类）



### 2.4 Loading Constraints（加载约束）



### 2.5 Deriving a Class from a class File Representation（从类的.class文件中派生出类对象）



## 3 Linking（链接）

### 3.1 Verification(校验)



### 3.2 Preparation（准备）



### 3.3 Resolution（解析）

很重要，可以说与程序的运行息息相关

#### 3.3.1 Class and Interface Resolution



#### 3.3.2 Field Resolution



#### 3.3.3 Method Resolution



#### 3.3.4 Interface Method Resolution 



#### 3.3.5 Method Type and Method Handle Resolution



#### 3.3.6 Call Site Specifier Resolution



### 3.4 Access Control



### 3.5 Overriding





## 4 Initializing（初始化）

理解The Run-Time Constant pool 与 per-type contstant pool





# The Structure of Java Virtual Machine（Java虚拟机结构）





## 1 Runtime Data Area（运行时数据区）



### 1.1 Program Counter Register（PC寄存器）







### 2.1 Java Virtual Machine Stacks（Java虚拟机栈）

#### 2.1.1 Stack Frames（栈帧）



##### 2.1.1.1 Local Variables（局部变量表）



##### 2.1.1.2 Operand Stacks（操作数栈）



##### 2.1.1.3 Dynamic Linking（动态链接）





与栈帧相关的还有returnAddress和一些附加信息







### 3.1 Natvie Method Stack（本地方法栈）









### 4.1 Heap（堆空间）

#### 4.1.1 Eden(新生代)



#### 4.1.2 Survior[From]（幸存者0区）



#### 4.1.3 Suvivor[To]（幸存者1区）



#### 4.1.4 Tenure（老年代）



#### 4.1.5 Permanent（永久代）

JDK1.7之后已经被移除了，取而代之的是MetaSpace





JDK1.7中已经将字面量，静态变量以及符号引用存放在了对象信息中，也就是放在了堆空间中，JDK1.8之后，永久代中的数据被转移到元空间中，元空间使用直接内存，就没有OOM:PermGen的情况出现了，减少了GC次数与OOM的情况发生。









### 5.1 MetaSpace（元空间）







### 6.1 Method Area（方法区）



#### 6.1.1 Run-Time Constant Pool（运行时常量池）

由Constant Pool在JVM运行时期产生。



## 2 Instruction Set Summary





# Just In Time（即时编译器）

java是解释与编译相结合的语言，由逃逸分析而生的标量替换、同步省略、栈上分配进行对效率进行了优化。









# Garbage Collector（垃圾收集器）

GC发生的主要区域在Heap区和Method Area，





# The Structure Of Java Virtual Machine

当前说明文档规定了一个抽象的(Java虚拟)机器。这些规范并不是针对某个特定的Java Virtual Machine的实现。

为了正确的实现Java虚拟机，你只需要能够读懂.class文件格式并且能够正确的执行其中指定的操作指令即可。实现的细节并不是"Java Virtual Machine's sepcification"的一部分，因为这将会不必要的产生一些对实现者创造力的制约。举个例子，像是Run-Time data Areas（运行时数据区）的内存布局设计，garbage-collection（垃圾回收）算法的使用，并且任何对Java Virtual Machine指令的内部调优（像是将这些指令转换为machine code(机器码)）等等这些，都将决定权留给我们的实现者。

## The .class File Format

Java Virtual Machine要执行的编译后代码，以使用硬件与操作系统独立的二进制格式的表现方式，通常来说（但不必要）存储在一个文件当中，这个文件就是大家常说的class格式类型的文件。class文件类型精确的定义了Class或者Interface的表现形式，包括像是可能会在特定平台中be taken for granted in（被视为是理所当然）的byte ordering（字节序）等等细节。

## Data Types

像是Java编程语言，Java虚拟机的操作都是对于两种数据类型，一种是Primitive Types（基本数据类型）还有一种是Reference Types（引用类型）。相应的两种数据类型的值可以被存储到变量中，作为参数被传递，作为方法的返回值，并且和原始类型与引用类型的值做一些操作。

Java虚拟机期望在处于运行时之前对所有的类型进行检查，通常是由编译器做此操作，而不必由Java虚拟机本身去做。基本类型的值不需要被标记或是可以在运行时进行检查以确认他们的类型，或者是与引用类型的值区分开来。相反的，Java虚拟机使用指定类型的操作指令来区分其操作数类型。对于实例，iadd, ladd, fadd, 和 dadd都是Java虚拟机指令，用来对两个数字进行加运算并且产生数字结果，但是这些指令都是为int,long,float和double类型准备的（与上面的指令顺序相同）。对Java虚拟机支持的类型的指令集总结请看2.11.1。

Java虚拟机包含对对象的显示支持，一个对象不是一个动态分配的class实例就是一个数组。对对象的引用被认为是Java虚拟机的引用类型。引用类型的值可以被称作是指向对象的指针。一个对象可能存在多个指向其的引用。对对象的操作像是传递或者是测试，都是通过引用传递的方式进行的。

## Primitive Types and Values

Java虚拟机支持的基本数据类型包括数字类型，布尔类型和returnAddress类型。

数字类型由整数类型和浮点数类型组成。

整数类型有：

byte，值是8位带符号的二进制补码整数，默认值为零。

short，值是16位带符号的二进制补码整数，默认值为零。

int，值是32位带符号的二进制补码整数，默认值为零。

long， 值是64位带符号的二进制补码整数，默认值为

char，值是16位无符号整数，表示[基本多文种平面](https://baike.baidu.com/item/%E5%9F%BA%E6%9C%AC%E5%A4%9A%E6%96%87%E7%A7%8D%E5%B9%B3%E9%9D%A2/10788078?fr=aladdin)中的Unicode码点，默认值为null码点('\u0000')

浮点类型有：

float，值是浮点数的集或是浮点扩展指数值的集（如果支持的话）的元素，默认值是一个符号为正的0

double，值是双精度数的集或是双精度扩展指数值的集（如果支持的话）的元素，默认值是一个符号为正的0

布尔类型的值是对真值true和false的编码，默认值是false。

returnAddress类型的值是一个对Java虚拟机指令的指针。对于基本类型来说，只有returnAddress类型与Java编程语言类型没有直接的关联（在使用Java编写相关程序的时候，你不会使用到该类型）













































# 垃圾回收

## 判断对象是否可回收

### 引用计数算法



### 可达性分析算法



## 引用类型

### 强引用

不会被垃圾回收器回收，引起OOM的最主要原因

### 软引用

堆空间不够则回收，够用则不回收

### 弱引用

只要垃圾回收器工作就回收

### 虚引用

无法通过引用获取对象，只做垃圾回收工作追踪使用





## 常见垃圾回收算法

### 复制算法

适用于垃圾产生较多的区域（也就是说总空间中只有少量的对象不会被回收）

### 标记清除算法

### 标记清除整理算法

### 分代收集算法

## 常见的垃圾收集器

### Serial/Serial Old收集器

### ParNew收集器

### Parallel Scavenge收集器

### Parallel Old收集器

### CMS（Current Mark Swep）

### G1收集器



















