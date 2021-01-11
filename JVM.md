# Compiling for Java Virtual Machine

甲骨文公司的JDK软件包括了将使用java编程语言编写的源文件编译成JVM指令集的编译器。JVM自身会实现一个运行时系统。在涉及到将JVM指令集转换为CPU指令集的翻译过程中，也会使用到编译器。JIT代码生成器就是一个例子，它可以将JVM加载的代码转换成平台指定的指令集。

使用javac进行.class文件的生成，使用javap命令可以反编译生成可阅读的JVM识别的格式文件。

每条指令会被翻译成：

-- index -- | -- opcode -- | [ -- oprand1-- [ -- oprand2 -- ]] [ -- comment --]

index表示指令在当前方法数组中的索引

opcode表示指令操作代码的助记符，还有0个或者多个操作数

comment是可选的一部分由javap产生；一部分则由代码作者提供。

index也可以当做是控制转换的索引地址，例如：goto 8，跳转到第8行指令继续。

还有一种指令：9 invokevirtual #4，#4就是指向运行时常量池中某个常量的引用



## Accessing the Run-Time Constant Pool

许多数字常量，同样的像对象，域，和方法，可以通过当前类的运行时常量进行获取。对象的获取后面再考虑。int，long，float和double以及对String实例的引用数据，都是使用ldc，ldc_w，和ldc2_w指令进行管理的。

