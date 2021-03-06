JVM 运行时数据区域:  程序计数器, Java虚拟机栈, 本地方法栈, Java堆(Heap), 方法区(Method Area), 直接内存;  

### 程序计数器 (Program Counter Register)   

程序计数器:  
程序计数器是一块较小的内存空间, 它可以看作是, 当前线程所执行的字节码, 的行号指示器;  
在虚拟机的概念模型里, 字节码解释器工作时, 就是通过改变这个计数器的值, 来选取下一条, 需要执行的字节码指令;  
执行分支, 循环, 跳转, 异常处理, 线程恢复等基础功能, 都需要依赖这个计数器来完成;  

程序计数器 与 多线程:  
由于Java 虚拟机的多线程, 是通过线程轮流切换, 并分配处理器执行时间, 的方式来实现的, 在任何一个确定的时刻, 一个处理器(对于多核处理器来说是一个内核),  
都只会执行一条线程中的指令, 因此, 为了线程切换后能恢复到, 正确的执行位置, 每条线程都需要, 有一个独立的程序计数器,   
各各线程之间的计数器, 互不影响, 独立存储, 我们称这类内存区域, 为“线程私有”的内存;  

如果线程正在执行的是一个Java 方法, 这个计数器记录的, 是正在执行的, 虚拟机字节码指令的地址;  
如果正在执行的是 Native 方法, 这个计数器值则为空(Undefined);  
此内存区域是, 唯一一个在Java 虚拟机规范中, 没有规定任何 OutOfMemoryError 情况的区域;  

### Java虚拟机栈 (Java Virtual Machine Stacks)  
与程序计数器一样, Java 虚拟机栈(Java Virtual Machine Stacks), 也是线程私有的, 它的生命周期与线程相同;  
每个方法被执行的时候, 都会同时创建一个栈帧(Stack Frame), 用于存储: 局部变量表, 操作数栈, 动态链接, 方法地址等信息;  
每一个方法从被调用, 到执行完成的过程, 就对应着一个栈帧, 在虚拟机栈中, 入栈和出栈的过程;  

局部变量表:  
局部变量表存放了, 编译期可知的, 各种基本数据类型, 对象的引用(指针);  
局部变量表所需的内存空间, 在编译期间完成分配, 在方法运行期间不会改变;    
其中64位长度的 long 和 double 类型的数据, 会占用2个局部变量空间(Slot), 其余的数据类型只占用1个;  
 
操作数栈:   
当一个方法刚执行的时候, 这个方法的操作数栈是空的, 在方法执行的过程中, 会有各种指令进行, 入栈和出栈;  

动态链接:  
每个栈帧都包含一个, 指向运行时常量池中, 该栈帧所属的方法的引用, 持有这个引用, 是为了支持方法在调用过程中的动态链接(Dynamic Linking);  
在 Class 文件的常量池中, 存有大量的符号引用, 字节码中的方法调用指令, 就以常量池中, 指向方法的符号引用作为参数;   
这些符号引用, 一部分会在类加载阶段, 或者第一次使用的时候, 就转化为直接引用, 这种转化称为静态解析;  
另外一部分, 将在每一次运行期间, 转化为直接引用, 这部分称为动态连接;   
动态链接, 是把符号形成的方法调用, 翻译成实际的方法调用, 装在必要的, 类解释还没有定义的符号;  
并把变量访问编译成, 与这些变量运行时, 的存储结构相应的偏移地址;    
动态链接方法和变量, 使得方法中使用的其他类的变化, 不会影响到本程序代码;  

返回地址:  
当一个方法开始执行后, 只有两种方式可以退出这个方法:   
第一种方式是, 执行引擎遇到任意一个, 方法返回的字节码指令, 这时候可能会有返回值, 传递给上层的方法调用者;  
是否有返回值和返回值的类型, 将根据遇到方法返回指令来决定, 这种退出方法的方式称为正常完成出口;  

