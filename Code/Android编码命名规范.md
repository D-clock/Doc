# Android编码命名规范

今年正式本科毕业，目前为止参与过的团队开发项目也有四五个。阅读过各式各样的混乱代码，最离谱的见过所有的变量都用中文拼音首字母，心中真是万千匹草泥马在奔腾。由此，也意识到命名对于编码的重要性。有人说，看一个开发者的水平如何，从看他代码的命名可以大致得出结论。好的命名除了可以让项目成员快速且更好的理解代码，自己读起来也赏心悦目。为此，特地根据自己平常的一些编码规范和网上一些资料进行整理汇总，方便自己时常查看对比。

### 基本的命名法

Java编程比较常见的有下面三种命名方式

1. 驼峰(Camel)命名法:又称小驼峰命名法，除首单词外，其余所有单词的第一个字母大写。
2. 帕斯卡(pascal)命名法:又称大驼峰命名法，所有单词的第一个字母大写
3. 下划线命名法:单词与单词间用下划线做间隔

一般建议拿来做命名的单词要比较精悍短小，这样即使两三个单词一起拼装成一个命名，也不至于显得很冗长。当然有些单词我们也可以直接写成一些约定俗成的缩写。诸如：msg(message)、init(initial)、img(image)等.....

个人认为，这些缩写可参照业界常见的缩写命名，也可以根据当前项目中的风格，进行团队成员间的约定。这样相对比较灵活，也方便团队成员之间相互理解。

### 包命名

采用反域名命名规则，全部使用小写字母。

1. 一级包名为com;
2. 二级包名为xx（可以是公司或则个人的随便）;
3. 三级包名应用的英文名app_name;
4. 四级包名为模块名或层级名;

下面罗列一些常见的命名划分方式

|命名格式| 作用 |
|-------|----------|
|com.xx.app_name.activities(或com.xx.app_name.activity)|存放app所有的Activity|
|com.xx.app_name.service|存放app所有的Service|
|com.xx.app_name.receiver|存放app所有的BroadcastReceiver|
|com.xx.app_name.provider|存放app所有的ContentProvider|
|com.xx.app_name.fragment|存放app所有的Fragment|
|com.xx.app_name.dialog|存放app所有的Dialog|
|com.xx.app_name.base|存放app一些共有的基础模块，诸如BaseActivity、BaseContentProvider、BaseService，BaseFragment等|
|com.xx.app_name.utils|存放app的工具类,诸如格式化日期的DateFormatUtils，处理字符串的StringUtils等|
|com.xx.app_name.bean(或com.xx.app_name.unity)|存放app自定义的实体类|
|com.xx.app_name.db)|存放app数据库操作相关的类|
|com.xx.app_name.view)|存放app自定义的控件|
|com.xx.app_name.adapter)|存放app所有的适配器类|


### 类命名

名词，采用大驼峰命名法，尽量避免缩写，除非该缩写是众所周知的。如HTML,URL,JSON，XML等.

|类|命名格式| 示例 |
|-------|-------|----------|
|Activity|XXX功能+Activity|如主界面HomeActivity,启动页LauncherActivity|
|Service|XXX功能+Service|如消息推送的Service，PushService或PushMessageService|
|BroadcastReceiver|XXX功能+Receiver|如在线的消息广播接受者，OnlineReceiver|
|ContentProvider|XXX功能+Provider|如联系人的内容提供者，ContactsProvider|
|Fragment|XXX功能+Fragment|如显示联系人的Fragment，ContactsFragment|
|Dialog|XXX功能+Dialog|如普通的选择提示对话框，ChoiceDialog|
|Adapter|XXX功能+XX类型控件Adapter|如联系人列表，ContactsListAdapter|
|基础功能类|Base+XX父类名|如BaseActivity，BaseFragment|
|工具类|XXX功能+Utils|如处理字符串的工具类，StringUtils|
|管理类|XXX功能+Manager|如管理联系人的类，ContactsManager|


### 接口命名

和类名基本一致。也可以在接口名前面再加一个大写的I，表明这是一个接口Interface。

如：可以表明一个信息是否可以分享的接口，可以命名为Shareable，也可以是IShareable。

### 方法

动词或动名词，采用小驼峰命名法。

下面是一些比较常见的命名风格和含义

|命名风格| 含义|
|-------|----------|
|initXX()|初始化，如初始化所有控件initView()|
|isXX()|是否满足某种要求，如是否为注册用户isRegister()|
|processXX()|对数据做某些处理，可以以process作为前缀|
|displayXX()|显示提示信息，如displayXXDialog，displayToast，displayXXPopupWindow|
|saveXX()|保存XX数据|
|resetXX()|重置XX数据|
|addXX()/insertXX()|添加XX数据|
|deleteXX()/removeXX()|删除XX数据|
|updateXX()|更新XX数据|
|searchXX()/findXX()/queryXX()|查找XX数据|
|draw()|控件里面使用居多，例如绘制文本drawText|


