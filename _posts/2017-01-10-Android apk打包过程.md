##Android资源管理框架（Asset Manager）简介
Android应用程序主要由两部分内容组成：代码和资源。资源主要是指那些与UI相关的东西，如UI布局，字符串和图片等。代码和资源分开可以使得应用程序在运行时根据实际需要来组织UI。这样的好处就是应用程序只需要编译一次，就可以支持不同的UI布局。这种特性使应用程序在运行时可以适应不同的屏幕大小和密度，以及不同的国家和语言。

Android应用程序资源可以分为两大类，分别时assets和res：

#####1.assets
assets类资源放在工程根目录的assets子目录下，它里面保存的时一些原始的文件，可以以任何方式来进行组织。这些文件最终会被原装不动地打包在apk文件中。如果我们要在程序中访问这些文件，那么就需要制定文件名来访问。例如，假设assets目录下有一个名称为filename的文件，那么就可以使用一下代码来访问它：
```java
AssetManager am = getAssets();
InputStream is = asset.open("filename");
```
#####2.res
res资源放在工程根目录res子目录下，它里面保存的文件大多数都会被编译，并且都会被赋予资源ID。这样我们就可以在程序中通过ID来访问res类的资源。res类资源按照不同的用途可以进一步划分为一下9中子类型：

--amimator ：这类资源以XML文件保存在res/animator目录下，用来描述属性动画。属性动画通过改变对象的属性来实现动画效果，例如通过不断地改变对象的坐标值来实现对象移动动画。

--anim：  这类资源以xml文件保存在res/anim目录下，用来描述补间动画。

--color：这类资源以xml文件保存在res/color目录下，用描述对象颜色状态。对象的状态可以划分为：pressed，focused，selected，checkable，checked，enabled和window_focused等7种。

--drawable：这类资源以xml或着Bitmap文件保存在res/drawable目录下，用来描述可绘制对象。

--layout：这类资源以xml文件保存在res/layout目录下，用来描述应用程序界面布局。

--menu：这类资源以xml文件保存在res/menu目录下。用来描述应用程序菜单，例如，Options Menu,Context Menu,和Sub Menu。

--raw：这类资源以任意格式的文件保存在res/raw目录下，它们和assets类资源一样，都是原装不动地打包在apk文件中，不过它们会被赋予资源ID，这样可以在程序中通过ID来访问它们。

--values：这类资源以XML文件保存在res/values目录下，用来描述一些简单值，例如数组，颜色，尺寸，字符串和色值。分别保存的名称是arrays.xml,colors.xml,dimens.xml,strings.xml和styles.xml文件中。

--xml：这类资源以XML文件保存在res/xml目录下，一般用来描述应用程序的配置信息。

上述9类资源文件，除了raw资源，以及bitmap文件的drawable类型资源之外，其它的资源文件均为文本给事的xml文件，它们在打包的过程中，会被编译成二进制格式的xml文件。这些二进制格式的xml文件分别有一个字符串资源池，用来保存文件中引用到的每一个字符串。这样原来在文本格式的xml文件中的每一个放置字符串的地方在二进制格式的xml文件中都被替换成一个索引到字符串资源池的整数值。这样做有两个好处：
1.文件占用更小-----例如，假设在原来的文本格式的XML文件中，有四个地方使用的都是同一个字符串，那么在最终编译出来的二进制格式的XML文件中，字符串资源池只有一份字符串值，而引用它的四个地方只占用一个整数值。
2.解析速度更快-----由于在二进制格式的XML文件中，所有的XML元素标签和属性等值都是使用整数来描述的，因此，在解析的过程中，就不再需要进行字符串解析，这样就可以提高解析速度。

另外，每一个res资源在编译的打包完成之后，都会被分配一个资源ID，这些资源ID最终会被定义为Java常量值，保存在R.java文件中，与其应用的其他源文件一起被编译到程序中，这样就可以在程序或者资源文件中通过这些ID常量来访问指定的资源。

我们接下来再看应用程序资源的组织。应用程序资源的组织方式有18个维度，如图1所示：
<div align="center">
	<img src="/images/posts/Android apk打包过程/应用程序资源的匹配算法.png" height="300" width="500">  