另外一种退出方式是, 在方法执行过程中遇到了异常, 并且这个异常, 没有在方法体内得到处理;  
无论是Java虚拟机内部产生的异常, 还是代码中, 使用 throw 字节码指令产生的异常, 只要在本方法, 的异常表中, 没有搜索到匹配的异常处理器, 就会导致方法退出;  
这种退出方法的方式, 称为异常完成出口, 一个方法使用异常完成出口的方式退出, 是不会给它的上层调用者产生任何返回值的;  

无论采用何种退出方式, 在方法退出之后, 都需要返回到方法被调用的位置, 程序才能继续执行;  
方法返回时, 可能需要在栈帧中保存一些信息, 用来帮助恢复它的上层方法的执行状态;   
一般来说, 方法正常退出时, 调用者的 PC 计数器的值, 可以作为返回地址, 栈帧中很可能会保存这个计数器值;   
而方法异常退出时, 返回地址, 是要通过异常处理器表来确定的, 栈帧中一般不会保存这部分信息;  

方法退出的过程, 实际上就等同于把当前栈帧出栈, 因此退出时可能执行的操作有:  
恢复上层方法的, 局部变量表和操作数栈;  
把返回值(如果有的话), 压入调用者栈帧的操作数栈中;  
调整PC计数器的值, 指向方法调用指令, 后面的一条指令等;  


### 本地方法栈  
本地方法栈(Native Method Stacks), 与虚拟机栈所发挥的作用是非常相似的,  
其区别不过是, 虚拟机栈为虚拟机执行Java 方法(也就是字节码)服务, 而本地方法栈, 则是为虚拟机使用到的 Native 方法服务;  
本地方法栈区域, 也可能会抛出 StackOverflowError 和 OutOfMemoryError 异常;  

### Java堆(Heap)  
对于大多数应用来说, Java 堆是 Java 虚拟机所管理的内存中, 最大的一块, Java 堆是被所有线程共享的一块内存区域;  
在虚拟机启动时, 创建此内存区域的唯一目的, 就是存放对象实例, 几乎所有的对象实例, 以及数组都在这里分配内存;    

Java 堆是垃圾收集器管理的主要区域, 现在的垃圾收集器, 基本都是采用的分代收集算法, 所以 Java 堆中还可以细分为:  
年轻代, 年老代, 持久代;  
新生对象被分配到 Young Generation 的 Eden 区, 大对象直接被分配到 Old Generation;  
Java 堆可以处于, 物理上不连续的内存空间中, 只要逻辑上是连续的即可;  
如果在堆中没有内存完成实例分配, 并且堆也无法再扩展时, 将会抛出 OutOfMemoryError 异常;  

### 方法区(Method Area)
与Java堆一样, 是线程间共享的内存区域, 它用于存储已被虚拟机加载的, 类信息, 常量, 静态变量, 即时编译器, 编译后的代码等数据;  
Java 虚拟机规范对这个区域, 的限制非常宽松, 除了和 Java 堆一样不需要连续的内存, 可以选择固定大小或者可扩展外, 还可以选择不实现垃圾收集;  
相对而言, 垃圾收集行为在这个区域是比较少出现的, 但并非数据进入了方法区就如永久代的名字一样“永久”存在了;  
这个区域的内存回收目标, 主要是针对常量池的回收, 和对类型的卸载, 一般来说这个区域的回收“成绩”难以令人满意, 尤其是类型的卸载, 条件相当苛刻;  
但是这部分区域, 的回收确实是有必要的;  
根据Java 虚拟机规范的规定, 当方法区无法满足内存分配需求时, 将抛出 OutOfMemoryError 异常;  

