title: iOS 中的runtime与消息转发
author: Jaren
date: 2018-07-07 19:01:16
tags:
---
<div align=left>
<img width="450" height="600" src="/assets/blogImg/rt1.png" "2018年的第一场雪"/>  
</div> 

 	在80年代初，小李和小王是异地恋的情侣，小王在改革号角的引领下毅然选择了南方的一个城市去奋斗，而那个时候没有手机，他们之间的互诉相思的方式主要依靠写信。但是由于小王又经常出差，居住地址会经常变动。所以小李每次给小王的回信，小王可能因为地址的变动而没有收到，他们后来想到了一个好办法来解决这个问题，具体的方法如下：
 <!-- more -->



80年代的消息转发


其实上面这张图，基本上就可以表达Runtime在iOS中的作用以及iOS的消息转发机制。Runtime的特性主要是消息(方法)传递，如果消息(方法)在对象中找不到，就进行转发，具体怎么实现的呢。我们从下面几个方面探寻Runtime的实现机制。     

Runtime介绍

&nbsp;&nbsp;&nbsp;&nbsp;Objective-C 扩展了 C 语言，并加入了面向对象特性和 Smalltalk 式的消息传递机制。而这个扩展的核心是一个用 C 和 编译语言 写的 Runtime 库。它是 Objective-C 面向对象和动态机制的基石。      
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Objective-C 是一个动态语言，这意味着它不仅需要一个编译器，也需要一个运行时系统来动态得创建类和对象、进行消息传递和转发。理解 Objective-C 的 Runtime 机制可以帮我们更好的了解这个语言，适当的时候还能对语言进行扩展，从系统层面解决项目中的一些设计或技术问题。了解 Runtime ，要先了解它的核心 - 消息传递 （Messaging）。          
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;高级编程语言想要成为可执行文件需要先编译为汇编语言再汇编为机器语言，机器语言也是计算机能够识别的唯一语言，但是OC并不能直接编译为汇编语言，而是要先转写为纯C语言再进行编译和汇编的操作，从OC到C语言的过渡就是由runtime来实现的。然而我们使用OC进行面向对象开发，而C语言更多的是面向过程开发，这就需要将面向对象的类转变为面向过程的结构体。     

上述都是官方的文档释义，有些晦涩无聊，接下来我们用代码来具体解释一下。

###### Runtime消息传递

一个对象的方法  [obj test]，编译器转成消息发送objc_msgSend(obj, test)，Runtime时执行的流程是这样的：

- 1.首先，通过obj的isa指针找到它的class;

- 2.在class的method list找test;

- 3.如果class中没到test，继续往它的superclass中找 ;

- 4.一旦找到test这个函数，就去执行它的实现IMP

&nbsp;&nbsp;&nbsp;&nbsp;当然了，由于效率的问题，每个消息都遍历一次objc_method_list并不合理。所以需要把经常被调用的函数缓存下来，去提高函数查询的效率。这也就是objc_class中另一个重要成员objc_cache做的事情 - 再找到test之后，把test的method_name作为key，method_imp作为value给存起来。当再次收到test消息的时候，可以直接在cache里找到，避免去遍历objc_method_list。从前面的源代码可以看到objc_cache是存在objc_class结构体中的。

###### objec_msgSend的方法：

OBJC_EXPORTidobjc_msgSend(idself, SEL op, ...)
我们看看对象(object)，类(class)，方法(method)这几个的结构体：
  ![ico原来的样子](/assets/blogImg/rt2.png) 

###### 类对象(objc_class)

Objective-C类是由Class类型来表示的，它实际上是一个指向objc_class结构体的指针
 ![ico原来的样子](/assets/blogImg/rt3.png) 

struct objc_class结构体定义了很多变量。结构体里保存了指向父类的指针、类的名字、版本、实例大小、实例变量列表、方法列表、缓存、遵守的协议列表等，由此可见，类对象就是一个结构体struct objc_class，这个结构体存放的数据就是元数据(metadata)。

