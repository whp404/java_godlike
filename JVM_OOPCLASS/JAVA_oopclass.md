#Java对象模型
一个Java对象可以分为三部分存储在内存中，分别是：**对象头(Header)、实例数据(Instance Data)和对齐填充(Padding)。**
###对象头
在HotSpot虚拟机中，对象头可以分为两部分

- 对象自身的运行时数据：这部分存储包括哈希码(HashCode)、GC分代年龄、锁状态标志、线程持有的锁、偏向线程ID、偏向时间戳等。这部分数据被官方称为Mark Word，在32位和64位的虚拟机中的大小分别为32bit和64bit。
![](https://img.mubu.com/document_image/d14b7173-73cf-4cd4-aa93-7350f5f575df-1023633.jpg)
- 对象的类型指针：
即指向对象的类元数据的指针。虚拟机可以通过该指针判定对象实例属于哪个类。在Java对象中比较特殊的是Java数组，一个数组实例的对象头中必须记录数组的长度。JVM可以通过对象头中的数组长度数据来判定数组的大小，这是访问数组类型的元数据无法得到的。
	- 类的元数据，即类的数据描述，也被存在**方法区**。我们知道对象头中会存有对象的类型指针，通过类型指针可以获取类的元数据。因此，对象的类型指针其实指向的是方法区的某个存有类信息的地址。

	          并不是每个对象实例都存有对象的类型指针。根据对象访问定位方法的不同，对象的类型指针被存放在不同的区域。
			
			    通过句柄访问对象
				对象的类型指针被存放在句柄池中；
			
				通过Reference指针直接访问对象（***hotspot采用该方式！）
				对象的类型指针被存放在对象本身的数据中。
###对象数据
实例数据才是一个对象实例存储的有效信息，也是在程序代码中所定义的各种类型的字段内容。这部分内容同时记录了子类从父类继承所得的各类型数据。

>Java对象保存在堆内存中。在内存中，一个Java对象包含三部分：对象头、实例数据和对齐填充。其中对象头是一个很关键的部分，**因为对象头中包含锁状态标志、线程持有的锁等标志。**这篇文章就主要从Java对象模型入手，找一找我们关系的对象头以及对象头中和锁相关的运行时数据在JVM中是如何表示的。**本文的所有分析均基于HotSpot虚拟机**。

###填充
对齐填充在对象数据中并不是必然的，只是起着占位符的作用，没有特别含义。HotSpot要求对象起始地址必须是8字节的整数倍。对象头的大小刚好符合要求，因此当实例数据没有对齐时，就需要通过填充来对齐数据。
##OOP-KLASS MODEL
OOP（Ordinary Object Pointer）指的是普通对象指针，而Klass用来描述对象实例的具体类型
###为什么要有oop-klass model?
HotSopt JVM的设计者不想让每个对象中都含有一个vtable（虚函数表)。
众所周知，C++和Java都是面向对象的语言，面向对象语言有一个很重要的特性就是多态。关于多态的实现，C++和Java有着本质的区别。
>多态是面向对象的最主要的特性之一，是一种方法的动态绑定，实现运行时的类型决定对象的行为。多态的表现形式是父类指针或引用指向子类对象，在这个指针上调用的方法使用子类的实现版本。多态是IOC、模板模式实现的关键。

>在C++中通过虚函数表的方式实现多态，每个包含虚函数的类都具有一个虚函数表（virtual table），在这个类对象的地址空间的最靠前的位置存有指向虚函数表的指针。在虚函数表中，按照声明顺序依次排列所有的虚函数。由于C++在运行时并不维护类型信息，所以在编译时直接在子类的虚函数表中将被子类重写的方法替换掉。

>在Java中，在运行时会维持类型信息以及类的继承体系。每一个类会在方法区中对应一个数据结构用于存放类的信息，可以通过Class对象访问这个数据结构。其中，类型信息具有superclass属性指示了其超类，以及这个类对应的方法表（其中只包含这个类定义的方法，不包括从超类继承来的）。而每一个在堆上创建的对象，都具有一个指向方法区类型信息数据结构的指针，通过这个指针可以确定对象的类型。