</div>
这里有一点需要说明的是，表格中的18个维度是按照优先级从最大到小排列的，这个优先级次序可以帮助系统根据机器的本地配置来在应用程序资源目录中找到最合适的资源来使用。
<div align="center">
	<img src="/images/posts/Android apk打包过程/应用程序资源的匹配算法.png" height="300" width="500">  
</div>
  注意，图2的算法流程图是来自于官方文档的，它的详细描述可以参考：http://developer.android.com/guide/topics/resources/providing-resources.html#BestMatch。我们同样是通过上述官方文档中的例子来说明上述应用程序资源匹配算法的执行过程。
   假设一个应用程序的drawable资源按照以下方式来组织：
```plain
	drawable/  
    drawable-en/  
    drawable-fr-rCA/  
    drawable-en-port/  
    drawable-en-notouch-12key/  
    drawable-port-ldpi/  
    drawable-port-notouch-12key/  
```
并且该应用程序所运行在的设置的配置情况如下所示：
```plain
	Locale = en-GB   
    Screen orientation = port   
    Screen pixel density = hdpi   
    Touchscreen type = notouch   
    Primary text input method = 12key   
```
根据图2所示的算法，Android资源管理框架按照以下步骤来选择一个drawable资源：

 Step 1. 消除与设备配置冲突的drawable目录，即drawable-fr-rCA目录，因为设备设置的语言是en-GB。

```palin
    drawable/  
    drawable-en/  
    drawable-en-port/  
    drawable-en-notouch-12key/  
    drawable-port-ldpi/  
    drawable-port-notouch-12key/  
```
Step 2. 从MMC开始，选择一个资源组织维度来过渡从Step 1筛选后剩下来的目录。

Step 3. 检查Step 2选择的维度是否有对应的资源目录。如果没有，就返回到Step 2继续处理。如果有，那么就继续往下执行Step 4。在我们这个例子中，要一直重复执行Step 2，直到检查到language这个维度时。

Step 4. 消除那些不包含有Step 2所选择的资源维度的目录。在我们这个例子中，就是要消除那些不包含有en这个language的目录：

```plain

    drawable-en/  
    drawable-en-port/  
    drawable-en-notouch-12key/  
```
Step 5.  继续执行Step 2、Step 3和Step 4，直到找到一个最匹配的资源目录为止，即剩下最后一个目录为止。在我们这个例子中，下一个要检查的维度是screen orienation。由于设备的screen orienation为port，因此，所有不包含有port资源维度的目录将被消除：
```plain
    drawable-en-port/  
```
最后剩下来的目录就只有drawable-en-port，因此，它就是最匹配的资源目录了，这时候所有drawable类型的资源都可以从这个目录中获取。

应用程序资源的编译、打包以及查找过程
<div align="center">
	<img src="/images/posts/Android apk打包过程/应用程序资源的编译、打包以及查找过程.jpg" height="300" width="500">  
</div>
通过图3可以发现：
1.除了assets和res/raw资源被原封不动地打包进apk之外，其他资源都会被编译或者处理。
2.除了assets资源之外，其他的资源都会被赋予一个资源id。
3.打包工具负责编译和打包资源，编译完成之后，会生成一个resources.arsc文件和一个R.java，前者保存的时一个资源索引表，后者保存各个资源id常量。
4.应用程序配置文件AndroidManifest.xml同样会被编译成二进制xml文件，然后再打包到apk中，
5.应用程序运行时通过AssetManager来访问资源，或者通过资源ID来访问，或通过文件名来访问。

###Android资源打包工具的执行过程(应用程序资源的编译和打包过程)
<div align="center">
	<img src="/images/posts/Android apk打包过程/应用程序资源的编译、打包以及查找过程.jpg" height="300" width="500">  
</div>

```java
project  
      --AndroidManifest.xml  
      --res  
        --drawable-ldpi  
          --icon.png  
        --drawable-mdpi  
          --icon.png  
        --drawable-hdpi  
          --icon.png  
        --layout  
          --main.xml  
          --sub.xml  
        --values  
          --strings.xml 
```
根据上图分析应用程序资源的编译和打包过程：
 一 解析AndroidMainfest.xml
 解析它时为了获得要变异的应用程序的包名称。有了包名后，可以创建资源表，即创建一个ResourceTable对象。
 
 二 添加被引用资源包
 我们在编译一个Android应用程序的资源的时候，至少会涉及到两个包，其中一个被引用的系统资源包，另外一个就是当前正在编译的应用程序资源包。每一个包都可以定义自己的资源，同时它也可以引用其它包的资源。那么，一个包是通过什么方式来引用其它包的资源的呢？这就是我们熟悉的资源ID了。资源ID是一个4字节的无符号整数，其中，最高字节表示Package ID，次高字节表示Type ID，最低两字节表示Entry ID。