多方考证，对StringTable和SymbolType描述最符合实际的一篇文章--[聊聊jvm的StringTable及SymbolTable](https://blog.csdn.net/weixin_34360651/article/details/91460994)



[StringTable详解](https://blog.csdn.net/zuodaoyong/article/details/107296585)

字符串对象:

```java
String s0 = new String("abc"); //这玩意存放在堆中
//1.s0 = s0.intern();
String s1 = "abc"; //这玩意存放在StringTable中，而StringTable是在运行时常量池中，运行时常量池是由常量池在运行时生成的一块存放数据的区域，存放在Method Area中！！！！！
//2.s0 = s0.intern();
System.out.println(s0 == s1);
```

那么问题来了，在JDK8中上面的两个变量还相等吗？！显然不等啊，一个是对堆空间对象"a"的引用地址，另一个是对StringTable中"a"的地址，能一样吗？！！！那怎么才能一样呢？！那就是让s0调用一下intern();那么是加在1的位置还是2的位置，才能让他俩相等呢？！！！1和2的位置都一样的啦！！！！

StringTable在Method Area中G1垃圾回收器会对其中的常量进行回收处理的！



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

## The returnAddress Type and Values

returnAddress类型是JVM的jsr、ret和jsr_w指令使用的。returnAddress类型的值是一个指向JVM指令操作数的指针。不像是数字的基本类型，returnAddress类型不对应任何Java编程语言类型并且也不能被运行程序所修改。

## The boolean Type

尽管JVM定义了布尔类型，但只对其提供很有限的支持。没有JVM指令solely dedicated to（专用于）布尔值的操作。相反的是，Java编程语言对布尔值的操作被编译成对JVMint数据类型值的使用。

Java虚拟机确实直接支持布尔数组。他的newarray指令可以创建布尔数组。布尔类型的数组可以使用byte数组指令bload和bastore进行获取与修改。

在oracle的JVM实现中，布尔数组在java编程语言中被编码为JVMbyte数组，每个布尔元素使用8位长度。

Java虚拟机将布尔数组的组成成员用1表示true，0表示false进行编码。如果Java编程语言被编译器对应映射成JVM的int类型，那么编译器必须使用对应的编码。

## Reference Types and Values

有三种reference类型：class类型、array类型和interface类型。他们的值是指向被动态创建的类实例，数组或是实现接口的类实例或数组（顺序同上）。

一个数组类型由一个一维成员类型组成（它的长度不是类型指定的）。组成数组类型的成员类型可能本身就是一个数组类型。如果从任何数组类型开始，考虑其组件类型，然后（如果成员本身也是数组类型）该成员类型也是数组类型，and so on（以此类推），最终必须到达一个不是数组类型的组件类型；这成为数组类型的元素类型。一个数组的元素类型不是一个基本类型或是class类型就是一个接口类型。

一个reference值也可能是特殊值null的引用，一个指向非对象的引用，在这儿将会以null表示。null引用在最初的时候没有运行时类型，但是可能会被转换为任何类型。一个reference类型的默认值是null。

当前规范不强制使用具体的值编码为null。



## 1 Runtime Data Area（运行时数据区）

在一个程序的执行过程中，JVM定义了各种各样的运行时数组区，有些数据区在JVM启动时就被创建了，并且在JVM退出时会被销毁。其他的数据区则属于每个线程的。Per-thread（线程私有）的数据区在线程创建时创建，在线程退出时销毁。

### 1.1 Program Counter Register（PC寄存器）

JVM可以支持许多线程在同一时刻的执行（JLS）。每个JVM线程都有属于他自己的pc（program counter）注册器。在任何时间点，任意一个JVM线程都只执行单个方法中的代码，称作当前线程的当前方法。如果方法不是native，那么pc寄存器包含JVM当前被执行指令的地址。如果当前线程当前执行的方法是native，那么JVM的pc寄存器的值为未定义的。JVM的pc寄存器在指定的平台上有足够的宽度可以容纳returnAddress或者本地方法指针。



### 2.1 Java Virtual Machine Stacks（Java虚拟机栈）

每个JVM线程都有一个私有的JVM栈，与线程同时被创建。JVM栈保存了很多帧。JVM栈与传统语言栈类似，像是C语言：它持有本地变量与中间结果，并且是方法调用与返回过程的参与者。由于JVM栈从不直接被操作除非栈帧的入栈或出栈，帧可以由堆来分配。给JVM栈分配的内存空间不需要是连续的。

在JVM规范的最初版本中，JVM栈被称为Java栈。

当前规范允许JVM栈不是大小固定就是根据计算需要动态扩展或压缩。如果JVM栈是固定大小的话，那么每个JVM栈可以在创建时选择独立。

JVM的实现可以提供程序员或者用户，堆JVM栈初始大小的控制权，同时，在JVM栈动态扩展或压缩的情况下，控制最大最小值的权限。

以下是与JVM栈相关的异常情况：

当一个线程的计算需要一个比JVM栈所允许的更大的空间，JVM会抛出StackOverflowError。

当JVM栈可以被动态扩展，并且在试图扩展中没有足够的内存空间可以被使用来满足此次扩展，或者在新线程JVM栈初始化时没有足够的内存空间可以使用，则会抛出OutOfMemoryError。



#### 2.1.1 Frames（栈帧）

帧是用来存储数据和中间结果，同样的也是用来动态链接的执行，方法结果的返回，和异常的调度。

每当一个方法被调用就会对应的创建一个帧。当他的方法调用完成，那么对应的帧也被销毁了，不论这个方法运行正常或者时被打断（方法抛出一个未捕获的异常）。帧是从创建帧线程的JVM栈中分配的。每个帧都有其自己的本地变量表，其自己的操作数栈，和指向当前线程类的运行时常量池的指针。

帧可能会被附加指定实现的信息扩展，例如debugging信息。

本地变量表数组和操作数栈的大小在编译器就被确定下来了，并且会和与栈相关的当前方法代码被一同提供。因此帧的数据结构大小只取决于JVM的实现，并且帧结构的内存空间在方法被调用时（simultaneously同时的）就会被分配。

线程中有且只有当前被执行的方法的栈帧是激活状态。这个激活的帧叫做当前帧，并且与其相关的方法叫做当前方法。当前方法所在的类叫做当前类。本地变量表和操作数栈上的操作通常都持有者对当前帧的引用。

如果一个帧的方法调用了其他方法或者他的方法执行完成了，那么它就不再是当前帧了。当一个方法被调用，一个新的栈被创建并成为当前帧，控制权也转换到新的方法上。当方法返回，当前帧将其方法调用的结果返回，如果存在的话，给前一帧。当前帧会被丢弃，前一帧成为当前帧。

注意线程创建的帧就位于那个线程并且不能被其他线程引用。



##### 2.1.1.1 Local Variables（局部变量表）

每一帧包含了变量的数组就是常说的局部变量表。帧的本地变量表的长度在编译器就已经确定下来了，并且一同提供了和栈相关的类或者接口的二进制表示形式的方法代码。

一个独立的本地变量表可以持有boolean、byte、char、short、int、float、reference或者returnAddress类型的值。一对本地变量可以持有long或者double类型的值。

本地变量以索引的形式标明地址。本地变量的第一位的索引为0。

当且仅当整数比局部变量数组的大小比局部变量数组小0到1之间，才会被视为局部变量数组的索引。

long或者double的值占据着两个连续的本地变量。这样的值只会以较小的索引进行地址标记。举个例子



##### 2.1.1.2 Operand Stacks（操作数栈）



##### 2.1.1.3 Dynamic Linking（动态链接）





与栈帧相关的还有returnAddress和一些附加信息







### 3.1 Natvie Method Stack（本地方法栈）

JVM的实现可能会使用传统栈，colloquially（通俗地来讲）叫做“C栈”，用以支持native方法（那些使用非Java编程语言编写地语言）。本地方法栈也可能会被JVM的C语言指令集的解释器实现使用。JVM的实现不能加载native方法并且他们本身依赖传统栈不需要直通本地方法栈。如果提供了，本地方法栈也会是每个线程分配一个本地方法栈。

当前规范允许本地方法栈不是固定大小就是一个根据计算进行动态扩展或者压缩的空间。如果本地方法栈设置为固定大小，那么每个本地方法栈当其被创建的时，可能会是一个独立的空间。

一个JVM实现可能提供给程序员或者使用者控制初始本地方法栈大小的控制权，同样的，在本地方法栈可变大小的情况下，控制本地方法栈空间最大最小值的权限。

以下是与本地方法栈相关的异常情况：

当某个线程的计算需要比被允许大小更大的内存空间时，JVM抛出StackOverflowError。

当本地方法栈可以被动态扩展并且内存空间不足以满足本地方法栈的扩展，或者当没有足够的内存空间用来满足新线程本地方法栈的初始化时，JVM将抛出OutOfMemoryError。



### 4.1 Heap（堆空间）

JVM拥有一个所有JVM线程共享的堆空间。堆是运行时数据区为所有类实例和数组分配的空间。

堆空间在虚拟机启动时被创建。对象的堆空间存储是由自动存储管理系统回收的（也就是大家常说的垃圾回收器）；对象从来不显示释放。JVM不采用特定类型的自动存储管理系统，并且存储管理技术可以由实现者的系统需求进行选择。堆可能会时一个固定大小或者时根据计算扩展或压缩的。堆的内存空间不需要是连续的。

JVM实现可提供给程序员或者用户控制初始堆空间大小的权限，同样的，如果堆空间可以被动态扩展或者压缩，也要提供对最大最小值的控制权限。

以下是与堆空间相关的异常情况：

如果如果一个计算需要比自动存储管理系统能提供的更多的堆空间，那么JVM会抛出OutOfMemoryError。



#### 4.1.1 Eden(新生代)



#### 4.1.2 Survior[From]（幸存者0区）



#### 4.1.3 Suvivor[To]（幸存者1区）



#### 4.1.4 Tenure（老年代）



#### 4.1.5 Permanent（永久代）

JDK1.7之后已经被移除了，取而代之的是MetaSpace





JDK1.7中已经将字面量，静态变量以及符号引用存放在了对象信息中，也就是放在了堆空间中，JDK1.8之后，永久代中的数据被转移到元空间中，元空间使用直接内存，就没有OOM:PermGen的情况出现了，减少了GC次数与OOM的情况发生。









### 5.1 MetaSpace（元空间）







### 6.1 Method Area（方法区）

JVM拥有所有JVM线程共享的方法区。所谓方法区就像是传统语言编译代码或类似于操作系统处理中的文本片段的存储区域。它存储每个class的结构，像是运行时常量池，域和方法数据，还有方法和构造器的编码，包括类或者实例和接口初始化使用的特殊方法。

方法区在虚拟机启动时就被创建了。尽管方法区在逻辑上是堆空间的一部分，JVM简单的实现可能会选择既不会对其进行垃圾回收也不会进行空间压缩。当前规范不会强制安排方法区的位置或者管理编译后代码的策略。方法区可以是一个固定大小或者是根据计算需求扩展或者压缩。方法区的内存空间可以是不连续的。

JVM实现可提供程序员或者用户堆方法区初始大小的控制权限，同样的，在不同大小方法区的情况下，控制其最大最小空间的权限。

以下是与方法区相关的异常情况：

如果内存空间不足以支撑方法区的按需分配，那么JVM会抛出OutOfMemoryError。



#### 6.1.1 Run-Time Constant Pool（运行时常量池）

运行时常量池是每个类或者每个接口.class文件中constant_pool表的运行时表示。它包含了几种常量，范围包括编译时期方法中的数字字面量和必须是在运行时解析出的方法引用。运行时常量池的功能类似于传统编程语言中的符号表，尽管它包含比传统符号表更大范围的数据。

每个运行时常量池都是由JVM的方法区分配的。当JVM创建class或者interface时，类或者接口的运行时常量池就会被构造出来。

以下是与class类或者interface接口的运行时常量池构造相关的异常情况：

当创建一个class或者interface，如果运行时常量池的构造需要比JVM方法区中能获得的内存空间更大时，JVM会抛出OutOfMemoryError。

通过Loading，Linking，and Initializing这一部分的相关讲解，可以获取更多有关运行时常量池构造的信息。

由Constant Pool在JVM运行时期产生。



## 2 Instruction Set Summary





# Just In Time（即时编译器）

java是解释与编译相结合的语言，由逃逸分析而生的标量替换、同步省略、栈上分配进行对效率进行了优化。













































# Garbage Collector（垃圾收集器）

GC发生的主要区域在Heap区和Method Area，

# 垃圾回收

## 判断对象是否可回收

### 引用计数算法

无法解决对象的循环引用问题，所以我们Java使用的垃圾回收器中，并不使用此算法进行对象是否为垃圾的判断。听说python中的垃圾回收使用的是此算法。



### 可达性分析算法

通过一系列的称为GC Roots的对象作为起始点,从这些节点开始向下搜索,搜索所经过的路径称为引用链（Reference Chain），当一个对象到GC Roots没有任何引用链相连（在图论中称为对象不可达）时，这个对象就是不可用的。找出所有可用对象，其余空间为”无用“。

那我们就来看看GC Roots，首先那些可以被称之为"Roots"：

​	--	java虚拟机栈中当前活跃的栈帧中的引用（引用类型参数、局部变量、临时值）

​	--	本地方法栈中的JNI（native方法）中的引用

​	--	方法区中的静态变量、常量的引用

​	--	所有当前被加载的Java类

​	-- 	对于分代收集来说，如果在进行minor GC/young GC时，那么从old gen中指向young gen中的引用就必须作为minor GC/young 			GC的GC Roots的一部分。







## 引用类型

### 强引用

不会被垃圾回收器回收，引起OOM的最主要原因

### 软引用

堆空间不够则回收，够用则不回收

### 弱引用

[弱引用的场景与示例](https://www.cnblogs.com/absfree/p/5555687.html)

只要垃圾回收器工作就回收

### 虚引用

[虚引用的场景与示例](https://www.cnblogs.com/mfrank/p/9837070.html)

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





## GC方法

System.gc()是Full GC，但是不建议使用，finalize()，可用于复活对象，只能调用一次





### HotSpot VM

其中的GC分为两种:

​	--	partial GC

​			并不回收整个GC堆的模式

​			--	Young GC : 只收集young gen年轻代

​			--	Old GC：只收集old gen老年代，只有CMS的Concurrent Collection是这个模式

​			--	Mixed GC：收集整个young gen、以及部分的old gen。只有G1有这个模式。

​	--	Full GC

​			收集整个堆空间，包括young gen、old gen、perm gen(如果存在的话)等所有部分的模式。





# JMX

Java Management Extension，Java管理扩展。









