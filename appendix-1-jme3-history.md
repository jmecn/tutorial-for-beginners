# 序章：jME3的故事

## 起源

早在2003年的时候，那时的jME还不叫jMonkeyEngine，也不是一款游戏引擎。作者只是想要尝试一下Java的3D图形性能，于是启动了一个项目，只实现了最基本的场景图管理和图形渲染。

作者的名字叫做`Mark Powell`，他在论坛的昵称叫做`MojoMonk`。这个项目被命名为`MojoMonkey`，后来更名为`jMonkeyEngine`。

`Mark Powell`在实现了一个基本的图形引擎(Display System)后，又逐步为这个引擎添加了：

 * 声音系统(Sound System)
 * 输入系统(Input System)
 * 动画系统(Animation Controller)
 * 碰撞检测(Collision Detection)
 * 物理系统(Physical System)
 * 特效系统(Effects System)

等模块，将其发展成了一个完整的游戏引擎。并在2005年的游戏开发者大会(GDC)上首次将其公开介绍给整个Java社区。

## 关于这个名字

为什么这个引擎要叫做`jMonkeyEngine`？作者的原话是这么说的：

> Yes, before jME existed I used MojoMonkey as my gamer tags. Early in my career I worked for a military contractor and started studying graphics engines for rendering some things they were working on. This was all done in C++ and became complex enough that I figured that it was an engine, so jokingly called it the Monkey Engine. Fast forward a few years and I am working in Java and again need to do some graphics. I dust off the Monkey Engine and start converting it to Java, so start calling it jMonkey Engine (j for Java obviously).
>
> Then it sort of took off from there, Joshua Slack jumped on board, NCsoft hired us to build games with it, NCsoft started having failures and cutting teams, I moved on to other pursuits and handed it off to others, then this thread was written. The end.

译文：

> 早些时候我曾为一家军事承包商工作过，在那里初次接触、学习了图形引擎，主要工作就是开发程序，用于渲染他们需要的东西。那个程序是用纯C++开发的，其复杂度足以让我搞懂什么是“引擎”，我戏称它为“猴子引擎”。

> 几年后，我开始从事Java开发工作，同样需要搞一些图形化的东西。于是我把“猴子引擎”从尘埃中拾起，并且尝试把它转成Java程序，并开始称其为jMonkeyEngine（显然字母j代表Java）。

> 后来这个玩意就起飞了，`Joshua Slack`上了我的贼船，NCsoft公司雇佣我们俩来开发游戏，就用这个引擎。再后来NCsoft经历了失败和裁员，我的兴趣也转向了其他东西，并将jME交给了别人。当我写下这个帖子的时候，结束了。

*注1：他提到的NCsoft公司，就是韩国那家开发了“永恒之塔”、“天堂”、“天堂II”以及“剑灵”的公司。*

