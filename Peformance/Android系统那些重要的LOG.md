# Android系统那些重要的LOG

LOG对于开发者的重要性自然不必多提，今天主要来总结一下平时在开发过程中遇到的一些重要的LOG。这些LOG是Android系统自带打印的，巧妙的利用这些LOG能够帮助我们熟悉一个程序、排除故障以及做一些相应的性能优化工作。

> **过滤这些LOG的方式是Android开发的程序猿都懂的，不再赘述~~~~~~**


## dalvikvm & ART

作为一名Android开发者，都知道是Dalvik和ART两个不同的虚拟机（Android开发者都不了解就要去厕所面壁了）。Android 4.4之前，都是只存在Dalvik虚拟机，而从Android4.4开始ART的虚拟机就已经存在，只不过需要手动进入系统设置里面切换到ART虚拟机运行环境。到Android5.0开始，系统就已经默认只用ART虚拟机。

如果你仔细观察过logcat，肯定可以看到下面这两类log（**log太长了，截不了全图，直接贴出来好了**）

**Android 4.4.x及以下的版本可见**

> ** com.tencent.mm D/dalvikvm( 9050): GC_CONCURRENT freed 2049K, 65% free 3571K/9991K, external 4703K/5261K, paused 2ms+2ms total 143.522ms**

**Android 5.x及以上的版本可见**

> ** com.tencent.mm I/art : Explicit concurrent mark sweep GC freed 104710(7MB) AllocSpace objects, 21(416KB) LOS objects, 33% free, 25MB/38MB, paused 1.230ms total 67.216ms **

系统的虚拟机打印出以上的log，目的是为了告诉我们一个app它对内存使用的变化情况。要利用这些log，首先就要弄明白它的信息含义。

先看dalvikvm的log格式信息含义,它的信息格式如下

> ** PackageName D/dalvikvm: GC_Reason Amount_freed, Heap_stats, External_memory_stats, Pause_time Total_time **

| 名词      |    含义  |
| -------- | -------- |
| PackageName  | 当前dalvik虚拟机属于哪个应用的，因为系统为每个app单独分配一个虚拟机  |
| GC_Reason     |   导致内存释放的原因，Dalvik虚拟机共有五类原因：GC_CONCURRENT、GC_FOR_MALLOC、GC_HPROF_DUMP_HEAP、GC_EXPLICIT、GC_EXTERNAL_ALLOC。  |
| Amount_freed      |    一次释放了多少内存  |
| Heap_stats      |    当前虚拟机堆内存的分配状态  |
| External_memory_stats      |    Native堆内存的分配状态，这部分内存的分配回收不受虚拟机控制  |
| Pause_time      |    GC线程垃圾搜集和释放引起的暂停时间，因为GC线程在做垃圾搜集和释放的时候，并发虚拟机会让其他Worker线程都挂起，让GC进行回收释放内存  |
| Total_time      |    整个GC流程的总耗时  |

GC_Reason 又分为以下几类

| 类型      |    含义 |
| -------- | --------|
| GC_CONCURRENT  | 当前分配给app的内存开始被占满时，而触发此类GC。如果GC不能腾出内存，则会分配APP一个更大的堆内存。不断的重复前面两个操作，直到OOM为止。此类现象属于内存泄露了。 |
| GC_FOR_MALLOC     |   内存配额已满，GC_CONCURRENT类型的GC多次执行无效后触发此类GC。此时，系统强制停止app来回收内存 |
| GC_HPROF_DUMP_HEAP      |    在需要打印app的堆转储（HPROF）文件时候触发的GC，用于分析堆内存 |
| GC_EXPLICIT      |    app显式调用System.gc时产生，Google是非常不建议开发者自行调用System.gc的 |
| GC_EXTERNAL_ALLOC      |    Native堆内存的GC，如NIO和Bitmap像素的内存回收。这一类GC只有在API10及其以下的Android版本才能见到 |

** com.tencent.mm D/dalvikvm( 9050): GC_CONCURRENT freed 2049K, 65% free 3571K/9991K, external 4703K/5261K, paused 2ms+2ms total 143.522ms** 这行dalvikvm log想向我们表达的意思是微信这个app（com.tencent.mm就是微信的包名）此时需要用到的堆内存已经快占满分配堆内存，所以释放到2049K的内存。当前app分配的堆内存总大小9991K，其中有65%是可以使用的，3571K是当前被使用的。外部堆内存当前分配了5261K，已用了4703K。整个GC线程搜集垃圾和释放内存总耗时2ms+2ms（这段时间内其它Worker线程都暂停挂起），整个过程的总耗时是143.522ms。

再接着说说ART虚拟机的log，其格式含义如下：

> **PackageName I/art: GC_Reason GC_Name Objects_freed(Size_freed) AllocSpace Objects, Large_objects_freed(Large_object_size_freed) Heap_stats LOS objects, Pause_time(s) Total_time**

| 名词      |    含义  |
| -------- | -------- |
| PackageName  | 同上  |
| GC_Reason     |   同上，但ART虚拟机共有下面这些导致GC的类型：Concurrent、Alloc、Explicit、NativeAlloc、CollectorTransition、HomogeneousSpaceCompact、DisableMovingGc、HeapTrim  |
| GC_Name      |    GC的名称。总共有以下几种：Concurrent mark sweep (CMS)、Concurrent partial mark sweep、Concurrent sticky mark sweep、Marksweep + semispace  |
| Objects freed      |   被回收的非大对象的个数   |
| Size freed      |    被回收的非大对象的总内存  |
| Large objects freed  |   被回收的大对象的个数   |
| Large object size freed      |   被回收的大对象的总内存   |
| Heap stats      |  同上    |
| Pause_time      | 同上     |
| Total_time      |  同上  |


