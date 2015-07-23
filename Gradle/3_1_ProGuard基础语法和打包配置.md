# ProGuard基础语法和打包配置

这是对Android打包混淆所需配置所做的总结.

## ProGuard常用语法

下面列出一些常用的语法

* -libraryjars class_path 应用的依赖包，如android-support-v4
* -keep [,modifier,...] class_specification 不混淆某些类
* -keepclassmembers [,modifier,...] class_specification 不混淆类的成员
* -keepclasseswithmembers [,modifier,...] class_specification 不混淆类及其成员
* -keepnames class_specification 不混淆类及其成员名
* -keepclassmembernames class_specification 不混淆类的成员名
* -keepclasseswithmembernames class_specification 不混淆类及其成员名
* -assumenosideeffects class_specification 假设调用不产生任何影响，在proguard代码优化时会将该调用remove掉。如system.out.println和Log.v等等
* -dontwarn [class_filter] 不提示warnning

更多关于ProGuard混淆规则和语法，直接上[ProGuard官网](http://proguard.sourceforge.net/index.html#manual/usage.html)

## Android的混淆原则

Android混淆打包混淆的原则如下
* 反射用到的类不混淆
* JNI方法不混淆
* AndroidMainfest中的类不混淆，四大组件和Application的子类和Framework层下所有的类默认不会进行混淆
* Parcelable的子类和Creator静态成员变量不混淆，否则会产生android.os.BadParcelableException异常
* 使用GSON、fastjson等框架时，所写的JSON对象类不混淆，否则无法将JSON解析成对应的对象
* 使用第三方开源库或者引用其他第三方的SDK包时，需要在混淆文件中加入对应的混淆规则
* 有用到WEBView的JS调用也需要保证写的接口方法不混淆

## 常用的混淆配置和解析

先看看下面Google默认配置的混淆文件，文件的位置：
> `AndroidStudio的安装目录\sdk\tools\proguard\proguard-android.txt`

```
# This is a configuration file for ProGuard.
# http://proguard.sourceforge.net/index.html#manual/usage.html

-dontusemixedcaseclassnames
-dontskipnonpubliclibraryclasses
-verbose

# Optimization is turned off by default. Dex does not like code run
# through the ProGuard optimize and preverify steps (and performs some
# of these optimizations on its own).
-dontoptimize
-dontpreverify
# Note that if you want to enable optimization, you cannot just
# include optimization flags in your own project configuration file;
# instead you will need to point to the
# "proguard-android-optimize.txt" file instead of this one from your
# project.properties file.

-keepattributes *Annotation*//使用注解需要添加
-keep public class com.google.vending.licensing.ILicensingService
-keep public class com.android.vending.licensing.ILicensingService

# For native methods, see http://proguard.sourceforge.net/manual/examples.html#native
-keepclasseswithmembernames class * {//指定不混淆所有的JNI方法
    native <methods>;
}

# keep setters in Views so that animations can still work.
# see http://proguard.sourceforge.net/manual/examples.html#beans
-keepclassmembers public class * extends android.view.View {//所有View的子类及其子类的get、set方法都不进行混淆
   void set*(***);
   *** get*();
}

# We want to keep methods in Activity that could be used in the XML attribute onClick
-keepclassmembers class * extends android.app.Activity {//不混淆Activity中参数类型为View的所有方法
   public void *(android.view.View);
}

# For enumeration classes, see http://proguard.sourceforge.net/manual/examples.html#enumerations
-keepclassmembers enum * {//不混淆Enum类型的指定方法
    public static **[] values();
    public static ** valueOf(java.lang.String);
}
//不混淆Parcelable和它的子类，还有Creator成员变量
-keep class * implements android.os.Parcelable {
  public static final android.os.Parcelable$Creator *;
}
//不混淆R类里及其所有内部static类中的所有static变量字段
-keepclassmembers class **.R$* {
    public static <fields>;
}

# The support library contains references to newer platform versions.
# Don't warn about those in case this app is linking against an older
# platform version.  We know about them, and they are safe.
-dontwarn android.support.**//不提示兼容库的错误警告


```
Google为我们提供好的混淆代码如上，但是往往我们还需要添加自己的混淆规则，这是因为我们可能会遇到以下几种情况：
* 代码中使用了反射，如一些ORM框架的使用
* 使用GSON、fastjson等JSON解析框架所生成的对象类
* 继承了Serializable接口的类
* 引用了第三方开源框架或继承第三方SDK，如开源的okhttp网络访问框架，百度定位SDK等
* 有用到WEBView的JS调用接口

针对以上几种情况，分别给出下列对应解决的混淆规则
>1.代码中使用了反射，加入下面的混淆规则即可.

```
-keepattributes Signature
-keepattributes EnclosingMethod
```
>2.使用GSON、fastjson等JSON解析框架所生成的对象类，加入下面的混淆规则即可.假设com.clock.bean包下所有的类都是JSON解析生成对象的类

```
-keep class com.clock.bean.**{*;}//不混淆所有的com.clock.bean包下的类和这些类的所有成员变量

```
>3.继承了Serializable接口的类，加上如下规则

```
//不混淆Serializable接口的子类中指定的某些成员变量和方法
-keepclassmembers class * implements java.io.Serializable {
    static final long serialVersionUID;
    private static final java.io.ObjectStreamField[] serialPersistentFields;
    private void writeObject(java.io.ObjectOutputStream);
    private void readObject(java.io.ObjectInputStream);
    java.lang.Object writeReplace();
    java.lang.Object readResolve();
}
```
>4.引用了第三方开源框架或继承第三方SDK情况比较复杂，开发过程中使用了知名成熟的开发框架都会有给出它的混淆规则，第三方的SDK也一样。这里以okhttp框架和百度地图sdk为例

```
//okhttp框架的混淆
-dontwarn com.squareup.okhttp.internal.http.*
-dontwarn org.codehaus.mojo.animal_sniffer.IgnoreJRERequirement
-keepnames class com.levelup.http.okhttp.** { *; }
-keepnames interface com.levelup.http.okhttp.** { *; }
-keepnames class com.squareup.okhttp.** { *; }
-keepnames interface com.squareup.okhttp.** { *; }

//百度sdk的混淆
-keep class com.baidu.** {*;}
-keep class vi.com.** {*;}
-dontwarn com.baidu.**

```
>5.有用到WEBView的JS调用接口，需加入如下规则

```
-keepclassmembers class fqcn.of.javascript.interface.for.webview {
   public *;
}
-keep class com.xxx.xxx.** { *; }//保持WEB接口不被混淆 此处xxx.xxx是自己接口的包名
```

## ProGuard语法的延伸
下面对ProGuard语法做一些小小的延伸和拓展

* 不混淆某个类

```
-keep class com.clock.**//不混淆所有com.clock包下的类，** 换成具体的类名则表示不混淆某个具体的类

```
* 不混淆某个类和成员变量

```
-keep class com.clock.**{*;}//不混淆所有com.clock包下的类和类中的所有成员变量，**可以换成具体类名，*可以换成具体的字段，可参照Serialzable的混淆

```
* 不混淆类中的方法或变量，参照AndroidStudio给的默认混淆文件的书写方式

* 使用`assumenosideeffects`移除代码

```
//移除Log类打印各个等级日志的代码，打正式包的时候可以做为禁log使用，这里可以作为禁止log打印的功能使用，另外的一种实现方案是通过BuildConfig.DEBUG的变量来控制
-assumenosideeffects class android.util.Log {
	public static *** v(...);
	public static *** i(...);
	public static *** d(...);
    public static *** w(...);
    public static *** e(...);
}
```
## 总结

大致的做了ProGuard的使用总结，光看不写是不会有好代码出来的，动手打码吧，感谢相关参考资料的作者。