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

.java file →javac →.class file → Class Loader Sub System[Loading（bootstrap、extension、application）| Linking（verifycation→preparation→resolution） | Initialization]

Bootstrap class loader加载rt.jar，将所有需要的材料添加到运行时环境中

Extension class loader加载jre/lib/ext中的jar

Application class loader加载ClassPath下用户定义的类





## 1 Java Virtual Machine Startup（java虚拟机的启动）



## 2 Creation And Loading（创建与加载）

ClassLoader分为两类：启动类加载器和用户定义类加载器

Loading阶段主要工作是从各种源中将类加载到Method Area，生成大Class对象，为接下来的操作提供材料。

1.通过类的全限定名获取这个类对应的byteCode二进制流

“材料”获取的途径：

​	--	jar、war包

​	--	网络二进制流中

​	--	动态获取，例如：动态代理使用的动态生成字节码

2.将二进制流转换成Method Area存储格式的，作为原始资料放入Method Area

3.将Method Area 的Class信息解析生成堆空间的大Class对象，供运行时操作，作为方法区各种数据访问的入口



### 2.1 Loading using the Bootstrap Class Loader（使用启动类加载器）

BootstrapClassLoader（启动类加载器）

### 2.2 Loading using a User-Defined Class Loader（使用用户定义类类加载器）

ExtClassLoader（扩展类加载器）

AppClassLoader（应用类加载器）

### 2.3 Creating Array Classes（创建数组类）



### 2.4 Loading Constraints（加载约束）



### 2.5 Deriving a Class from a class File Representation（从类的.class文件中派生出类对象）



## 3 Linking（链接）



进行Linking操作的必要条件及必要操作：

​	--	A class or interface is completely loaded before it is linked

​	--	A class or interface is completely verified and prepared before it is initialized

​	--	Errors deteced during linkage are thrown at a point in the program where some action is taken by the program that might,directly or indirectly, require linkage to the class or interface involved in the error

举个例子，一个JVM实现可能会选择独立的解析类或者接口中的每一个符号引用，当这个类或者接口使用（"lazy"或者"late"的方案），或者在class是验证（"eager"或者"static"的方案）情况下时立刻被加载。这意味着resolution处理可能继续，在某些实现中，在类或者接口已经被初始化后。无论接下来采取哪种策略，任何在resolution过程中检测到的error必须被抛出，（在程序中直接或者间接使用对类或者几口的符号引用的时间点）。

由于Linking操作需要对新的数据结构进行内存空间的分配，可能会因此引发OutOfMemoryError！

### 3.1 Verification(校验)

verification之前的操作还有两个重要的操作

#### 3.1.1 Format Checking

​	--	The first four bytes must contain the right magic number.

​	-- 	All recognized attributes must be of the proper length.

​	--	The class file must not be truncated（缩减的，删节的） or have extra bytes at the end.

​	--	The constant pool must satisfy the constraints documented throughout（自始至终的）（The Constant Pool就是满足常量池的相关约束）

​	--	All field references and method references in the constant pool must have valid names, valid classes, and valid descriptors.

format checking不保证指定类中的指定的field或者method实际是存在的，也不保证描述符指向实际存在的类。format checking只保证以上这些东西是格式正确的。在bytecodes自身被校验和解析中会进行更多细节的检查。

这些对基本class格式文件的检查，对class格式文件内容的翻译来说是有必要的。Format checking有别于bytecode的verification，在历史上由于他们都是一种完整性校验的形式而经常被人们混淆

#### 3.1.2 Constraints Checking

constraints on java virtual machine code，先来介绍一下

一个方法、实例的初始化方法，亦或是类或者接口的初始化方法都存储在.class文件结构中method info的Code属性中的code数组中(.class→method_info→Code属性→code数组)

下面介绍下与Code_attribute结构中内容相关的约束。

##### 3.1.2.1 Static Constraints

static cnstraints，类文件上的静态约束表示文件格式良好的约束。除了对.class文件中代码的静态约束，这些约束在上一个部分Format Checking中已经给出。.class文件中对code的静态约束详细说明了，JVM指令应该怎样在code数组中布局和每个独立指令的操作数应该是怎样的。

下面是code数组中对指令的静态约束：