###oop体系
![](http://www.hollischuang.com/wp-content/uploads/2017/12/OOP%E7%BB%93%E6%9E%84.png)

**OOPS类的共同基类型为oopDesc**



这些OOPS在JVM内部有着不同的用途，例如，instanceOopDesc表示类实例，arrayOopDesc表示数组。也就是说，当我们使用new创建一个Java对象实例的时候，JVM会创建一个instanceOopDesc对象来表示这个Java对象。同理，当我们使用new创建一个Java数组实例的时候，JVM会创建一个arrayOopDesc对象来表示这个数组对象。

>以instanceOopDesc为例，instanceOopDesc中主要包含以下几部分数据：markOop _mark和union _metadata 以及一些不同类型的 field。

- markOop _mark：是因为对象头中有和锁相关的运行时数据，这些运行时数据是synchronized以及其他类型的锁实现的重要基础，而关于锁标记、GC分代等信息均保存在_mark中

- union _metadata：前面介绍到的_metadata是一个共用体，其中_klass是普通指针，_compressed_klass是压缩类指针。这些指针指向方法区的类的元数据！

###klass数据
klass模型的功能：

- 实现语言层面的Java类（在Klass基类中已经实现）
- 实现Java对象的分发功能，即多态功能（由Klass的子类提供虚函数实现）

>HotSopt JVM的设计者把对象一拆为二，分为klass和oop，其中oop的职能主要在于表示对象的实例数据，所以其中不含有任何虚函数。而klass为了实现虚函数多态，所以提供了虚函数表。所以，关于Java的多态，其实也有虚函数的影子在。


在JVM中，对象在内存中的基本存在形式就是oop。那么，对象所属的类，在JVM中也是一种对象，因此它们实际上也会被组织成一种oop，即klassOop。同样的，对于klassOop，也有对应的一个klass来描述，它就是klassKlass，也是klass的一个子类。在这种设计下，JVM对内存的分配和回收，都可以采用统一的方式来管理。oop-klass-klassKlass关系如图

![](http://www.hollischuang.com/wp-content/uploads/2017/12/2579123-5b117a7c06e83d84.png)


###内存存储

关于一个Java对象，他的存储是怎样的，一般很多人会回答：对象存储在堆上。稍微好一点的人会回答：对象存储在堆上，对象的引用存储在栈上。今天，再给你一个更加显得牛逼的回答：

>对象的实例（instantOopDesc)保存在堆上，对象的元数据（instantKlass）保存在方法区，对象的引用保存在栈上。


再次强调：instantOopDesc包括mark-word(hashcode,锁信息)，union _metadata（元素类型指针），field域(类中成员变量)


因为我们都知道。方法区用于存储虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。 所谓加载的类信息，其实不就是给每一个被加载的类都创建了一

	class Model
	{
	    public static int a = 1;
	    public int b;
	
	    public Model(int b) {
	        this.b = b;
	    }
	}
	
	public static void main(String[] args) {
	    int c = 10;
	    Model modelA = new Model(2);
	    Model modelB = new Model(3);
	}


[![Pp2bDI.md.png](https://s1.ax1x.com/2018/06/22/Pp2bDI.md.png)](https://imgchr.com/i/Pp2bDI)
从上图中可以看到，在方法区的instantKlass中有一个int a=1的数据存储。在堆内存中的两个对象的oop中，分别维护着int b=3,int b=2的实例数据。和oopDesc一样，instantKlass也维护着一些fields，用来保存类中定义的类数据，比如int a=1。
##总结
每一个Java类，**在被JVM加载的时候，JVM会给这个类创建一个instanceKlass**，保存在方法区，用来在JVM层表示该Java类。当我们在Java代码中，使用new创建一个对象的时候，JVM会创建一个instanceOopDesc对象，这个对象中包含了两部分信息，对象头以及元数据。对象头中有一些运行时数据，其中就包括和多线程相关的锁的信息。元数据其实维护的是指针，指向的是对象所属的类的instanceKlass。