❀ 运行时常量池(Runtime Constant Pool)   
运行时常量池, 是方法区的一部分, 用于存放编译期生成的, 各种字面量和符号引用(类和方法的符号引用), 这部分内容将在类加载后存放;  
Java 虚拟机对 Class 文件每一部分(包括常量池)的格式都有严格规定:  
一般来说, 除了保存 Class 文件中描述的符号引用外, 还会把翻译出来的直接引用, 也存储在运行时常量池中;  
运行时常量池, 相对于 Class 文件常量池, 的另外一个重要特征是具备动态性, 运行期间也可能将新的常量放入池中;  
这种特性被开发人员利用得比较多的, 便是String类的 intern()方法;  
既然运行时常量池是方法区的一部分, 自然受到方法区内存的限制, 当常量池无法再申请到内存时会抛出OutOfMemoryError异常。  

常量池:  
常量池可以理解为, Class 文件之中的资源仓库, 它是 Class 文件结构中, 与其他项目关联最多的数据类型,  
也是占用Class文件, 空间最大的数据项目之一, 同时它还是在Class文件中, 第一个出现的表类型数据项目;  
常量池中主要存放两大类常量: 字面量(Literal), 和符号引用(Symbolic References);    
字面量比较接近于Java语言层面的常量概念, 如文本字符串,  声明为final的常量值等;   
符号引用则属于编译原理方面的概念, 包括了下面三类常量:  
类和接口的全限定名(Fully Qualified Name);  
字段的名称和描述符(Descriptor);  
方法的名称和描述符(Descriptor);  



### 直接内存(DirectMemory)
直接内存(Direct Memory)并不是虚拟机运行时数据区的一部分, 也不是Java虚拟机规范中定义的内存区域。   
但是这部分内存也被频繁地使用, 而且也可能导致OutOfMemoryError异常出现,     
在JDK 1.4中新加入了NIO(New Input/Output)类, 引入了一种基于通道(Channel)与缓冲区(Buffer)的I/O方式,   
它可以使用Native函数库直接分配堆外内存, 然后通过一个存储在Java堆中的DirectByteBuffer对象作为这块内存的引用进行操作。     
这样能在一些场景中显著提高性能, 因为避免了在Java堆和Native堆中来回复制数据。  
显然, 本机直接内存的分配不会受到Java堆大小的限制, 但是, 既然是内存, 肯定还是会受到本机总内存大小以及处理器寻址空间的限制。   
服务器管理员在配置虚拟机参数时, 会根据实际内存设置-Xmx等参数信息, 但经常忽略直接内存,   
使得各个内存区域总和大于物理内存限制(包括物理的和操作系统级的限制), 从而导致动态扩展时出现OutOfMemoryError异常。  




### 类与对象  
对象的创建;  对象的内存布局;  类加载的生命周期;  类加载器ClassLoader, 与双亲委派模型;  
[链接](jvm/object_info.md)  

[Class类文件结构](jvm/class_file_structure.md)    


### 内存回收  
可达性分析算法;  GC Roots;  JVM怎么判断对象是否已死;  finalize;  
[链接](jvm/class_lifecycle.md)  

[垃圾收集算法](jvm/garbage_collection.md)   
[垃圾收集器](jvm/garbage_coloector.md)   
### 判断对象是否存活与引用有关  
强引用: 就是指在程序代码之中普遍存在的, 类似“Object obj = new Object()”这类的引用, 只要强引用还存在, 垃圾收集器永远不会回收掉被引用的对象。  
软引用: 在系统将要发生内存溢出异常之前, 将会把这些对象列进回收范围之中进行第二次回收。    
弱引用: 被弱引用关联的对象只能生存到下一次垃圾收集发生之前。当垃圾收集器工作时, 无论当前内存是否足够, 都会回收掉只被弱引用关联的对象。
虚引用: 一个对象是否有虚引用存在, 完全不会对其生存时间构成影响, 也无法通过虚引用来取得一个对象实例。
为一个对象设置虚引用的唯一目的就是能在这个对象被收集器回收时刻得到一个系统通知。


