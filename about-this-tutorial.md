# jMonkeyEngine游戏开发课程

## 选题概述

### 选题名称

基于jMonkeyEngine引擎的3D游戏开发

### 职业方向

Android游戏开发、Java游戏开发

### 课程背景综述

目前国内3D游戏开发几乎Unity3D一家独大，很多Java、Android工程师想转向游戏开发时不得不重新进行技能培训，为何不直接使用我们熟悉的语言、熟悉的IDE来进行开发呢？

jMonkeyEngine是一款免费、开源、纯Java的游戏引擎，虽然在性能上无法与Unity3D比肩，但这个引擎具备几乎Unity3D的全部功能。更进一步来说，使用这款引擎，就等于拥抱无数开源的Java类库。随着现代智能手机的性能不断提升，性能方面的差距也会不断缩小。

jMonkeyEngine拥有相当详细的注释和文档，但全部都是英文的。国内有一些开发者因为兴趣而尝试翻译部分文档，贡献给开源社区。但是一直不成体系和规模。通过这门课程，我将系统地介绍JME的引擎的内含，并通过实践案例来让开发者迅速掌握这个引擎，并运用到实际开发中去。

至少，为所有想做3D游戏开发的Java/Android开发工程师提供多一种选择！

## 选题技术需求

### 技术需求
* 使用场景

 游戏开发, 虚拟现实(VR), 动画制作, 教育

* 技术趋势

 引擎功能已经趋于成熟，企业应用还比较少，但已经有一些公司在快速开发Android 3D应用时尝试这种引擎。

* 热点关键词

 `免费`,`开源`,`Java`,`地形`,`GUI图形用户界面`,`GLSL着色器`,`网络游戏`,`粒子特效`,`Bullet物理引擎`,`人工智能`

* 参考资料

 * [源代码](https://github.com/jMonkeyEngine/jmonkeyengine)
 * [官方网站](https://jmonkeyengine.org)
 * [中文网站](http://www.jmecn.net)

### 市场需求
* 市场招聘：不明
* 媒体报道：不明

## 教学内容

### 选题所属阶段
 * 入门、`基础`、进阶(就业)、专项
### 选题对应岗位
 * Android游戏开发岗
 * 独立游戏制作人
### 对应的岗位技能必要性
 * 精通、`掌握`、了解
### 前置课程
 * Java面向对象编程语言
 * Android应用开发(可选)
 * 3D游戏数学基础(可选)

## 教学大纲

### [1. jME3简介](http://blog.jmecn.net/chapter-1-introduce-jme3/)

![](/content/images/2017/03/android_studio2.png)

1.1 概述
1.2 jME3 SDK
1.3 获取jME3
1.4 官方教程和例子

### [2. JME3基本概念](http://blog.jmecn.net/chapter-2-basic-concepts/)

![](/content/images/2017/03/FlyCam.png)

2.1 应用程序主类SimpleApplication
2.2 生命周期
2.3 主循环
2.4 场景结构 Spatial、Node、Geometry
2.5 资源管理 AssetManager
2.6 输入管理 InputManager
2.7 状态机管理 AppStateManager

### [3. 模型](http://blog.jmecn.net/chapter-3-model/)

![](/content/images/2017/03/Ashe_AA_4x.png)

3.1 理解3D模型
3.2 模型的来源
3.3 实例：寒冰射手-艾希
3.4 实例：加载3D模型

### [4. 网格](http://blog.jmecn.net/chapter-4-mesh/)

![](/content/images/2017/03/sphere.png)

4.1 定义模型的形状
4.2 实例：自定义网格
4.3 程序生成网格
4.4 扩展阅读：渲染管线

### [5. 材质，障眼法](http://blog.jmecn.net/chapter-5-material-the-light-magic/)

![](/content/images/2017/04/PostCartoonEdge.png)

5.1 五色令人目盲
5.2 jME3的材质
5.3 加载j3md材质
5.4 改变材质参数
5.5 扩展阅读：UV坐标

### [6. 材质系统](http://blog.jmecn.net/chapter-6-material-system/)

![](/content/images/2017/04/Materials_shininess.png)

6.1 材质系统
6.2 材质实例：j3m文件
6.3 材质模板：j3md文件
6.4 附录

### [7. 光与影](http://blog.jmecn.net/chapter-7-light-and-shadow/)

![](/content/images/2017/04/PointLight.png)

7.1 感受光影
7.2 光源
7.3 阴影
7.4 光与材质

### [8. 场景图](http://blog.jmecn.net/chapter-8-scene-graph/)

![](/content/images/2017/04/SceneGraph.svg)

8.1 概念
8.2 实例：HelloNode
8.3 Node
8.4 遍历场景图

### [9. 用户交互](http://blog.jmecn.net/chapter-9-user-interaction/)

![](/content/images/2017/05/wheel.png)

9.1 键盘、鼠标、手柄、触屏
9.2 ActionListener
9.3 RawInputListener
9.4 动作触发器

### [10. 图形用户界面](http://blog.jmecn.net/chapter-10-graphics-user-interface/)

![](/content/images/2017/05/Picture.png)

10.1 GuiNode
10.2 屏幕坐标系
10.3 BitmapFont
10.4 Lemur GUI插件

### [11. 3D音效](http://blog.jmecn.net/chapter-11-3d-audio/)

![](/content/images/2017/05/LemurMusicPlayer.png)

11.1 3D音效
11.2 音效系统分析

### [12. 动画](http://blog.jmecn.net/chapter-12-animation)

![HelloAnimation](/content/images/2017/05/HelloAnimation.png)

12.1 概述
12.2 骨骼蒙皮动画
12.3 播放动画
12.4 操纵骨骼
12.5 运动路径
12.6 剧情动画

### [13. 控制游戏逻辑](http://blog.jmecn.net/chapter-13-controlling-game-logic)
![](/content/images/2017/06/AppState-Lift-Cycle.svg)
13.1 导读：游戏主循环
13.2 jME3的主循环
13.3 AppState
13.4 Control
13.5 多线程优化

### [14. 特效](http://blog.jmecn.net/chapter-14-special-effects/)
![](/content/images/2017/06/ParticleFire.png)
14.1 特效概述
14.2 后期滤镜
14.3 场景处理器
14.4 粒子系统
14.5 性能问题

### [15. 碰撞检测](http://blog.jmecn.net/chapter-15-collision-detection)
15.1 AABB包围盒
15.2 网格检测
15.3 射线检测
15.3.1 实例：屏幕拣选

### [16. 物理引擎](http://blog.jmecn.net/chapter-16-physics-engine)
16.1 牛顿的苹果
16.2 Bullet的物理空间
16.3 刚体碰撞
16.4 创建一个胶囊体
16.5 控制模型在地图中走动

### [17. 户外场景](http://blog.jmecn.net/chapter-17-outdoor-scene)
17.1 创建一个户外场景
17.2 天空
17.3 地形
17.4 水面