三. 收集资源文件

  在编译应用程序资源之前，Android资源打包工具aapt会创建一个AaptAssets对象，用来收集当前需要编译的资源文件。这些需要编译的资源文件就保存在AaptAssets类的成员变量mRes中，如下所示：
  ```java
  class AaptAssets : public AaptDir  
{  
    ......  
  
private:  
    ......  
  
    KeyedVector<String8, sp<ResourceTypeSet> >* mRes;  
}; 
```
 AaptAssets类定义在文件frameworks/base/tools/aapt/AaptAssets.h中。

        AaptAssets类的成员变量mRes是一个类型为ResourceTypeSet的KeyedVector，这个KeyedVector的Key就是资源的类型名称。由此就可知，收集到资源文件是按照类型来保存的。例如，对于我们在这篇文章中要用到的例子，一共有三种类型的资源，分别是drawable、layout和values，于是，就对应有三个ResourceTypeSet。

        从前面的图3可以看出，ResourceTypeSet类本身描述的也是一个KeyedVector，不过它里面保存的是一系列有着相同文件名的AaptGroup。例如，对于我们在这篇文章中要用到的例子：

        1. 类型为drawable的ResourceTypeSet只有一个AaptGroup，它的名称为icon.png。这个AaptGroup包含了三个文件，分别是res/drawable-ldpi/icon.png、res/drawable-mdpi/icon.png和res/drawable-hdpi/icon.png。每一个文件都用一个AaptFile来描述，并且都对应有一个AaptGroupEntry。每一个AaptGroupEntry描述的都是不同的资源配置信息，即它们所描述的屏幕密度分别是ldpi、mdpi和hdpi。

        2. 类型为layout的ResourceTypeSet有两个AaptGroup，它们的名称分别为main.xml和sub.xml。这两个AaptGroup都是只包含了一个AaptFile，分别是res/layout/main.xml和res/layout/sub.xml。这两个AaptFile同样是分别对应有一个AaptGroupEntry，不过这两个AaptGroupEntry描述的资源配置信息都是属于default的。

        3. 类型为values的ResourceTypeSet只有一个AaptGroup，它的名称为strings.xml。这个AaptGroup只包含了一个AaptFile，即res/values/strings.xml。这个AaptFile也对应有一个AaptGroupEntry，这个AaptGroupEntry描述的资源配置信息也是属于default的。

 四. 将收集到的资源增加到资源表
前面收集到的资源只是保存在一个AaptAssets对象中，这一步需要将这些资源同时增加到一个资源表中去，即增加到前面所创建的一个ResourceTable对象中去，因为最后我们需要根据这个ResourceTable来生成资源索引表，即生成resources.arsc文件。

五. 编译values类资源
 类型为values的资源描述的都是一些简单的值，如数组、颜色、尺寸、字符串和样式值等，这些资源是在编译的过程中进行收集的。
 
 六. 给Bag资源分配ID
类型为values的资源除了是string之外，还有其它很多类型的资源，其中有一些比较特殊，如bag、style、plurals和array类的资源。这些资源会给自己定义一些专用的值，这些带有专用值的资源就统称为Bag资源。例如，Android系统提供的android:orientation属性的取值范围为｛“vertical”、“horizontal”｝，就相当于是定义了vertical和horizontal两个Bag。

 七. 编译Xml资源文件
前面的六步操作为编译Xml资源文件准备好了所有的素材，因此，现在就开始要编译Xml资源文件了。除了values类型的资源文件，其它所有的Xml资源文件都需要编译。
这里我们只挑layout类型的资源文件来说明Xml资源文件的编译过程，也就是这篇文章中要用到的例子中的main.xml文件，它的内容如下所示：
```java
    <?xml version="1.0" encoding="utf-8"?>  
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"  
        android:orientation="vertical"  
        android:layout_width="fill_parent"  
        android:layout_height="fill_parent"   
        android:gravity="center">  
        <Button   
            android:id="@+id/button_start_in_process"  
            android:layout_width="wrap_content"  
            android:layout_height="wrap_content"  
            android:gravity="center"  
            android:text="@string/start_in_process" >  
        </Button>  
        <Button   
            android:id="@+id/button_start_in_new_process"  
            android:layout_width="wrap_content"  
            android:layout_height="wrap_content"  
            android:gravity="center"  
            android:text="@string/start_in_new_process" >  
        </Button>  
    </LinearLayout>  
    ```