###  内存分配与回收策略  
对象主要分配在新生代的Eden区上, 大多数情况下, 对象在新生代Eden区中分配。 当Eden区没有足够空间进行分配时, 虚拟机将发起一次Minor GC。  
大对象直接进入老年代, 所谓的大对象是指, 需要大量连续内存空间的Java对象, 最典型的大对象就是那种很长的字符串以及数组。    
大对象对虚拟机的内存分配来说就是一个坏消息, 经常出现大对象容易导致内存还有不少空间时就提前触发垃圾收集以获取足够的连续空间来存储它们。    

长期存活的对象将进入老年代, 如果对象在Eden出生并经过第一次Minor GC后仍然存活, 并且能被Survivor容纳的话, 将被移动到Survivor空间中, 并且对象年龄设为1。    
对象在Survivor区中每 "熬过" 一次Minor GC, 年龄就增加1岁, 当它的年龄增加到一定程度(默认为15岁), 就将会被晋升到老年代中。     

动态对象年龄判定, 为了能更好地适应不同程序的内存状况, 如果在Survivor空间中相同年龄所有对象大小的总和大于Survivor空间的一半, 年龄大于或等于该年龄的对象就可以直接进入老年代。  

空间分配担保, 在发生Minor GC之前, 虚拟机会先检查老年代最大可用的连续空间是否大于新生代所有对象总空间, 如果这个条件成立, 那么Minor GC可以确保是安全的。  
因此当出现大量对象在Minor GC后仍然存活的情况(最极端的情况就是内存回收后新生代中所有对象都存活), 就需要老年代进行分配担保, 把Survivor无法容纳的对象直接进入老年代。  

新生代GC(Minor GC): 指发生在新生代的垃圾收集动作, 因为Java对象大多都具备朝生夕灭的特性, 所以Minor GC非常频繁, 一般回收速度也比较快。  
老年代GC(Major GC/Full GC): 指发生在老年代的GC, 出现了Major GC, 经常会伴随至少一次的Minor GC,  Major GC的速度一般会比Minor GC慢10倍以上。   

### 什么情况下触发垃圾回收  

GC有两种类型: Scavenge GC 和 Full  GC  
Scavenge GC  
一般情况下, 当新对象生成, 并且在 Eden 申请空间失败时, 就会触发 Scavenge  GC,     
对 Eden 区域进行GC, 清除非存活对象, 并且把尚且存活的对象移动到 Survivor 区, 然后整理 Survivor 的两个区。    
这种方式的GC是对年轻代的 Eden 区进行, 不会影响到年老代。因为大部分对象都是从 Eden 区开始的, 同时 Eden 区不会分配的很大, 所以 Eden 区的GC会频繁进行。    
因而, 一般在这里需要使用速度快、效率高的算法, 使 Eden 去能尽快空闲出来。  


Full GC  
对整个堆进行整理, 包括 Young、Old、Permanent。Full GC需要对整个对进行回收, 所以比 Scavenge  GC要慢, 因此应该尽可能减少Full GC的次数。  
在对JVM调优的过程中, 很大一部分工作就是对于Full GC的调节。有如下原因可能导致Full GC: 
年老代(Old Generation)被写满
持久代(Permanent Generation)被写满 
System.gc()被显示调用 
上一次GC之后Heap的各域分配策略动态变化

### Java内存模型  
[Java内存模型(JMM)](jvm/jmm_concept.md)  
[volatile 关键字](/Java/basic/library/volatile.md)  


### 参考  
http://www.importnew.com/17770.html  
http://www.cnblogs.com/dingyingsi/p/3760447.html  
http://www.cnblogs.com/dingyingsi/p/3760730.html  
https://github.com/LRH1993/android_interview/blob/master/java/virtual-machine/Garbage-Collector.md  
http://blog.csdn.net/ns_code/article/details/17565503  
https://juejin.im/post/5b8f7e43f265da0afe62b9f8  
https://blog.csdn.net/qq_26222859/article/details/73135660  
https://blog.csdn.net/vegetable_bird_001/article/details/51278339  
https://juejin.im/post/5c6b987ae51d45362c3631ca 