​	--	只有在 [6.5章 Instruction](https://docs.oracle.com/javase/specs/jvms/se8/jvms8.pdf) 中提到的指令才能出现在code数组中。只有[6.2章 opcodes](https://docs.oracle.com/javase/specs/jvms/se8/jvms8.pdf)中指定的操作码才能出现，任何文档中没有定义的指令和操作码都不能出现在code数组中。如果class格式文件的版本号是51.0及以上，那么无论是操作码jsr或者是jsr_w都不会出现在code数组中。

​	--	在code数组中第一个指令的操作码的索引为0

​	--	对于code数组中的每一个指令除了最后一个，下一个指令的操作码的索引等于当前指令操作码的索引加上该指令的长度，包括其所有的操作码。出于以下目的wide指令将会像其他任何指令一样被处理；  指定宽指令要修改的操作的操作码将会被当作该wide指令的操作数中的一员进行处理。

​	--	在code数组中的最后一个指令的最后一个字节的索引必须是code_length-1

以下是在code数组中对指令中操作数的静态约束：

​	--	在当前方法中每个跳转或者是分支指令都必须是这些（jsr, jsr_w, goto, goto_w, ifeq, ifne, ifle, iflt, ifge, ifgt, ifnull, ifnonnull, if_icmpeq, if_icmpne, if_icmple, if_icmplt, if_icmpge, if_icmpgt, if_acmpeq, if_acmpne）。以跳转或者分支的指令必须不能是被用来指定wide指令修改的操作的操作码。

​	--	在当前方法中，对于每个[tableswitch](https://blog.csdn.net/john1337/article/details/88420273)指令的每个目标，包括默认，必须是指令操作码。同样也适用于lookupswitch指令，对操作数的规范。

​	--	操作数指令ldc和ldc_w指令，必须是正确的常量池表的索引。不同version的.class文件中对一些常量的使用是不同的，需要注意。

更多内容可以参照[JVM规范4.9.1](https://docs.oracle.com/javase/specs/jvms/se8/jvms8.pdf)大多是对指令的操作数和操作码进行规范的，保证索引，对常量池的引用的正确性，保证代码符合规范。



##### 3.1.2.2 Structural Constraints

code数组中的structural constraints指定了JVM指令之间的关系。

以下是一些structual constraints：

​	--	每个指令的执行，必须满足在操作数栈和本地变量表中存在合适的类型与数量的参数，不管导致他被调用的执行路径是什么。对int类型的操作指令，也同样适用与操作boolean、byte、char和short类型的值。

​	--	在不同的执行路径下执行一个指令，在指令的执行之前，操作数栈必须拥有同样的深度。

​	--	在执行期间的任何时候，操作数栈增长的深度都不能超过max_stack指定的深度。

​	--	任何指令的执行都不能弹出比该指令包含的在操作数栈中的更多的值。

​	--	在分配值之前，局部变量不能被获取

更多内容可以参照[JVM规范4.9.2](https://docs.oracle.com/javase/specs/jvms/se8/jvms8.pdf)大多是对操作指令进行的规范，像是special和virtual方法的调用，方法的返回值，操作数栈的规范等，保证代码指令间相互的引用没什么问题。

Format Checking与Constural Checking结束，那么就正式开始Verfication。

尽管Java编程语言的编译器必须只生成满足前面部分中所有静态和结构约束的类文件，但Java虚拟机不能保证它被要求加载的任何文件都是由该编译器生成的或格式正确。像web浏览器这样的应用程序不下载源代码，然后编译源代码；这些应用程序下载已经编译的类文件。文件浏览器需要确定类文件是由可信的编译器生成的，还是由试图利用Java虚拟机的对手生成的。

​	--	编译时检查的另一个问题是版本倾斜。用户可能已经成功地将一个类（比如PurchaseStockOptions）编译为TradingClass的子类。但是TradingClass的定义可能已经改变了，因为该类的编译方式与现有的二进制文件不兼容。方法可能已被删除或其返回类型或修饰符已更改。字段可能已经更改了类型或从实例变量更改为类变量。方法或变量的访问修饰符可能已从public更改为private。有关这些问题的讨论，请参阅Java语言规范JavaSE8版第13章“二进制兼容性”。

由于这些潜在的问题，Java虚拟机需要自己验证它试图合并的类文件是否满足了所需的约束。Java虚拟机的实现验证每个类文件在linking time链接时是否满足必要的约束（§5.4）。

链接时验证提高了运行时解释器的性能。可以消除在运行时为验证每个已解释指令的约束而必须执行的昂贵检查。Java虚拟机可以假设已经执行了这些检查。例如，Java虚拟机已经知道以下内容：

​	--	没有操作数堆栈溢出或下溢。

​	--	所有局部变量的使用和存储都是有效的。

​	--	所有Java虚拟机指令的参数都是有效类型。

Java虚拟机实现可以使用两种策略进行验证：

​	--	必须使用类型检查验证版本号大于或等于50.0的类文件。

​	--	除了那些符合 Java ME CLDC和Java Card概要文件的Java虚拟机实现之外，所有Java虚拟机实现都必须支持类型推断验证，以便验证版本号小于50.0的类文件。支持 Java ME CLDC和Java Card配置文件的Java虚拟机实现的验证由它们各自的规范管理。

在这两种策略中，验证主要涉及对代码属性的代码数组（§4.7.3）实施§4.9中的静态和结构约束。但是，在验证过程中必须执行代码属性之外的三个附加检查：

​	--	确保最终类不是子类。

​	--	确保最终方法不被覆盖（§5.4.5）。

​	--	检查每个类（对象除外）是否有一个直接超类。



Verification操作是用来确保类或者接口的二进制形式结构是正确的。Verification可能会导致附加类或者接口的加载，但是并不需要对他们进行验证或者准备操作

如果类或者接口的二进制形式不满足静态或者结构性约束，那么将会在不满足条件的类或者接口的点抛出VerifyError

如果JVM尝试验证一个类或者接口由于抛出了LinkageError（或者是它的子类Error）而失败，那么接下来的对该类或者接口的验证尝试也将会失败，并抛出同样的错误。

##### 3.1.2.3 Verification by Type Checking

版本号为50.0或以上（§4.1）的类文件必须使用本节给出的类型检查规则进行验证。

如果并且仅当类文件的版本号等于50.0，那么如果类型检查失败，Java虚拟机实现可以选择尝试通过类型推断执行验证（§4.10.2）。

​	--	这是一个务实的调整，旨在缓和向新的核查纪律的过渡。许多操作类文件的工具可能会以需要调整方法堆栈映射帧的方式更改方法的字节码。如果工具没有对堆栈映射帧进行必要的调整，那么即使字节码原则上是有效的，类型检查也可能失败（因此将在旧的类型推断方案下进行验证）。为了让实现者有时间调整他们的工具，Java虚拟机实现可能会退回到较旧的验证规程，但时间有限。

​	--	在类型检查失败但调用类型推断并成功的情况下，预期会有一定的性能损失。这样的处罚是不可避免的。它还应该向工具供应商发出信号，表明他们的输出需要调整，并为供应商提供额外的动力来进行这些调整。

​	--	总之，通过类型推断将故障转移到验证支持将堆栈映射帧逐渐添加到Java SE平台（如果50.0版类文件中不存在这些帧，则允许进行故障转移）以及从Java SE平台中逐渐删除jsr和jsr_w指令（如果它们存在于50.0版类文件中，允许故障转移）。

如果Java虚拟机的实现曾经尝试对version50.0类文件执行类型推断验证，那么在类型检查验证失败的所有情况下，它都必须这样做。

​	--	这意味着Java虚拟机实现不能选择在一种情况下而不是在另一种情况下使用类型推断。它必须拒绝未通过类型检查进行验证的类文件，或者在类型检查失败时始终故障转移到类型推断验证器。

类型检查器强制执行通过Prolog子句指定的类型规则。英语文本是用来描述类型规则的非正式方式，而Prolog子句提供了一个正式的规范

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

首先来个正规的介绍，what is garbage collector?

垃圾收集器可以自动的管理应用的动态内存分配需求。

垃圾收集器通过以下操作来执行自动动态内存管理：

​	--	从操作系统中分配和归还内存

​	--	将内存分发给有需要的应用

​	--	确定哪一部分的内存还被应用所使用

​	--	回收不用的内存给应用重新使用

Java的HotSpot垃圾收集器使用各种各样的技术来提高效率，比如以下的操作：

​	--	使用generational（代与代之间）的scavenging（清理）与aging（老化）in conjunction with（相结合）的方式，将精力集中在堆中最有可能包含大量可回收内存区域的地方

​	--	使用多线程来aggressively（积极的）让操作并行，或者在程序的后台并发的执行一些long-running（长时间运行）的操作

​	--	尝试通过压缩存活对象来recover(恢复)更大的连续自由内存空间







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









