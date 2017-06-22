# 物理引擎

## 牛顿的苹果

长久以来，哲学领域有一个争论不休的话题，就是这个世界有没有独立意志？

有一种观点认为世界是有意志的，但是它的意志与人的表现形式不同。天道无私，对世间万物并无任何偏爱。另一种观点认为世界是无意识的，它的背后并无一个主宰来决定应该怎么做，只是按照大道规则自然运转。事实上这是一个无法证伪的问题，因为具有独立意志，就意味着不需要任何外在来证明其存在。

这个问题只对思考它的人有意义，最终取决于人们如何与世界相处。世界孕育了各种生灵，其中人类天生便开启了灵智。如果我们有意识，那这个孕育了我们的世界呢？

汉语中有个话说得很有意思，叫做**“采天地之灵气，吸日月之精华”**。很多人幻想世界上有种看不见摸不着的“灵气”或“能量”，要在风水好的地方才能吸收到，吸收这种“灵气”可以达到改造体质的目的。这都是胡扯。**其实天地真正的精华是什么？是万物自然运行的规律。**成语**“仰观俯察”**出自《易经·系辞》，原文是**“仰以观于天文，俯以察于地理，是故知幽明之故。”**意思是把世界当做我们的老师。人类的诸多科技文明，都是从世界中学来的，代代积累传承至今。

