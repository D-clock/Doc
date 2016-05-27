# 如何查看aar的源代码

AndroidStudio中有一种aar格式的依赖包文件（和jar包差不多一样，就是能包含一些资源文件在其中），那么如果我们想看看其中源代码要怎么做呢？下面以我的开源项目 [AndroidStudyCode](https://github.com/D-clock/AndroidStudyCode) 为例子，简单演示一遍。


- 将 **AndroidStudyCode** 以 **Import Module** 的形式导入，可以看到libs下存在aar文件 **AndroidUtils**

![](http://c.hiphotos.baidu.com/image/pic/item/b999a9014c086e06ffe53c9605087bf40bd1cb9a.jpg)

- 找到 **AndroidStudyCode** 中某个调用 **AndroidUtils** 包中的方法，然后 **Ctrl+鼠标单击**

![](http://a.hiphotos.baidu.com/image/pic/item/91ef76c6a7efce1bd5fbc317a851f3deb58f651e.jpg)

![](http://a.hiphotos.baidu.com/image/pic/item/0b55b319ebc4b745f334c4cbc8fc1e178b82154e.jpg)

到此即可以看到源代码，此时的源码已经经过一定处理，没有了注释，而且有些地方还做了优化调整。为了能看到 **AndroidUtils** 中完整的源代码，我们需要找到它的源代码工程 [AndroidUtils](https://github.com/D-clock/AndroidUtils) , 将其下载到本地。接着开始点击下面的 **Choose Sources**

![](http://a.hiphotos.baidu.com/image/pic/item/b03533fa828ba61ec8b4b7f24634970a314e5990.jpg)

选中我们下载好的源码目录

![](http://d.hiphotos.baidu.com/image/pic/item/c9fcc3cec3fdfc03e9b19c65d33f8794a5c22625.jpg)

点击OK，可以看到下面的界面

![](http://c.hiphotos.baidu.com/image/pic/item/7dd98d1001e939010ba5f3e37cec54e737d19643.jpg)

再点击一次OK，即可看到下面的原封不动的源代码了

![](http://d.hiphotos.baidu.com/image/pic/item/d52a2834349b033b587a820712ce36d3d439bd78.jpg)

如果想要方便修改 **AndroidUtils** 中的源代码，并应用在 **AndroidStudyCode** 中，则可以参考以下的做法：

1、删除 libs 下的 **AndroidUtils** aar包，并去到  **AndroidStudyCode**  的 **build.gradle** 中移除aar配置

![](http://b.hiphotos.baidu.com/image/pic/item/2f738bd4b31c87011880d43b207f9e2f0708ff3d.jpg)

2、以 **Import Library** 的形式导入 **AndroidUtils** 的源代码，并在  **AndroidStudyCode**  的 **build.gradle** 添加以下配置信息

![](http://f.hiphotos.baidu.com/image/pic/item/d01373f082025aaf31661f16fcedab64024f1a92.jpg)

配置到此完成。
