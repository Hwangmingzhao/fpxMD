### 你能保证 GC 执行吗？

不能，虽然你可以调用 System.gc() 或者 Runtime.gc()，但是没有办法保证 GC的执行



### 直接内存

Java1.8后，类的元数据放入nativememory, 字符串池和类的静态变量放入 java 堆中， 这样可以加载多少类的元数据就不再由MaxPermSize 控制, 而由系统的实际可用空间来控制。



### 永久代和方法区之间的联系

这里的 “PermGen space”其实指的就是方法区。不过方法区和“PermGen space”又有着本质的区别。**前者是 JVM 的规范，而后者则是 JVM 规范的一种实现**，并且只有 **HotSpot 才有 “PermGen space”**，而对于其他类型的虚拟机，如 JRockit（Oracle）、J9（IBM） 并没有“PermGen space”。由于方法区主要存储类的相关信息，所以对于动态生成类的情况比较容易出现永久代的内存溢出。最典型的场景就是，在 jsp 页面比较多的情况，容易出现永久代内存溢出。



## 两种GC:MinorGC（先）/FullGC（后）



### 什么时候触发MinorGC

#### （1） eden区满

即申请一个对象时，发现eden区不够用，则触发一次MinorGC。

#### （2）在FullGC触发之前

就是在重度垃圾回收之前先进行一次轻度的回收



### 什么时候会触发FullGC

除直接调用System.gc外，触发Full GC执行的情况有如下四种。

#### （1）旧生代空间不足

旧生代空间只有在新生代对象转入及创建为大对象、大数组时才会出现不足的现象，当执行Full GC后空间仍然不足，则抛出如下错误：java.lang.OutOfMemoryError: Java heap space

为避免以上两种状况引起的FullGC，**调优时应尽量做到让对象在Minor GC阶段被回收**、让对象在新生代多存活一段时间及不要创建过大的对象及数组。

#### （2） 永久代空间满（1.8后就不叫永久代了哦）

永久代中存放的为一些class的信息等，当系统中要加载的类、反射的类和调用的方法较多时，永久代可能会被占满，在未配置为采用CMS GC的情况下会执行Full GC。如果经过Full GC仍然回收不了，那么JVM会抛出如下错误信息：

java.lang.OutOfMemoryError: PermGen space

**为避免Perm Gen占满造成Full GC现象，可采用的方法为增大Perm Gen空间或转为使用CMS GC。**

#### （3）CMS GC时出现promotion failed和concurrent mode failure（还是旧生代空间不足的原因）

对于采用CMS进行旧生代GC的程序而言，尤其要注意GC日志中是否有promotion failed和concurrent mode failure两种状况，当这两种状况出现时可能会触发Full GC。

promotion failed是在进行Minor GC时，**survivor space放不下、对象只能放入旧生代，而此时旧生代也放不下造成的；**concurrentmode failure是在执行CMS GC的过程中同时**有对象要放入旧生代，而此时旧生代空间不足**造成的。

应对措施为：增大survivorspace、旧生代空间或调低触发并发GC的比率

#### （4）统计得到的Minor GC晋升到旧生代的平均大小大于旧生代的剩余空间

这是一个较为复杂的触发情况，Hotspot为了避免由于新生代对象晋升到旧生代导致旧生代空间不足的现象，在进行Minor GC时，做了一个判断，如果之前统计所得到的Minor GC晋升到旧生代的平均大小大于旧生代的剩余空间，那么就直接触发Full GC。

例如程序第一次触发MinorGC后，有6MB的对象晋升到旧生代，那么当下一次Minor GC发生时，首先检查旧生代的剩余空间是否大于6MB，如果小于6MB，则执行Full GC。

当新生代采用PSGC时，方式稍有不同，PS GC是在Minor GC后也会检查，例如上面的例子中第一次Minor GC后，PS GC会检查此时旧生代的剩余空间是否大于6MB，如小于，则触发对旧生代的回收。除了以上4种状况外，对于使用RMI来进行RPC或管理的Sun JDK应用而言，默认情况下会一小时执行一次Full GC。可通过在启动时通过- java-Dsun.rmi.dgc.client.gcInterval=3600000来设置Full GC执行的间隔时间或通过-XX:+ DisableExplicitGC来禁止RMI调用System.gc





### 对象分配规则

（1）对象**优先分配在Eden区**，如果Eden区没有足够的空间时，虚拟机执行一次Minor GC。

（2）**大对象直接进入老年代**（大对象是指需要大量连续内存空间的对象）。这样做的目的是避免在Eden区和两个Survivor区之间发生大量的内存拷贝（新生代采用复制算法收集内存）。

（3）长期存活的对象进入老年代。虚拟机为每个对象定义了一个年龄计数器，如果对象经过了1次Minor GC那么对象会进入**Survivor区**，**之后每经过一次Minor GC那么对象的年龄加1**，直到达到阀值对象进入老年区。

（4） 动态判断对象的年龄。如果Survivor区中相同年龄的所有对象大小的总和大于Survivor空间的一半，年龄大于或等于该年龄的对象可以直接进入老年代。（真实……）

（5） 空间分配担保。每次进行Minor GC时，JVM会计算Survivor区移至老年区的对象的平均大小，如果这个值大于老年区的剩余值大小则进行一次Full GC，如果小于检查HandlePromotionFailure设置，如果true则只进行Monitor GC,如果false则进行Full GC



