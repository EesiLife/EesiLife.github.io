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
![](file:///home/siy/md_haroopad/Android apk打包过程/应用程序资源的组织方式.jpg)
这里有一点需要说明的是，表格中的18个维度是按照优先级从最大到小排列的，这个优先级次序可以帮助系统根据机器的本地配置来在应用程序资源目录中找到最合适的资源来使用。
![](file:///home/siy/md_haroopad/Android apk打包过程/应用程序资源的匹配算法.png)
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
![](file:///home/siy/md_haroopad/Android apk打包过程/应用程序资源的编译、打包以及查找过程.jpg)
通过图3可以发现：
1.除了assets和res/raw资源被原封不动地打包进apk之外，其他资源都会被编译或者处理。
2.除了assets资源之外，其他的资源都会被赋予一个资源id。
3.打包工具负责编译和打包资源，编译完成之后，会生成一个resources.arsc文件和一个R.java，前者保存的时一个资源索引表，后者保存各个资源id常量。
4.应用程序配置文件AndroidManifest.xml同样会被编译成二进制xml文件，然后再打包到apk中，
5.应用程序运行时通过AssetManager来访问资源，或者通过资源ID来访问，或通过文件名来访问。

Android资源打包工具的执行过程
![](file:///home/siy/md_haroopad/Android apk打包过程/Android资源打包工具的执行过程.jpg)

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
 




















##应用程序资源的编译和打包过程
   在一个apk文件中，除了代码文件之外，还有资源文件。资源文件时通过安卓资源打包工具aapt（Android Asset Package Tool）打包到APK文件中的。在打包前，大部分文本格式的XML资源文件还会被编译成二进制格式的xml资源文件。
   