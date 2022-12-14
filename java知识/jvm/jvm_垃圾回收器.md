[TOC]
# 垃圾回收器
* **Serial和Serial Old垃圾回收器**:分别用来回收新生代和老年代的垃圾对象

>工作原理就是单线程运行，垃圾回收的时候会停止我们自己写的系统的其他工作线程，让我们系统直接卡死不动，然后让他们垃圾回收，这个现在一般写后台Java系统几乎不用。

* **ParNew和CMS垃圾回收器**
>ParNew现在一般都是用在新生代的垃圾回收器，CMS是用在老年代的垃圾回收器，他们都是多线程并发的机制，性能更好，现在一般是线上生产系统的标配组合。

* **G1垃圾回收器**(重点)
>统一收集新生代 和老年代，采用了更加优秀的算法和设计机制

## 1.最常用的新生代垃圾回收器---ParNew
新生代的ParNew垃圾回收器主打的就是多线程垃圾回收机制ParNew垃圾回收器如果一旦在合适的时机执行Minor GC的时候，就会把系统程序的工作线程全部停掉，禁止程序继续运行创建新的对象，然后自己就用多个垃圾回收线程去进行垃圾回收.如果你只有一个cpu还用这个模式就是在一个cpu上开多线程,得不偿失
![](https://raw.githubusercontent.com/Haiyang-coder/ImageRepository/main/202210251504918.png)

## 2.GMS垃圾回收器原理
一般老年代我们选择的垃圾回收器是CMS，他采用的是标记清理算法,这种算法的缺点就是处理的时间很慢.如果停止一切工作线程，然后慢慢的去执行“标记-清理”算法，会导致系统卡死时间过长，很多响应无法处理所以
>所以CMS垃圾回收器采取的是垃圾回收线程和系统工作线程尽量同时执行的模式来处理的

**CMS如何实现系统一边工作的同时进行垃圾回收?**
1. 初试标记(标记GC Roots对象,在Stop World状态)
2. 并发标记(耗时)
3. 重新标记(Stop World)
4. 并发清理(耗时)
   
>首先，CMS要进行垃圾回收时，会先执行初始标记阶段，这个阶段会让系统的工作线程全部停止，进入“Stop the World”状态，所谓的“初始标记”，他是说标记出来所有GC Roots直接引用的对象.

GC Roots对象就是类的静态变量指向的实例和方法的局部变量指向的实例对象.
>接着第二个阶段，是并发标记，这个阶段会让系统线程可以随意创建各种新对象，继续运行在运行期间可能会创建新的存活对象，也可能会让部分存活对象失去引用，变成垃圾对象。在这个过程中，垃圾回收线程，会尽可能的对已有的对象进行GC Roots追踪

GC Roots追踪:就是把除了GC Roots对象的其他对象也看一下谁在引用他
>随后就是重新标记,停下java,把刚才并发标记过程中新来到老年代的对象和刚才失去引用的对象重新标记一下

>最后就可以并发的清理标记为垃圾的内存了

<font color=red>那么如果CMS垃圾回收期间，系统程序要放入老年代的对象大于了可用内存空间:</font>
>这个时候，会发生Concurrent Mode Failure,此时就会自动用“Serial Old”垃圾回收器替代CMS，就是直接强行把系统程序“Stop the World”，重新进行长时间的GC Roots追踪，标记出来全部垃圾对象，不允许新的对象产生


## G1垃圾回收器
G1垃圾回收器是可以同时回收新生代和老年代的对象的，不需要两个垃圾回收器配合起来运作，他一个人就可以搞定所有的垃圾回收。
**他最大的一个特点，就是把Java堆内存拆分为多个大小相等的Region**,然后G1也会有新生代和老年代的概念，但是只不过是逻辑上的概念,而且G1最大的一个特点，就是可以让我们设置一个垃圾回收的预期停顿时间
### (1)如何做到控制GC的停顿时间的?
>通过追踪每个Region里的回收价值,G1可以做到让你来设定垃圾回收对系统的影响，他自己通过把内存拆分为大量小Region，以及追踪每个Region中可以回收的对象大小和预估时间，最后在垃圾回收的时候，尽量把垃圾回收对系统造成的影响控制在你指定的时间范围内，同时在有限的时间内尽量回收尽可能多的垃圾对象
### (2)Region可能属于新生代也可能属于老年代
>其实在G1对应的内存模型中，Region随时会属于新生代也会属于老年代，所以没有所谓新生代给多少内存，老年代给多少内存这 一说了.实际上新生代和老年代各自的内存区域是不停的变动的，由G1自动控制

### (3)大对象Region
>在G1中，大对象的判定规则就是一个大对象超过了一个Region大小的50%，比如按照上面算的，每个Region是2MB，只要一个大对象超过了1MB，就会被放入大对象专门的Region中,如果更大可以横跨几个Region保存.其实新生代、老年代在回收的时候，会顺带带着大对象Region一起回收，所以这就是在G1内存模型下对大对象的分配和回收的策略