###### 实例(objc_object)

 ![ico原来的样子](/assets/blogImg/rt4.png) 


&nbsp;&nbsp;&nbsp;&nbsp;类对象中的元数据存储的都是如何创建一个实例的相关信息，就是从isa指针指向的结构体创建，类对象的isa指针指向的我们称之为元类(metaclass)

元类中保存了创建类对象以及类方法所需的所有信息，因此整个结构应该如下图所示:
 ![ico原来的样子](/assets/blogImg/rt5.png) 

实例对象、类对象与元类简图
struct objc_object结构体它的isa指针指向类对象；

类对象的isa指针指向了元类；

super_class指针指向了父类的类对象；

而元类的super_class指针指向了父类的元类；

有点绕口令的感觉，那么就可以用网上的一个神图来表示了：
 ![ico原来的样子](/assets/blogImg/rt6.png) 

通过上图我们可以看出整个体系构成了一个自闭环，如果是从NSObject中继承而来的上图中的Root class就是NSObject。
 ![ico原来的样子](/assets/blogImg/rt7.png) 



c1是通过一个实例对象获取的Class，实例对象可以获取到其类对象，类名作为消息的接受者时代表的是类对象，因此类对象获取Class得到的是其本身。

如果我们想要获取ISA指针的对象的话，可以用下面这两个函数

<font color=RED size=3>OBJC_EXPORTBOOLclass_isMetaClass(Classcls) OBJC_AVAILABLE(10.5, 2.0, 9.0, 1.0)</font> ;

<font color=RED size=3>OBJC_EXPORTClassobject_getClass(idobj) OBJC_AVAILABLE(10.5, 2.0, 9.0, 1.0)</font> ;
class_isMetaClass用于判断Class对象是否为元类，object_getClass用于获取对象的isa指针指向的对象。     

 ![ico原来的样子](/assets/blogImg/rt8.png) 


    &nbsp;&nbsp; 通过代码可以看出，一个实例对象通过class方法获取的Class就是它的isa指针指向的类对象，而类对象不是元类，类对象的isa指针指向的对象是元类。         

    &nbsp;&nbsp;&nbsp;&nbsp; 所以关于Runtime部分，我们总结一下就是：<font color=GREEN size=3>首先实例对象是一个结构体，这个结构体只有一个成员变量，指向构造它的那个类对象，这个类对象中存储了一切实例对象需要的信息包括实例变量、实例方法等，而类对象是通过元类创建的，元类中保存了类变量和类方法，这样就完美解释了整个类和实例是如何映射到结构体的。所以理解Runtime就是理解iOS在运行时他的数据存储以及他的类、实例、类对象、元类他们之间的关系和作用。 </font>      

**消息转发机制**    

   &nbsp;&nbsp;&nbsp;&nbsp;上述讲了很多Runtime的基本理解和概念，那到底他和消息转发有什么关系呢，以及怎么去运用它呢？这就要讲到iOS的消息转发机制了。    

&nbsp;&nbsp;&nbsp;&nbsp;归根到底，Objective-C中所有的方法调用本质就是向对象发送消息。    

&nbsp;&nbsp;&nbsp;&nbsp;1.类中创建方法  <font color=RED size=3>-(void)todoSomething。</font> ;   

&nbsp;&nbsp;&nbsp;&nbsp;2.iOS系统为这个方法创建一个编号即：SEL(todoSomething);并添加到方法列表中。 <font color=RED size=3>（selector是SEL的一个实例，这点和IMP是不一样的，IMP是指向最终实现程序的内存地址的指针）</font>      

&nbsp;&nbsp;&nbsp;&nbsp;3.当调用这个方法的时候： <font color=RED size=3>-(void)todoSomething。</font> ; 系统去方法列表中插手这个方法编号，查到就执行。
  <font color=RED size=3>注意：我们在写C代码的时候，经常会用到函数重载，就是函数名相同，参数不同，但是这在Objective-C中是行不通的，因为selector只记了method的name，没有参数，所以没法区分不同的method。</font>   

