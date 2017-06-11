# 第十五章：碰撞检测

## 碰撞与相交

**碰撞检测**这个词通常有两种含义，一种是物理意义上的碰撞检测，另一种是数学意义上的碰撞检测。本章讨论的是纯数学的碰撞检测，即判断物体之间是否相交（或包含、重合）、计算交点、预测相交时刻等。“物理引擎”则是在碰撞检测的基础上又增加了“物理”因素，模拟力与力之间的相互作用，这个内容将在下一章讨论。

显而易见，数学碰撞检测比物理碰撞检测的速度要快，而且消耗的内存也更少，因为压根就不用考虑质量、能量、动量、速度、加速度等物理因素。类似于鼠标拾取、视线范围等功能，使用射线与物体进行相交检测就可以轻易实现。有经验的游戏开发者会尽量使用数学方法来模拟物理引擎的功能，以此来优化游戏的性能。

![2d碰撞检测](content/images/2017/06/2d-collision-detection.png)

比如一款赛车游戏，你可以把赛车的4个轮子都加上物理特性，并实时计算车身的重量、轮胎的弹性、发动机的马力、地面的阻力、轴承的摩擦力等各种条件对车辆的影响。听起来是不是很精准？事实上压根就没这个必要，而且在手机上运行的速度可能会慢到无法忍受。更有效率的做法是向地面发射4条射线，计算射线与地面的交点。后一种方法每帧只需要进行一次运算，却能达到与前一种方法差不多的效果，而玩家很难区分这两种方法的差异。这正是数学方法的魅力所在。

对大量物体进行碰撞检测是个十分耗时的工作，因此，优化工作从最初就开始了。常用的优化方法有两种：其一是使用简单形状的**碰撞体**来代替复杂形状的物体，例如包围体、射线、平面等；其二是利用**数据结构**来减少实际参与检测的物体数量，例如九宫格、四叉树（2D）、八叉树（3D）等等。

以 Minecraft 这个游戏为例，无论玩家如何更换角色的皮肤、模型，史蒂夫的碰撞体积始终是 2 格高，小黑是 3 格，蠹虫是 1 格。无论世界有多大，永远只有玩家附近的区块（Chunk）是激活的，超过一定距离后连水都不会流动了。

### 包围体

在2D游戏中，通常使用矩形、圆形、射线/线段来代替复杂图形的相交检测。如果要求提高精度，则会使用三角形等多边形来进行碰撞检测；如果精度还是不够，最细粒度将会使用像素进行检测。3D游戏只是比2D游戏多了一个维度，一般使用方块、球体、射线/线段来进行相交检测。若嫌精度不够，则会使用三角形或网格来进行检测。

这个方法的基本思路，是使用简化的包围形状（2D, Bounding Shape）或包围体（3D, Bounding Volume）来代替本体进行碰撞检测。最常用的两种形状为**包围盒**（Bounding Box）与**包围球**（Bounding Sphere），这两种形状的碰撞检测速度是最快的。圆形或球体的相交很容易判断，只要比较圆心距离与两个圆的半径即可。矩形或方块的相交也不太复杂，通过检查各个边是否相交即可。球体和矩形之间的相交，通过圆心到边或面的距离就可以快速判断。

![Bounding Volume](content/images/2017/06/bounding-volume.png)

包围盒又可以分为**轴对齐包围盒**（AABB, Axis Aligned Bounding Box）与**转向包围盒**（OBB, Oriented Bounding Box）。AABB 与 OBB 的差别在于物体旋转之后，AABB的大小可能发生改变，但是每条边依然与坐标系保持平行；OBB的大小不会变，且将随物体一起旋转。OBB 看起来比 AABB 更接近物体本身的形状，但是碰撞检测的算法要比 AABB 复杂许多。很多游戏会尽量避免使用 OBB，或者使用 AABB 或包围球代替它。

![AABB vs OBB](content/images/2017/06/aabb-vs-obb.png)

除了包围盒和包围球以外，根据实际情况，游戏中也会使用其他形状的包围体。比较常见的当属圆柱体（Cylinder）和胶囊体（Capsule），主要用于模拟玩家、怪物等模型的碰撞检测。圆柱体只在3D游戏中存在，而胶囊体在2D和3D游戏中都有所应用。2D的胶囊体是由一个矩形外加倒扣在两端的半圆组成，3D的胶囊体则由一个圆柱外加倒扣在两端的半球组成。

![胶囊体](content/images/2017/06/capsule.png)

### 射线

射线检测（Ray cast）是另一种常用的检测方法。选择一个位置（position）作为原点，朝某个方向（direction）发射出一根射线，计算射线途径路线上是否和物体的边（2D）或表面（3D）相交。如果相交，交点坐标是多少？距离原点有多远？

![Ray cast](content/images/2017/06/ray-cast.png)

在2D游戏中，射线检测常用于计算视线（LOS, Line of sight）与视野范围（FOV, Field of view），也有用射线来模拟光照的。这部分内容我推荐读者阅读 INDIENOVA 上的两篇文章。

