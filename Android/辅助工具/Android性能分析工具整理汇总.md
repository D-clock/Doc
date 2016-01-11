# Android性能分析工具整理汇总

把做Android开发以来碰到的一些不错的性能分析工具做个整理汇总...

## Debug GPU Overdraw

**类型**：系统自带功能UI渲染检测功能（**打开Settings，然后到 Developer Options -> Debug GPU Overdraw 选择 Show overdraw areas，手机系统设置中文的孩纸，自行对照翻译进去哈**)
**作用**：用来检测UI的重绘次数，开发者可以用来优化UI的性能。
**使用心得**：检测UI性能的利器，对于开发者做UI优化的帮助挺大的。因为大量的重绘容易让app造成卡顿或者直接导致丢帧的现象。开发者熟悉View的绘制原理可以结合对一些布局或者自定义控件做相应的优化。诸如：在ListView或GridView里面的item使用layout_weight设置就会造成多余重绘。其他情况还有很多，不一一例举。至于怎么用，可以自行Google

## Profile GPU Rendering

**类型**：系统自带功能UI渲染检测功能（**打开Settings，然后到 Developer Options -> Profile GPU Rendering. 选择 On screen as bars **)
**作用**：用来检测UI绘制帧的速率和耗时，同样开发者可以用来优化UI的性能。
**使用心得**：跟Debug GPU Overdraw功能类似，但它反应的是UI绘制帧的速率，同样可以用来检测自己的app是否丢帧或者绘制过度，具体操作可以自行Google

## Hierarchy Viewer

**类型**：SDK自带工具（**打开Settings，然后到 Developer Options -> Profile GPU Rendering. 选择 On screen as bars **)
**作用**：检测UI渲染用的
**使用心得**：老牌工具了，Google一下

## Memory Monitor、Heap Viewer、Allocation Tracker

**类型**：AndroidStudio自带的工具
**作用**：均是内存检测分析的工具
**使用心得**：不用多说，大家懂的...


## Memory Analyzer Tool (MAT)

**类型**：ADT时代的插件，也有独立的MAT版本
**作用**：内存详尽分析的神器啊！
**使用心得**：它是我在ADT下唯一的美好回忆啊，AS现在的工具就差它了，希望快点跟上。为了隆重介绍我的挚爱，果断献上它的官方文档：http://help.eclipse.org/mars/index.jsp

![MAT Doc](http://a.hiphotos.baidu.com/image/pic/item/b21bb051f819861846864c164ced2e738ad4e65b.jpg)

## Traceview、Systrace

**类型**：SDK自带
**作用**：CPU使用分析的工具
**使用心得**：排除CPU性能瓶颈的利器，TraceView能让我知道个个函数调用的CPU耗时，以及总CPU耗时等，方便排查优化。Systrace能够让我了解各个AP子模块的使用情况，同样利于瓶颈排查，性能优化工作等，总之，很赞就是了。

## Battery Historian

**类型**：独立开源软件 **(Google IO大会上的推荐的工具)**
**作用**：耗电分析工具
**使用心得**：在耗电分析上Google亲自推的东西自然不用说，Battery Historian 1.0的基本使用在网上挺多，可以自行查看。2.0的功能更加perfect，但是国内资料少，国外的资料算还可以，so，翻墙吧，骚年！使用 Battery Historian 需要注意两点，一是它只对5.x及其以上的系统生效，二是搭建环境的时候注意要使用Python2.x的，不要使用Python3.x。因为两个版本的语法变法很大，Python 3.x下Battery Historian会报错。最后，这个是开源项目 https://github.com/google/battery-historian

-----------------------------------分割线-----------------------------------

上面主要都是官方的工具，下面是一些第三方apk工具...

# WakeLock Detector

**功能简介**：对手机的运行状态进行探测记录，能统计那些应用触发了CPU运行消耗cpu，那些应用触发了屏幕点亮。同时还可以对运行时间进行统计，可以查看应用内使用细节。

**使用心得**：之前做了一款app被用户投诉耗电太快。偶然发现了它，拿做电量损耗检测。同时，它也能够统计其他安装在手机上的app的电量消耗，方便做出对比，向顶级体验的应用看齐。

**使用前提**：手机需要root，该app需要获取root权限


# GSam Battery Monitor
**功能简介**：检测手机电池电量的消耗去向，能够用折线图进行统计展示。

**使用心得**：不错的产品，能够计算出你的电量被手机的哪部分功能所消耗的，可以追溯到这部分功能是哪些app在使用，从而定位到手机耗电过快的元凶。

**使用前提**：手机需要root，该app需要获取root权限

# Trepn Profiler
**功能简介**：高通出品的，杠杠的赞啊！分析检测手机CPU的消耗，而且能够分析特地的分析某个app。

**使用心得**：用来调试分析自己的app，实时的用折线图展示了app对CPU的消费情况，赞赞赞。

**使用前提**：手机需要root，该app需要获取root权限，且只支持手机的CPU是高通的。

# Root Explorer
**功能简介**：一款文件浏览器，可以查看app没有加密过的数据库，读取里面的数据，且支持简单的条件查询。

**使用心得**：在开发的时候，需要确认是否成功把数据插入数据库，有了它就可以直接打开database文件浏览查找了。

**使用前提**：手机需要root，该app需要获取root权限


-----------------------------------分割线-----------------------------------

除了上面这些apk工具外，最后是一些知名IT公司开发的工具（包含SDK），很好用...

## Bugly

揪BUG、揪ANR的SDK。腾讯出品的东东，杠杠的。对发布出去的产品你想准确定位各种闪退的BUG，用它准行。而且bugly的更新频率还挺快的，大公司的效率真是任性（只能说鹅厂越来越会用技术赚钱了~~~~~）

**官网地址**：http://bugly.qq.com/

## BugTags

官网说的：测试，从未如此简单！新一代的、专为移动测试而生的缺陷发现及管理工具。个人觉得很不错，同样推荐！

**官网地址**：https://bugtags.com/

## GT

这款神器，可能并不多人知道（我猜的）。腾讯MIG专项测试组开发出来的狂拽酷炫屌炸天的神器，只要多神，不多说了，直接进去官网看吧，我已泪奔（腾讯的技术真心叼）

**官网地址**：http://gt.tencent.com/index.html

## iTest

科大讯飞出品的测试工具，直接安装使用。是一款服务于Android测试人员的专业手机自动化性能监控工具。

**官网地址**：http://itest.iflytesting.com/?p=1

## Emmagee

网易出品的测试工具，和iTest差不多，最大的好处在于，能够对应用的常用性能指标进行检测，并以csv的格式保存方便查看应用的各项参数。测试结果看起来更加直观，还有很重要一点是，它开源!!!!

**官网地址**：https://github.com/NetEase/Emmagee


## 待续...

目前大体就这些了，后续有更好的工具也会接着更新，有些工具过时失效了，也会在这里移除...