&nbsp;&nbsp;&nbsp;&nbsp; 所以如果调用了一个方法，就会进行一次发送消息会在相关的类对象中搜索方法列表，如果找不到则会沿着继承树向上一直搜索知道继承树根部（通常为NSObject），如果还是找不到并且消息转发都失败了就回执行doesNotRecognizeSelector:方法报unrecognized selector错。那么消息转发到底是什么呢？接下来将会逐一介绍最后的三次机会。

 ![ico原来的样子](/assets/blogImg/rt9.png) 
###### 1.动态方法解析     
&nbsp;&nbsp;Objective-C运行时会调用 +resolveInstanceMethod:或者 +resolveClassMethod:，让你有机会提供一个函数实现。如果你添加了函数并返回YES， 那运行时系统就会重新启动一次消息发送的过程。如下图实例
 ![ico原来的样子](/assets/blogImg/rt10.png) 

打印出了“Doing foo”
可以看到虽然没有实现foo:这个函数，但是我们通过class_addMethod动态添加fooMethod函数，并执行fooMethod这个函数的IMP。从打印结果看，成功实现了。

如果resolve方法返回 NO ，运行时就会移到下一步：forwardingTargetForSelector。

###### 备用接收者

如果目标对象实现了-forwardingTargetForSelector:，Runtime 这时就会调用这个方法，给你把这个消息转发给其他对象的机会。

实现一个备用接收者的例子如下：
 ![ico原来的样子](/assets/blogImg/rt11.png) 

可以看到我们通过forwardingTargetForSelector把当前ViewController的方法转发给了Person去执行了。打印结果也证明我们成功实现了转发。

###### 完整消息转发

如果在上一步还不能处理未知消息，则唯一能做的就是启用完整的消息转发机制了。

&nbsp;&nbsp;&nbsp;&nbsp;首先它会发送-methodSignatureForSelector:消息获得函数的参数和返回值类型。如果-methodSignatureForSelector:返回nil，Runtime则会发出-doesNotRecognizeSelector:消息，程序这时也就挂掉了。如果返回了一个函数签名，Runtime就会创建一个NSInvocation对象并发送-forwardInvocation:消息给目标对象。

 ![ico原来的样子](/assets/blogImg/rt12.png) 

这就是Runtime的三次转发流程。下面我们讲讲Runtime的实际应用



当系统自带的方法功能不够，可以给系统自带的方法扩展一些功能，并保持原有的功能。例如我想知道当前的URL是否为空如果每次都判断一下的话会很麻烦，如果我创建扩展来写，又不知道内部是如何实现的.

###### 一、可以使用runtime交换方法。

 ![ico原来的样子](/assets/blogImg/rt13.png) 
###### 二、也可以动态添加方法

 ![ico原来的样子](/assets/blogImg/rt14.png) 
###### 三、 给分类添加属性

 ![ico原来的样子](/assets/blogImg/rt15.png) 


###### 四、KVO实现

KVO的实现依赖于 Objective-C 强大的 Runtime，当观察某对象 A 时，KVO 机制动态创建一个对象A当前类的子类，并为这个新的子类重写了被观察属性 keyPath 的 setter 方法。setter 方法随后负责通知观察对象属性的改变状况。

###### 五、消息转发(热更新)解决Bug(JSPatch)

关于消息转发，消息转发分为三级，我们可以在每级实现替换功能，实现消息转发，从而不会造成崩溃。JSPatch不仅能够实现消息转发，还可以实现方法添加、替换能一系列功能

###### 六、实现NSCoding的自动归档和自动解档

原理描述：用runtime提供的函数遍历Model自身所有属性，并对属性进行encode和decode操作。

核心方法：在Model的基类中重写方法：

 ![ico原来的样子](/assets/blogImg/rt16.png) 


总结：在整个Objective-C运行中，所有的方法调用都是消息的发送或转发的过程，最后可以把的第一个图大致变成下面这样的，方便理解

 ![ico原来的样子](/assets/blogImg/rt17.png) 