GC_Reason 又分为以下几类


| 类型      |    含义 |
| -------- | --------|
| Concurrent  | 后台并发类型的GC，不需要停止其他Worker线程，也不妨碍内存的分配。 |
| Alloc     |   此类GC是因分配给app的内存已经占满而触发，这时候GC发生在内存分配的线程 |
| Explicit      | app显式调用System.gc触发，同样不鼓励做这样的操作，因为ART下显式调用gc会造成线程分配被阻塞和占用更多的CPU周期，有可能引发app闪退  |
| NativeAlloc      |   由Native内存分配而出发的GC，如Bitmap或者RenderScript的内存分配 |
| CollectorTransition      |    由堆内存垃圾搜集时对象迁移而触发的GC，通常发生在app后台长期不使用时切换回前台使用的情况下产生 |
| HomogeneousSpaceCompact      |    整理堆内存随便而产生的GC，通常发生在app从前台的使用状态切换到后台长时间不使用的状态下发生，这样做好处是为了减少内存的使用而内存碎片的产生 |
| DisableMovingGc      |    这个并不是GC的原因,而是使用了GetPrimitiveArrayCritical引起的阻塞。一般情况下不建议使用GetPrimitiveArrayCritical的，因为会阻碍内存的移动而限制了垃圾搜集 |
|HeapTrim|	同样不是GC的原因，但需要注意此时GC会被阻塞，直到堆内存的整理完成之后才会进行	|

GC_Name 又分为以下几类

| 类型      |    含义 |
| -------- | --------|
| Concurrent mark sweep (CMS)  | 除了图片所占内存外,释放其他所有对象的内存空间 |
| Concurrent partial mark sweep     |   除了图片和zygote进程所占内存外，对其他所用内存空间做垃圾搜集 |
| Concurrent sticky mark sweep      | 分代垃圾搜集器只能释放上次GC时产生的对象，通常发生在对所有垃圾内存做搜集或对内存进行标记整理的时候，  |
| Marksweep + semispace      |   非并发的复制类型GC，用于压缩整理堆内存的空间 |

大致总结到这里，后面ART那句LOG代表的意思，大致参照以上解释相信自己也能够看懂了，所以不再赘述。了解内存的变化有助于我们对app的性能调优，以上如果诸多不理解的话，就需要先学习一下虚拟机的运行机制了。

## Timeline

这个log如下图所示，为了方便，我直接拿微信朋友圈作为演示

![wechat](http://b.hiphotos.baidu.com/image/pic/item/ca1349540923dd54bfac21f9d709b3de9d824840.jpg)

我打开微信朋友圈的Activity（你随便打开哪个Activity都行），就会打印出一句log。这个log带给我们的信息有两个：

* **朋友圈Activity名叫SnsTimeLineUI**
* **它的全路径是com.tencent.mm.plugin.sns.ui.SnsTimeLineUI**

得知这两个对我们的意义在哪呢？大致如下几种场景会有所帮助：

* **在新接触项目代码时，把项目跑起来后想知道哪个界面叫啥名就可以用到了，可以帮助快速熟悉项目代码；**
* **在对某些项目做逆向工程分析时，可以快速定位到一个Activity界面，方便我们入手做分析工作；**
* **可以直接跨app的调用应用的界面，例如可以在不使用微信的SDK的情况下，调用它的多图发朋友圈功能，当然如果还有涉及到权限或其他问题的话就需要再作分析进行调用了（因为是奇技淫巧，所以不赘述，普通开发也不推荐大家用，哈哈）**


## ActivityManager

ActivityManager所带给我们的消息比较多，例如

* 可以让我们了解各个堆栈信息

![ActivityManager Task Info](http://c.hiphotos.baidu.com/image/pic/item/960a304e251f95ca23ee6727cf177f3e6709520c.jpg)

* 可以让我们了解哪些Activity被哪些信息出发弹出、哪些BroadcastReceiver接收哪些广播、哪些进程被强制停止或者直接kill

![ActivityManager Activity Broadcast Process Info](http://f.hiphotos.baidu.com/image/pic/item/6f061d950a7b02082eb1cd7064d9f2d3572cc823.jpg)

* 可以让我们知道Service被杀死，和被重启，已经中间的耗时

![ActivityManager Service Info](http://g.hiphotos.baidu.com/image/pic/item/f703738da9773912ad4d54f0fe198618367ae23c.jpg)

## Choreographer

这个log在优化自定义View和Animation的时候，可以用到

如上图可以知道小米运动这个app某些界面出现了跳帧的现象，Choreographer提示我们，可能在app的ui线程做了太多事情导致了丢帧。这时候就可以考虑一下是否要优化了。

![Choreographer Info](http://a.hiphotos.baidu.com/image/pic/item/80cb39dbb6fd526638bfe785ad18972bd507368f.jpg)

## 总结

最后，本文的内容涉及到了一定的虚拟机方面（dalvikvm & ART）和自定义控件和动画方面（Choreographer）的知识，如果看不明白可能要先了解一下这些方面的知识点。好了，大致就总结了这些常用的log。当然，系统肯定还有其他一些目前我尚未发现到的，接下来如果看到一些有用的也会不断更新本文。其他有兴趣的开发者有空也可以多留意一下，说不定能在log中发现更多的惊喜，哈哈。