[基于 Tile 地图的视觉范围生成方法以及实现](https://indienova.com/indie-game-development/tile-based-line-of-sight-explained/)

[![视觉范围生成](content/images/2017/06/line-of-sight.png)](https://indienova.com/includes/los/final.html)

[视线和光线：如何创建 2D 视觉范围效果](https://indienova.com/indie-game-development/sight-light-how-to-create-2d-visibility-shadow-effects-for-your-game/)

*请在下面的区域内移动鼠标，如果没有反应请点击一下该区域再尝试*

<div><iframe src="https://indienova.com/includes/2dlight/draft7.html" height="288" width="672"></iframe></div>

3D 游戏中的典型应用是**鼠标拾取**（mouse picking）。玩家经常需要通过鼠标来拣选 3D 场景中的物体，无论是捡起道具，还是选择NPC对话，都离不开射线检测。由于在3D游戏中，玩家所看到的画面，实质上是三维**物体**在**视锥近平面**上的二维**投影**。因此，可以把**摄像机位置**作为原点，朝投影平面上的**鼠标**发射一根“视线”。“视线”与哪个物体相交，就说明鼠标选择的是哪个物体。

![视锥投影](content/images/2017/06/world-view-projection.png)

在 CS:GO 这种第一人称射击（FPS, First Person Shoot）游戏中，玩家射击的方向是通过**摄像机位置**与**准星**的连线来确定的。不过由于“准星”一般都处于画面的正中央，因此可以直接使用**摄像机的正面朝向**作为射线的方向，这就不需要再计算射线方向了。

![准星](content/images/2017/05/Picture.png)

### 空间分割技术

利用简单形状来进行碰撞检测的例子还有很多，难以一一列举。本文的目的并不是讨论如何进行碰撞检测，而是阐述为什么要这么做，以及在jMonkeyEngine中如何做。

虽然包围体很好用，但在游戏开发中我们总会遇到不能使用包围体的情况。而且就算使用包围体，在面对大量物体时也难以保证效率。比如一款3D网络游戏，世界地图中可能有高山、丘陵、平原、湖泊等复杂地形，每个区域中可能同时有大量玩家活动。

做个简单的数学题：假设平均每张地图的模型面数为1万面；每个玩家使用胶囊体（capsule）作为包围体，每个地图平均200 人在线。玩家要和地图进行碰撞检测，避免掉到地图外面。玩家之间也要进行碰撞检测，避免出现“穿人”的现象。游戏的刷新速度为30帧/秒，那么一张地图每秒需要进行多少次碰撞检测？

在不做任何优化的条件下，玩家和一个1万面的地图进行碰撞检测，意味着要和1万个三角形进行相交检测，200个玩家就是**200万**次相交检测。而200个玩家互相进行碰撞检测，计算的最少次数为 `1+2+3+4+...+199 = 19900`。每秒30帧，那么这个运算量还得乘以30。**注意，我们还只讨论了碰撞检测的次数，每次碰撞检测本身又需要多少次计算呢？**如果我们想做大世界无缝地图，每个地图的负载量想要做到500、1000人呢？

这就是碰撞检测需要进行优化的原因。

事实上，玩家根本就不需要和整个地图进行碰撞检测，因为每个玩家都只处于地图上的一小块区域，最好是之和附近的地面进行碰撞检测。玩家之间也不需要全部互相检测，有的玩家在地图西边，有的在东边，压根就碰不着面。

所以，游戏中会使用不同的数据结构和算法把场景“肢解”掉，划分成更小的区域。这样碰撞检测就可以在更小的区域内完成，无人活动的区域甚至也可以直接跳过。常见的数据结构如：

1. BVH（bounding volume hierarchies，包围体层次结构）
2. BSP（binary space partitioning，二叉空间分割）树
3. Quad Tree （四叉树，多用于2D游戏）
4. Oct Tree（八叉树，多用于3D游戏）
5. 九宫格（多用于2D游戏，把场景划分成坐标格，每个玩家之和周围8格进行碰撞检测）

除了这些算法以外，更简单的做法是把场景中的物体分组，即不要总是对整个场景中的全部物体进行碰撞检测。比如落在地上的道具，可以把它们单独分为一组。玩家运动时不与道具进行碰撞检测，只有鼠标拾取时才用射线与道具组中的物体进行检测。

具体的数据结构和算法不是本文要讨论的内容，如果你对这些感兴趣，请自行查阅更多其他资料。我推荐下面这本书：

    3D数学基础：图形与游戏开发
    作者：(美)Fletcher Dunn (美)Ian Parberry
    翻译：史银雪　陈洪　王荣静
    出版社：清华大学出版社
    ISBN：978-7-302-10946-4

### 检测的方式

前面我们主要从数据结构的角度对不同的碰撞检测方法进行了介绍。根据检测方式的不同，又可以把碰撞检测分为两类：

1. 离散点的碰撞检测
离散点的碰撞检测是指定某一时刻T的两个静态碰撞体，看它们之间是否相交。如果没有相交则返回它们最近点的距离；如果相交则返回交迭深度、交迭方向等。
2. 连续碰撞检测
连续碰撞检测则是分别指定在T1、T2两个时刻两个碰撞体的位置，看它们在由T1运动到T2时刻的过程中是否发生碰撞，如果碰撞则返回第一碰撞点的位置和法线。

连续碰撞检测（CCD continuous collisiondetection）是最为自然的碰撞检测，可以大大方便碰撞响应逻辑的编写，可以很容易避免物体发生相交或者穿越。离散点的碰撞检测则没有那么友好，当检测到碰撞时两个物体已经相交了，可能两个物体的网格中已经有许多三角形发生了交迭，如何将两个交迭的对象分开并按合理的方式运动是一个挑战。

虽然连续碰撞检测是最自然的方式，但它的实现非常复杂，运算开销也很大，所以目前大部分成熟的物理引擎和碰撞检测引擎还是采用了基于离散点的碰撞检测，为了避免物体交迭过深或者彼此穿越，它们都要采用比较小的模拟步长。

## Collidable

前面我们已经讨论了很多关于碰撞检测的概念，下面来看看 jMonkeyEngine 中是如何进行碰撞检测的。

在jME3中，所有能够进行碰撞检测的物体都实现了 `com.jme3.collision.Collidable` 接口。这个接口中只声明了一个方法，用于计算两个碰撞器之间有多少次碰撞：`collideWith(Collidable other, CollisionResults results)`。

* jME3中的碰撞检测是纯数学检测，所谓有多少次碰撞，其实是指有多少个交点。
* `com.jme3.collision.CollisionResult` 对象保存了一次碰撞的结果。
* `com.jme3.collision.CollisionResults` 对象使用ArrayList保存了多次碰撞检测的结果。

注意，jME3记录了所有的碰撞结果，这意味着射线与包围盒会有两个交点，前一个是射线进入包围盒的位置，后一个是射线离开包围盒的位置。

下面是CollisionResults 中的方法说明：

<table>
  <tr><th>方法</th><th>说明</th></tr>
  <tr>
    <td>size()</td>
    <td>返回碰撞结果的数量。</td>
  </tr>
  <tr>
    <td>getClosestCollision()</td>
    <td>返回距离最近的碰撞结果。</td>
  </tr>
  <tr>
    <td>getFarthestCollision()</td>
    <td>返回距离最远的碰撞结果。</td>
  </tr>
  <tr>
    <td>getCollision(i)</td>
    <td>返回下标为 i 的的碰撞结果。</td>
  </tr>
</table>

碰撞结果（CollisionResult）对象中保存了诸多信息，通过下列方法可以获取：

<table>
  <tr><th>方法</th><th>说明</th></tr>
  <tr>
    <td>getContactPoint()</td>
    <td>返回交点坐标，结果为 Vector3f 类型。</td>
  </tr>
  <tr>
    <td>getContactNormal()</td>
    <td>返回交点处的法线向量，结果为 Vector3f 类型。</td>
  </tr>
  <tr>
    <td>getDistance()</td>
    <td>返回两个碰撞体之间的距离，结果为 float 类型。</td>
  </tr>
  <tr>
    <td>getGeometry()</td>
    <td>返回实际产生碰撞的 Geometry 对象。</td>
  </tr>
  <tr>
    <td>getTriangle(t)</td>
    <td>判断被撞物体的网格中，具体是哪个三角形被击中。使用一个Triangle对象来记录发生碰撞的三角形。</td>
  </tr>
  <tr>
    <td>getTriangleIndex()</td>
    <td>判断被撞物体的网格中，具体是哪个三角形被击中。返回此三角形在网格中的索引号。</td>
  </tr>
</table>

### Collidable的实现类

jME3中有很多Collidable接口的实现类，主要分布于数学（com.jme3.math）、包围体（com.jme3.bounding）、场景（com.jme3.scene）这三包中。

Collidable的主要实现类如下：

    AbstractTriangle //com.jme3.math
        Triangle //com.jme3.math
    Ray //com.jme3.math
    BoundingVolume // com.jme3.bounding
        BoundingBox // com.jme3.bounding
        BoundingSphere // com.jme3.bounding
    Spatial // com.jme3.scene
        Geometry // com.jme3.scene
        Node // com.jme3.scene
 
 值得一提的是，Mesh内部定义了 collideWith 方法，但并没有实现Collidable接口。这个方法是专门提供给 Geometry 内部调用的。
 
### Collidable的使用

假设我们有两个碰撞体（Collidable）a 和 b 要进行碰撞检测，参加碰撞检测的双方可是以 Geometry、Node、Mesh、包围体、平面、直线或射线。需要特别注意是，你只能用Geometry与包围体或摄像进行碰撞检测。这意味着本例中的 a 可以是 Geometry、Node等任意物体，但 b 只能是某种包围体或射线。

下面的代码演示了jME3中如何进行碰撞检测。

    /**
     * 打印碰撞检测结果
     * 
     * @param results
     */
    private void printCollisionResults(Collidable a, Collidable b) {
    
        CollisionResults results = new CollisionResults();
        a.collideWith(b, results);// 碰撞检测
        
        System.out.println("碰撞结果数量：" + results.size());

        /**
         * 判断检测结果
         */
        if (results.size() > 0) {

            // 从近到远，打印出所有的结果。
            for (int i = 0; i < results.size(); i++) {
                CollisionResult result = results.getCollision(i);

                float dist = result.getDistance();
                Vector3f point = result.getContactPoint();
                Vector3f normal = result.getContactNormal();
                Geometry geom = result.getGeometry();

                System.out.printf("序号：%d, 距离：%.2f, 物体名称：%s, 交点：%s, 交点法线：%s\n", i, dist, geom.getName(), point, normal);
            }

            // 离b最近的交点
            Vector3f closest = results.getClosestCollision().getContactPoint();
            // 离b最远的交点
            Vector3f farthest = results.getFarthestCollision().getContactPoint();

            System.out.printf("最近点：%s, 最远点：%s\n", closest, farthest);
        }
    }


## jME3中的射线检测

在jME3中，`com.jme3.math.Ray`表示射线。它有3个参数：原点坐标(origin)、射线方向(direction)、线段长度(limit)。使用无参构造方法实例化射线时，原点坐标默认在Vector3f(0, 0, 0,)处，射线方向为Vector3f(0, 0, 1)，线段长度为正无穷。

你可以通过下列构造方法来初始化射线的原点坐标和射线方向：

    Ray ray = new Ray(Vector3f origin, Vector3f direction);

通过下列方法来改变射线的参数：

    ray.setOrigin(new Vector3f(32.1f, 17.82f, -23.4f));// 改变原点坐标
    ray.setDirection(new Vector3f(0.4f, 0.3f, 0f).normalize());// 改变方向
    ray.setLimit(100f);// 改变长度

当你把射线长度改为定长时，它就变成了线段。

### 拾取

站在开发者的视角，诸如开枪、拾取物品、选择目标、拖拽物体等操作，都可以简化为一种相似的模式：用户首先瞄准或选中3D场景中的某个目标，然后触发对该目标的操作。这种模式称为拾取，拾取是射线最典型的应用。通过点击鼠标来选择目标称为鼠标拾取，鼠标拾取可以通过固定准星，也可以通过指针来实现。

我们在前文中已经介绍过射线的这种用法。对于固定准星的第一人称视角而言，可以使用**摄像机位置**作为原点，并把**摄像机正方向**作为射线方向，生成一根射线。

    // 生成射线
    Ray ray = new Ray(cam.getLocation(), cam.getDirection);

使用鼠标指针来拾取时，需要先计算从**摄像机位置**到**鼠标位置**的方向向量。用户在屏幕上看到的鼠标，其坐标是在(x, y)平面上的二维坐标，必须通过**视锥投影**把它恢复到世界坐标系中，获得三维坐标后才能计算射线方向。

    // 先计算鼠标在世界坐标系中的位置。
    Vector2f screenCoord = inputManager.getCursorPosition();
    Vector3f worldCoord = cam.getWorldCoordinates(screenCoord, 1f);

    // 然后计算视线方向
    Vector3f dir = worldCoord.subtract(cam.getLocation());
    dir.normalizeLocal();

    // 生成射线
    Ray ray = new Ray(cam.getLocation(), dir);

在上面的代码中，`cam.getWorldCoordinates(screenCoord, 1f);` 方法用于把鼠标的二维屏幕坐标转换成三维世界坐标。也许你对这个方法的第二个参数 `1f` 有些疑问，不明白这个数字是怎么来的，下面我来解释一下。

3D场景中的摄像机使用**视锥**来决定用户在屏幕上看到什么内容。视锥最左侧的顶点是摄像机的位置，也代表人眼的位置。凡处于视锥**近平面**到**远平面**区间内的所有3D物体，都会通过**小孔成像原理**投影到**近平面**上，成为2维的画面，即是我们在屏幕上看到的内容。

![视锥投影](content/images/2017/06/world-view-projection-2.png)

视锥**近平面**到屏幕的距离为0，**远平面**的距离为1，`getWorldCoordinates(screenCoord, 1f);` 表示计算鼠标在最远平面上的三维坐标。此方法的第二个参数可以取0~1之间的任何数值，它表示把鼠标映射到视锥中的某个具体位置。无论取值为多少，从摄像机角度看过去，方向都是一致的。

你可以通过下面的方法来计算视线方向，结果也没有什么不同。

    // 先计算鼠标在世界坐标系中的位置。
    Vector2f screenCoord = inputManager.getCursorPosition();
    Vector3f near = cam.getWorldCoordinates(screenCoord, 0f);// 鼠标在近平面上的坐标。
    Vector3f far = cam.getWorldCoordinates(screenCoord, 0.3f);// 离近平面稍远的某个坐标，依然在视锥内。
    
    // 然后计算视线方向
    Vector3f dir = far.subtract(near);
    dir.normalizeLocal();
    
    // 生成射线
    Ray ray = new Ray(near, dir);
 
生成射线后，就可以用它来和场景中的物体进行碰撞检测。例如：

    CollisionResults results = new CollisionResults();
    rootNode.collideWith(ray, results);
    if (results.size() > 0) {
        // 获得离射线原点最近的交点
        Vector3f closest = results.getClosestCollision().getContactPoint();
    }
 
 然而，这并非是最好的做法。直接和rootNode中的所有物体进行碰撞检测是没有必要的，我们可以把场景中的物体分组，然后专门检测需要拾取的物体。使用一个单独的Node，很容易就可以做到这一点。
 
     Node pickable = new Node("pickable");// 创建可拾取物品的节点
     rootNode.attachChild(pickable);
 
把地上的道具统统添加到 pickable 节点中，然后在拾取时只用 pickable 和射线进行检测。

    pickable.attachChild(item1);
    pickable.attachChild(item2);
    ..
    pickable.collideWith(ray, results);// 检测可拾取的物体

这都是我们在前面章节中介绍过的技巧，关键在于学会灵活运用。

### 实例：拾取

下面这个例子演示了游戏中常见的两种拾取模式。其一是第一人称视角，你通过准星来瞄准场景中的物体；另一种是第三人称视角（上帝视角），通过鼠标点击来选择场景中的物体。使用空格键来切换这两种模式。

	package net.jmecn.collision;
	
	import com.jme3.app.DebugKeysAppState;
	import com.jme3.app.FlyCamAppState;
	import com.jme3.app.SimpleApplication;
	import com.jme3.collision.CollisionResult;
	import com.jme3.collision.CollisionResults;
	import com.jme3.font.BitmapText;
	import com.jme3.input.KeyInput;
	import com.jme3.input.MouseInput;
	import com.jme3.input.controls.ActionListener;
	import com.jme3.input.controls.KeyTrigger;
	import com.jme3.input.controls.MouseButtonTrigger;
	import com.jme3.material.Material;
	import com.jme3.material.TechniqueDef.LightMode;
	import com.jme3.math.ColorRGBA;
	import com.jme3.math.Quaternion;
	import com.jme3.math.Ray;
	import com.jme3.math.Vector2f;
	import com.jme3.math.Vector3f;
	import com.jme3.scene.Geometry;
	import com.jme3.scene.Node;
	import com.jme3.scene.Spatial;
	import com.jme3.scene.shape.Sphere;
	
	import net.jmecn.state.CubeAppState;
	
	/**
	 * 利用射线检测，实现拾取。
	 * 
	 * @author yanmaoyuan
	 *
	 */
	public class HelloPicking extends SimpleApplication implements ActionListener {
	
	    // 空格键：切换摄像机模式
	    public final static String CHANGE_CAM_MODE = "change_camera_mode";
	    // 鼠标左键：拾取
	    public final static String PICKING = "pick";
	
	    // 准星
	    private Spatial cross;
	    // 拾取标记
	    private Spatial flag;
	
	    // 射线
	    private Ray ray;
	
	    public static void main(String[] args) {
	        // 启动程序
	        HelloPicking app = new HelloPicking();
	        app.start();
	    }
	
	    public HelloPicking() {
	        super(new FlyCamAppState(), new DebugKeysAppState(), new CubeAppState());
	
	        // 初始化射线
	        ray = new Ray();
	        // 设置检测最远距离，可将射线变为线段。
	        // ray.setLimit(500);
	    }
	
	    @Override
	    public void simpleInitApp() {
	        // 初始化摄像机
	        flyCam.setMoveSpeed(20f);
	        cam.setLocation(new Vector3f(89.0993f, 10.044929f, -86.18647f));
	        cam.setRotation(new Quaternion(0.063343525f, 0.18075047f, -0.01166729f, 0.9814177f));
	
	        // 设置灯光渲染模式为单通道，这样更加明亮。
	        renderManager.setPreferredLightMode(LightMode.SinglePass);
	        renderManager.setSinglePassLightBatchSize(2);
	
	        // 做个准星
	        cross = makeCross();
	
	        // 做个拾取标记
	        flag = makeFlag();
	
	        // 用户输入
	        inputManager.addMapping(PICKING, new MouseButtonTrigger(MouseInput.BUTTON_LEFT));
	        inputManager.addMapping(CHANGE_CAM_MODE, new KeyTrigger(KeyInput.KEY_SPACE));
	        inputManager.addListener(this, PICKING, CHANGE_CAM_MODE);
	    }
	
	    /**
	     * 在摄像机镜头正中央贴一张纸，充当准星。
	     */
	    private Spatial makeCross() {
	        // 采用Gui的默认字体，做个加号当准星。
	        BitmapText text = guiFont.createLabel("+");
	        text.setColor(ColorRGBA.Red);// 红色
	
	        // 居中
	        float x = (cam.getWidth() - text.getLineWidth()) * 0.5f;
	        float y = (cam.getHeight() + text.getLineHeight()) * 0.5f;
	        text.setLocalTranslation(x, y, 0);
	
	        guiNode.attachChild(text);
	
	        return text;
	    }
	
	    /**
	     * 制作一个小球，用于标记拾取的地点。
	     * 
	     * @return
	     */
	    private Spatial makeFlag() {
	        Material mat = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");
	        mat.setColor("Color", ColorRGBA.Green);
	        mat.getAdditionalRenderState().setWireframe(true);
	
	        Geometry geom = new Geometry("flag", new Sphere(8, 8, 1));
	        geom.setMaterial(mat);
	
	        return geom;
	    }
	
	    @Override
	    public void onAction(String name, boolean isPressed, float tpf) {
	        if (isPressed) {
	            if (PICKING.equals(name)) {
	                // 拾取
	                pick();
	            } else if (CHANGE_CAM_MODE.equals(name)) {
	
	                if (flyCam.isDragToRotate()) {
	                    // 自由模式
	                    flyCam.setDragToRotate(false);
	                    guiNode.attachChild(cross);
	                } else {
	                    // 拖拽模式
	                    flyCam.setDragToRotate(true);
	                    cross.removeFromParent();
	                }
	            }
	        }
	
	    }
	
	    /**
	     * 使用射线检测，判断离摄像机最近的点。
	     */
	    private void pick() {
	
	        Ray ray = updateRay();
	        CollisionResults results = new CollisionResults();
	
	        // rootNode.collideWith(ray, results);// 碰撞检测
	
	        Node cubeSceneNode = stateManager.getState(CubeAppState.class).getRootNode();
	        cubeSceneNode.collideWith(ray, results);// 碰撞检测
	
	        // 打印检测结果
	        print(results);
	
	        /**
	         * 判断检测结果
	         */
	        if (results.size() > 0) {
	
	            // 放置拾取标记
	            Vector3f position = results.getClosestCollision().getContactPoint();
	            flag.setLocalTranslation(position);
	
	            if (flag.getParent() == null) {
	                rootNode.attachChild(flag);
	            }
	        } else {
	            // 移除标记
	            if (flag.getParent() != null) {
	                flag.removeFromParent();
	            }
	        }
	    }
	
	    /**
	     * 更新射线参数
	     * 
	     * @return
	     */
	    private Ray updateRay() {
	        // 使用摄像机的位置作为射线的原点
	        ray.setOrigin(cam.getLocation());
	
	        if (!flyCam.isDragToRotate()) {
	            /**
	             * 自由模式下，直接使用摄像机方向即可。
	             */
	            ray.setDirection(cam.getDirection());
	        } else {
	            /**
	             * 拖拽模式下，通过鼠标的位置计算射线方向
	             */
	            Vector2f screenCoord = inputManager.getCursorPosition();
	            Vector3f worldCoord = cam.getWorldCoordinates(screenCoord, 1f);
	
	            // 计算方向
	            Vector3f dir = worldCoord.subtract(cam.getLocation());
	            dir.normalizeLocal();
	
	            ray.setDirection(dir);
	        }
	
	        return ray;
	    }
	
	    /**
	     * 打印检测结果
	     * 
	     * @param results
	     */
	    private void print(CollisionResults results) {
	        System.out.println("碰撞结果：" + results.size());
	        System.out.println("射线：" + ray);
	
	        /**
	         * 判断检测结果
	         */
	        if (results.size() > 0) {
	
	            // 从近到远，打印出射线途径的所有交点。
	            for (int i = 0; i < results.size(); i++) {
	                CollisionResult result = results.getCollision(i);
	
	                float dist = result.getDistance();
	                Vector3f point = result.getContactPoint();
	                Vector3f normal = result.getContactNormal();
	                Geometry geom = result.getGeometry();
	
	                System.out.printf("序号：%d, 距离：%.2f, 物体名称：%s, 交点：%s, 交点法线：%s\n", i, dist, geom.getName(), point, normal);
	            }
	
	            // 离射线原点最近的交点
	            Vector3f closest = results.getClosestCollision().getContactPoint();
	            // 离射线原点最远的交点
	            Vector3f farthest = results.getFarthestCollision().getContactPoint();
	
	            System.out.printf("最近点：%s, 最远点：%s\n", closest, farthest);
	        }
	    }
	}

使用鼠标左键拾取场景中的某个点后，该位置就会出现一个绿色的小球，代表拾取的定点坐标。

## jME3中的包围体

在jME3中，包围体被定义为一个抽象类BoundingVolume，有两个实现类：

* 轴对齐包围盒 BoundingBox
* 包围球 BoundingSphere

jME3中并没有 OrientedBoundingBox(OBB) 的实现，因为 OBB 在性能方面不够优秀，而且需要使用 OBB 的地方绝大多数时候都能用AABB或包围球代替。

### 设置包围体

BoundingVolume 是和 Mesh 绑定的，因为包围体需要根据 Mesh 中的顶点数据来计算自身的形状。在默认情况下，所有Mesh都将使用 AABB 作为自己的包围体，因为 AABB 的效率最高。

下面是 Mesh 类中的源码：

    /**
     * The bounding volume that contains the mesh entirely.
     * By default a BoundingBox (AABB).
     */
    private BoundingVolume meshBound =  new BoundingBox();

如果想要更改网格的包围体，需要调用 Mesh 的 `setBound(BoundingVolume bv)` 方法来设置包围体，并调用 `updateBound()` 方法来更新包围体的形态。

例如，把网格的包围体设为 BoundingSphere ，需要这么做：

    // 设置网格包围球
    Mesh mesh = new Box(1, 1, 1);
    mesh.setBound(new BoundingSphere());
    mesh.updateBound();	

### 获得包围体

通过 getBound()方法可以获得 Mesh 的包围体。

    BoundingVolume bv = mesh.getBound();

然而，在大多数情况下，这么做并不能满足我们的需要。比如玩家的模型可能是由很多个Geometry组成的，我们需要整个玩家的包围体，此时应该使用 Spatial 中的 getWorldBound() 方法。

    // 加载模型
    Spatial player = assetManager.loadModel("...");
    BoundingVolumne bv = player.getWorldBound();

如果这个 Spatial 对象是 Geometry 类型，那么问题就简单了。因为每个
 Geometry 内部只有一个Mesh，所以它会直接调用 `Mesh#getBound()` 来获得包围体。同理，你还可以通过调用 `Geometry#setModelBound()` 和 `Geometry#updateModelBound()` 方法来间接调用 `Mesh#setBound()` 和 `Mesh#updateBound()`。

如果是 Node 类型的话，它会根据自身内部所有子节点的包围体，计算出一个更大的包围体！

### 实例：轴对齐包围盒

下面的例子演示了jME3中的AABB，并使用WireBox将AABB显示了出来。在程序运行的过程中，能够看到AABB会随着物体姿态的变化而改变大小和位置，但每条边始终与坐标轴平行。

	package net.jmecn.collision;
	
	import com.jme3.app.SimpleApplication;
	import com.jme3.bounding.BoundingBox;
	import com.jme3.light.AmbientLight;
	import com.jme3.light.DirectionalLight;
	import com.jme3.material.Material;
	import com.jme3.math.ColorRGBA;
	import com.jme3.math.Quaternion;
	import com.jme3.math.Vector3f;
	import com.jme3.scene.Geometry;
	import com.jme3.scene.debug.WireBox;
	import com.jme3.scene.shape.Cylinder;
	
	import net.jmecn.logic.FloatControl;
	import net.jmecn.logic.RotateControl;
	import net.jmecn.state.AxisAppState;
	
	/**
	 * 演示轴对齐包围盒（Axis Align Bounding Box）
	 * 
	 * @author yanmaoyuan
	 *
	 */
	public class HelloAABB extends SimpleApplication {
	
	    private Geometry debug;
	    private Geometry cylinder;
	
	    @Override
	    public void simpleInitApp() {
	        // 初始化摄像机
	        cam.setLocation(new Vector3f(4.5114727f, 6.176994f, 13.277485f));
	        cam.setRotation(new Quaternion(-0.038325474f, 0.96150225f, -0.20146479f, -0.18291113f));
	        flyCam.setMoveSpeed(10);
	
	        viewPort.setBackgroundColor(ColorRGBA.LightGray);
	
	        // 参考坐标系
	        stateManager.attach(new AxisAppState());
	
	        // 圆柱体
	        Material mat = new Material(assetManager, "Common/MatDefs/Light/Lighting.j3md");
	        mat.setColor("Diffuse", ColorRGBA.Yellow);
	        mat.setColor("Ambient", ColorRGBA.Yellow);
	        mat.setColor("Specular", ColorRGBA.White);
	        mat.setFloat("Shininess", 24);
	        mat.setBoolean("UseMaterialColors", true);
	
	        cylinder = new Geometry("cylinder", new Cylinder(2, 36, 1f, 8, true));
	        cylinder.setMaterial(mat);
	        // 让圆柱体运动，这样才能看到包围盒的变化。
	        cylinder.addControl(new RotateControl());
	        cylinder.addControl(new FloatControl(2, 2));
	
	        // 用于显示包围盒
	        mat = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");
	        mat.setColor("Color", ColorRGBA.Magenta);
	        mat.getAdditionalRenderState().setLineWidth(2);
	        debug = new Geometry("debug", new WireBox(1, 1, 1));
	        debug.setMaterial(mat);
	
	        rootNode.attachChild(cylinder);
	        rootNode.attachChild(debug);
	
	        // 光源
	        rootNode.addLight(new DirectionalLight(new Vector3f(-1, -2, -3)));
	        rootNode.addLight(new AmbientLight(new ColorRGBA(0.2f, 0.2f, 0.2f, 1f)));
	    }
	
	    @Override
	    public void simpleUpdate(float tpf) {
	        // 根据圆柱体当前的包围盒，更新线框的位置和大小。
	        BoundingBox bbox = (BoundingBox) cylinder.getWorldBound();
	        debug.setLocalScale(bbox.getExtent(null));
	        debug.setLocalTranslation(bbox.getCenter());
	    }
	
	    public static void main(String[] args) {
	        HelloAABB app = new HelloAABB();
	        app.start();
	    }
	
	}

### 实例：包围球

下面的例子演示了如何使用BoundingSphere来替换物体原本的包围盒，并使用WireSphere来显示BoundingSpehre的形态。

	package net.jmecn.collision;
	
	import com.jme3.app.SimpleApplication;
	import com.jme3.bounding.BoundingSphere;
	import com.jme3.light.AmbientLight;
	import com.jme3.light.DirectionalLight;
	import com.jme3.material.Material;
	import com.jme3.math.ColorRGBA;
	import com.jme3.math.Quaternion;
	import com.jme3.math.Vector3f;
	import com.jme3.scene.Geometry;
	import com.jme3.scene.Mesh;
	import com.jme3.scene.debug.WireSphere;
	import com.jme3.scene.shape.Cylinder;
	
	import net.jmecn.logic.FloatControl;
	import net.jmecn.logic.RotateControl;
	import net.jmecn.state.AxisAppState;
	
	/**
	 * 演示包围球（Bounding Sphere）
	 * 
	 * @author yanmaoyuan
	 *
	 */
	public class HelloBoundingSphere extends SimpleApplication {
	
	    private Geometry debug;
	    private Geometry cylinder;
	
	    @Override
	    public void simpleInitApp() {
	        // 初始化摄像机
	        cam.setLocation(new Vector3f(4.5114727f, 6.176994f, 13.277485f));
	        cam.setRotation(new Quaternion(-0.038325474f, 0.96150225f, -0.20146479f, -0.18291113f));
	        flyCam.setMoveSpeed(10);
	
	        viewPort.setBackgroundColor(ColorRGBA.LightGray);
	
	        // 参考坐标系
	        stateManager.attach(new AxisAppState());
	
	        // 圆柱体
	        Material mat = new Material(assetManager, "Common/MatDefs/Light/Lighting.j3md");
	        mat.setColor("Diffuse", ColorRGBA.Yellow);
	        mat.setColor("Ambient", ColorRGBA.Yellow);
	        mat.setColor("Specular", ColorRGBA.White);
	        mat.setFloat("Shininess", 24);
	        mat.setBoolean("UseMaterialColors", true);
	
	        // 设置网格包围球
	        Mesh mesh = new Cylinder(2, 36, 1f, 8, true);
	        mesh.setBound(new BoundingSphere());
	        mesh.updateBound();
	
	        cylinder = new Geometry("cylinder", mesh);
	        cylinder.setMaterial(mat);
	        // 让圆柱体运动，这样才能看到包围盒的变化。
	        cylinder.addControl(new RotateControl());
	        cylinder.addControl(new FloatControl(2, 2));
	
	        // 用于显示包围球
	        mat = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");
	        mat.setColor("Color", ColorRGBA.Magenta);
	        mat.getAdditionalRenderState().setLineWidth(2);
	        mat.getAdditionalRenderState().setWireframe(true);
	        debug = new Geometry("debug", new WireSphere(1));
	        debug.setMaterial(mat);
	
	        rootNode.attachChild(cylinder);
	        rootNode.attachChild(debug);
	
	        // 光源
	        rootNode.addLight(new DirectionalLight(new Vector3f(-1, -2, -3)));
	        rootNode.addLight(new AmbientLight(new ColorRGBA(0.2f, 0.2f, 0.2f, 1f)));
	    }
	
	    @Override
	    public void simpleUpdate(float tpf) {
	        // 根据圆柱体当前的包围球，更新线框的位置和大小。
	        BoundingSphere bs = (BoundingSphere) cylinder.getWorldBound();
	        debug.setLocalScale(bs.getRadius());
	        debug.setLocalTranslation(bs.getCenter());
	    }
	
	    public static void main(String[] args) {
	        HelloBoundingSphere app = new HelloBoundingSphere();
	        app.start();
	    }
	
	}

### 实例：基于包围体的碰撞检测

下面我们分别使用方块和球体来进行碰撞检测。方块使用BoundingBox作为包围体，固定于坐标系中间不动；球体使用BoundingSphere作为包围体，你可以通过鼠标来控制球体的位置。当球体与方块相交时，球体的颜色会发生改变。

	package net.jmecn.collision;
	
	import com.jme3.app.SimpleApplication;
	import com.jme3.bounding.BoundingSphere;
	import com.jme3.collision.CollisionResults;
	import com.jme3.input.RawInputListener;
	import com.jme3.input.event.JoyAxisEvent;
	import com.jme3.input.event.JoyButtonEvent;
	import com.jme3.input.event.KeyInputEvent;
	import com.jme3.input.event.MouseButtonEvent;
	import com.jme3.input.event.MouseMotionEvent;
	import com.jme3.input.event.TouchEvent;
	import com.jme3.material.Material;
	import com.jme3.math.ColorRGBA;
	import com.jme3.math.Vector3f;
	import com.jme3.scene.Geometry;
	import com.jme3.scene.shape.Box;
	import com.jme3.scene.shape.Sphere;
	
	import net.jmecn.state.AxisAppState;
	
	/**
	 * 基于包围体的碰撞检测。
	 * 
	 * @author yanmaoyuan
	 *
	 */
	public class TestCollisionWith extends SimpleApplication implements RawInputListener {
	
	    private Geometry green;
	    private Geometry pink;
	
	    public static void main(String[] args) {
	        TestCollisionWith app = new TestCollisionWith();
	        app.start();
	    }
	
	    public TestCollisionWith() {
	        super(new AxisAppState());
	    }
	
	    @Override
	    public void simpleInitApp() {
	        cam.setLocation(new Vector3f(0f, 18f, 22f));
	        cam.lookAt(Vector3f.ZERO, Vector3f.UNIT_Y);
	
	        viewPort.setBackgroundColor(ColorRGBA.White);
	
	        // 绿色物体
	        Material mat = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");
	        mat.setColor("Color", ColorRGBA.Green);
	
	        green = new Geometry("Green", new Sphere(6, 12, 1));
	        green.setMaterial(mat);
	        green.setModelBound(new BoundingSphere());// 使用包围球
	        green.updateModelBound();// 更新包围球
	
	        // 粉色物体
	        mat = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");
	        mat.setColor("Color", ColorRGBA.Pink);
	        pink = new Geometry("Pink", new Box(1, 1, 1));
	        pink.setMaterial(mat);
	
	        rootNode.attachChild(green);
	        rootNode.attachChild(pink);
	
	        inputManager.addRawInputListener(this);
	    }
	
	    @Override
	    public void onMouseMotionEvent(MouseMotionEvent evt) {
	        // 根据鼠标的位置来改变绿色方块的坐标，并将其限制在 (-10, -10) 到 (10, 10)的范围内。
	        float x = evt.getX();
	        float y = evt.getY();
	
	        // 坐标系大小为 20 * 20
	        x = x * 20 / cam.getWidth() - 10;
	        y = y * 20 / cam.getHeight() - 10;
	
	        green.setLocalTranslation(x, 0, -y);
	
	        // 碰撞检测
	        CollisionResults results = new CollisionResults();
	        pink.collideWith(green.getWorldBound(), results);
	        
	        if (results.size() > 0) {
	            green.getMaterial().setColor("Color", ColorRGBA.Red);
	        } else {
	            green.getMaterial().setColor("Color", ColorRGBA.Green);
	        }
	    }
	
	    @Override public void beginInput() {}
	    @Override public void endInput() {}
	    @Override public void onJoyAxisEvent(JoyAxisEvent evt) {}
	    @Override public void onJoyButtonEvent(JoyButtonEvent evt) {}
	    @Override public void onMouseButtonEvent(MouseButtonEvent evt) {}
	    @Override public void onKeyEvent(KeyInputEvent evt) {}
	    @Override public void onTouchEvent(TouchEvent evt) {}
	
	}

## 模拟物理现象

我们介绍了那么多方法，却一直没有回答一个根本问题：怎么样才能让我的模型不要掉落到地图外面去？

实现这个功能需要模拟一点点物理定律，并在检测到碰撞后改变物体的物理状态，比如运动速度、阻力等。下面我们来模拟一个简单的物理现象：物体会做自由落体运动，落到地面上会弹起来，最终在地面上滚动。

即使是最简单的物理现象，使用代码实现起来也有些麻烦。我提供了两个例子，其一是使用jME3中的纯数学方法来模拟，另一种是使用Bullet物理引擎来实现。希望通过比较两种实现的异同点，你能进一步理解碰撞检测在游戏中的用途。

### 实例：使用纯数学方法

首先，创建一个Physical类，继承并实现 AbstractControl。这个Physical 类用来为物体增加物理特性，比如速度、加速度等。在主循环中，Physical对象将会实时计算物体的运动状态，并更新物体的坐标。

	package net.jmecn.collision;
	
	import com.jme3.math.Vector3f;
	import com.jme3.renderer.RenderManager;
	import com.jme3.renderer.ViewPort;
	import com.jme3.scene.Spatial;
	import com.jme3.scene.control.AbstractControl;
	
	/**
	 * 经典物理定理运动。
	 * 
	 * @author yanmaoyuan
	 *
	 */
	public class Physical extends AbstractControl {
	    private Vector3f position = new Vector3f(0, 0, 0);// 位置
	    private Vector3f velocity = new Vector3f(1, 0, 0);// 运动速度
	    private Vector3f gravity = new Vector3f(0, 0, 0);// 重力加速度
	
	    public Vector3f getPosition() {
	        return position;
	    }
	
	    public void setPosition(Vector3f position) {
	        this.position = position;
	    }
	
	    public Vector3f getVelocity() {
	        return velocity;
	    }
	
	    public void setVelocity(Vector3f velocity) {
	        this.velocity = velocity;
	    }
	
	    public Vector3f getGravity() {
	        return gravity;
	    }
	
	    public void setGravity(Vector3f gravity) {
	        this.gravity = gravity;
	    }
	
	    @Override
	    public void setSpatial(Spatial spatial) {
	        super.setSpatial(spatial);
	        position.set(spatial.getLocalTranslation());
	    }
	
	    @Override
	    protected void controlUpdate(float tpf) {
	        velocity.addLocal(gravity.mult(tpf));
	        position.addLocal(velocity.mult(tpf));
	        spatial.setLocalTranslation(position);
	    }
	
	    @Override
	    protected void controlRender(RenderManager rm, ViewPort vp) {
	    }
	}
	
其次，创建游戏场景。在游戏中使用一个平面来代表地板，用方块来代表自由下落的物体。在主循环中进行碰撞检测，若方块落到地板上了，就把它“弹”起来。

	package net.jmecn.collision;
	
	import com.jme3.app.SimpleApplication;
	import com.jme3.collision.CollisionResults;
	import com.jme3.material.Material;
	import com.jme3.math.ColorRGBA;
	import com.jme3.math.FastMath;
	import com.jme3.math.Vector3f;
	import com.jme3.scene.Geometry;
	import com.jme3.scene.shape.Box;
	import com.jme3.scene.shape.Quad;
	
	import net.jmecn.state.AxisAppState;
	
	/**
	 * 使用碰撞检测，让地板变“坚固”。
	 * 
	 * @author yanmaoyuan
	 *
	 */
	public class TestSolidFloor extends SimpleApplication {
	
	    private Geometry cube;
	    private Geometry floor;
	    private CollisionResults results = new CollisionResults();
	
	    public TestSolidFloor() {
	        super(new AxisAppState());
	        this.setPauseOnLostFocus(false);
	    }
	
	    @Override
	    public void simpleInitApp() {
	        cam.setLocation(new Vector3f(0f, 16f, 20f));
	        cam.lookAt(Vector3f.ZERO, Vector3f.UNIT_Y);
	
	        viewPort.setBackgroundColor(ColorRGBA.White);
	
	        // 方块
	        Material mat = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");
	        mat.setColor("Color", ColorRGBA.Green);
	        cube = new Geometry("Green", new Box(1, 1, 1));
	        cube.setMaterial(mat);
	        cube.move(-10, 0, 0);
	
	        // 添加物理组件
	        Physical physical = new Physical();
	        physical.setVelocity(new Vector3f(3, 10, 0));// 速度
	        physical.setGravity(new Vector3f(0, -9.8f, 0));// 重力加速度
	        cube.addControl(physical);
	
	        // 地板
	        mat = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");
	        mat.setColor("Color", ColorRGBA.Pink);
	
	        floor = new Geometry("Floor", new Quad(16, 4));
	        floor.rotate(-FastMath.HALF_PI, 0f, 0f);
	        floor.move(-10, 0, 2);
	        floor.setMaterial(mat);
	
	        rootNode.attachChild(cube);
	        rootNode.attachChild(floor);
	    }
	
	    @Override
	    public void simpleUpdate(float tpf) {
	        // 清空上次的检测结果
	        if (results.size() > 0) {
	            results.clear();
	        }
	
	        // 碰撞检测
	        floor.collideWith(cube.getWorldBound(), results);
	        if (results.size() > 0) {
	
	            Physical physical = cube.getControl(Physical.class);
	            Vector3f v = physical.getVelocity();
	
	            // 改变速度方向
	            if (v.y <= 0) {
	                v.y = -v.y;
	            }
	
	            if (v.y > 0.0000001f) {
	                v.y *= 0.9f;// 碰撞后地板吸收一定的能量。
	            } else {
	                v.y = 0f;
	            }
	
	        }
	    }
	
	    public static void main(String[] args) {
	        TestSolidFloor app = new TestSolidFloor();
	        app.start();
	    }
	
	}

运行这个程序，你将看到一个绿色方块从高处落下，然后在“地板”上弹了几次，然后在“地板”边缘“掉”了出去。

### 实例：使用Bullet物理引擎

下面我们使用Bullet物理引擎来重新实现上例中的物理现象。

使用Bullet物理引擎之前，你需要确保自己的项目中包含对 `jme3-bullet`和 `jme3-bullet-native`这两个模块的依赖。

    // 支持bullet 3D物理引擎
    compile "$jme3.g:jme3-bullet:$jme3.v"
    compile "$jme3.g:jme3-bullet-native:$jme3.v"
    
添加依赖后，我们创建一个新的测试类，并使用BulletAppState来处理碰撞检测。需要注意的是，由于我们现在使用的是jME3之外的物理引擎来进行碰撞检测，因此jME3中的碰撞体都没法使用了。不能用BoundingVolumne来进行碰撞检测，而应该使用Bullet的 SphereCollisionShape 等碰撞体；也不能用我们定义的 Physical 类来更新物体的坐标，而应使用 Bullet 的 RigidBodyControl 等控件。

	package net.jmecn.collision;
	
	import com.jme3.app.SimpleApplication;
	import com.jme3.bullet.BulletAppState;
	import com.jme3.bullet.PhysicsSpace;
	import com.jme3.bullet.collision.shapes.MeshCollisionShape;
	import com.jme3.bullet.collision.shapes.SphereCollisionShape;
	import com.jme3.bullet.control.RigidBodyControl;
	import com.jme3.material.Material;
	import com.jme3.math.ColorRGBA;
	import com.jme3.math.FastMath;
	import com.jme3.math.Vector3f;
	import com.jme3.scene.Geometry;
	import com.jme3.scene.shape.Quad;
	import com.jme3.scene.shape.Sphere;
	
	import net.jmecn.state.AxisAppState;
	
	/**
	 * 使用Bullet物理引擎来进行碰撞检测。
	 * 
	 * @author yanmaoyuan
	 *
	 */
	public class TestSolidFloor_Bullet extends SimpleApplication {
	
	    @Override
	    public void simpleInitApp() {
	        stateManager.attach(new AxisAppState());
	
	        cam.setLocation(new Vector3f(0f, 16f, 20f));
	        cam.lookAt(Vector3f.ZERO, Vector3f.UNIT_Y);
	
	        viewPort.setBackgroundColor(ColorRGBA.White);
	
	        /**
	         * 场景
	         */
	        // 方块
	        Material mat = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");
	        mat.setColor("Color", ColorRGBA.Green);
	        Geometry sphere = new Geometry("Green", new Sphere(8, 8, 1));
	        sphere.setMaterial(mat);
	        sphere.move(-10, 0, 0);
	
	        // 地板
	        mat = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");
	        mat.setColor("Color", ColorRGBA.Pink);
	
	        Geometry floor = new Geometry("Floor", new Quad(12, 4));
	        floor.rotate(-FastMath.HALF_PI, 0f, 0f);
	        floor.move(-10, 0, 2);
	        floor.setMaterial(mat);
	
	        rootNode.attachChild(sphere);
	        rootNode.attachChild(floor);
	
	        /**
	         * 物理引擎
	         */
	        BulletAppState bullet = new BulletAppState();
	        stateManager.attach(bullet);
	
	        PhysicsSpace space = bullet.getPhysicsSpace();
	
	        // 地板
	        // 刚体控件(形状, 质量)。质量为0将不受任何力的影响，适合当做地面。
	        RigidBodyControl rigid = new RigidBodyControl(new MeshCollisionShape(new Quad(12, 4)), 0);
	        floor.addControl(rigid);
	        space.add(rigid);
	
	        // 球体
	        rigid = new RigidBodyControl(new SphereCollisionShape(1f), 1f);
	        sphere.addControl(rigid);
	        space.add(rigid);
	
	        rigid.setLinearVelocity(new Vector3f(3, 10, 0));// 线速度
	        rigid.setGravity(new Vector3f(0, -9.8f, 0));// 重力加速度
	    }
	
	    public static void main(String[] args) {
	        TestSolidFloor_Bullet app = new TestSolidFloor_Bullet();
	        app.start();
	    }
	
	}

运行此程序，你将看到一个绿色小球从高处落下，然后滚到地板边缘，最后“掉出”这个世界。