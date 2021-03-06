#第2章 Java内存区域
--------------------------------
内存动态分配  垃圾收集技术

###Java虚拟机管理的内存包含以下几个运行时数据区域
	1. 程序计数器
       这个区域是线程隔离的，表示当前线程执行的字节码的行号指示器，是“线程私有”的内存。
	   如果正在执行的是一个Java方法，那么计数器记录的是执行的虚拟机字节码指令的地址；如果执行的Native方法那么则为空
	2. Java虚拟机栈
       线程私有的，每个Java方法执行的时候都会创建一个栈帧，存储局部变量表、操作数栈、动态链接、方法出口等信息。
	   局部变量表存放基本数据类型、对象引用类型。long和double类型占据2个局部变量空间(Slot)，其余的占用一个。局部变量表所需的内存空间在编译期间就完成分配。
	3. 本地方法栈
       和虚拟机栈的作用类似，虚拟机栈为执行Java方法服务的，而本地方法栈是为执行Native方法服务的。
	4. Java堆(GC堆)
       被所有线程共享，在虚拟机启动时创建。用来存放对象实例，所有的对象实例和数组都要在堆上分配。
	   从内存回收的角度来看：可以分为 新生代和老生代。细分点为：Eden空间、From Survivor空间、To Survivor空间。
	   甚至还可以存在多个线程私有的分配缓冲区TLAB
	5. 方法区
       线程共享的，用户存储已经被虚拟机家在的类信息、常量、静态变量、JIT编译后的代码等
	6. 运行时常量池
       是方法区的一部分，用于存放编译期生成的各种字面量和符号引用，这部分内容在类加载后进入运行时常量池。
       运行期间也可以将新的常量放入池中。 如String类的intern()。
	7. 直接内存
       并不是虚拟机运行时数据区的一部分，但是频繁被使用。基于通道channel和缓冲区Buffer的NIO类可以使用Native函数库直接分配堆外内存，然后通过Java堆中的DirectByteBuffer对象作为这块内存的引用进行操作。这样子能够显著提高性能。

###对象创建的过程
	1. 虚拟机遇到new指令时候，检查这个指令的参数是否能在常量池中定位到一个类的符号引用，并且检查这个符号引用代表的类是否已被加载、解析和初始化过。
	2. 接下来将为新生对象分配内存，根据内存是否规整的分为“指针碰撞”和空闲列表“方法。另外还需要考虑线程安全性，一种是对分配内存空间的动作进行同步处理，另一种是把内存分配的动作划分在不同的空间里，即上面提到的在堆中为每个线程分配一个缓冲TLAB，线程申请内存的时候现在TLAB上分配，只有TLAB用完才同步锁定。
	3. 对对象进行必要的设置，这里主要指的是对象头。
	4. 执行<init>方法。

###对象的内存布局
	对象在内存中的布局分为：对象头、实例数据和对齐填充。
	1. 对象头包括对象自身的运行时数据(Mark Word)， 另外还包含类型指针，即指向它的类元数据的指针，虚拟机通过这个指针可以确定这个对象是哪个类的实例。如果对象是数组的话，对象头还需要保存数组的长度。
	2. 实例数据是对象真正的有效数据，相同宽度的字段总是会被分配到一起，父类的变量出现在子类之前。
	3. 对齐填充，对象的大小必须是8字节的整数倍。

###对象的定位访问
	1. 使用句柄访问。堆中会有一块区域作为句柄池，reference存储的是对象的句柄地址，句柄中包含了对象实例数据与类型数据各自的地址。
	2. 使用直接指针。那么reference存放的就是对象的直接地址，Java堆就还要考虑存放访问类型数据的相关信息。
使用句柄的好处就是稳定，对象变化的话reference无需变化。而使用指针的话就是效率高，速度快。


#第3章 垃圾收集器与内存分配策略
------------------------------
主要解决 1.哪些内存需要回收 2.什么时候回收  3.如何回收

第二章我们分析了Java运行时区域，垃圾收集器主要是是堆和方法区进行处理

###引用计数算法
现在很少使用引用计数算法，很难解决对象的循环引用问题。

###可达性分析算法
通过一系列成为GC Roots的对象作为起始点，搜索走过的路径称为引用链，当对象到GC Roots没有任何的引用链证明此对象不可用。
可作为GC Roots对象的几种：
	1. 虚拟机栈中引用的对象
	2. 方法区中类静态属性引用的对象
	3. 方法区中常量引用的对象
	4. 本地方法栈JNI引用的对象

###再谈引用
	Java对引用分为：
	1. 强引用。垃圾收集器永远不会回收被引用的对象
	2. 软引用。 SoftReference 描述一些还有用但并非必须的对象。系统将要发生内存溢出的异常之前，会把这些对象进行第二次回收。
	3. 弱引用。WeakReference 描述非必需对象 垃圾收集器工作时就会回收弱引用关联的对象。
	4. 虚引用。PhantomReference 唯一目的就是能在这个对象被回收的时候收到一个系统通知，不对生存时间构成影响。

###生存还是死亡