还有一个很有趣的词，叫做**“天心”**。天有心吗？我不知道，我只知道人有心。所谓**体悟天心**，其实是以人心感悟天道。你到底悟没悟，悟出了什么，谁也看不出来。好在中国人从来都不太在意这些虚无缥缈的东西，我们只关心**天心神用**。什么叫“天心神用”？徐公子胜治曾在其小说[《神游》](http://book.qidian.com/info/65875)中描述了一番非常经典的问论：**何者为神？用者为神。何者为用？当者为用。**那么这句话翻译过来的意思就是：什么是神？就是为了用。就好比一瓶水，我要用的就是它能充润于我，我口渴的时候它能喝，我不渴的时候它也可有可无。这就是“用者为神”。它的实用就是滋养一个人，充润一个人。我口渴了，解渴只需要一瓶水就够。一滴水不够，一桶水太多，太多太少都不恰当，不恰当的那就是“祸”了。只有恰当才称得上是“用”，这就是“当者为用”。

一个人从诞生的那一刻起，就拥有了整个世界，并且无时无刻不在吸收天地的精华。无论你是否在意，天地都将这一切赋予了你。所以玩游戏时，如果你操纵的角色直接从平地上掉出了世界，你就会非常鄙视这款游戏的开发者，因为这违背了常识。到底什么是常识？常识就是道，大道常常隐藏在平凡的事物当中。每个人对世界的规律都有自己的认知，但是少有人能把它总结成规律。人人都知道苹果熟了会掉到地上，只有牛顿这样的人才会总掉落的苹果身上看到万有引力定律。

现在，你是游戏的开发者，**你正在创造一个世界。**这个世界有意志吗？游戏本身是没有意志的，但开发游戏的你有意志，将来玩这个游戏的人也有意志。你是这个游戏的主宰，可以让这个世界表现出更加接近现实世界的规律，或者只符合你定下的规则。无论如何，你需要一个规则来运转这个世界。我们用类和对象来抽象自己对世界的理解，并用代码逻辑来描述世界的规律，越是接近现实规律，程序就越是复杂。并不是每个人都能用代码实现苹果落地的过程，物理引擎将使你最好的帮手。

## 物理引擎概述

物理引擎是一种用于模拟真实物理现象的中间件，可以用来创建虚拟的物理环境，并在其中运行来自物理世界的规则。物理引擎应用的最多的地方就是动画和游戏行业，例如3D游戏开发常用的三大物理引擎：

* [Havok](http://www.havok.com/)
* [PhysX](http://www.nvidia.cn/object/physx_new_cn.html)
* [Bullet](http://bulletphysics.org/wordpress/)

Havok引擎的授权则比较昂贵和严格，光环4、上古卷轴5等游戏大作使用的都是这款引擎。PhysX虽然现在不开源，但也实行免费推广政策，是Unity3D、CryEngine等游戏引擎的首选。Bullet物理引擎是一款开源、免费的物理引擎，jMonkeyEngine3 选择了与Bullet引擎集成。很多电影、游戏大作都选择使用这些物理引擎，网上有很多关于它们的介绍和对比，所以本文就不再赘述了，感兴趣的读者可以自行探索。

顺便提一下2D物理引擎，应用最广泛的2D物理引擎当属[Box2D](https://github.com/erincatto/Box2D)，以及它的衍生项目 Chipmunk。Cocos 2d、libGDX等2D游戏引擎中默认集成的就是Box2D。Java世界中也有一些比较出色的作品，比如[Dyn4j](https://github.com/wnbittle/dyn4j)。

除这些知名物理引擎以外，还有其他数不清的物理引擎，使用各种各样的编程语言进行开发（Java、C#、JavaScript、Python……）。我并不想一一介绍这些引擎，而是希望在学习具体的应用方法之前，能够对物理引擎背后的**共性**有所了解。我的思路是：首先大致了解物理引擎的**理论基础**，然后进一步分析各种引擎的基本**设计思路**，最后是探讨物理引擎在游戏引擎中的**使用方法**。在对**物理引擎**这个概念已经有所理解之后，再分别学习如何在 jMonkeyEngine 中集成3D物理引擎（Bullet）和2D物理引擎（Dyn4j），甚至自己集成任意物理引擎。

### 理论力学

如同3D图形引擎是建立在**计算机图形学**和**光学建模**的理论基础之上，现代物理引擎也有其理论根基，最主要的就是物理学的分支——**理论力学**。

**理论力学（theoretical mechanics）**是研究物体机械运动的基本规律的学科，是力学的一个分支。它是大部分工程技术科学的基础，也称**经典力学**。其理论基础是**牛顿运动定律**。20世纪初建立起来的量子力学和相对论，表明牛顿力学所表述的是相对论力学在物体速度远小于光速时的极限情况，也是量子力学在量子数为无限大时的极限情况。对于速度远小于光速的宏观物体的运动，包括超音速喷气飞机及宇宙飞行器的运动，都可以用经典力学进行分析 。

理论力学通常分为三个部分：静力学、运动学与动力学。**静力学（statics）**研究作用于物体上的力系的简化理论及力系平衡条件；**运动学（kinematics）**只从几何角度研究物体机械运动特性而不涉及物体的受力；**动力学（dynamics）**则研究物体机械运动与受力的关系。动力学是理论力学的核心内容。理论力学的研究方法是从一些由经验或实验归纳出的反映客观规律的基本公理或定律出发，经过数学演绎得出物体机械运动在一般情况下的规律及具体问题中的特征。理论力学中的物体主要指**质点**、**刚体**及**刚体系**，当物体的变形不能忽略时，则成为**变形体力学（如材料力学、弹性力学等）**的讨论对象。静力学与动力学是工程力学的主要部分。

质点和刚体的运动是游戏物理引擎中运用最多的技术，因为刚体在发生碰撞时可以忽略物体的形态变化。对于需要实时运算的游戏来说，这意味着只需要针对质点的运动进行模拟，可以极大得简化模拟过程。比刚体更为复杂的是软体，例如布娃娃、弹球、布料等物体，在发生碰撞后会导致模型变形。在游戏中，这种变形通常意味着需要实时计算成千上万个顶点的位置。

### 设计思路

#### 架构

从架构设计的角度来看，物理引擎是一种**中间件（Middleware）**。如同汽车发动机一样，它从设计之初就没有打算单独存在，而是作为其它系统的一个组成部分。无论是制作电影还是游戏，都可以根据需要来使用物理引擎。

同时，物理引擎又是一种独立的系统软件或服务程序，它不依赖其他特定的软件或平台而存在。这就导致几乎每个物理引擎都会自己实现一套数据结构、数学库、内存管理、碰撞检测、物理模拟等。但是，没有可视化系统。虽然我们使用物理引擎是为了获得更加真实的游戏效果，但是物理引擎本身并不在乎你的游戏画面，它的作用只是近似模拟物理现象。正因为专注，所以才更加专业。关于这个问题，我曾在知乎上看到过一个相关讨论，刷新了我对物理引擎的认知。

> 很多人搞错了一点，以为数学库要效率高，这也是我刚开始写引擎和图形程序时候的错觉。后来发现，不是的，游戏引擎的数学库，不是全都要求效率高的，数学库最重要的是“稳定”。因为真正在跑的，AAA游戏，或者重度的MMORPG，最重要的，不是数学运算，而是“架构”。反而数学运算，需要稳。稳到什么程度？havok曾经推销他们的物理引擎，最引以为傲的的，并不是它有多快，而是它的所有物理运算在所有平台上的计算结果都完全一致（评论有朋友修正了这个说法，应该是同样的计算在同一平台上每次计算都一样）。这简直吊炸天啊有木有！！！如果你不明白这句话背后的意义，那么我想你没必要讨论数学库了。

*注：这个答主的内容已经删掉了，不过[此处](http://lib.csdn.net/article/unity3d/24530)还有一个备份，有兴趣的朋友可以去看看。*

#### 封装

下面，让我们把目光从宏观的架构设计转移到微观的类和对象设计。站在面向对象分析和设计（OOA & OOD）的视角，物理引擎对现实世界中的物理现象进行了封装。一般来说，物理引擎中总是有这样几类对象：

* 世界（World）或空间（Space）
* 物体（Body）
* 关节（Joint）
* 形状（Shape）或网格（Mesh）
* 质量（Mess）
* 速度（Velocity）
* 力（Force）

**物理世界**

在诸多物理引擎中，“世界（World）”是各种物理规则的载体，比如“重力（Gravity）”。与游戏引擎中的场景图类似，“世界”也需要管理场景中的物体。如果你希望模拟物理现象，那么你就得先创造一个世界，再把各种形状的物体添加到这个世界中，然后开动世界的马达，让物体在世界中运动、碰撞、相互作用。

通常我们只需要一个“世界”，但你也可以创造不同的世界。在多个世界中定义不同的物理规则，以此来模拟不同的物理现象，如地表、水下、太空等。除了物理规则以外，“世界”最常见的属性是“边界（Bound）”。这个属性的存在主要是考虑了程序的性能问题，用于剔除超出世界边缘的物体。毕竟我们的游戏画面就那么大，对那些超出玩家感知的物体进行运算可能没有太大的必要。

多世界同样也是一种分组手段。有时同一个场景中的物体，并不一定要在一个世界中进行碰撞检测。例如赛车游戏，车辆和赛道需要进行碰撞检测，但是观众和其他自然景观应该在另一个组中。不过大多数物理引擎会直接定义用于“分组（Group）”的对象，这样就没有必要为另一个世界分配内存资源了。

**物体和关节**

“物体（Body）”代表物理引擎中实际需要进行计算的对象。据我所知，不同的物理引擎对物体的称呼可能不太一样。不过但根据理论力学中的概念，“物体”的分类应该是有规律的，通常可以分为刚体（RigidBody）、软体（SoftBody）、液体（Liquid）、布料（Cloth）、粒子（Particle）等。

有些物体在运动时会影响其他物体，或者受到一定的约束，例如滑轮组、铁链、钟摆和骨骼。约束物体的东西被称为“关节（Joints）”。根据使用场景不同，关节也有不同的分类，诸如铰链（Hinge Joint）、弹簧关节（Spring Joint）、骨骼关节（Bone Joint）、固定关节（Fixed Joint）等等。

在有些物理引擎中，“关节（Joint）”和“物体（Body）”是两种不同的东西，有些则把关节当做一类特殊物体。这对我们来说没有什么本质区别，重点是了解有这么两种东西，以及它们的作用。实际使用时，我们更关心如何配置各种物体和关节的参数，如何才能达到我们想要的效果。

**物体的属性**

“物体（Body）”是物理引擎中最重要的对象，它们一般会有很多属性，物理引擎根据这些属性来进行模拟运算。我把这些属性分为两大类，分别是**几何属性（Geometric attributes）**和**物理属性（Physical attributes）**。几何属性用于进行碰撞检测，物理属性用于计算物体在碰撞前后的运动状态。

“几何属性”其实就是物体的形状，主要用于进行碰撞检测。2D物体的形状通常是AABB、圆形、三角形、胶囊或者其他凸多边形（Convex），3D物体的形状则多了一个维度。这些内容我们在上一章已经有所讨论，这里就不再赘述了。

上一章有个问题我们没有讨论，我想很多人应该能自己想到，即复杂物体的形状可以由多个简单形状组合而来。比如一个人形动物，可以用多个圆柱体、方块、球体来代表四肢、躯干和脑袋，再用关节把各个部件链接起来。毕竟游戏中的物理引擎只是进行近似模拟，这样做的精确度已经足够了。

“物理属性”则更加复杂，包括物理学中的质量（Mess）、速度（Velocity）、力（Force）等。质量是数量（Value），而速度和力都是向量（Vector）。

质量物体的密度（density）有关，动能定理、动量守恒定理、物质守恒定理、牛顿第二定律等诸多物理定律都需要根据质量来进行计算。在物理引擎中，对于那些不希望它们运动的物体，比如游戏场景，一般会把质量设为 0 或无穷大（Infinity）。

物体的运动状态是根据速度和时间来计算的，分为线速度、加速度，角速度、角加速度。线速度让物体直线运动，角速度让物体绕质点旋转，加速度随着时间迁移而改变速度的大小和方向。静止物体的速度和加速度都为0。

力分为推力、拉力、弹力、摩擦力等。各种力作用在一个物体上，最后的合力产生了物体的加速度，从而改变物体的运动状态。

### 运行机制

世界的运行方式与游戏引擎很相似，主要是基于时间的主循环。根据每次循环经过的时间分量，就可以推动世界中所有的物体发生运动。下面是一段伪代码，用来描述这个循环：

    public class World {
        List<Body> bodies = ..
        public void update(double deltaTime) {
            // 计算力的作用
            // 计算速度
            // 计算物体的运动
            // 碰撞检测
            // 计算碰撞后的运动
            // ..
        }
    }

不过物理引擎也有与游戏引擎不同的地方，主要在于物理引擎要考虑物体运动的真实性。例如，当 deltaTime 的取值很大时，会发生什么情况？也许两个物体在1秒后就会发生碰撞，但是deltaTime的值却是10秒。按物体闪现到10秒后的位置来计算，它们可能已经互相“穿透”了。

为了保证模拟的真实性，物理引擎一般会使用**连续碰撞检测（CCD continuous collisiondetection）**，并且限制每两次碰撞检测之间的最小时间步长（timeStep）。假如运动的步长是0.1秒，当deltaTime为10秒时，物理世界的主循环可能要执行100次完整的碰撞检测。

因此，虽然物理引擎大多使用基于时间的主循环，但考虑到执行效率，通常也会配合线程或定时器来按固定帧率执行。一般物理引擎建议的执行速率是每秒60次计算，但是30帧、16帧甚至10帧的频率也是可以接受的。

**急速断点**

说句题外话。在很多模仿WOW的MMORPG中，玩家的装备上如果有加速、急速之类的属性，通常都会有“急速断点”的概念。这是由于服务器按照固定帧率运行造成的。

下文是WOW某个版本中关于治疗德鲁伊如何堆“急速”属性的讨论帖：

>急速：减少读条技能的读条时间，减少HOT技能每一跳之间的间隔(但HOT持续时间不变)每增加一部分急速，HOT都会增加一次不完整的一跳。增加到完整一跳后，继续再增加一次不完整的一跳。所以急速对于回春来说永远是有作用的。

>急速断点：急速增加到某个值，能使HOT增加一跳。增加完一跳后，如果急速达不到下一个阈值，则HOT的效果就跟之前没有变化。所以每当急速达到断点后，多出的急速是无用的。直到达到下一个断点。

>* 当面板有0.16%急速的时候，野性成长就有了第八跳
>* 当面板有14.34%急速的时候，野性成长有了第九跳
>* 当面板急速达到30%急速左右的时候，野性成长有了第十跳
>* 当面板急速达到51%急速左右的时候，野性成长有了第十一跳。 

>以上数字并不是计算获得的，而是一点一点堆急速，试出来的。

按照游戏的设定，急速属性会让玩家施法速度得到提升，效果应该是递增的。然而玩家实际上却发现，当急速属性达到某些特定的“阈值”之间时，再怎么堆急速属性都没用了，除非能到达下一个“断点”值。

这是因为服务器可能每秒只会进行16次运算，任何指令都会被分摊在1/16秒内执行。所谓“急速断点”，其实是让技能在服务器的一个单位时钟内能够执行更多次。如果想让技能从8次变成9次，那么技能的间隔时间就得从 1/8 个单位时间变成 1/9 个单位时间。除非服务器改变运行机制，否则这种“跳跃式”的急速断点就必然存在。

### 与游戏引擎的集成

在游戏引擎中使用物理引擎，一般要解决下面几个问题：

* 实现引擎之间的交互通信
* 处理引擎之间对象的对应关系
* 实现不同线程的数据同步

第一个问题的起因很简单，物理引擎通常是c++开发的，而游戏引擎则不一定。比如jMonkeyEngine是使用Java开发的，Unity3D则以C#作为编程语言，网页游戏主要使用JavaScript。Java的实现机制是JNI，Android则是NDK（并没有什么太大区别）。假设我们要和Havok引擎进行通信，就需要通过JNI来开发一套调用其dll或so库的接口。

实际上jMonkeyEngine就是通过这种方式来集成Bullet物理引擎的，`jme3-bullet-native`模块中含有`bullet`物理引擎的dll文件和so文件。因此我们才可以通过Java接口来使用Bullet物理引擎的功能。

第二个问题的来源也很容易理解。物理引擎一般都要定义自己的数据结构和数学库，而游戏引擎也会这么干。就拿jME3来说，我们使用 `com.jme3.math.Vector3f` 来表示一个3D向量，在Bullet中这个对象就没用了，因为它会定义自己的3D向量。同理，我们虽然知道物理引擎需要网格形状来进行碰撞检测，但就是没办法直接用jME3中的网格数据，需要使用物理引擎中定义的几何形状才行。

这个问题倒不难解决，只是比较麻烦，需要在JNI接口上一层对两个引擎的对象进行转换和处理。比如把jME3中的 Mesh 对应为Bullet中的 MeshCollisionShape 。它们看起来很像，但完全不是一个同一个东西。

第三个问题的起因，主要是从运行效率考虑，物理引擎的线程应该和渲染线程分离。当游戏场景需要更新时，主线程再从物理引擎中获得物体的位置和姿态，用来同步渲染对象。这项工作的处理，一般是使用一个中间对象，通过它来进行同步。

例如，在jME3中我们可以定义一个 BodyControl 类，用它来保存物理引擎中的物体对象，并在update 方法中根据物体的状态来改变 Spatial 的位置和旋转，然后jME3的主线程就会根据Spatial的信息来渲染场景。下面是一段伪代码，用来演示这个机制：

    public class BodyControl extends AbstracControl {
        private Body body;
        public BodyControl(Body body) {this.body = body;}
        @Override
        public void controlUpdate(float tpf) {
            spatial.setLocalTranslation(body.getPhysicalPosition());// 更新位置
            spatial.setLocalRotation(body.getPhysicalRotation());// 更新旋转
        }
    }

关于物理引擎的介绍，就到此为止。下面我们通过 Bullet 和 Dyn4j 物理引擎，来了解如何在jME3中使用它们。

## Bullet物理引擎

Havok、PhysX、Bullet，这三大物理引擎均是使用c++进行开发的，我们没法直接在Java程序中使用它们。好在 jME3 使用 JNI 技术实现了 Bullet 物理引擎的Java接口，因此我们可以通过 `jme3-bullet` 和 `jme3-bullet-native`库来使用Bullet 物理引擎。如果你对Havok或PhysX更感兴趣，就得自己实现它们的JNI接口了。我建议参考jME3的方式，这里是JNI接口的源代码：[jme3-bullet-native](https://github.com/jMonkeyEngine/jmonkeyengine/tree/master/jme3-bullet-native)。

### 添加Bullet依赖库

在jME3中使用Bullet引擎，我们需要添加两个依赖库，分别是：

    dependencies {
        // 添加 Bullet 物理引擎依赖库
        compile 'org.jmonkeyengine:jme3-bullet:3.1.0-stable'
        compile 'org.jmonkeyengine:jme3-bullet-native:3.1.0-stable'
    }

如果是在Android平台，则要把 `jme3-bullet-native` 替换成 `jme3-bullet-native-android`。

    dependencies {
        // 添加 Bullet 物理引擎依赖库
        compile 'org.jmonkeyengine:jme3-bullet:3.1.0-stable'
        compile 'org.jmonkeyengine:jme3-bullet-native-android:3.1.0-stable'
    }
    
我们在程序中实际使用的是 `jme3-bullet` 库中的 Java 类，`jme3-bullet-native` 和 `jme3-bullet-native-android` 分别是不同平台上的 C++ 接口，用来调用 Bullet 物理引擎的功能。

### 初始化Bullet

BulletAppState 是jME3为Bullet物理引擎开发的核心接口，我们将会通过它来使用Bullet引擎。为了提升引擎工作的效率，BulletAppState内部有独立的工作线程，物理引擎运行时并不依赖jME3的主线程。

使用的方法很简单，在
 `simpleInitApp()` 方法中实例化一个 `BulletAppState` 对象，并把它添加到 `stateManager` 中。

	package net.jmecn.physics3d;
	
	import com.jme3.app.SimpleApplication;
	import com.jme3.bullet.BulletAppState;

	/**
	 * 使用Bullet物理引擎
	 * 
	 * @author yanmaoyuan
	 */
	public class TestBullet extends SimpleApplication {
		
		@Override
		public void simpleInitApp() {		
			// 初始化物理引擎
			BulletAppState bulletAppState = new BulletAppState();
			stateManager.attach(bulletAppState);

		}
	}

物理引擎只负责计算，不管怎么显示。实际使用时，需要把场景图中的模型和物理引擎中物体关联起来。

然而，有时我们还是希望能够更加直观地看到物理引擎中的各种物体，尤其是在开发调试的过程中。

因此，jME3实现了一个BulletDebugAppState，用于可视化观察Bullet物理引擎中的各种物体。你不需要手动把它添加到stateManager中，只需要调用BulletAppState的`setDebugEnabled()`方法，它会自动把所有的活干好。

	package net.jmecn.physics3d;
	
	import com.jme3.app.SimpleApplication;
	import com.jme3.bullet.BulletAppState;

	/**
	 * 使用Bullet物理引擎
	 * 
	 * @author yanmaoyuan
	 */
	public class TestBullet extends SimpleApplication {
		
		@Override
		public void simpleInitApp() {		
			// 初始化物理引擎
			BulletAppState bulletAppState = new BulletAppState();
			stateManager.attach(bulletAppState);

			// 开启调试模式，这样能够可视化观察物体的运动情况。
			bulletAppState.setDebugEnabled(true);
		}
	}

### 使用方法

对于游戏中的任何一个物体，若想为其增加物理支持，差不多都要经历下面个过程：

1. 为模型创建一个“碰撞形状（CollisionShape）”，它是物体的几何属性，用于碰撞检测。
2. 创建一个PhysicsControl，设置好它的形状和质量。这个PhysicsControl就代表了物理引擎中的“物体”。
3. 把这个PhysicsControl和Spatial绑定。
4. 把这个PhysicsControl添加到Bullet的“物理空间（PhysicsSpace）”中。
5. 把Spatial添加到场景图（rootNode）中。
6. （可选）设置“物体”的物理属性，诸如速度、力、弹性等。
7. （可选）实现PhysicsCollisionListener接口，监听物体的碰撞事件。

下面我们来生成一个场景，做个小实验。场景中有一个固定不动的地板，作为其它物体的载体；一个以一定初速度飞出去的小球，从地板左侧向右侧运动；几块挡在小球行进路上的木板，它们将被小球逐一击倒。

示意图如下：
![一个简单的实验场景](/content/images/2017/06/ball_and_board.png)

### 物理空间

BulletAppState 中保存着对 Bullet “物理空间”的引用，即我们在前文介绍过的“世界”对象。一切需要进行物理模拟的物体，都要添加到这个空间中。

通过下面的方法就可以拿到这个物体空间对象，并把“物体”、“关节”等对象添加到空间中。

        // 获得Bullet的物理空间，它代表了运转物理规则的世界。
        PhysicsSpace physicsSpace = bulletAppState.getPhysicsSpace();
        // 将刚体添加到物理空间中
        physicsSpace.add(rigidBody);

这个空间是有边界的，边界的形状是一个盒子。在构造BulletAppState时，通过盒子对角线上最小和最大的两个顶点坐标，就可以设定物理空间的边界。例如：

    // 初始化物理引擎，设置物理空间的边界。
    BulletAppState bulletAppState = new BulletAppState(new Vector3f(-100f, -100f, -100f), new Vector3f(100f, 100f, 100f));

通过 `setGravity` 可以设置物理空间中的全局重力加速度，默认值是 `new Vector3f(0,-9.81f,0);`。

    physicsSpace.setGravity(new Vector3f(0,-9.81f,0));

### 碰撞物体

Bullet物理引擎支持很多种不同的物体，称为物理碰撞对象（PhysicsCollisionObject）。jME3只实现了3种最常用的物体：

* 角色（PhysicsCharacter）
* 刚体（PhysicsRigidBody）
* 幽灵体（PhysicsGhostObject）

角色代表的是游戏中的玩家、怪物、NPC这些东西。其运动模式一般是走、跑、蹲、跳。遇到较高的障碍物会被卡住，但是遇到较矮的台阶却可以通行。

刚体代表游戏中的简单物体，诸如苹果、箱子、车辆、树、地形等等。一般只需要考虑碰撞、运动和力的相互作用，不会像玩家控制的角色那么智能。

幽灵体比较特别。发生碰撞并不会改变物体原本的运动状态，而会穿透幽灵的身体。从作用上来说，幽灵体更偏向于纯粹的数学碰撞检测，很多物理规则对它是无效的。利用它的这种特性，很容易就可以实现NPC的警戒范围，甚至是视线检测。

jME3并没有实现Bullet中的软体、液体、布料、粒子等碰撞体，一方面是为游戏性能考虑，另一方面是因为开源项目缺人。在大多数时刻，现有的3种物体已经足够满足游戏开发的需要了。如果你需要这些特性，恐怕得自己实现。

在我们要做的实验中，地板、小球、木板都可以用刚体来表示。

#### 物体控制器

对应3种碰撞物体，jME3实现了3种不同的物体控制器（PhysicsControl）：

* CharacterControl
* RigidyBodyControl
* GhostControl

利用多个简单刚体组合成复杂物体，jME3另外设计实现了车辆（VehicleControl）、布娃娃（KinematicRagdollControl）等物体控件。

有意思的是，jME3并不希望我们直接使用前面提到的碰撞物体，而是使用这些不同类型的 PhysicsControl。例如创建一个刚体小球，做法是这样的：

		// 创建球形刚体，质量为0.65kg，半径为0.123m。
		RigidBodyControl rigidBodyBall = new RigidBodyControl(0.65f);
		rigidBodyBall.setCollisionShape(new SphereCollisionShape(0.123f));
		rigidBodyBall.setPhysicsLocation(new Vector3f(-10, 1, 0));// 在物理世界中的坐标
		rigidBodyBall.setLinearVelocity(new Vector3f(8, 5, 0));// 线速度
		rigidBodyBall.setFriction(0.2f);// 摩擦系数
		// 把小球添加到物理空间中
		physicsSpace.add(rigidBodyBall);

还记得我们在前面讨论过的一个问题吗：物理引擎要如何与游戏引擎集成？**如何实现不同线程的数据同步**？

**刚体是在物理空间中运行的，而游戏模型是在场景图中展示的，分别处于两个不同的线程中**。我们需要把模型和刚体绑定起来，并且**实时同步**它们的位置、旋转等状态，这就需要一个PhysicsControl来进行同步。

因此，如果你希望把这个小球刚体和某个模型绑定，就可以这样干：

		Spatial spatial = ...;// 加载某个模型
		spatial.addControl(rigidBodyBall);

		rootNode.attachChild(spatial);// 把模型添加到场景图
		physicsSpace.add(rigidBodyBall);// 把刚体添加到物理空间

这样做之后，场景中的模型就会保持与刚体小球同步运动。

### 几何属性

所谓几何属性，指的是物体的外形，用于进行碰撞检测。jME3称其为Collidable，意为“可进行碰撞检测的东西”。Unity3D称其为碰撞器（Collider），指的是同样的东西。Bullet引擎把物体的几何属性称为碰撞形（ColiisionShape）。

在这个实验中，地面、小球、木板的CollisionShape是不一样的。小球应该使用 `SphereCollisionShape` ，地面和木板可以使用 `BoxCollisionShape` 。创建对应的CollisionShape后，通过 `setCollisionShape()` 方法就可以把它和刚体关联在一起。

		// 创建球形刚体，质量为0.65kg，半径为0.123m。
		RigidBodyControl rigidBodyBall = new RigidBodyControl(0.65f);
		rigidBodyBall.setCollisionShape(new SphereCollisionShape(0.123f));

如果提前创建好了CollisionShape，还可以使用RigidBodyControl的另一个构造方法来进行初始化。

		// 创建球形刚体，质量为0.65kg，半径为0.123m。
		CollisionShape collisionShape = new SphereCollisionShape(0.123f)
		RigidBodyControl rigidBodyBall = new RigidBodyControl(collisionShape, 0.65f);

Bullet 提供的 CollisionShape 都位于 `com.jme3.bullet.collision.shapes` 包中，既有一些简单形状，又有一些复杂的网格形状。

#### 简单碰撞形状

简单形状是最常用的，因为碰撞检测的效率较高。下面是 jME3 对应 Bullet 物理引擎实现的CollisionShape 类。

<table>
  <tr><th>形状</th><th>用途</th><th>例子</th></tr>
  <tr>
    <td>BoxCollisionShape</td>
    <td>方块，不会在平地上滚动。</td>
    <td>长方形或立方形物体，如砖、箱子、家具。</td>
  </tr>
  <tr>
    <td>SphereCollisionShape</td>
    <td>球体，在平地上会滚动。</td>
    <td>像苹果、足球、大炮球、小型宇宙飞船之类的紧凑物体。</td>
  </tr>
  <tr>
    <td>CylinderCollisionShape</td>
    <td>管状或柱状物体。</td>
    <td>像柱子一样的长条物体；盘状物体，如轮子、盘子。</td>
  </tr>
  <tr>
    <td>CompoundCollisionShape</td>
    <td>可以用多个形状来自定义复杂形状，通过
 <code>addChildShape()</code> 方法可以向其中添加形状，并设定相对位置。</td>
    <td>车厢和车轮（一个方块+4个圆盘）等。</td>
  </tr>
  <tr>
    <td>CapsuleCollisionShape</td>
    <td>胶囊体，由一个竖直的圆筒加上下两头的半球体组成。圆筒形的身体不会被墙角或垂直边缘卡住；半球形的顶端和底部不会被楼梯台阶或地面的浅坑卡住。</td>
    <td>人、动物。</td>
  </tr>
  <tr>
    <td>SimplexCollisionShape</td>
    <td>极简形状。由一到个四点定义而成的物理点、线、三角形或矩形。</td>
    <td>空气墙、栅栏。</td>
  </tr>
  <tr>
    <td>PlaneCollisionShape</td>
    <td>2D平面。</td>
    <td>平坦坚实的地面或墙壁。</td>
  </tr>
</table>

上面这些CollisionShape可用于场景中的运动物体，当然也可用于静态物体。不过，如果你需要对模型做精确的碰撞检测，上面那些形状明显是不够用的。

#### 精确网格形状

Bullet为此提供了**精确网格形状**，可以进行三角网格级别的碰撞检测。出于游戏的性能考虑，jME3不支持对两个**精确网格形状**进行碰撞检测。比如 `MeshCollisionShape`，它适合用作游戏地图的碰撞形状，但是不适合用来当做玩家或动物的形状。

<table>
  <tr><th>形状</th><th>用途</th><th>例子</th></tr>
  <tr>
    <td>MeshCollisionShape</td>
    <td>一个精确网格形状，能够表现具有开口和附属物的复杂形状。<br/>
局限性：两个精确网格形状之间无法进行碰撞检测，只有简单形状可以与此形状碰撞。使用这种形状的的物体将无法运动。</td>
    <td>一个全局静态的精确网格模型。</td>
  </tr>
  <tr>
    <td>HullCollisionShape</td>
    <td>这是一种不太精确的网格形状，用来表示难以被CompoundCollisionShape描述的物体。它给人的感觉像是把东西装进了布袋中，你看得出来里面有个物体，但布袋的外形并没有与物体表面完全贴合。<br/>局限性：生成的形状是凸多边形。</td>
    <td>动态三D模型。</td>
  </tr>
  <tr>
    <td>GImpactCollisionShape</td>
    <td>一种用于动态物体的精确网格形状，算法来源： http://gimpact.sourceforge.net/。<br/>
局限性：CPU密集型，节约使用！我们建议使用HullCollisionShape（或CompoundCollisionShape）来提高性能。两个精确网格形状之间无法进行碰撞检测，只有简单形状可以与此形状碰撞。</td>
    <td>虚拟现实或科学模拟中复杂的动态对象（如蜘蛛）。</td>
  </tr>
  <tr>
    <td>HeightfieldCollisionShape</td>
    <td>为静态地形优化过的精确网格形状。这种形状比其他精确网格形状要快得多。<br/>局限性：需要高度图数据。两个精确网格形状之间无法进行碰撞检测，只有简单形状可以与此形状碰撞。</td>
    <td>基于高度图实现的静态地形。</td>
  </tr>
</table>

#### 使用CollisionShape

使用下面的方法，可以为Geometry的网格生成静态的精确网格形状。

    MeshCollisionShape townShape = new MeshCollisionShape(townGeom.getMesh());

jME3还为生成**精确网格形状**提供了一个工具类：CollisionShapeFactory。

例如，为一座小镇生成静态的精确网格形状：

    CollisionShape shape = CollisionShapeFactory.createMeshShape(town);

或者，为一个动态物体生成网格形状（结果可能是
 HullCollisionShape 或 CompoundCollisionShape）：

    CollisionShape shape = CollisionShapeFactory.createDynamicMeshShape(spaceCraft);

回到我们的实验，小球使用SphereCollisionShape，地面和木板都可以使用BoxCollisionShape。例如创建地面的碰撞形状，尺寸参考标准篮球场的大小：

		// 创建地板，尺寸为长28m，宽15m，厚0.1m。
		BoxCollisionShape shape = new BoxCollisionShape(new Vector3f(14f, 0.05f, 7.5f));

创建用来挡住小球的木板形状：

		// 创建木板形状，尺寸为横宽1.8m * 竖高1.05m * 厚0.03m。
		BoxCollisionShape shape = new BoxCollisionShape(new Vector3f(0.015f, 0.525f, 0.9f));

### 物理属性

#### 质量

刚体的物理属性很多，最基本的是质量。当质量为0时，这个物体就会成为一个静态物体，不会受到任何力的作用。

	// 地面刚体的质量设为0，这样就不会受到任何力的作用。
	RigidBodyControl rigidBodyFloor = new RigidBodyControl(0);

Bullet引擎中的质量单位为kg，刚体的默认质量为1kg。通过
 `setMass()` 方法可以调整物体的质量。

	RigidBodyControl rigidBodyFloor = new RigidBodyControl();
	rigidBodyFloor.setMass(0);

注意，对于使用精确网格形状的物体，质量必须设为0，否则为报错，因为这种物体通常都是全局静态的。

#### 速度

线速度（LinearVelocity）是用3D向量来表示的，向量的方向即速度的方向，向量的长度即速度的大小，速度大小的单位是 **米/秒**。

下面的代码意味着让小球斜向上飞出，速率为 5m/s

    rigidBodyBall.setLinearVelocity(new Vector3f(4, 3, 0));

![Linear Velocity](/content/images/2017/06/linear_velocity.png)

重力加速度（Gravity）与速度的表示类似，让物体受到一个垂直向下的重力加速度，是这样的：

    rigidBodyBall.setGravity(new Vector3f(0, -9.8f, 0));

角速度（AngularVelocity）也是用3D向量来表示的。Bullet使用欧拉角来描述物体的旋转，3D向量的每个分量代表绕x、y、z轴旋转的速度，单位是 **弧度/秒**。

下面的代码意味着让小球绕X轴顺时针旋转，速度为 3.14 rad/s。

    rigidBodyBall.setAngularVelocity(new Vector3f(3.14f, 0, 0));

改变物体的速度，能让一个静止的物体运动起来。

#### 阻尼

物体在运动的时候，由于受到阻尼（Damping）的影响，运动速率会逐渐降低直至归零。

    // 设置物体运动的阻尼
    rigidBodyBall.setLinearDamping(0.1f);
    rigidBodyBall.setAngularDamping(0.1f);

摩擦力（Friction）是一种产生阻尼的方式，物体表面并不完全光滑，在受到压力作用时，就会产生摩擦力。一般物体表面的摩擦系数大于0，只有绝对光滑的物体摩擦系数为0。

    // 设置物体表面的摩擦系数
    rigidBodyBall.setFriction(0.2f);

#### 弹性系数

弹性系数（Restitution）表示物体在发生碰撞后速度的改变程度，取值范围为0.0到1.0。

在两个发生碰撞的物体之中，如果任何一个物体的弹性系数为0时，垂直于碰撞面法线方向的速度会完全消除。小球落地后完全不会弹起来，感觉就像一拳打在了棉花上。

当两个物体的弹性系数为1时，垂直于碰撞面法线方向的速度会直接取反。小球从多高的地方落下，就会弹起多高。

在Bullet中，所有物体的弹性系数默认为0。我们需要手动设置地板和小球的弹性系数，才能让小球在落地后弹起来。

    rigidBodyFloor.setRestitution(1.0f);
    rigidBodyBall.setRestitution(0.8f);

#### 力

力（Force）使用3D向量表示。当力的作用方向穿透物体的质心时，物体将会产生线性的加速度。当力的作用位置不在质心上时，物体可能就会发生旋转（例如开门）。

有3种方法来对物体施加力的影响。 `applyForce(..)` 方法可以对物体施加持续的力；`applyTorque(..)` 方法可以施加一个扭矩；`applyImpulse(..)` 方法可以施加一个瞬间的冲击力。

对物体施加力的作用，能让一个静止的物体运动起来。

#### 位置

物体是处于物理空间中的，通过 `setPhysicsLocaltion()` 和 `setPhysicsRotation()` 可以改变物体的位置和旋转。

在调用 `spatial.addControl()` 把物理控件和模型绑定时，物体会被自动放置到模型的位置。此后，物体会根据自己在物理空间中的位置来改变Spatial在场景图中的位置。

### 例一

下面是这个实验的完整代码。在这个例子中，我们没有把刚体和任何模型关联，只是利用
 `bulletAppState.setDebugEnabled(true);` 方法来可视化观察物体的运动状态。另外，我使用 `jmeClone()` 方法重复创建了10个木板，减少了代码量。

    package net.jmecn.physics3d;
    
    import com.jme3.app.SimpleApplication;
    import com.jme3.bullet.BulletAppState;
    import com.jme3.bullet.PhysicsSpace;
    import com.jme3.bullet.collision.shapes.BoxCollisionShape;
    import com.jme3.bullet.collision.shapes.SphereCollisionShape;
    import com.jme3.bullet.control.RigidBodyControl;
    import com.jme3.math.Quaternion;
    import com.jme3.math.Vector3f;
    
    /**
     * 使用Bullet物理引擎
     * 
     * @author yanmaoyuan
     */
    public class TestBullet extends SimpleApplication {
    
        @Override
        public void simpleInitApp() {
            cam.setLocation(new Vector3f(-18.317675f, 16.480816f, 13.418682f));
            cam.setRotation(new Quaternion(0.13746259f, 0.86010045f, -0.3107305f, 0.38049686f));
    
            // 初始化物理引擎
            BulletAppState bulletAppState = new BulletAppState();
            stateManager.attach(bulletAppState);
    
            // 获得Bullet的物理空间，它代表了运转物理规则的世界。
            PhysicsSpace physicsSpace = bulletAppState.getPhysicsSpace();
    
            // 创建地板的刚体对象，尺寸为长28m，宽15m，厚0.1m。
            // 刚体的质量设为0，这样地板就不会受到任何力的作用。
            RigidBodyControl rigidBodyFloor = new RigidBodyControl(0);
            Vector3f halfExtents = new Vector3f(14f, 0.05f, 7.5f);
            rigidBodyFloor.setCollisionShape(new BoxCollisionShape(halfExtents));
            rigidBodyFloor.setRestitution(0.8f);// 弹性系数
            // 将刚体添加到物理空间中
            physicsSpace.add(rigidBodyFloor);
    
            // 创建球形刚体，质量为0.65kg，半径为0.123m。
            RigidBodyControl rigidBodyBall = new RigidBodyControl(0.65f);
            float radius = 0.123f;
            rigidBodyBall.setCollisionShape(new SphereCollisionShape(radius));
            rigidBodyBall.setPhysicsLocation(new Vector3f(-10, 1, 0));// 在物理世界中的坐标
            rigidBodyBall.setLinearVelocity(new Vector3f(8, 5, 0));// 线速度
            rigidBodyBall.setFriction(0.2f);// 摩擦系数
            rigidBodyBall.setRestitution(0.8f);// 弹性系数
            physicsSpace.add(rigidBodyBall);
    
            // 创建挡板的刚体对象，质量为0.2kg，尺寸为横宽1.8m * 竖高1.05m * 厚0.03m。
            RigidBodyControl rigidBodyBoard = new RigidBodyControl(0.2f);
            halfExtents = new Vector3f(0.015f, 0.525f, 0.9f);
            rigidBodyBoard.setCollisionShape(new BoxCollisionShape(halfExtents));
    
            // 克隆10个挡板，从半场开始，间隔一米摆放。
            for (int i = 0; i < 10; i++) {
                RigidBodyControl board = (RigidBodyControl) rigidBodyBoard.jmeClone();
                board.setPhysicsLocation(new Vector3f(i * 1, 0.52f, 0f));
                // 将刚体添加到Bullet的物理空间中。
                physicsSpace.add(board);
            }
    
            // 开启调试模式，这样能够可视化观察物体之间的运动。
            bulletAppState.setDebugEnabled(true);
        }
    
        public static void main(String[] args) {
            TestBullet app = new TestBullet();
            app.start();
        }
    
    }


效果截图：

![TestBullet](/content/images/2017/06/TestBullet.png)

小球从“篮球场”的一端发射出去，逐一击倒了挡在路上的木板。随着击倒木板的数量增加，小球的运动速度也渐渐归零。

### 例二

在上一个例子中，我们简单地使用了刚体碰撞，让小球击中了地面上的木板。比较遗憾的是，这些物体仅仅是在物理世界中运行，并没有真正和场景图中的物体绑定，而且我们也没法控制小球。

下面我们来做第二个案例，它的主要代码来自于官方教程的[HelloPhysics](https://jmonkeyengine.github.io/wiki/jme3/beginner/hello_physics.html)。在这个案例中，首先我们会把场景中的Geometry和刚体绑定，并且可以通过按键来发射小球，用小球去轰击地面上的砖墙。

    package net.jmecn.physics3d;
    
    import com.jme3.app.SimpleApplication;
    import com.jme3.bullet.BulletAppState;
    import com.jme3.bullet.collision.shapes.BoxCollisionShape;
    import com.jme3.bullet.collision.shapes.SphereCollisionShape;
    import com.jme3.bullet.control.RigidBodyControl;
    import com.jme3.input.KeyInput;
    import com.jme3.input.MouseInput;
    import com.jme3.input.controls.ActionListener;
    import com.jme3.input.controls.KeyTrigger;
    import com.jme3.input.controls.MouseButtonTrigger;
    import com.jme3.light.AmbientLight;
    import com.jme3.light.DirectionalLight;
    import com.jme3.material.Material;
    import com.jme3.math.ColorRGBA;
    import com.jme3.math.Vector2f;
    import com.jme3.math.Vector3f;
    import com.jme3.scene.Geometry;
    import com.jme3.scene.shape.Box;
    import com.jme3.scene.shape.Sphere;
    import com.jme3.scene.shape.Sphere.TextureMode;
    import com.jme3.texture.Texture;
    import com.jme3.texture.Texture.WrapMode;
    
    /**
     * 按键发射小球轰击砖墙。
     * 
     * @author yanmaoyuan
     *
     */
    public class HelloPhysics extends SimpleApplication implements ActionListener {
    
        /**
         * 开火，发射小球。鼠标左键触发。
         */
        public final static String FIRE = "fire";
        /**
         * 显示或隐藏BulletAppState的debug形状。按空格键触发。
         */
        public final static String DEBUG = "debug";
    
        /** 砖块的尺寸 */
        private static final float brickLength = 0.48f;
        private static final float brickWidth = 0.24f;
        private static final float brickHeight = 0.12f;
    
        private BulletAppState bulletAppState;
    
        @Override
        public void simpleInitApp() {
            cam.setLocation(new Vector3f(0, 4f, 6f));
            cam.lookAt(new Vector3f(2, 2, 0), Vector3f.UNIT_Y);
    
            bulletAppState = new BulletAppState();
            stateManager.attach(bulletAppState);
    
            // 初始化按键
            initKeys();
    
            // 初始化光照
            initLight();
    
            // 初始化场景
            initScene();
        }
    
        @Override
        public void onAction(String name, boolean isPressed, float tpf) {
            if (isPressed) {
                if (FIRE.equals(name)) {
                    shootBall();
                } else if (DEBUG.equals(name)) {
                    boolean debugEnabled = bulletAppState.isDebugEnabled();
                    bulletAppState.setDebugEnabled(!debugEnabled);
                }
            }
        }
    
        /**
         * 初始化按键输入
         */
        private void initKeys() {
            inputManager.addMapping(FIRE, new MouseButtonTrigger(MouseInput.BUTTON_LEFT));
            inputManager.addMapping(DEBUG, new KeyTrigger(KeyInput.KEY_SPACE));
            inputManager.addListener(this, FIRE, DEBUG);
        }
    
        /**
         * 初始化光照
         */
        private void initLight() {
            // 环境光
            AmbientLight ambient = new AmbientLight();
            ambient.setColor(new ColorRGBA(0.3f, 0.3f, 0.3f, 1f));
    
            // 阳光
            DirectionalLight sun = new DirectionalLight();
            sun.setDirection(new Vector3f(-1, -2, -3).normalizeLocal());
    
            rootNode.addLight(ambient);
            rootNode.addLight(sun);
        }
    
        /**
         * 初始化场景
         */
        private void initScene() {
            makeFloor();
            makeWall();
        }
    
        /**
         * 制作地板
         */
        private void makeFloor() {
            // 网格
            Box floor = new Box(10f, 0.1f, 5f);
            floor.scaleTextureCoordinates(new Vector2f(3, 6));
    
            // 材质
            Material mat = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");
            Texture tex = assetManager.loadTexture("Textures/Terrain/Pond/Pond.jpg");
            tex.setWrap(WrapMode.Repeat);
            mat.setTexture("ColorMap", tex);
    
            // 几何体
            Geometry geom = new Geometry("floor", floor);
            geom.setMaterial(mat);
            geom.setLocalTranslation(0, -0.1f, 0);// 将地板下移一定距离，让表面和xoz平面重合。
    
            // 刚体
            RigidBodyControl rigidBody = new RigidBodyControl(0);
            geom.addControl(rigidBody);
            rigidBody.setCollisionShape(new BoxCollisionShape(new Vector3f(10f, 0.1f, 5f)));
    
            rootNode.attachChild(geom);
            bulletAppState.getPhysicsSpace().add(rigidBody);
        }
    
        /**
         * 建造一堵墙
         */
        private void makeWall() {
            // 利用for循环生成一堵由众多砖块组成的墙体。
            float startpt = brickLength / 4;
            float height = 0;
            for (int j = 0; j < 15; j++) {
                for (int i = 0; i < 6; i++) {
                    Vector3f vt = new Vector3f(i * brickLength * 2 + startpt, brickHeight + height, 0);
                    makeBrick(vt);
                }
                startpt = -startpt;
                height += 2 * brickHeight;
            }
        }
    
        /**
         * 在指定位置放置一个物理砖块
         * 
         * @param loc
         *            砖块的位置
         */
        private void makeBrick(Vector3f loc) {
            // 网格
            Box box = new Box(brickLength, brickHeight, brickWidth);
            box.scaleTextureCoordinates(new Vector2f(1f, .5f));
    
            // 材质
            Material mat = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");
            Texture tex = assetManager.loadTexture("Textures/Terrain/BrickWall/BrickWall.jpg");
            mat.setTexture("ColorMap", tex);
    
            // 几何体
            Geometry geom = new Geometry("brick", box);
            geom.setMaterial(mat);
            geom.setLocalTranslation(loc);// 把砖块放在指定位置
    
            // 刚体
            RigidBodyControl rigidBody = new RigidBodyControl(2f);
            geom.addControl(rigidBody);
            rigidBody.setCollisionShape(new BoxCollisionShape(new Vector3f(brickLength, brickHeight, brickWidth)));
    
            rootNode.attachChild(geom);
            bulletAppState.getPhysicsSpace().add(rigidBody);
        }
    
        /**
         * 从摄像机所在位置发射一个小球，初速度方向与摄像机方向一致。
         */
        private void shootBall() {
            // 网格
            Sphere sphere = new Sphere(32, 32, 0.4f, true, false);
            sphere.setTextureMode(TextureMode.Projected);
    
            // 材质
            Material mat = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");
            Texture tex = assetManager.loadTexture("Textures/Terrain/Rock/Rock.PNG");
            mat.setTexture("ColorMap", tex);
    
            // 几何体
            Geometry geom = new Geometry("cannon ball", sphere);
            geom.setMaterial(mat);
    
            // 刚体
            RigidBodyControl rigidBody = new RigidBodyControl(1f);
            geom.addControl(rigidBody);
            rigidBody.setCollisionShape(new SphereCollisionShape(0.4f));
            rigidBody.setPhysicsLocation(cam.getLocation());// 位置
            rigidBody.setLinearVelocity(cam.getDirection().mult(25));// 初速度
    
            rootNode.attachChild(geom);
            bulletAppState.getPhysicsSpace().add(rigidBody);
        }
    
        public static void main(String[] args) {
            HelloPhysics app = new HelloPhysics();
            app.start();
        }
    }

下图为被小球击倒的墙体截图。

![HelloPhysics](/content/images/2017/06/HelloPhysics.png)

### 例三

下面我们来解决一个问题：如何才能让玩家在地图上行走，并且不从地图上掉落？

不想让玩家从地图上落下去很容易，只要对地图模型使用RigidBodyControl，并把质量设为0即可。

        /**
         * 初始化场景
         */
        private void initScene() {
            // 从zip文件中加载地图场景
            assetManager.registerLocator("town.zip", ZipLocator.class);
            Spatial sceneModel = assetManager.loadModel("main.scene");
    
            // 为地图创建精确网格形状
            CollisionShape sceneShape = CollisionShapeFactory.createMeshShape(sceneModel);
            RigidBodyControl landscape = new RigidBodyControl(sceneShape, 0);
            sceneModel.addControl(landscape);
            
            rootNode.attachChild(sceneModel);
            bulletAppState.getPhysicsSpace().add(landscape);
        }

*注：如果你没有town.zip文件，可以从此处下载：[jME3Tutorials/town.zip](https://github.com/jmecn/jME3Tutorials/raw/master/town.zip)。下载后，将其放在工程的根目录下即可，不必放到assets目录中。*

对于玩家而言，RigidBodyControl就有点不够用了。玩家会走、会跳，遇到障碍物还能跨越过去，这些特性可以通过CharacterControl来实现。
使用胶囊体来作为玩家的碰撞形状，并使用CharecterControl来控制角色物体。代码如下：

        /**
         * 初始化玩家
         */
        private void initPlayer() {
            // 使用胶囊体作为玩家的碰撞形状
            CapsuleCollisionShape capsuleShape = new CapsuleCollisionShape(0.3f, 1.8f, 1);
            float stepHeight = 0.5f;// 玩家能直接登上多高的台阶？
            
            // 使用CharacterControl来控制玩家物体
            CharacterControl player = new CharacterControl(capsuleShape, stepHeight);
            player.setJumpSpeed(10);// 起跳速度
            player.setFallSpeed(55);// 坠落速度
            player.setGravity(9.8f * 3);// 重力加速度
            player.setPhysicsLocation(new Vector3f(0, 10, 0));// 位置
            
            bulletAppState.getPhysicsSpace().add(player);
        }

在上述代码中，我们使用3个参数构造了一个胶囊体（`new CapsuleCollisionShape(0.3f, 1.8f, 1);`），这3个参数的含义分别是：胶囊的半径，胶囊的高度，胶囊的纵轴与Y轴平行 (0=X,1=Y,2=Z)。

实例化CharacterControl时，我们使用了胶囊体形状，并设置了“角色”的“步高”，“步高”指的是“角色”在行走时能够上多高的台阶。比较特别的是，“角色”是没有“质量”这个属性的，即不能直接对其施加力的作用。但是如果有刚体和角色发生碰撞，角色会被撞飞。

为了让玩家能够操纵角色，CharacterControl对象应该定义为全局对象，这样我们就可以在玩家触发按键输入后控制它。

比如，调用 `player.jump()` 方法可以让角色跳跃。可以通过 `player.onGround()` 方法判断角色是否站在地面上。

        public void onAction(String name, boolean isPressed, float tpf) {
            if ("Jump".equals(name) && isPressed) {
                player.jump();
            }
        }
    
想让角色行走，可以调用 `player.setWalkDirection()` 方法。行走方向的参数是一个3D向量。向量的方向即行走的方向，向量的长度则是行走的速度，单位是 **m/s**。

在第一人称视角游戏中，摄像机的位置应该与CharacterControl的位置保持同步，这样玩家就能从角色的视角去观察世界。玩家通过 AWSD 键或↑↓ ←→ 键来控制角色移动时，角色行走的方向要根据摄像机的朝向来计算。

        private CharacterControl player;
        private Vector3f walkDirection = new Vector3f(0, 0, 0);
        private boolean left = false, right = false, up = false, down = false;
        
        public void simpleUpdate(float tpf) {
            camDir.set(cam.getDirection()).multLocal(0.6f);
            camLeft.set(cam.getLeft()).multLocal(0.4f);
            walkDirection.set(0, 0, 0);
            if (left) {// 玩家按下了←
                walkDirection.addLocal(camLeft);
            }
            if (right) {// 玩家按下了→
                walkDirection.addLocal(camLeft.negate());
            }
            if (up) {// 玩家按下了↑
                walkDirection.addLocal(camDir);
            }
            if (down) {// 玩家按下了↓
                walkDirection.addLocal(camDir.negate());
            }
            player.setWalkDirection(walkDirection);
            cam.setLocation(player.getPhysicsLocation());// 保持同步
        }

另外，在第一人称视角游戏中，我们可以不用加载玩家自身的角色模型，因为玩家通过摄像机很难看到自己的模样。如果想要让玩家能够看到身体的一部分（手、脚、肚子之类的），可以使用一个简化的模型，例如无头人。

第三人称视角的游戏，如果是通过鼠标拾取地面坐标来控制角色，则需要根据目标点来计算运动的方向。通常来说，这种移动方式需要另外的 MotionControl 或 AiControl 来计算运动的方向或路径。计算出运动方向后，依然是调用 `player.setWalkDirection()` 方法来让角色行动。

        public void setTarget(Vector3f target) {
            Vector3f walkDirection = target.subtract(player.getPhysicsLocation();
            walkDirection.y = 0;// 把移动方向限制在水平面上。
            walkDirection.normalizeLocal();// 单位化
            walkDirection.multLocal(25);// 改变速率
            player.setWalkDirection(walkDirection);
        }

最后，本例的完整代码如下：

    package net.jmecn.physics3d;
    
    import com.jme3.app.SimpleApplication;
    import com.jme3.asset.plugins.ZipLocator;
    import com.jme3.bullet.BulletAppState;
    import com.jme3.bullet.collision.shapes.CapsuleCollisionShape;
    import com.jme3.bullet.collision.shapes.CollisionShape;
    import com.jme3.bullet.control.CharacterControl;
    import com.jme3.bullet.control.RigidBodyControl;
    import com.jme3.bullet.util.CollisionShapeFactory;
    import com.jme3.input.KeyInput;
    import com.jme3.input.controls.ActionListener;
    import com.jme3.input.controls.KeyTrigger;
    import com.jme3.light.AmbientLight;
    import com.jme3.light.DirectionalLight;
    import com.jme3.math.ColorRGBA;
    import com.jme3.math.Vector3f;
    import com.jme3.scene.Spatial;
    
    /**
     * 演示角色在地图中自由行走。
     * 
     * @author yanmaoyuan
     *
     */
    public class HelloCollision extends SimpleApplication implements ActionListener {
        /**
         * 显示或隐藏BulletAppState的debug形状。按F1键触发。
         */
        public final static String DEBUG = "debug";
    
        // 前、后、左、右、跳跃
        public final static String FORWARD = "forward";
        public final static String BACKWARD = "backward";
        public final static String LEFT = "left";
        public final static String RIGHT = "right";
        public final static String JUMP = "jump";
    
        private BulletAppState bulletAppState;
    
        // 角色控制器
        private CharacterControl player;
        private Vector3f walkDirection = new Vector3f();
        private boolean left = false, right = false, up = false, down = false;
    
        // 临时变量，用于保存摄像机的方向。避免在simpleUpdate中重复创建对象。
        private Vector3f camDir = new Vector3f();
        private Vector3f camLeft = new Vector3f();
    
        @Override
        public void simpleInitApp() {
            bulletAppState = new BulletAppState();
            stateManager.attach(bulletAppState);
    
            // 初始化按键
            initKeys();
    
            // 初始化光照
            initLight();
    
            // 初始化场景
            initScene();
    
            // 初始化玩家
            initPlayer();
        }
    
        @Override
        public void simpleUpdate(float tpf) {
            camDir.set(cam.getDirection()).multLocal(0.6f);
            camLeft.set(cam.getLeft()).multLocal(0.4f);
            walkDirection.set(0, 0, 0);
    
            // 计算运动方向
            boolean changed = false;
            if (left) {
                walkDirection.addLocal(camLeft);
                changed = true;
            }
            if (right) {
                walkDirection.addLocal(camLeft.negate());
                changed = true;
            }
            if (up) {
                walkDirection.addLocal(camDir);
                changed = true;
            }
            if (down) {
                walkDirection.addLocal(camDir.negate());
                changed = true;
            }
            if (changed) {
                walkDirection.y = 0;// 将行走速度的方向限制在水平面上。
                walkDirection.normalizeLocal();// 单位化
                walkDirection.multLocal(0.5f);// 改变速率
            }
    
            player.setWalkDirection(walkDirection);
            cam.setLocation(player.getPhysicsLocation());
        }
    
        /**
         * 初始化按键输入
         */
        private void initKeys() {
            inputManager.addMapping(DEBUG, new KeyTrigger(KeyInput.KEY_F1));
            inputManager.addMapping(LEFT, new KeyTrigger(KeyInput.KEY_A));
            inputManager.addMapping(RIGHT, new KeyTrigger(KeyInput.KEY_D));
            inputManager.addMapping(FORWARD, new KeyTrigger(KeyInput.KEY_W));
            inputManager.addMapping(BACKWARD, new KeyTrigger(KeyInput.KEY_S));
            inputManager.addMapping(JUMP, new KeyTrigger(KeyInput.KEY_SPACE));
    
            inputManager.addListener(this, DEBUG, LEFT, RIGHT, FORWARD, BACKWARD, JUMP);
        }
    
        @Override
        public void onAction(String name, boolean isPressed, float tpf) {
            if (DEBUG.equals(name) && isPressed) {
                boolean debugEnabled = bulletAppState.isDebugEnabled();
                bulletAppState.setDebugEnabled(!debugEnabled);
            } else if (LEFT.equals(name)) {
                left = isPressed;
            } else if (RIGHT.equals(name)) {
                right = isPressed;
            } else if (FORWARD.equals(name)) {
                up = isPressed;
            } else if (BACKWARD.equals(name)) {
                down = isPressed;
            } else if (JUMP.equals(name) && isPressed) {
                player.jump();
            }
        }
    
        /**
         * 初始化光照
         */
        private void initLight() {
            // 环境光
            AmbientLight ambient = new AmbientLight();
            ambient.setColor(new ColorRGBA(0.3f, 0.3f, 0.3f, 1f));
    
            // 阳光
            DirectionalLight sun = new DirectionalLight();
            sun.setDirection(new Vector3f(-1, -2, -3).normalizeLocal());
    
            rootNode.addLight(ambient);
            rootNode.addLight(sun);
        }
    
        /**
         * 初始化场景
         */
        private void initScene() {
            // 从zip文件中加载地图场景
            assetManager.registerLocator("town.zip", ZipLocator.class);
            Spatial sceneModel = assetManager.loadModel("main.scene");
    
            // 为地图创建精确网格形状
            CollisionShape sceneShape = CollisionShapeFactory.createMeshShape(sceneModel);
            RigidBodyControl landscape = new RigidBodyControl(sceneShape, 0);
            sceneModel.addControl(landscape);
    
            rootNode.attachChild(sceneModel);
            bulletAppState.getPhysicsSpace().add(landscape);
        }
    
        /**
         * 初始化玩家
         */
        private void initPlayer() {
            // 使用胶囊体作为玩家的碰撞形状
            CapsuleCollisionShape capsuleShape = new CapsuleCollisionShape(0.3f, 1.8f, 1);
            float stepHeight = 0.5f;// 玩家能直接登上多高的台阶？
    
            // 使用CharacterControl来控制玩家物体
            player = new CharacterControl(capsuleShape, stepHeight);
            player.setJumpSpeed(10);// 起跳速度
            player.setFallSpeed(55);// 坠落速度
            player.setGravity(9.8f * 3);// 重力加速度
            player.setPhysicsLocation(new Vector3f(0, 10, 0));// 位置
    
            bulletAppState.getPhysicsSpace().add(player);
        }
    
        public static void main(String[] args) {
            HelloCollision app = new HelloCollision();
            app.start();
        }
    }

下图为玩家在一人称视角看到的场景。

![HelloCollision](/content/images/2017/06/HelloCollision.png)

### 例四

前面几个例子看起来都很美好，下面我们要解决几个实际开发中的问题。

#### 问题分析

首先，请观察下面几幅图。

图一：Jaime的模型和胶囊体的尺寸不一样，而且原点也不重合，导致Jaime看起来是浮在天上的。

![比例问题](/content/images/2017/06/ScaleProblem.png)

图二：我们把Jaime放大为与原来2倍，使它与胶囊体的尺寸保持一致，但由于原点坐标不重合，它看起来依然是浮空的。

![原点坐标不重合](/content/images/2017/06/OriginProblem.png)

图三：寒冰射手艾希的模型，与胶囊体的原点坐标有偏差，而且比例不统一，看起来像是陷进地里了。

![原点不重合，比例不统一。](/content/images/2017/06/ScaleProblem2.png)

上图中出现的问题，都是在尝试把3D模型和物理控件绑定时出的问题。不仅是角色模型，就算是简单的方块、球体，也同样会出现类似的问题，为什么？

这是因为，模型和物理碰撞体分别处于两个世界中，这两个世界的标准不统一。Bullet物理引擎采用公制单位，但谁知道3D模型采用的是什么单位呢？虽然不太可能有物理引擎采用英制单位，但不同引擎的标准不是我们能够控制的。

再者，我们加载的3D模型，在建模时的原点坐标位于何处，这也不确定。有些游戏模型喜欢以角色的脚底为原点，有些则以模型的中心为原点。这都无可厚非，重要的是游戏开发前应该统一标准和规范。不要等到出了问题再回头修改，重做的成本太大了。

在前面几幅图中，我只是随手拣了几个模型来做示范。在实际的游戏开发中，最好是先建立标准，然后再进行建模。如果你是从其他途径获得的游戏模型，最好先搞清楚建模时的标准再进行开发。

#### 模型适配

下面，我来介绍一种解决标准不统一问题的方法，对模型和胶囊体的位置、尺寸进行适配。

到目前为止，我们都是直接把物理控制器（PhysicsControl）和Spatial关联在一起（如下图）。由于物体和Spatial的标准不统一，就会造成前面的问题。

![ModelNode](/content/images/2017/06/ModelNode.png)

我们换一个方式，先创建一个“角色”节点，把物体控制器（PhysicsControl）和它关联在一起。同时，把模型的Spatial对象也挂在这个“角色”节点下方。如果你想灵活调整摄像机的位置，还可以再创建一个摄像机节点挂在“角色”节点中。

![CharacterNode](/content/images/2017/06/CharacterNode.png)

由于3D模型是这个“角色”节点的子节点，因此在“角色”节点随物体运动时，模型也会随父节点运动。同时，我们可以调整模型的尺寸、朝向，并改变它相对父节点的位置（比如下移一点距离）。通过这种方式，就可以调整角色模型，使其能够和碰撞体近似重合。

        /**
         * 初始化玩家
         */
        private void initPlayer() {
            float radius = 0.3f;// 胶囊半径0.3米
            float height = 1.8f;// 胶囊身高1.8米
            float stepHeight = 0.5f;// 角色步高0.5米
            
            /**
             * 创建角色根节点
             */
            character = new Node("Character");
            rootNode.attachChild(character);
            
            // 加载角色的模型
            Spatial model = assetManager.loadModel("Models/Jaime/Jaime.j3o");
            model.move(0, -(height/2+radius), 0);
            model.scale(1.8f);
            character.attachChild(model);// 挂到角色根节点下

            // 使用胶囊体作为玩家的碰撞形状
            CapsuleCollisionShape capsuleShape = new CapsuleCollisionShape(radius, height, 1);
    
            // 使用CharacterControl来控制玩家物体
            player = new CharacterControl(capsuleShape, stepHeight);
            character.addControl(player);// 绑定角色控制器
            
            player.setJumpSpeed(10);// 起跳速度
            player.setFallSpeed(55);// 坠落速度
            player.setGravity(9.8f * 3);// 重力加速度
            player.setPhysicsLocation(new Vector3f(0, height/2+radius, 0));// 位置
    
            bulletAppState.getPhysicsSpace().add(player);
        }

结果：

![Adjusted](/content/images/2017/06/Adjusted.png)

对于胶囊体而言，模型需要下移多少距离是比较容易计算的，即半个圆筒的高度+球的半径(`height/2+radius`)。模型缩放的比例就不太好计算了。对于未知来源的模型，要么通过包围盒的高度来计算，要么还是手动试吧。

#### 修正摄像机位置

由于胶囊体的原点在正中心，直接使用 `player.getPhysicsLocation()` 来同步摄像机的坐标，玩家会感觉眼睛长在腰上。我们可以创建一个辅助节点，专门用来同步摄像机的位置。

        // 创造一个辅助节点，用于修正摄像机的位置。
        Node camNode = new Node("Camera");
        character.attachChild(camNode);
        camNode.setLocalTranslation(0, height/2, 0);// 将此节点上移一段距离，使摄像机位于角色的头部。

作为“角色”的子节点，此辅助节点会随父节点一起运动。我们用它的世界坐标来同步摄像机的位置，就能让玩家感觉摄像机处于角色“眼睛”的位置。

        @Override
        public void simpleUpdate(float tpf) {
            // 同步摄像机位置
            cam.setLocation(camNode.getWorldTranslation());
        }

这种做法更适合第一人称游戏，而第三人称游戏的摄像机焦点一般位于角色的中心。如果你要切换成第三人称视角，可以使用ChaseCamera，并直接把“角色”根节点设为焦点摄像机的焦点。

    ChaseCamera chaseCam = new ChaseCamera(cam, character, inputManager);

#### 参考代码

下面是一个更复杂的例子，使用第三人称控制Jaime在地图中走来走去。

![PhysicsJaime](/content/images/2017/06/PhysicsJaime.png)

这个例子基于例三，不过我对它进行了重构，把主要功能分解到了多个AppState中，并额外增加了一些功能。

* [Main](https://github.com/jmecn/jME3Tutorials/blob/master/src/main/java/net/jmecn/physics3d/jaime/Main.java) 这是主类，程序启动的入口。
* [SceneAppState](https://github.com/jmecn/jME3Tutorials/blob/master/src/main/java/net/jmecn/physics3d/jaime/SceneAppState.java) 这是游戏场景类，主要作用就是加载小镇模型，并设为不可动的刚体。
* [CharacterAppState](https://github.com/jmecn/jME3Tutorials/blob/master/src/main/java/net/jmecn/physics3d/jaime/CharacterAppState.java) 这是用来控制角色的类。出了加载Jaime的模型，并为其增加物理属性以外，我还把在以前介绍过的骨骼动画技术也应用到了其中。
* [InputAppState](https://github.com/jmecn/jME3Tutorials/blob/master/src/main/java/net/jmecn/physics3d/jaime/InputAppState.java) 这个类专门用来处理用户输入。你可以通过ASWD按键来移动Jaime，空格键跳跃，F1键开关调试网格。

由于代码实在太多，我就不直接贴出来了。点击上面的超链接即可阅读源代码。

在这个例子中，我耍了一点小花招。角色的运动方向，先是在InputAppState的 `walk()` 方法中计算了一次，确定了运动的**相对方向**。然后，在CharacterAppState的 `update()` 方法中，根据摄像机的朝向，把相对方向**旋转成了绝对方向**。如果你在阅读这段代码时有困难，可能意味着需要复习一下3D数学知识。

关于Bullet物理引擎，就先介绍到这里。

## Dyn4j物理引擎

jME3是一个3D游戏引擎。虽然它具有开发2D游戏的能力，不过似乎很少有人用它做2D游戏。jME3没有集成2D物理引擎，需要我们自己动手搞定。dyn4j是一款纯Java开发的开源2D物理引擎，我推荐使用它作为首选的2D物理引擎。

也许Box2d是比dyn4j更好的选择，但我并没有选择它，主要原因是集成起来太麻烦了。Box2d是用C++开发的，在Java中使用Box2d需要开发额外的JNI接口，而且要带着一大堆dll、so文件。虽然已经有人开发过Box2d的Java接口（例如[jbox2d](https://github.com/jbox2d/jbox2d)以及[libgdx](https://github.com/libgdx/libgdx)的扩展项目[gdx-box2d](https://github.com/libgdx/libgdx/tree/master/extensions/gdx-box2d)），但我总嫌Box2d太过重量级。

dyn4j的优点很多，首先它是纯Java开发的，天然具有跨平台优势。第二，dyn4j的文档很丰富，而且有很多学习的例子。第三，集成 dyn4j 只需要一个不到400 KiB 大小的jar文件，非常方便。第四，dyn4j的实际运算效率并不低，至少在我看来是可以接受的。第五，我不喜欢杀鸡用牛刀，如果开发游戏时真的遇到dyn4j解决不了的问题，我会尝试box2d（目前还没有遇到这种情况）。第六，jME3社区有人模仿BulletAppState的模式，为dyn4j开发了一个插件[jME3-Dyn4j-plugin](https://github.com/Hec-P/jME3-Dyn4j-plugin)。虽然我嫌他做得太复杂，但是有个人提前帮我们踩坑毕竟也是件不错的事。

我还可以继续列出很多理由，但说得再多也没有意义，不如直接上代码看看效果如何。

dyn4j的官方资料很丰富，但全部都是英文的。相关链接如下：

* [首页](http://www.dyn4j.org/)
* [下载](https://github.com/wnbittle/dyn4j/releases)
* [源码](https://github.com/wnbittle/dyn4j)
* [JavaDoc](http://docs.dyn4j.org/)
* [教程](http://www.dyn4j.org/documentation/)
* [论坛](http://forum.dyn4j.org/)
* [博客](http://www.dyn4j.org/category/blog/)

### 添加dyn4j依赖库

dyn4j引擎只需要一个jar文件，在这里可以下载到最新的发布版本：[https://github.com/wnbittle/dyn4j/releases](https://github.com/wnbittle/dyn4j/releases)。

除了直接把dyn4j的jar文件添加到项目的classpath，你还可以通过maven或gradle来添加dyn4j的依赖。

**Maven**

    <dependency>
        <groupId>org.dyn4j</groupId>
        <artifactId>dyn4j</artifactId>
        <version>3.2.4</version>
    </dependency>

**Gradle**

    dependencies {
        // 添加dyn4j依赖库
        compile 'org.dyn4j:dyn4j:3.2.4'
    }

### 使用方法

物理引擎的使用方法大同小异。想要使用dyn4j来为物体增加物理属性，一般分为下面几个步骤。

1. 创建一个物理世界（World）的实例。
2. 创物体（Body）对象，设置它的碰撞形状（Rectangle、Circle、Triangle等），并把它添加到物理世界中。
3. （可选）创建关节（Joint）对象，用它来关联物体，把这个关节添加到物理世界中。
4. 把物体和Spatial绑定。
5. 把Spatial添加到场景图（rootNode）中。
6. （可选）设置物体的物理属性，诸如质量、速度、作用力等。
7. 在游戏主循环中调用 World.update(double) 方法，驱动世界运行。

dyn4j有很多参考例子，[Examples](https://github.com/wnbittle/dyn4j/tree/master/examples/org/dyn4j/examples)文件夹中包含分别使用 java Graphics 技术和 JOGL 渲染的小程序；[Samples](https://github.com/wnbittle/dyn4j/tree/master/examples/org/dyn4j/samples)文件夹中有不少具体的案例。

在本文中，我们依然做那个小球撞木板的小实验。前面我们用Bullet物理引擎实现过了，现在用dyn4j引擎再实现一次。

### 物理世界

dyn4j的物理世界类为 `org.dyn4j.dynamics.World`。所有物体、关节都要添加到这个世界中才可以运动。

我们可以直接实例化 World 对象，并在游戏主循环中调用它的 update() 方法，即可驱动其运行。

    package net.jmecn.physics2d;

    import org.dyn4j.dynamics.World;
    import com.jme3.app.SimpleApplication;
    
    /**
     * 演示Dyn4j物理引擎
     * 
     * @author yanmaoyuan
     *
     */
    public class TestDyn4j extends SimpleApplication {
    
        // Dyn4j的物理世界
        private World world = new World();
        
        @Override
        public void simpleInitApp() {
            // TODO ..
        }
        @Override
        public void simpleUpdate(float tpf) {
            // 更新Dyn4j物理世界
            world.update(tpf);
        }
    
        public static void main(String[] args) {
            TestDyn4j app = new TestDyn4j();
            app.start();
        }
    
    }

World是有边界的，可以使用dyn4j的AABB包围盒来设置它的边界。

        AxisAlignedBounds aabb = new AxisAlignedBounds(600, 480);// 矩形大小
        aabb.translate(300, 240);// 中心坐标
        world.setBounds(aabb);// 设置边界

World中也有一个公共物理属性，最常用的就是重力加速度（Gravity）。

        world.setGravity(new Vector2(0, -9.82));// 设置重力
        world.setGravity(World.EARTH_GRAVITY);// 地球重力
        world.setGravity(World.ZERO_GRAVITY);// 失重

dyn4j的物理世界，统一使用公制单位。

* 1 单位长度 = 1 m
* 1 单位质量 = 1 kg
* 1 单位速度 = 1 m/s
* 1 单位力 = 1 N
* 1 单位密度 = 1 kg/m²。
* 旋转采用弧度值，3.1415926 单位旋转 = 180°。

考虑到物理引擎的工作效率，我们可以使用一个单独的线程来管理 World 的工作。然而如何管理这个线程并不是我们的重点，所以就不详细介绍了。如果你对具体实现感兴趣，不妨观摩一下这几个类：

* [Dyn4jAppState](https://github.com/Hec-P/jME3-Dyn4j-plugin/blob/master/src/com/jme3/physics/dyn4j/Dyn4jAppState.java)
* [ThreadingType](https://github.com/Hec-P/jME3-Dyn4j-plugin/blob/master/src/com/jme3/physics/dyn4j/ThreadingType.java)
* [PhysicsSpace](https://github.com/Hec-P/jME3-Dyn4j-plugin/blob/master/src/com/jme3/physics/dyn4j/PhysicsSpace.java)

这几个类都是模仿BulletAppState的工作模式而改写的。

### dyn4j的基本概念

dyn4j中只有一种物体，即刚体，使用 `org.dyn4j.dynamics.Body`来表示。关节则有很多不同用途的子类，共同继承 `org.dyn4j.dynamics.joint.Joint` 类表示。

Body的**质量**不能直接设置，主要是通过枚举类型 MassType 来设定的。`MassType.INFINITE` 表示无穷大质量，一般用于固定不动的物体，比如地形。 `MassType.NORMAL` 表示普通质量的物体，一般用于活动物体。如果希望精确设置物体的质量，可以通过设置物体各个部位的密度来间接设置。

Body的**物理属性**包括重力加速度（Gravity），线速度（Linear Velocity），角速度（Angular Velocity）、力（Force）等。用法和 Bullet 引擎几乎差不多，比如通过 `applyForce(Vector2 )` 方法可以对物体施加一个瞬间的作用力。

关于Body的**几何属性**，dyn4j定义了很多基本的2D几何形状，位于 `org.dyn4j.geometry` 包中。包括AABB、射线、矩形、圆形、椭圆、三角形、胶囊、线段、折线、顶点等。这些几何形状通称为凸多边形（Convex），均可用于碰撞检测。

每个物体的碰撞形状，可以使用多种几何形状来定义。物体的每个部件称为一个 `BodyFixture `，每个 BodyFixture 中可以存储一个 `Convex` 对象。每个 BodyFixture 可以有自己的物理属性，例如密度（Density）、摩擦系数（Friction）、弹性系数（Restitution）等。

![body-structure](http://www.dyn4j.org/wp-content/uploads/2010/02/body-structure.png)

当一个几何形状刚创建时，默认是处于原点(0, 0)处的，可以使用 `traslate(x, y)` 方法来改变它在物体中的相对位置。

下面的代码，定义了一个圆形的刚体。

        // 创建一个圆形
        Circle circle = new Circle(0.5);// 创建一个半径0.5米的圆
        circle.translate(0, 3);// 上移3米

        // 创建物体的部件
        BodyFixture fixture = new BodyFixture(circle);
        fixture.setRestitution(0.7);// 弹性系数0.7

        // 创建刚体
        Body body = new Body();
        body.addFixture(fixture);// 设置碰撞形状
        body.setMass(MassType.NORMAL);// 质量
        body.translate(-10, 1);// 位置坐标
        body.setLinearVelocity(8, 5);// 线速度
        body.setLinearDamping(0.05);// 阻尼
        
        world.addBody(body);
    
### 同步刚体和模型

通过实现AbstractControl，我们可以轻松完成 Body 和 Spatial 的同步。唯一的问题在于，Body处于2D空间，而Spatial处于3D空间，需要把2D的**位移**和**旋转**转化到为3D空间中。我的方案是，把2D位移转化到XOY平面中，然后利用Quaternion来实现的绕Z轴旋转。

为此，我定义了一个BodyControl，用于同步 Body 和 Spatial。代码如下：

    package net.jmecn.physics2d;
    
    import org.dyn4j.dynamics.Body;
    
    import com.jme3.math.Quaternion;
    import com.jme3.math.Vector3f;
    import com.jme3.renderer.RenderManager;
    import com.jme3.renderer.ViewPort;
    import com.jme3.scene.control.AbstractControl;
    import com.jme3.util.TempVars;
    
    /**
     * 物体控制器
     * 
     * @author yanmaoyuan
     *
     */
    public class BodyControl extends AbstractControl {
    
        private Body body;
    
        public BodyControl(final Body body) {
            this.body = body;
        }
    
        public Body getBody() {
            return body;
        }
    
        @Override
        protected void controlUpdate(float tpf) {
            TempVars temp = TempVars.get();
    
            float x = (float) body.getTransform().getTranslationX();
            float y = (float) body.getTransform().getTranslationY();
            float z = spatial.getLocalTranslation().z;
            spatial.setLocalTranslation(x, y, z);
    
            Quaternion rotation = temp.quat1;
            float angle = (float) body.getTransform().getRotation();
            rotation.fromAngleNormalAxis(angle, Vector3f.UNIT_Z);
            spatial.setLocalRotation(rotation);
    
            temp.release();
        }
    
        @Override
        protected void controlRender(RenderManager rm, ViewPort vp) {
        }
    
    }

算法本身很简单，只是单纯的同步。不过我在 `controlUpdate()` 方法中使用了 jME3 的工具类 `TempVars`，减少内存泄露。`TempVars` 中定义了很多全局静态变量，适合作为临时变量来使用，可以避免在主循环中反复实例化对象。注意时候后要调用 `release()` 方法释放 TempVars 对象。

### 例五

小球撞木板实验的完整代码如下：

    package net.jmecn.physics2d;
    
    import org.dyn4j.dynamics.Body;
    import org.dyn4j.dynamics.BodyFixture;
    import org.dyn4j.dynamics.World;
    import org.dyn4j.geometry.Circle;
    import org.dyn4j.geometry.MassType;
    import org.dyn4j.geometry.Rectangle;
    
    import com.jme3.app.SimpleApplication;
    import com.jme3.light.DirectionalLight;
    import com.jme3.material.Material;
    import com.jme3.math.ColorRGBA;
    import com.jme3.math.Quaternion;
    import com.jme3.math.Vector3f;
    import com.jme3.scene.Geometry;
    import com.jme3.scene.shape.Box;
    import com.jme3.scene.shape.Sphere;
    
    /**
     * 演示Dyn4j物理引擎
     * 
     * @author yanmaoyuan
     *
     */
    public class TestDyn4j extends SimpleApplication {
    
        // Dyn4j的物理世界
        private World world = new World();
    
        @Override
        public void simpleInitApp() {
            // 调整摄像机的位置，便于观察场景。
            cam.setLocation(new Vector3f(-5.326285f, 13.811526f, 22.174831f));
            cam.setRotation(new Quaternion(0.024544759f, 0.9620277f, -0.2556783f, 0.09235176f));
    
            // 背景色改成白色
            viewPort.setBackgroundColor(ColorRGBA.White);
    
            // 添加光源
            rootNode.addLight(new DirectionalLight(new Vector3f(-1, -2, -3).normalizeLocal()));
    
            makeFloor();
    
            makeCircle();
    
            makeRectangle();
        }
    
        /**
         * 创建随机颜色的感光材质
         * 
         * @return
         */
        private Material getMaterial() {
            ColorRGBA color = ColorRGBA.randomColor();
    
            Material mat = new Material(assetManager, "Common/MatDefs/Light/Lighting.j3md");
            mat.setColor("Diffuse", color);
            mat.setColor("Ambient", color.mult(0.7f));
            mat.setColor("Specular", ColorRGBA.Black);
            mat.setFloat("Shininess", 1f);
            mat.setBoolean("UseMaterialColors", true);
            return mat;
        }
    
        /**
         * 建造一个固定的“地板”
         */
        private void makeFloor() {
            // 矩形的高和宽
            float width = 28f;
            float height = 0.1f;
    
            // 刚体
            Body body = new Body();
            body.addFixture(new Rectangle(width, height));// 碰撞形状
            body.setMass(MassType.INFINITE);// 质量无穷大
    
            world.addBody(body);
    
            // 几何体
            // 2D的物体并没有厚度，但jME3毕竟是一个3D引擎。所以也可以用3D方块来表示2D的矩形。 :)
            Geometry geom = new Geometry("body", new Box(width / 2, height / 2, 0.1f));
            geom.setMaterial(getMaterial());
            
            // 绑定刚体和模型
            geom.addControl(new BodyControl(body));
    
            rootNode.attachChild(geom);
        }
    
        /**
         * 制造一个圆形
         */
        private void makeCircle() {
            // 半径
            float radius = 0.123f;
    
            // 刚体
            Body body = new Body();
    
            // 碰撞形状
            BodyFixture fixture = body.addFixture(new Circle(radius));
            fixture.setRestitution(0.7);// 弹性
    
            body.setMass(MassType.NORMAL);// 普通质量
            body.translate(-10, 1);// 位置坐标
            body.setLinearVelocity(8, 5);// 线速度
            body.setLinearDamping(0.05);// 阻尼
    
            world.addBody(body);
    
            // 几何体
            Geometry geom = new Geometry("body", new Sphere(32, 32, radius));
            geom.setMaterial(getMaterial());
    
            // 绑定刚体和模型
            geom.addControl(new BodyControl(body));
    
            rootNode.attachChild(geom);
        }
    
        /**
         * 制造矩形木板
         */
        private void makeRectangle() {
            float width = 0.03f;
            float height = 1.05f;
    
            // 生成10个木板，从半场开始，间隔一米摆放。
            for (int i = 0; i < 10; i++) {
                // 刚体
                Body body = new Body();
                // 碰撞形状
                body.addFixture(new Rectangle(width, height));
                body.setMass(MassType.NORMAL);
                body.translate(i * 1, height / 2);// 设置位置
    
                world.addBody(body);
    
                // 几何体
                Geometry geom = new Geometry("body", new Box(width / 2, height / 2, 0.1f));
                geom.setMaterial(getMaterial());
    
                // 绑定刚体和模型
                geom.addControl(new BodyControl(body));
    
                rootNode.attachChild(geom);
            }
        }
    
        @Override
        public void simpleUpdate(float tpf) {
            // 更新Dyn4j物理世界
            world.update(tpf);
        }
    
        public static void main(String[] args) {
            TestDyn4j app = new TestDyn4j();
            app.start();
        }
    
    }

运行效果如图所示：

![TestDyn4j](/content/images/2017/06/TestDyn4j.png)

### 例六

上面的例子演示了如何在jME3中使用2D物理引擎，有没有办法把画面也“变成”2D的？

#### 平行投影

首先，应该把摄像机的工作模式改为“平行投影”。

    cam.setParallelProjection(true);// 平行投影

只调用这个方法是有隐患的，一般还需要调整摄像机的分辨率。换言之，我们可能需要把画面放大一点。

当摄像机处于平行投影模式时，画面中的物体不会给人近大远小的感觉。此时，1单位距离 = 1像素，这意味着一个半径为0.5的小球，看上去只有1个像素那么大。假如摄像机的分辨率是1280 * 720，恐怕没有人能够看清它。

注意，我们这里讨论的是**摄像机的分辨率**，不是窗口的分辨率，也不是显示器的分辨率。前者指的是摄像机观察场景的范围有多大，这个范围内的场景最终将会被渲染到游戏窗口中。摄像机的分辨率越小，物体在屏幕上看起来就越大，因为窗口的大小是不变的。

> 如果你无法理解，请随便找两张同样大小的纸来。在第一张纸上画1个尽量大的圆，力求把整张纸占满；在另一张张纸上画10个互不相交的圆，力求让这10个圆把整张纸占满。最后，比较一下两张纸上圆的大小。

前面已经介绍过，dyn4j中的长度单位是“米”，因此，我们最好同样使用“米”作为画面的长度单位。假设我们希望屏幕最宽能看到10米范围内的物体，可以使用下面的方式来调整摄相机分辨率：

    /**
     * 将摄像机改变为平行模式，并调整分辨率。
     */
    private void resetCamera() {
        // 摄像机原分辨率
        float width = cam.getWidth();
        float height = cam.getHeight();

        // 调整后的宽度
        float w = 10;// 米
        // 调整后的高度
        float h = w * height / width;

        // 画面边框离到中心的距离
        float right = w / 2;// 右
        float left = -right;// 左
        float top = h / 2; // 上
        float bottom = -top;// 下
        float far = 1000;// 远
        float near = -1000;// 近

        cam.setFrustum(near, far, left, right, top, bottom);
        cam.setParallelProjection(true);// 开启平行投影模式
        cam.lookAtDirection(new Vector3f(0, 0, -1), Vector3f.UNIT_Y);
    }

上在面的代码中，摄像机的分辨率是通过 `setFrustum()` 方法来改变的。

为摄像机的位置为中心（原点），摄像机的边框分别与X、Y轴相交于4个点，left、right、top、bottom的值即为交点在数轴上的值。

![摄相机分辨率](/content/images/2017/06/CameraDimension.png)

以摄像机所在平面为中心，在垂直方向平行“伸出”两个平面，到摄像机平面的距离分别为near、far。这两个平面之间形成了一个空间，只有处于这个空间内的3D物体才会被渲染到画面上。这个空间也称为“视锥空间”。

当摄像机处于“平行投影”模式时，near平面和far平面的大小的一样的，视锥空间的形状是一个长方体。

![平行投影](/content/images/2017/06/frustum.png)

至于“正交投影”模式，空间的形状则是一个锥形，这也是为什么称其为“视锥”的原因。

![正交投影](/content/images/2017/06/world-view-projection.png)

#### 模型纸片化

在2D游戏中使用3D模型有些浪费，使用方形网格+图片能够节省很大的开销。不过，具体选择什么样的素材与游戏的艺术风格有关，本文只是介绍一种方法。

在例五中，我们使用球体网格（Sphere）来表示2D的圆形，其实完全可以用方形纸片来表示。

下面是一张篮球图片，来源是dyn4j引擎自带的例子。

![篮球](/content/images/2017/06/Basketball.png)

图片本身是方形的，但是画面上除篮球以外的部分都是透明的。我们可以把这个篮球图片用作纸片的纹理贴图，并把材质的“混色模式（BlendMode）”设为“Alpha混色”，这样透明部分就看不见了。

除此之外，还有模型适配的问题，我们在例四中已经处理过一次了。dyn4j中的几何形状，原点都在正中心，但Quad网格的原点却在矩形的左下角。我们可以使用一个 Node 来调整 Geometry 的相对位置，再把 BodyControl 绑定到这个Node上，这个问题就解决了。

下面是重新制作的小球模型。

    /**
     * 制造一个圆形
     */
    private void makeCircle() {
        // 半径
        float radius = 0.123f;

        // 刚体
        Body body = new Body();

        // 碰撞形状
        BodyFixture fixture = body.addFixture(new Circle(radius));
        fixture.setRestitution(0.7);// 弹性

        body.setMass(MassType.NORMAL);// 普通质量
        body.translate(-5, 1.5);// 位置坐标
        body.setLinearVelocity(8, 5);// 线速度
        body.setLinearDamping(0.05);// 阻尼

        world.addBody(body);

        // 纹理贴图
        Texture tex = assetManager.loadTexture("Textures/Dyn4j/Samples/Basketball.png");
        tex.setWrap(WrapMode.Repeat);
        // 材质
        Material mat = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");
        mat.setTexture("ColorMap", tex);
        mat.getAdditionalRenderState().setBlendMode(BlendMode.Alpha);

        // 几何体
        Quad quad = new Quad(radius * 2, radius * 2);
        Geometry geom = new Geometry("body", quad);
        geom.setMaterial(mat);

        // 辅助节点
        Node node = new Node("Ball");
        node.addControl(new BodyControl(body));

        // 调整相对位置。
        node.attachChild(geom);
        geom.setLocalTranslation(-radius, -radius, 0);

        rootNode.attachChild(node);
    }

#### 完整代码

    package net.jmecn.physics2d;
    
    import org.dyn4j.dynamics.Body;
    import org.dyn4j.dynamics.BodyFixture;
    import org.dyn4j.dynamics.World;
    import org.dyn4j.geometry.Circle;
    import org.dyn4j.geometry.MassType;
    import org.dyn4j.geometry.Rectangle;
    
    import com.jme3.app.SimpleApplication;
    import com.jme3.light.DirectionalLight;
    import com.jme3.material.Material;
    import com.jme3.material.RenderState.BlendMode;
    import com.jme3.math.ColorRGBA;
    import com.jme3.math.Vector2f;
    import com.jme3.math.Vector3f;
    import com.jme3.scene.Geometry;
    import com.jme3.scene.Node;
    import com.jme3.scene.shape.Quad;
    import com.jme3.texture.Texture;
    import com.jme3.texture.Texture.WrapMode;
    
    /**
     * 演示2D物理
     * 
     * @author yanmaoyuan
     *
     */
    public class TestDyn4j2D extends SimpleApplication {
    
        // Dyn4j的物理世界
        private World world = new World();
    
        @Override
        public void simpleInitApp() {
            // 重置摄像机参数
            resetCamera();
    
            // 背景色改成白色
            viewPort.setBackgroundColor(ColorRGBA.White);
    
            // 添加光源
            rootNode.addLight(new DirectionalLight(new Vector3f(-1, -2, -3).normalizeLocal()));
    
            makeFloor();
    
            makeCircle();
    
            makeRectangle();
        }
    
        /**
         * 将摄像机改变为平行模式，并调整分辨率。
         */
        private void resetCamera() {
            /**
             * 改变摄像机的分辨率。
             */
            // 摄像机原分辨率
            float width = cam.getWidth();
            float height = cam.getHeight();
    
            // 调整后的宽度
            float w = 10;
            // 调整后的高度
            float h = w * height / width;
    
            // 画面边框离到中心的距离
            float right = w / 2;// 右
            float left = -right;// 左
            float top = h / 2; // 上
            float bottom = -top;// 下
    
            cam.setFrustum(-1000, 1000, left, right, top, bottom);
    
            cam.setParallelProjection(true);// 开启平行投影模式
    
            cam.setLocation(new Vector3f(0, 2, 0));
            cam.lookAtDirection(new Vector3f(0, 0, -1), Vector3f.UNIT_Y);
        }
    
        /**
         * 建造一个固定的“地板”
         */
        private void makeFloor() {
            // 矩形的高和宽
            float width = 8f;
            float height = 0.3f;
    
            // 刚体
            Body body = new Body();
            body.addFixture(new Rectangle(width, height));// 碰撞形状
            body.setMass(MassType.INFINITE);// 质量无穷大
    
            world.addBody(body);
    
            // 纹理贴图
            Texture tex = assetManager.loadTexture("Textures/Terrain/BrickWall/BrickWall.jpg");
            tex.setWrap(WrapMode.Repeat);
            // 材质
            Material mat = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");
            mat.setTexture("ColorMap", tex);
    
            // 几何体
            Quad quad = new Quad(width, height);
            quad.scaleTextureCoordinates(new Vector2f(width, height));
            Geometry geom = new Geometry("floor", quad);
            geom.setMaterial(mat);
    
            // 使用辅助节点，用于调整模型的相对位置
            Node node = new Node("Floor");
            node.addControl(new BodyControl(body));
    
            node.attachChild(geom);
            geom.setLocalTranslation(-width / 2, -height / 2, 0);
    
            rootNode.attachChild(node);
        }
    
        /**
         * 制造一个圆形
         */
        private void makeCircle() {
            // 半径
            float radius = 0.123f;
    
            // 刚体
            Body body = new Body();
    
            // 碰撞形状
            BodyFixture fixture = body.addFixture(new Circle(radius));
            fixture.setRestitution(0.7);// 弹性
    
            body.setMass(MassType.NORMAL);// 普通质量
            body.translate(-5, 1.5);// 位置坐标
            body.setLinearVelocity(8, 5);// 线速度
            body.setLinearDamping(0.05);// 阻尼
    
            world.addBody(body);
    
            // 纹理贴图
            Texture tex = assetManager.loadTexture("Textures/Dyn4j/Samples/Basketball.png");
            tex.setWrap(WrapMode.Repeat);
            // 材质
            Material mat = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");
            mat.setTexture("ColorMap", tex);
            mat.getAdditionalRenderState().setBlendMode(BlendMode.Alpha);
    
            // 几何体
            Quad quad = new Quad(radius * 2, radius * 2);
            Geometry geom = new Geometry("body", quad);
            geom.setMaterial(mat);
    
            // 辅助节点
            Node node = new Node("Ball");
            node.addControl(new BodyControl(body));
    
            // 调整相对位置。
            node.attachChild(geom);
            geom.setLocalTranslation(-radius, -radius, 0);
    
            rootNode.attachChild(node);
        }
    
        /**
         * 制造矩形木板
         */
        private void makeRectangle() {
            float width = 0.3f;
            float height = 0.3f;
    
            // 纹理贴图
            Texture tex = assetManager.loadTexture("Textures/Dyn4j/Samples/Crate.png");
            tex.setWrap(WrapMode.Repeat);
            // 材质
            Material mat = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");
            mat.setTexture("ColorMap", tex);
    
            // 网格
            Quad quad = new Quad(width, height);
    
            // 生成10个木板，从半场开始，间隔一米摆放。
            for (int i = 0; i < 10; i++) {
                // 刚体
                Body body = new Body();
                // 碰撞形状
                body.addFixture(new Rectangle(width, height));
                body.setMass(MassType.NORMAL);
                body.translate(3, height / 2 + 0.2f + i * height);// 设置位置
    
                world.addBody(body);
    
                // 几何体
                Geometry geom = new Geometry("body", quad);
                geom.setMaterial(mat);
    
                Node node = new Node("Box");
                node.addControl(new BodyControl(body));
    
                node.attachChild(geom);
                geom.setLocalTranslation(-width / 2, -height / 2, 0);
    
                rootNode.attachChild(node);
            }
        }
    
        @Override
        public void simpleUpdate(float tpf) {
            // 更新Dyn4j物理世界
            world.update(tpf);
        }
    
        public static void main(String[] args) {
            TestDyn4j2D app = new TestDyn4j2D();
            app.start();
        }
    
    }


效果如下：

![飞起的小球](/content/images/2017/06/FlyingBall.png)

![篮球击垮了木箱](/content/images/2017/06/Crash.png)

### 例七

在了解了2D摄像机、2D物理引擎之后，我准备了一个投篮小游戏的例子。通过鼠标拖拽，控制投篮的方向和力量，视图把它扔到篮筐里。

![投篮小游戏](/content/images/2017/06/PlayBasketball.png)

源码：[投篮小游戏](https://github.com/jmecn/jME3Tutorials/tree/master/src/main/java/net/jmecn/physics2d/basket)

虽然我称其为一个“游戏”，但实质上它只是一个demo，用于进一步演示dyn4j在jME3中的应用。

如果你对dyn4j物理引擎感兴趣，建议去看看它官网提供的例子，前文中已经全部列出来了。