### 变量

采用小驼峰命名法。同样比较简单，但为了更好表明含义，我建议做一下的的区分

- 成员变量命名前面加m（member，表示成员变量之意），如，控件的宽高 mWidth，mHeight
- 静态类变量前面加s（static，表示静态变量之意），如，一个静态的单例 sSingleInstance


### 常量

同样较为简单，全部大写,采用下划线命名法.如：MIN_WIDTH,MAX_SIZE

### 布局资源文件(layout文件夹下)

全部小写，采用下划线命名法.

下面是一些比较常见的命名风格和含义

|布局类型| 命名风格 |
|-------|----------|
|Activity的xml布局|activity_+XX功能，如主页面activity_home|
|Fragment的xml布局|fragment_+XX功能，如联系人模块fragment_contacts|
|Dialog的xml布局|dialog_+XX功能，如选择日期dialog_select_date|
|抽取出来复用的xml布局（include）|include_+XX功能，如底部tab栏include_bottom_tabs|
|ListView或者RecyclerView的item xml布局|XX功能+_list_item，如联系人的contact_info_list_item|
|GridView的item xml布局|XX功能+_grid_item，如相册的album_grid_item|

### 动画资源文件(anim文件夹下)

全部小写，采用下划线命名法，加前缀区分.

下面是一些比较常见的命名风格和含义

|动画效果| 命名风格 |
|-------|----------|
|淡入/淡出|fade_in/fade_out|
|从某个方向淡入/淡出|fade_方向_in(out),右边淡入淡出fade_right_in(out)|
|从某个方向弹入/弹出|push_方向_in(out),右边推入推出push_right_in(out)|
|从某个方向滑入/滑出|slide_in(out)_from_方向,右边滑入滑出slide_in(out)_from_right|

### strings和colors资源文件

小驼峰命名法,命名风格大致如下：

- string命名格式：XX界面_XX功能_str,如 activity_home_welcome_str
- color命名格式：color_16进制颜色值，如红色 color_ff0000

像string通常建议把同一个界面的所有string都放到一起，方便查找。而color的命名则省去我们头疼的想这个颜色怎么命名。

### selecor、drawable、layer-list资源文件

小驼峰命名法。命名风格通常都是XX_selector、XX_drawable、XX_layer。

下面举两个比较常用的栗子：

- 按钮按压效果button_selector，正常状态命名为button_normal(XX_normal)，按压状态命名为button_pressed(XX_pressed)
- 选择效果checkbox_selector,未选中状态命名为checkbox_unchecked(XX_unchecked),选中状态为checkbox_checked(XX_checked)


### styles、dimens资源文件

- style采用大驼峰命名法，主题可以命名为XXTheme,控件的风格可以命名为XXStyle
- dimen采用小驼峰命名法，如所有Activity的titlebar的高度，activity_title_height_dimen

### 控件id命名

因为有些控件平常不常用，所以上网搜罗了一份下来。集大家之所长，以作参考。

|控件| 前缀缩写 |
|-------|----------|
|RelativeLayout| rl |
|LinearLayout| ll |
|FrameLayout| fl |
|TextView| tv |
|Button| btn |
|ImageButton| imgBtn |
|ImageView| imgView或iv |
|CheckBox| chk |
|RadioButton | rdoBtn |
|analogClock| anaClk |
|DigtalClock| dgtClk |
|DatePicker| dtPk |
|EditText | edtTxt或et |
|TimePicker| tmPk |
|toggleButton| tglBtn |
|ProgressBar| proBar |
|SeekBar | skBar |
|AutoCompleteTextView| autoTxt |
|ZoomControl| zmCtl |
|VideoView| vdoVi |
|WebView| webVi |
|Spinner| spn |
|Chronometer| cmt |
|ScollView| sclVi |
|TextSwitch| txtSwt |
|ImageSwitch| imgSwt |
|ListView| lv |
|GridView| gv |
|ExpandableList| epdLt |
|MapView| mapVi |

以上就是从网上搜罗整理下来的，大家也可以做纠正或者补充。


## 总结

以上就是综合很多网上很多资料，再加上平时开发经验进行汇总整理的命名规范。好的命名规则能够提高代码质量，使得新人加入项目的时候降低理解代码的难度。最后我还要啰嗦一下几点：

- 规矩终究是死的，适合团队的才是最好的
- 命名规范需要团队一起齐心协力来维护执行，在团队生活里，谁都不可能独善其身
- 如果你觉得你的命名晦涩难懂，那么，你可以开始动手了
- 一开始总会不习惯的，一步一个脚印，持之以恒，总会成功的

