# 第一章：jME3简介

## 概述 

**jMonkeyEngine3**是一个用纯Java开发的免费3D游戏引擎。拜开源社区所赐，它拥有非常非常多的功能，远多于一般游戏开发的需要。而且它的API非常简洁明了，只需要花少量的学习时间就可以掌握。

* 场景管理
* 可编程材质(着色器)
* 图形用户界面
* 内存资源管理
* 输入系统
* 声音系统
* 碰撞检测
* 物理引擎
* 特效系统
* 骨骼动画
* 地形系统
* 灵活易扩展的逻辑模块
* 性能优化(八叉树、层次细节、硬件加速等)

从本质上来讲，jME3就是一个第三方Java类库，一旦在你的Java/Android项目中引用了jME3的jar文件，就可以获得3D引擎的能力。

下面只用7行有效代码，就可以在手机上显示钢铁侠3D模型。

![钢铁侠](/content/images/2017/03/android_studio2.png)

下图是jME3的一些主要的jar。这些类库按不同的功能分别打包，可以根据需要来添加到项目中。

![jar文件](/content/images/2017/03/jME3.png)

jME3的这些组件已经发布到了公共仓库中：

* [JCenter](https://jcenter.bintray.com/org/jmonkeyengine/)

* [Bintray repo: org.jmonkeyengine](https://bintray.com/jmonkeyengine/org.jmonkeyengine)

jME3类库的Group ID是`org.jmonkeyengine`，最新的版本号(version)为`3.1.0-stable`。下列组件已经全部上传到公共仓库中，你可以通过Gradle或Maven去获取它们：

*    `jme3-core` - 任何jME3项目都需要的核心库
*    `jme3-effects` - 各种滤镜、粒子、水面等特效。
*    `jme3-networking` - jME3的网络模块(别名SpiderMonkey)。
*    `jme3-plugins` - 加载orge、fbx等模型文件的插件。
*    `jme3-jogg` - 加载jogg格式的音频文件。
*    `jme3-terrain` - 地形生成API，可使用高度图来生成3D地形。
*    `jme3-blender` - 加载blender模型文件，仅适用于桌面开发，手机显卡不支持。
*    `jme3-jbullet` - 基于`jbullet`的物理引擎（仅适用于桌面开发，手机用不了，而且JCenter上没有这个组件）。`jme3-jbullet`和`jme3-bullet`只能二选一，不能同时存在于同一个项目中。
*    `jme3-bullet` - 基于`BulletPhysics`的物理引擎，需要`jme3-bullet-native`或`jme3-bullet-native-android`。
*    `jme3-bullet-native` - `BulletPhysics`所需的静态库文件（dll、so），仅适用于桌面开发。注意：`jbullet`跟`BulletPhysics`是两码事，它不需要这些本地库文件。
*    `jme3-bullet-native-android` - `BulletPhysics`所需的静态库文件（dll、so），仅适用于Android开发。
*    `jme3-niftygui` - 为jME3添加NiftyGUI支持，可以使用NiftyGUI来制作图形用户界面 (JCenter上没有这个组件)。
*    `jme3-desktop` - jME3桌面应用开发所需的核心API。
*    `jme3-lwjgl` - jME3的桌面应用渲染模块，依赖LWJGL。
*    `jme3-lwjgl3` - jME3.1新增的模块! 使用LWJGL3为桌面进行渲染。
*    `jme3-jogl` - jME3的桌面应用渲染模块，依赖JOGL。它是LWJGL和LWJGL3的替代品，可选。有LWJGL你就不需要JOGL，用JOGL就不需要LWJGL。
*    `jme3-android` - jME3的Android应用核心模块。
*    `jme3-android-native` - jME3开发Android应用所需的本地库文件。
*    `jme3-ios` - jME3开发iOs应用的核心API (JCenter上没有这个组件)

jME3开发桌面应用，你至少需要下面这几个组件：

*    `jme3-core`
*    `jme3-desktop`
*    `jme3-lwjgl` 或 `jme3-lwjgl3`

jME3开发Android应用，你至少需要下面这几个组件：

*    `jme3-core`
*    `jme3-android`
*    `jme3-android-native`

## jME3 SDK

**jMonkeyEngine3 SDK**是开发团队基于NetBeans平台开发的jME3集成开发环境。它包含场景制作、模型预览、材质编辑等诸多游戏开发所需的功能。jMonkeyPlatform跟jME3 SDK是一码事，说的都是这个东西。

![SDK](/content/images/2017/03/jME3_SDK.png)

![SDK 3.0](/content/images/2017/03/jME3_SDK_3-0.png)

官方推荐初学者使用jME3 SDK来开发游戏。官方下载地址为：
https://github.com/jMonkeyEngine/sdk/releases

无论你习惯用jME3 SDK，还是Eclipse、IntelliJ IDEA，或者Android Studio，都可以使用jMonkeyEngine3来开发游戏。毕竟这个引擎的本质就是一堆jar文件，选择自己熟悉的工具可以提升你的开发效率。

注意：在jME3.1.0之后SDK就出了一个对中国开发者不太友好的bug，菜单上的中文变成了方框乱码。

![SDK 3.1](/content/images/2017/03/jME3_SDK_3-1.png)

这个bug的原因是NetBeans会自动根据用户电脑的语言来进行本地化，但却使用了一种不支持中文的字体。我向官方团队反馈了这个bug[#73](https://github.com/jMonkeyEngine/sdk/issues/73)，但是到目前为止都没有修复。

如果使用SDK的话，我建议你把它的语言换成英文，反而会更好用一些。

修复方法很简单，在jME3 SDK的安装目录下找到`etc`文件夹中的`jmonkeyplatform.conf`文件，使用记事本或者Editplus之类的文本编辑工具打开它
。找到 `default_options` 配置项，在末尾加上`-J-Duser.country=US`，将地区设置成美国，这样再次打开jME3 SDK的时候界面就变成了全英文。修改后的`default_options`看起来差不多是这样的：

    default_options="--branding jmonkeyplatform -J-Xms24m -J-Dsun.java2d.dpiaware=true -J-Dapple.laf.useScreenMenuBar=true -J-Dawt.useSystemAAFontSettings=lcd -J-Dswing.aatext=true -J-Xmx512m -J-XX:MaxDirectMemorySize=2048m -J-Dsun.zip.disableMemoryMapping=true -J-Dapple.awt.graphics.UseQuartz=true -J-Dsun.java2d.noddraw=true -J-Duser.country=US"

## 获取jME3

官方下载分流：

* 百度网盘分流-3.1核心包：
http://pan.baidu.com/s/1hrRPeJE
* 百度网盘分流-SDK-3.1-stable-EXE：
http://pan.baidu.com/s/1pK7ZQ5d
* 百度网盘分流-SDK-3.1-stable-ZIP：
http://pan.baidu.com/s/1kUPddkV

### JAR与源代码

jME3在github.com的地址是 [github.com/jMonkeyEngine/jmonkeyengine](https://github.com/jMonkeyEngine/jmonkeyengine)，你可以通过[Clone or download](https://github.com/jMonkeyEngine/jmonkeyengine/archive/master.zip)直接下载它的源代码，或者从[release](https://github.com/jMonkeyEngine/jmonkeyengine/releases)中下载最新发布的jar包。

![github](/content/images/2017/03/jME3_github.png)

如果从github下载比较慢，你也可以通过我的百度网盘下载这个引擎。

jME3.1 稳定版：http://pan.baidu.com/s/1hrRPeJE

如果你是使用Eclipse做开发，我曾写过一个[教程](http://bbs.jmecn.net/t/topic/25)，还录过一套[视频](http://www.bilibili.com/video/av6442041/)。质量不高，但还是推荐给你。

如果你更喜欢maven或gradle之类的工具，那就更好了。在你的项目依赖中添加如下的代码：

### Gradle
	
	repositories {
	    jcenter()
	    //maven { url "http://dl.bintray.com/jmonkeyengine/org.jmonkeyengine" }
	}
	
	dependencies {
	    def jme3 = [g:'org.jmonkeyengine', v:'3.1.0-stable']
	    compile "$jme3.g:jme3-core:$jme3.v"
	    compile "$jme3.g:jme3-desktop:$jme3.v"
	    compile "$jme3.g:jme3-lwjgl:$jme3.v"
	}

例如，在Android Studio中：

![Android Studio](/content/images/2017/03/android_studio.png)

例如，Eclipse Neon 在安装了 BuildShip插件后，也可以使用Gradle。

![Eclipse with Gradle](/content/images/2017/03/eclipse2.png)

### Maven

    <properties>
      <jme3_g>org.jmonkeyengine</jme3_g>
      <jme3_v>3.1.0-stable</jme3_v>
    </properties>

    <repositories>
      <repository>
        <id>jcenter</id>
        <url>http://jcenter.bintray.com</url>
      </repository>
    </repositories>

    <dependencies>
      <dependency>
        <groupId>${jme3_g}</groupId>
        <artifactId>jme3-core</artifactId>
        <version>${jme3_v}</version>
      </dependency>
      <dependency>
        <groupId>${jme3_g}</groupId>
        <artifactId>jme3-desktop</artifactId>
        <version>${jme3_v}</version>
        <scope>runtime</scope>
      </dependency>
      <dependency>
        <groupId>${jme3_g}</groupId>
        <artifactId>jme3-lwjgl</artifactId>
        <version>${jme3_v}</version>
      </dependency>
    </dependencies>

## 官方教程和例子

在jME3的github页面中，有一个名为jme3-examples的模块，其中有大量的示例源代码，全面展示了jME3的各种功能和特性。

所有例子中，`jme3test.helloworld`包中的13个例子是最基础的。官方推荐的学习顺序是：

    第一个JME3程序(HelloJME3) – 实现一个简单的程序
    场景管理(HelloNode) – 在场景图中改变几何体和节点属性
    资源管理(HelloAssets) – 加载三维模型、场景和其他的资源
    事件循环(HelloLoop) – 在事件循环中实现事件控制功能
    输入处理(HelloInput) – 对于键盘和鼠标的输入作出响应
    材质(HelloMaterial) – 设置材质、纹理、透明度
    动画(HelloAnimation) – 控制动画模型
    拣选(HelloPicking) – 射击、压下按钮、选择、捡起选项
    碰撞(HelloCollision) – 建造墙壁和固体地板
    地形(HelloTerrain) – 使用贴图创建小山的风景
    音效(HelloAudio) – 按照位置和事件来实现三维音效
    特效(HelloEffects) – 创建粒子特效，比如：火焰、爆炸、魔法
    物理(HelloPhysics) – 撞球和坠落的砖头

如果你下载了`jME3.1-stable`，解压缩后可以在文件夹中看到一个名为`jMonkeyEngine.jar`的可执行jar文件，其中就是编译过的example。运行这个程序，可以查看官方提供的诸多例子。

![examples](/content/images/2017/03/examples.png)

![examples2](/content/images/2017/03/examples2.png)

如果你下载了jME3 SDK，通过选择菜单 File -> New Project，可以创建一个JME3的测试项目。这个项目会自动包含exmaples中的源代码，并可以编译运行。

![examples3](/content/images/2017/03/jme3test.png)

![jme3test](/content/images/2017/03/jme3test2.png)

如果你用的是Intellij IDEA 或者 Eclipse，我想你应该不介意下载源代码，然后像我这样把jme3-exmaples文件夹里面的东西直接拷贝到自己的项目里面吧？

![eclipse](/content/images/2017/03/eclipse.png)