Xml资源文件main.xml的编译过程如图8所示：
 <div align="center">
	<img src="/images/posts/Android apk打包过程/Xml资源文件的编译过程.jpg" height="300" width="500">  
</div>
  1. 解析Xml文件
  每一个XMLNode都表示一个Xml元素，其中：

        --mElementName，表示Xml元素标签。

        --mChars，表示Xml元素的文本内容。

        --mAttributes，表示Xml元素的属性列表。

        --mChildren，表示Xml元素的子元素。

  Xml文件解析完成之后，就可以得到一个用来描述根节点的XMLNode，接下来就可以通过这个根节点来完成其它的编译操作。
  
  2. 赋予属性名称资源ID
  这一步实际上就是给每一个Xml元素的属性名称都赋予资源ID。例如，对于main.xml文件的根节点LinearLayout来说，就是要分别给它的属性名称“android:orientation”、“android:layout_width”、“android:layout_height”和“android:gravity”赋予一个资源ID。注意，上述这些属性都是在系统资源包里面定义的，因此，Android资源打包工具首先是要在系统资源包里面找到这些名称所对应的资源ID，然后才能赋给main.xml文件的根节点LinearLayout。
  3. 解析属性值
  对Xml元素的属性的值进行解析。例如，对于对于main.xml文件的根节点LinearLayout来说，前面我们已经给它的属性android:orientation的名称赋予了一个资源ID，这里就要给它的值“vertical”进行解析。
  4. 压平Xml文件
 经过前面的三步操作之后，所需要的基本材料都已经准备好了，接下来就可以对Xml文件的内容进行扁平化处理了，实际上就是将Xml文件从文本格式转换为二进制格式
 
  八. 生成资源符号
   这里生成资源符号为后面生成R.java文件做好准备的。从前面的操作可以知道，所有收集到的资源项都按照类型来保存在一个资源表中，即保存在一个ResourceTable对象。因此，Android资源打包工具aapt只要遍历每一个Package里面的每一个Type，然后取出每一个Entry的名称，并且根据这个Entry在自己的Type里面出现的次序来计算得到它的资源ID，那么就可以生成一个资源符号了，这个资源符号由名称以及资源ID所组成。

        例如，对于strings.xml文件中名称为“start_in_process”的Entry来说，它是一个类型为string的资源项，假设它出现的次序为第3，那么它的资源符号就等于R.string.start_in_process，对应的资源ID就为0x7f050002，其中，高字节0x7f表示Package ID，次高字节0x05表示string的Type ID，而低两字节0x02就表示“start_in_process”是第三个出现的字符串。
        
 九. 生成资源索引表
 经过上述八个操作之后，所获得的资源列表
 <div align="center">
	<img src="/images/posts/Android apk打包过程/收集到的所有资源项.jpg" height="300" width="500">  
</div>
 有了这些资源项之后，Android资源打包工具aapt就可以生成资源索引表resources.arsc了
 
  十. 编译AndroidManifest.xml文件
    经过前面的九个步骤之后，应用程序的所有资源项就编译完成了，这时候就开始将应用程序的配置文件AndroidManifest.xml也编译成二进制格式的Xml文件。之所以要在应用程序的所有资源项都编译完成之后，再编译应用程序的配置文件，是因为后者可能会引用到前者。

        应用程序配置文件AndroidManifest.xml的编译过程与其它的Xml资源文件的编译过程是一样的，可以参考前面的第七步。注意，应用程序配置文件AndroidManifest.xml编译完成之后，Android资源打包工具appt还会验证它的完整性和正确性，例如，验证AndroidManifest.xml的根节点mainfest必须定义有android:package属性。
        
十一. 生成R.java文件
  在前面的第八步中，我们已经将所有的资源项及其所对应的资源ID都收集起来了，因此，这里只要将直接将它们写入到指定的R.java文件去就可以了。
   注意，每一个资源类型在R.java文件中，都有一个对应的内部类，例如，类型为layout的资源项在R.java文件中对应的内部类为layout，而类型为string的资源项在R.java文件中对应的内部类就为string。