*注2：我在官方论坛找到了2007年时NCsoft通过mojomonk在论坛招人的帖子：
[NCsoft looking for engineers](https://hub.jmonkeyengine.org/t/ncsoft-looking-for-engineers/4136)*

## Logo和吉祥物

jMonkeyEngine3以猴子为名，它的吉祥物也是只小猴子，名叫Jaime。

![Jaime](/content/images/2017/03/Jaime.png)

`Nehon`使用Blender制作一个Jaime的3D动画模型，它现在是jME3-testdata中的一部分。

![Jaime2](/content/images/2017/03/Jaime2.png)

不知道你有没有注意到，Jaime的名字是一个双关语。它的发音就是 Jaime = 'J' 'M' 'E'三个字母连读。

## 我知道jME3，那jME1和jME2去哪了？

从2003年到2008年的这段时间，jME的主要开发工作是由Mark Powel来完成的。jME的版本从0.1一直升级到了2.0，最后不再更新。在他放弃这个项目之后，jME改由社区团队来接手，经过重新设计，在2007年愚人节那天推出了jME3的第一个预览版本。

总的来说，jME2和jME3是完全不同的两个东西。原作者在jME2时期之后就不再参与这个项目，目前的jME3完全是另一伙极客组成的核心团队在开发。在有了jME3之后，核心团队就放弃了jME2。jME2的源代码应该还留存在[GoogleCode](jmonkeyengine.googlecode.com)的代码仓库中，但是在2.0之后就没有继续维护下去了。

### Ardor3D

`Joshua Slack`是`Mark Powell`在jMonkeyEngine项目上的合作人，也是这个项目的核心开发者，他在论坛的昵称是`Renanse`。

当jME2项目被社区放弃之后，`Joshua Slack`和`Rikard Herlitz`在2008年9月23日发起了由jME2衍生而来的另一个项目，Ardor3D。

这个项目在2009年1月2日首次发布，此后几乎每个月都会有一次版本更新发布，直到2014年3月11日，`Joshua Slack`宣布放弃这个项目。

### JogAmp's Ardor3D Continuation

当Ardor3D被宣布废弃之后不久，一位名为Julien Gouesse的开发者宣布自己不愿放弃这个他倾注了大量心血的引擎，[打算自行维护这个项目](http://forum.jogamp.org/JOGL-2-support-for-Ardor3D-JMonkeyEngine-3-jzy3d-and-NiftyGUI-td1706747i60.html#a4033608)。他fork了Ardor3D的一个分支，称其为`JogAmp's Ardor3D Continuation`。

下面是作者为这个项目编写的手册：
https://gouessej.wordpress.com/2014/11/22/ardor3d-est-mort-vive-jogamps-ardor3d-continuation-ardor3d-is-dead-long-life-to-jogamps-ardor3d-continuation/

这个项目的一直处于活动状态，代码仓库位于Github：https://github.com/gouessej/Ardor3D

Github上最后一次提交是2016年8月份。

## jME编年史

jMonkeyEngine的历史分为2个时期：jME时期和jME3时期。

### jME 0.1 ~ 2.0

* 2003年6月
 - jMonkeyEngine项目启动。(只是一些图形工具集，还称不上一个引擎)
* 2003年9月
 - jME转变成一个基于场景图(scenegraph)的图形引擎。绝大部分早期API都是参考David Eberly所著的《3D Game Engine Design》，书中使用的是C++语言，Mark完成了一个Java实现。
* 2003年10月3日
 - jME 0.1被上传到一个CVS服务器。从这一天开始，用户需要通过CVS下载代码来安装jMonkeyEngine。主要功能：
 * 场景管理器(Scenegraph Manager)
* 2003年11月14日 - jME 0.2发布。增加了很多特性：
 * 渲染系统(RenderStates)
 * 声音系统(Sound System)
 * 输入系统(Input System)
 * 文字显示(Text output)
* 2003年12月
 - jME 0.3发布。增加功能如下：
 * 碰撞检测(Collision Detection)
 * 拣选(Picking)
 * 摄像机节点(CameraNode)
 * 背面剔除(Face culling)
 * 节点控制器(Node controllers)
 * 包围盒(BoundingBox)
 * 初始化实体(Initial Entity)
* 2004年1月16日
 - jME 0.4发布。新增功能：
 * 光源节点(Light Node)
 * 贝塞尔曲面(Bezier Mesh)
 * 贝塞尔曲线(Bezier Curve)
* 2004年2月
 - Joshua Slack ("Renanse")加入开发。
 * 新增功能：动画
* 2004年6月
 - 新增功能：RenderQueue
 - 同时Mark正在开发特效系统、粒子系统、镜头光晕
* 2004年8月
 - Jack Lindamood ("Cep21")加入开发，并为jME新增了.jme格式的模型文件。
* 2005年3月
 - jME在游戏开发者大会(GDC)上公开展示。
* 2005年6月
 - llama 和 irrisor 加入团队。
* 2008年4月
 * jME2.0 pre-alpha 预览版发布
* 2008年8月15日August 15, 2008
 * Joshua Slack宣布退出jMonkeyEngine开发
* 2009年1月24日
 * jME 2.0 发布
*2009年9月9日
 * jME 2.0.1 发布
* 2011年1月28日
 * jME 2.1 最终版发布

### jME 3.0 ~ 现在

* 2009年4月1日
 * jME3实验版首次亮相
* 2009年6月24日
 * jME3实验版成为官方版。
 * 主要开发者、设计者：Kirill Vainer
 * 项目管理：Erlend Sogge Heggen
 * 成员：Skye Book.
* 2010年5月17日
 * jME3 alpha版发布
 * jME3 SDK alpha版发布
* 2010年9月7日
 * jME官网经过重新设计，域名从jmonkeyengine.com更换为jmonkeyengine.org。
* 2011年10月22日
 * jME3 SDK Beta版发布
* 2014年2月15日。
 * jME3 SDK 3.0 稳定版发布
* 2014年3月21日
 * jME源代码迁移到Github和Gradle
* 2015年8月18日
 * jME3.1-alpha1 发布
* 2016年2月29日
 * jME3.1-alpha2 发布
* 2016年3月14日
 * jME3.1-alpha3 发布
 * **SDK不再和引擎同时发布，改为独立发布。**
* 2016年3月28日
 * jME3.1-alpha4 发布
* 2016年4月11日
 * jME3.1-alpha5 发布
* 2016年4月15日
 * jME3.1-beta1 发布
* 2016年11月22日
 * jME3.1-beta2 发布
* 2017年1月29日
 * jME3.1-beta3 发布
* 2017年2月11日
 * jME3.1-beta4 发布
* 2017年2月13日
 * jME3.1-stable 发布

[1]:https://en.wikipedia.org/wiki/Scene_graph
[2]:http://www.cnblogs.com/soroman/archive/2009/12/31/1637176.html