十二. 打包APK文件
所有资源文件都编译以及生成完成之后，就可以将它们打包到APK文件去了，包括：

1. assets目录。
2. res目录，但是不包括res/values目录， 这是因为res/values目录下的资源文件的内容经过编译之后，都直接写入到资源项索引表去了。
3. 资源项索引文件resources.arsc。
4. 
当然，除了这些资源文件外，应用程序的配置文件AndroidManifest.xml以及应用程序代码文件classes.dex，还有用来描述应用程序的签名信息的文件，也会一并被打包到APK文件中去，这个APK文件可以直接拿到模拟器或者设备上去安装。

至此，我们就分析完成Android应用程序资源的编译和打包过程了，其中最重要的是要掌握以下四个要点：

        1.  Xml资源文件从文本格式编译为二进制格式的过程。

        2. Xml资源文件的二进制格式。

        3. 资源项索引表resources.arsc的生成过程。

        4. 资源项索引表resources.arsc的二进制格式。

   在一个apk文件中，除了代码文件之外，还有资源文件。资源文件时通过安卓资源打包工具aapt（Android Asset Package Tool）打包到APK文件中的。在打包前，大部分文本格式的XML资源文件还会被编译成二进制格式的xml资源文件。
   
##应用程序资源的编译和打包过程
第一步：打包资源文件，生成R.java文件

【输入】Resource文件（就是工程中res中的文件）、Assets文件（相当于另外一种资源，这种资源Android系统并不像对res中的文件那样优化它）、AndroidManifest.xml文件（包名就是从这里读取的，因为生成R.java文件需要包名）、Android基础类库（Android.jar文件）

【输出】打包好的资源（一般在Android工程的bin目录可以看到一个叫resources.ap_的文件就是它了）、R.java文件（在gen目录中，大家应该很熟悉了）

【工具】aapt工具，它的路径在${ANDROID_SDK_HOME}/build-tools/21.1.2/aapt。

第二步：处理AIDL文件，生成对应的.java文件（当然，有很多工程没有用到AIDL，那这个过程就可以省了）

【输入】源码文件、aidl文件、framework.aidl文件

【输出】对应的.java文件

【工具】aidl工具

第三步：编译Java文件，生成对应的.class文件

【输入】源码文件（包括R.java和AIDL生成的.java文件）、库文件（.jar文件）

【输出】.class文件

【工具】javac工具

第四步：把.class文件转化成Davik VM支持的.dex文件

【输入】源码文件（包括R.java和AIDL生成的.java文件）、库文件（.jar文件）

【输出】.class文件

【工具】javac工具

第五步：打包生成未签名的.apk文件

【输入】打包后的资源文件、打包后类文件（.dex文件）、libs文件（包括.so文件，当然很多工程都没有这样的文件，如果你不使用C/C++开发的话）

【输出】未签名的.apk文件

【工具】apkbuilder工具

第六步：对未签名.apk文件进行签名

【输入】未签名的.apk文件

【输出】签名的.apk文件

【工具】jarsigner

第七步：对签名后的.apk文件进行对齐处理（不进行对齐处理是不能发布到Google Market的）

【输入】签名后的.apk文件

【输出】对齐后的.apk文件

【工具】zipalign工具



1. 先把java源文件自动编译成classes文件
2. 把classes文件编译和打包成classes.dex文件(dx目录${ANDROID_SDK_HOME}/build-tools/21.1.2/dx)
3. 把dex文件，资源映射文件，未压缩的资源，清单文件打包成apk
4. 给应用程序进行签名
5. 使用adb（android debug briage）工具上传并安装apk
6.  apk在手机上的安装过程：拷贝xxx.apk 到 /data/app/xxx-1.apk（系统应用存在/system/app/目录下），在 /data/data 目录下创建文件夹，名称就是包名，同时会在/data/system/packages.xml注册表文件里面添加纪录
```java
package name="包名" codePath="/data/app/包名-1.apk" nativeLibraryPath="/data/app-lib/包名-1" flags="572998" ft="15302011250" it="15302011475" ut="15302011475" version="1" userId="10048">

<sigs count="1">

<cert index="4" key="1000位的签名" />

</sigs>

<perms />

</package>
```