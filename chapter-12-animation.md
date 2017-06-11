# 第十二章：动画

本章我们将学习如何在jME3中播放3D动画。

## 概述

3D动画一般使用Blender、Maya、3DS Max、ZBrush等专业工具制作。jME3不能用于制作3D动画，但它可以导入包含3D动画数据的模型，然后在游戏中播放。

根据游戏开发的一般需求，jME3.1目前支持3种动画：

1. 骨骼动画（Skeleton Animation）
 * 骨骼动画用于制作动画角色，可以表演出角色的各种行为，例如“行走”、“攻击”、“跳跃”、“打招呼”等。
 * 播放骨骼动画时，模型一般都是在原地不动的。例如单独播放“行走”动画，看起来更像是在走“太空步”。
 * 骨骼动画的制作非常困难，需要专业人士付出大量精力和时间。
2. 运动路径（Motion Path）
 * 运动路径的作用是让模型按一定的轨迹移动。
 * 一条路径上可以有多个路径点（way point）。
 * 路径点之间的连线可以是直线，也可以是平滑的曲线。
 * 运动路径是由程序生成的，不需要专业建模工具。
3. 电影化（Cinematic）
 * jME3的“电影化”工具是用来制作游戏中的剧情动画的。
 * 你可以使用“电影化”工具来控制时间线，把骨骼动画、运动路径和声音结合起来做成关键帧，几个关键帧就可以表演出复杂的故事剧情。
 * 除了角色模型以外，你还可以使用“电影化”工具来控制摄像机。
 * 可以同时操纵多条时间线、多个角色的剧情动画。
 * 剧情动画是由代码生成的，不需要专业的建模工具。
 * 你需要像个编剧一样先设计好剧情脚本，然后再用代码实现它。

## 骨骼蒙皮动画

骨骼蒙皮动画，简称骨骼动画，因其占用磁盘空间少并且动画效果好被广泛用于3D游戏中。它把网格顶点(皮)绑定到一个骨骼层次上面，当骨骼层次变化之后，可以根据绑定信息计算出新的网格顶点坐标，进而驱动该网格变形。

一个完整的骨骼动画一般由骨架层次（Bone Hierarchy）、绑定网格（Rigged and Skinned mesh）以及一系列关键帧（Key Pose）组成，一个关键帧对应于骨架的一个新状态，两个关键帧之间的状态可以通过插值得到。

jME3游戏引擎只能加载、播放录制好的动画。因此，你必须使用其他工具(比如Blender)创建角色动画。

究竟什么样的模型才算是一个可以播放的模型呢？

1. 绑定骨骼（Skeleton Rigging）: 必须为模型构造骨架。
2. 蒙皮（Skinning）: 把单个骨头与相应的皮肤区域关联起来。
3. 关键帧动画: 一幅关键帧就是某个动作序列的快照记录。

你有3种途径获得制作好的动画模型：

1. 从网上下载免费的模型。
2. 从其他动画制作师哪里购买做好的模型。
3. 自己使用专业工具（Blender）制作。

3D骨骼动画的制作过程非常复杂，下面我会介绍制作的过程，但是不会涉及到具体的方法（因为我也不太会）。

讲解的过程中，我将会请jME3社区的吉祥物Jaime（吉米）来做我的模特。

![Jaime](/content/images/2017/03/Jaime.png)

> Jaime是jME3核心开发者`Nehon`使用Blender制作的3D动画模型，它现在是jme3-testdata模块中的一部分。模型的加载路径为：[Models/Jaime/Jaime.j3o](https://github.com/jMonkeyEngine/jmonkeyengine/tree/master/jme3-testdata/src/main/resources/Models/Jaime)。

> Jaime的原始文件为blender格式，保存在googlecode.com上。我在github上保留了一个备份文件，下载：[Jaime.blend](https://github.com/jmecn/jME3Tutorials/blob/master/jME3Tutorials/src/main/resources/Models/Jaime/Jaime.blend?raw=true)(9.5MB)。

### 设计

在开始做动画之前，首先要进行设计。据说专业制作3D动画时，制作人员会考虑剧本、人设、场景、分镜之类的东西。我们做的是游戏，主要根据游戏的需要来设计角色的外形、姿态、动作等等。

Jaime的动作包括：

* Idle 站在原地
* Wave 挥手打招呼
* Walk 用双足行走
* Run 四肢着地跑
* Punches 像正前方连续出拳
* SideKick （左）侧踢
* JumpStart、Jumping、JumpEnd 跳跃（这个动作被分解成了起跳、滞空、落地3个部分）
* Taunt 嘲讽（双拳捶胸，作咆哮状）

### 建模

建模这个就不多说了，它的结果就是我们常见的静态模型。

为了绑定骨骼动画方便，一般人形的模型都会做成一个标准的大字：双臂张开、双腿站直、眼睛目视前方。

![建模](/content/images/2017/05/1_Jaime_Modeling.png)

### 绑定骨骼（Rigging）

想让静态模型动起来，关键就靠绑定骨骼。首先要制作骨骼，然后要把骨骼和模型的网格绑定，后一步也称为蒙皮。

#### 制作骨骼

这个步骤的重点是根据模型的形状来制作骨骼，让骨骼的初始位置和模型的各个位置保持一致。

在不同的建模工具中，骨骼有不同的表现形式，诸如：

八面锥
![八面锥](/content/images/2017/05/2_Jaime_Rigging_oct.png)

线框
![线框](/content/images/2017/05/2_Jaime_Rigging_wire.png)

样条骨
![样条骨](/content/images/2017/05/2_Jaime_Rigging_box.png)

棍形
![棍形](/content/images/2017/05/2_Jaime_Rigging_stick.png)

注意！！骨骼动画中的“骨骼”，并不是你在上图中看到的锥体、方块或者线条，而是各个连接处的“关节点”！！这些关节点以父子层次关系链接在一起，最能表现骨骼本质是上图中的“线框”形态。

骨骼应当遵循命名规则，这样是不仅是为了方便3D引擎区别哪是哪，后面在蒙皮时也会用到。

骨头之间的连接采用父子层次结构（树形结构）：移动一块骨头会拉着别的骨头跟它一起动(比如运动胳膊会带动手掌)。

下图为Jaime全身骨骼的名字和层次结构：

![骨骼的名字](/content/images/2017/05/2_Jaime_Skeleton.png)

模型的动作越复杂，需要的骨骼越多。以Jaime为例，为了让Jaime做出各种面部表情，它的头部多很多额外的骨骼是用来控制眉毛、眼睛、嘴巴的。

为了游戏效率考虑，应该尽量减少骨骼个数量。jME3支持最多128根骨头。

#### 蒙皮（Skinning）

蒙皮中的“皮肤”指的是模型的网格。刚做好的骨骼和网格是分离的，蒙皮就是将骨骼和网格绑定的过程。

每块骨头都与部分皮肤相连。动画制作的时候，(看不见的)骨头拽着(看得见的)皮肤跟它一起动。例子：大腿骨与大腿上部皮肤的连接。

每个骨头只能控制皮肤中的一部分，一块皮肤可能受不止一块骨头的影响。例如：膝盖、肘部。jME3支持每个顶点最多受4个骨头影响。每个骨头对同一位置的影响称为“权重”。

骨头与皮肤的连接过程是渐进的：你先制定每块皮肤受骨头移动影响的权重。比如：当大腿动作的时候，整条腿全部受影响，髋关节受的影响小一些，而头部完全不受影响。

例如，Jaime的膝关节主要影响膝盖的转向，同时也少量影响大腿的运动。

![蒙皮](/content/images/2017/05/3_Jaime_Skinning.png)

动画制作师需要使用一些蒙皮工具来设定每一块骨头对表皮的影响，也称为“刷权重”。

除了手绘权重以外，一些插件、工具也是必要的。例如Blender中的“封套”工具可以显示骨骼的影响范围。

![封套](/content/images/2017/05/2_Jaime_Rigging_ball.png)

### 动作制作

绑定骨骼之后，模型就可以动起来了。

Jaime，给大家挥个手。

![挥手](/content/images/2017/05/4_Jaime_Pose.png)

#### 关键帧

每个动画是由多个关键帧（Key Pose或Key Frame）组成的，而每个关键帧其实就是一个具体的姿势（Pose），例如“挥手（Wave）”动作就是由35个关键帧组成的。

下图是“挥手”动作的部分关键帧截图：
![Pose0](/content/images/2017/05/4_Jaime_Pose0.png)
![Pose1](/content/images/2017/05/4_Jaime_Pose1.png)
![Pose2](/content/images/2017/05/4_Jaime_Pose2.png)
![Pose3](/content/images/2017/05/4_Jaime_Pose3.png)
![Pose4](/content/images/2017/05/4_Jaime_Pose4.png)
![Pose5](/content/images/2017/05/4_Jaime_Pose5.png)
![Pose6](/content/images/2017/05/4_Jaime_Pose6.png)

在游戏引擎中，通过插值计算可以得到关键帧之间任意时刻的姿态，这样就能够平滑地播放动画。

![插值](/content/images/2017/05/interpolate.png)

#### 动作制作方法

使用真人演员进行动作捕捉，可以获得最为生动的动作。不过这么做成本也很高，光是动作捕捉设备的价格就相当不菲。

![动作捕捉](/content/images/2017/05/5_Actor.jpg)

直接用建模工具手动调整，这要求专业、经验、耐心、细心缺一不可。当然，也不少不了一些辅助工具。例如：

* 改变骨骼的外形，以便于制作者控制骨骼的姿态；
* 使用反向运动（IK，Inverse Kinematics），通过改变末端骨骼的位置来牵动整体姿态。

*使用IK来编辑动画时，一般会在原骨骼的基础上增加一些辅助骨骼。例如Jaime目光的焦点、尾巴的尖端、手指、脚趾等。这些骨骼不和网格绑定，仅用于调整骨骼的姿态。*

![形变](/content/images/2017/05/4_Jaime_Pose_shift.png)

#### 动画分轨

每个模型可以有多个动画。每个动画都有一个名字，用于辨识(例如：“走”，“攻击”，“跳”)。

在建模工具中制作动画时，可以将一长段动画分解成若干个子动画，这样在游戏中就可以分别引用。通过程序控制动画的播放顺序，就可以组合出新的动画。

![多轨动画](/content/images/2017/05/5_Jaime_AnimTrack.png)

### 合成

动画制作的后期，一般要将角色和场景、天空、灯光等合成，然后渲染成3D动画。

对于3D游戏来说，合成的过程将在游戏引擎中完成，模型会在场景图中实时渲染。为了将模型加入到游戏中，就需要将制作好的模型导出为包含动画数据的模型文件。

jME3.1目前支持obj、blender、orge、fbx格式的3D模型，其中orge和fbx是含动画数据的。jME3建议的制作过程为：

1. 使用Blender或3ds Max等工具建立动画模型。
2. 使用Orge插件，将模型导出为orge格式。
3. 在jME3 SDK中导入orge模型，然后转成j3o文件。
4. 在游戏中加载j3o模型文件。

----
附录：

* [使用Blender创建3D资源](https://jmonkeyengine.github.io/wiki/jme3/external/blender.html)
* [使用3DS Max创建骨骼动画](https://jmonkeyengine.github.io/wiki/jme3/external/3dsmax.html)
* [MakeHuman+OrgeXML插件+Blender制作角色动画](https://jmonkeyengine.github.io/wiki/jme3/advanced/makehuman_blender_ogrexml_toolchain.html)

Unity3D引擎有两篇文章，同样具有相当的参考价值：

* [Unity3D游戏美术全攻略：从入门到精通](http://www.gameres.com/msg_674582.html)
* [Unity游戏动画 从入门到住院：动画状态机](http://www.gameres.com/680484.html)

其他文章：

[悟：动效设计法则【转载】](http://www.cgjoy.com/thread-180436-1-1.html)

## 播放动画

我们已经了解了什么是骨骼动画，下面介绍如何导入动画模型，并使用AnimControl和AnimChannel来播放骨骼动画。

样例代码中使用的模型依然是Jaime。程序启动后Jaime将会站着不动；按住键盘上的w键Jaime就会开始行走；按空格键Jaime会跳起来。

![HelloAnimation](/content/images/2017/05/HelloAnimation.png)

### 样例代码

    package net.jmecn;
    
    import com.jme3.animation.AnimChannel;
    import com.jme3.animation.AnimControl;
    import com.jme3.animation.AnimEventListener;
    import com.jme3.animation.LoopMode;
    import com.jme3.app.SimpleApplication;
    import com.jme3.input.KeyInput;
    import com.jme3.input.controls.ActionListener;
    import com.jme3.input.controls.KeyTrigger;
    import com.jme3.light.AmbientLight;
    import com.jme3.light.DirectionalLight;
    import com.jme3.material.Material;
    import com.jme3.math.ColorRGBA;
    import com.jme3.math.FastMath;
    import com.jme3.math.Vector3f;
    import com.jme3.scene.Geometry;
    import com.jme3.scene.Spatial;
    import com.jme3.scene.shape.Quad;
    
    /**
     * 动画
     * 
     * @author yanmaoyuan
     *
     */
    public class HelloAnimation extends SimpleApplication {
    
        /**
         * 按W键行走
         */
        private final static String WALK = "walk";
        
        /**
         * 按空格键跳跃
         */
        private final static String JUMP = "jump";
        
        /**
         * 记录Jaime的行走状态。
         */
        private boolean isWalking = false;
        
        /**
         * 动画模型
         */
        private Spatial spatial;
        
        private AnimControl animControl;
        private AnimChannel animChannel;
        
        public static void main(String[] args) {
            // 启动程序
            HelloAnimation app = new HelloAnimation();
            app.start();
        }
    
        @Override
        public void simpleInitApp() {
            // 初始化摄像机
            initCamera();
            
            // 初始化灯光
            initLight();
            
            // 初始化按键输入
            initKeys();
            
            // 初始化场景
            initScene();
            
            // 动画控制器
            animControl = spatial.getControl(AnimControl.class);
            animControl.addListener(animEventListener);
            
            // 显示这个模型中有多少个动画，每个动画的名字是什么。
            System.out.println(animControl.getAnimationNames());
    
            animChannel = animControl.createChannel();
            // 播放“闲置”动画
            animChannel.setAnim("Idle");
        }
        
        /**
         * 初始化摄像机
         */
        private void initCamera() {
            // 禁用第一人称摄像机
            flyCam.setEnabled(false);
            
            cam.setLocation(new Vector3f(1, 2, 3));
            cam.lookAt(new Vector3f(0, 0.5f, 0), new Vector3f(0, 1, 0));
        }
        
        /**
         * 初始化光源
         */
        private void initLight() {
            // 定向光
            DirectionalLight sunLight = new DirectionalLight();
            sunLight.setDirection(new Vector3f(-1, -2, -3));
            sunLight.setColor(new ColorRGBA(0.8f, 0.8f, 0.8f, 1f));
    
            // 环境光
            AmbientLight ambientLight = new AmbientLight();
            ambientLight.setColor(new ColorRGBA(0.2f, 0.2f, 0.2f, 1f));
    
            // 将光源添加到场景图中
            rootNode.addLight(sunLight);
            rootNode.addLight(ambientLight);
        }
        
        /**
         * 初始化按键
         */
        private void initKeys() {
            // 按W键行走
            inputManager.addMapping(WALK, new KeyTrigger(KeyInput.KEY_W));
            // 按空格键跳跃
            inputManager.addMapping(JUMP, new KeyTrigger(KeyInput.KEY_SPACE));
            
            inputManager.addListener(actionListener, WALK, JUMP);
        }
        
        /**
         * 初始化场景
         */
        private void initScene() {
            // 加载Jaime模型
            spatial = assetManager.loadModel("Models/Jaime/Jaime.j3o");
            rootNode.attachChild(spatial);
            
            // 创建一个平面作为舞台
            Geometry stage = new Geometry("Stage", new Quad(2, 2));
            Material mat = new Material(assetManager, "Common/MatDefs/Light/Lighting.j3md");
            mat.setColor("Diffuse", ColorRGBA.White);
            mat.setColor("Specular", ColorRGBA.White);
            mat.setColor("Ambient", ColorRGBA.Black);
            mat.setFloat("Shininess", 0);
            mat.setBoolean("UseMaterialColors", true);
            stage.setMaterial(mat);
            
            stage.rotate(-FastMath.HALF_PI, 0, 0);
            stage.center();
            rootNode.attachChild(stage);
        }
        
        /**
         * 按键动作监听器
         */
        private ActionListener actionListener = new ActionListener() {
            @Override
            public void onAction(String name, boolean isPressed, float tpf) {
                /**
                 * 若Jaime已经处于JumpStart/Jumping/JumpEnd状态，就不要再做其他动作了。
                 */
                // 查询当前正在播放的动画
                String curAnim = animChannel.getAnimationName();
                if (curAnim != null && curAnim.startsWith("Jump")) {
                    return;
                }
                
                if (WALK.equals(name)) {// 走
                    
                    // 记录行走状态
                    isWalking = isPressed;
                    
                    if (isPressed) {
                        // 播放“行走”动画
                        animChannel.setAnim("Walk");
                        animChannel.setLoopMode(LoopMode.Loop);// 循环播放
                    } else {
                        // 播放“闲置”动画
                        animChannel.setAnim("Idle");
                        animChannel.setLoopMode(LoopMode.Loop);
                    }
                    
                } else if (JUMP.equals(name)) {// 跳
                    
                    if (isPressed) {
                        
                        // 播放“起跳”动画
                        animChannel.setAnim("JumpStart");
                        animChannel.setLoopMode(LoopMode.DontLoop);
                        animChannel.setSpeed(1.5f);
                    }
                }
            }
        };
        
        /**
         * 动画事件监听器
         */
        private AnimEventListener animEventListener = new AnimEventListener() {
            @Override
            public void onAnimCycleDone(AnimControl control, AnimChannel channel, String animName) {
                if ("JumpStart".equals(animName)) {
                    // “起跳”动作结束后，紧接着播放“着地”动画。
                    channel.setAnim("JumpEnd");
                    channel.setLoopMode(LoopMode.DontLoop);
                    channel.setSpeed(1.5f);
                    
                } else if ("JumpEnd".equals(animName)) {
                    // “着地”后，根据按键状态来播放“行走”或“闲置”动画。
                    if (isWalking) {
                        channel.setAnim("Walk");
                        channel.setLoopMode(LoopMode.Loop);
                    } else {
                        channel.setAnim("Idle");
                        channel.setLoopMode(LoopMode.Loop);
                    }
                }
            }
    
            @Override
            public void onAnimChange(AnimControl control, AnimChannel channel, String animName) {
            }
        };
    }

### 加载动画模型

这个步骤比较简单，使用assetManager直接加载Jaime.j3o模型即可。

        // 加载Jaime模型
        spatial = assetManager.loadModel("Models/Jaime/Jaime.j3o");
        rootNode.attachChild(spatial);

需要注意的是，Jaime采用了感光材质，因此场景中必须要添加光源才能看到Jaime。

        /**
         * 初始化光源
         */
        private void initLight() {
            // 定向光
            DirectionalLight sunLight = new DirectionalLight();
            sunLight.setDirection(new Vector3f(-1, -2, -3));
            sunLight.setColor(new ColorRGBA(0.8f, 0.8f, 0.8f, 1f));
    
            // 环境光
            AmbientLight ambientLight = new AmbientLight();
            ambientLight.setColor(new ColorRGBA(0.2f, 0.2f, 0.2f, 1f));
    
            // 将光源添加到场景图中
            rootNode.addLight(sunLight);
            rootNode.addLight(ambientLight);
        }

### 动画控制器

#### AnimControl

如果你加载的模型带有骨骼动画数据，就可以通过AnimContorl来控制动画。AnimControl中保存了所有的动画数据，它的作用是根据这些数据实时计算骨骼（Skeleton）的姿态。

    /**
     * 动画模型
     */
    private Spatial spatial;
        
    private AnimControl animControl;
    private AnimChannel animChannel;

    public void simpleInitApp() {
        ...

        // 动画控制器
        animControl = spatial.getControl(AnimControl.class);
        animControl.addListener(animEventListener);

        // 播放“Idle”动画
        animChannel = animControl.createChannel();
        animChannel.setAnim("Idle");
    }

静态模型是没有动画数据的，为避免空指针异常，一般需要判断一下animControl对象是否为空。AnimControl对象为空说明这个模型没有动画数据，或者jME3加载时没有正确加载它的动画数据。

    animContorl = spatial.getControl(AnimControl.class);
    if (animControl != null) {
        ...
    }

*本章的样例代码中没有加这个判断，因为我十分确定Jaime的模型中是有动画数据的。:P*

AnimControl中保存了所有的动画数据，通过这个控制器，你能够访问模型的动画数据。例如`getAnimationNames()`方法可用于查询动画的名称：

    // 显示这个模型中有多少个动画，每个动画的名字是什么。
    System.out.println(animControl.getAnimationNames());

结果是：

    [Walk, Jumping, Idle, JumpEnd, Punches, SideKick, Wave, Taunt, JumpStart, Run]

对于那些从网上下载的免费模型来说，这个方法可以让你搞清楚模型内部有哪些动画。

#### AnimChannel

知道了动画的名字，就可以通过AnimChannel来播放对应的动画。首先使用`AnimControl#createChannel()`创建动画通道（AnimChannel），然后调用`AnimChannel#setAnim(String name)`播放对应的动画。

    // 播放“Idle”动画
    animChannel = animControl.createChannel();
    animChannel.setAnim("Idle");

一般情况下，单个动画通道（AnimChannel）就可以满足播放动画的需要了。但是根据游戏的需要，有时3D动画师会把角色的动作分解成多个动画，以实现各种组合动作的需要。

例如，一个角色身体的不同部位可以有不同的动作：

* 角色的下肢有“行走”、“站立”、“下蹲”、“蹲行”、“跳跃”等动作。角色做出这些动作时，躯干和上肢保持不动。
* 躯干部分有“挺直”、“弯腰”、“后仰”、“匍匐”等动作。角色做出这些动作时，下肢和上肢保持不动。
* 上肢部分有“挥手”、“戒备”、“攻击”、“持枪”、“持刀”、“空手握拳”等动作。角色做出这些动作时，躯干和下肢保持不动。

想要让角色做出“站立挥手”、“下蹲攻击”、“跳起挥手”、“持枪匍匐前进”等复杂动作，可以通过已有的动画进行排列组合。这样一方面节省了3D动画师的工作量，另一方面也减少了动画模型资源的体积。

一个AnimControl可以拥有多个动画通道（AnimChannel），每个通道同一时刻都可以用于播放一个动画序列。创建多个AnimChannel，就可以同时播放多个动画。

下面是一段伪代码，演示2个AnimChannel同时工作：

    // 创建2个动画通道
    AnimChannel legChannel = animControl.createChannel();
    AnimChannel armChannel = animControl.createChannel();
    // 让角色做出“跑动攻击”动作
    legChannel.setAnim("Run");
    armChannel.setAnim("Attack");

### 动画状态监听器

游戏中的动画通常使用“有限状态机（FSM）”来设计和管理，为此我们需要知道某个动画什么时候播放完毕。

AnimControl中可以注册AnimEventListener接口。当动画播放完毕后，AnimEventListener的实现类将会得到通知。通过实现这个接口，我们就可以在动画结束或改变后进行进一步的处理。

例如：跳跃着陆后，根据跳跃前的状态恢复成“Walk”或“Idle”动画。

        public void simpleInitApp() {
            ...

            // 动画控制器
            animControl = spatial.getControl(AnimControl.class);
            animControl.addListener(animEventListener);

            ...
        }

        /**
         * 动画事件监听器
         */
        private AnimEventListener animEventListener = new AnimEventListener() {
            @Override
            public void onAnimCycleDone(AnimControl control, AnimChannel channel, String animName) {
                if ("JumpStart".equals(animName)) {
                    // “起跳”动作结束后，紧接着播放“着地”动画。
                    channel.setAnim("JumpEnd");
                    channel.setLoopMode(LoopMode.DontLoop);
                    channel.setSpeed(1.5f);
                    
                } else if ("JumpEnd".equals(animName)) {
                    // “着地”后，根据按键状态来播放“行走”或“闲置”动画。
                    if (isWalking) {
                        channel.setAnim("Walk");
                        channel.setLoopMode(LoopMode.Loop);
                    } else {
                        channel.setAnim("Idle");
                        channel.setLoopMode(LoopMode.Loop);
                    }
                }
            }
    
            @Override
            public void onAnimChange(AnimControl control, AnimChannel channel, String animName) {
            }
        };
    }

### 用户控制动画

游戏场景中有些动画是不需要玩家控制的，比如树枝随风摆动、小动物跑跑跳跳、NPC到处闲逛等。而玩家控制的角色模型，需要根据用户输入来进行即时交互。

根据我们前面在学过的知识，首先应当在InputManager中绑定动作消息和触发器，然后再绑定消息和监听器。当监听器得到动作消息后，就可以通过`AnimChannel#setAnim(String name)`方法来播放对应的动画。

        private boolean isWalking = false;

        /**
         * 初始化按键
         */
        private void initKeys() {
            // 按W键行走
            inputManager.addMapping(WALK, new KeyTrigger(KeyInput.KEY_W));
            // 按空格键跳跃
            inputManager.addMapping(JUMP, new KeyTrigger(KeyInput.KEY_SPACE));
            
            inputManager.addListener(actionListener, WALK, JUMP);
        }

        /**
         * 按键动作监听器
         */
        private ActionListener actionListener = new ActionListener() {
            @Override
            public void onAction(String name, boolean isPressed, float tpf) {
                /**
                 * 若Jaime已经处于JumpStart/Jumping/JumpEnd状态，就不要再做其他动作了。
                 */
                // 查询当前正在播放的动画
                String curAnim = animChannel.getAnimationName();
                if (curAnim != null && curAnim.startsWith("Jump")) {
                    return;
                }
                
                if (WALK.equals(name)) {// 走
                    
                    // 记录行走状态
                    isWalking = isPressed;
                    
                    if (isPressed) {
                        // 播放“行走”动画
                        animChannel.setAnim("Walk");
                        animChannel.setLoopMode(LoopMode.Loop);// 循环播放
                    } else {
                        // 播放“闲置”动画
                        animChannel.setAnim("Idle");
                        animChannel.setLoopMode(LoopMode.Loop);
                    }
                    
                } else if (JUMP.equals(name)) {// 跳
                    
                    if (isPressed) {
                        
                        // 播放“起跳”动画
                        animChannel.setAnim("JumpStart");
                        animChannel.setLoopMode(LoopMode.DontLoop);
                        animChannel.setSpeed(1.5f);
                    }
                }
            }
        };

除了`setAnim`方法以外，AnimChannel还可以额外设置一些其他的参数：

* `setLoopMode(LoopMode lm)` 设置循环模式，默认为LoopMode.Loop。
 * LoopMode.DontLoop 只播放一次
 * LoopMode.Loop 循环播放。使用这种模式时，动画播放完一遍后，会继续从头开始播放。
 * LoopMode.Cycle 轮循播放。使用这种模式时，动画会先顺序播放一遍，然后从末尾逆序再播放一遍，整个过程像荡秋千一样。
* `setSpeed(float speed)` 设置动画播放的速度，默认为1f。
 * speed不可以小于0.0f
 * speed小于1时，播放速度会放慢。
 * speed等于1时，按正常速度播放。
 * speed大于1时，播放速度会加快。

上面两个方法要在`setAnim`方法之后调用才有效果，因为这个方法将会重置循环模式和播放速度。

## 操纵骨骼

前文中我们介绍了“骨骼蒙皮”的概念，即每一块骨骼通过一定的“权重”来影响“皮肤”。为了让骨骼动画能够正常工作，3D模型中除了保存网格、材质等数据，还必须保存“骨骼蒙皮”数据。

从数据结构的角度来看，每个骨骼（Skeleton）由多个骨头（Bone）以父子关系组成，每个骨头保存了自己的名称、姿态等数据；皮肤实际上就是模型的网格（Mesh），每个网格由多个顶点（Vertex）组成。

“骨骼蒙皮”数据记录的是每个顶点（Vertex）受多个骨头（Bone）之间的影响程度（即权重）。在jME3中，每个顶点最多受4个骨头的影响，这些骨骼的权值之和应该等于1.0。

下面我们将通过一个样例来学习SkeletonControl、SkeletonDebugger的作用，以及如何利用骨骼来给角色模型添加附件。

你能看出这个骨骼中的“骨头”吗？

![Jaime的骨骼](/content/images/2017/05/Jaime_Skeleton-1.png)

在下面的例子中，按F1可以显示/隐藏绿色的骨骼调试器；按F2可以启用/禁用骨骼控制器；按F3可以移除/添加Jaime右手的道具。

### 样例代码

    package net.jmecn;
    
    import com.jme3.animation.AnimControl;
    import com.jme3.animation.SkeletonControl;
    import com.jme3.app.SimpleApplication;
    import com.jme3.input.KeyInput;
    import com.jme3.input.controls.ActionListener;
    import com.jme3.input.controls.KeyTrigger;
    import com.jme3.light.AmbientLight;
    import com.jme3.light.DirectionalLight;
    import com.jme3.material.Material;
    import com.jme3.math.ColorRGBA;
    import com.jme3.math.FastMath;
    import com.jme3.math.Quaternion;
    import com.jme3.math.Vector3f;
    import com.jme3.scene.Geometry;
    import com.jme3.scene.Node;
    import com.jme3.scene.Spatial;
    import com.jme3.scene.debug.SkeletonDebugger;
    import com.jme3.scene.shape.Cylinder;
    
    /**
     * 演示Jaime的骨骼
     * @author yanmaoyuan
     *
     */
    public class HelloSkeleton extends SimpleApplication {
    
        /**
         * 按F1显示/隐藏SkeletonDebugger。
         */
        public final static String TOGGLE_SKELETON_DEBUGGER = "toggle_SkeletonDebugger";
        
        /**
         * 按F2启用/禁用SkeletonContorl。
         */
        public final static String TOGGLE_SKELETON_CONTROL = "toggle_SkeletonControl";
        
        /**
         * 按F3移除/添加Jaime右手的附件。
         */
        public final static String TOGGLE_ATTACHMENT = "toggle_attachment";
        
        // 我们的模特：Jaime
        private Node jaime;
        
        // 骨骼调试器
        private SkeletonDebugger sd;
        
        // 骨骼控制器
        private SkeletonControl sc;
        
        // Jaime的右手
        private Node rightHand;
        // Jaime的棍子
        private Spatial stick;
        
    
        @Override
        public void simpleInitApp() {
            /**
             * 摄像机
             */
            cam.setLocation(new Vector3f(8.896082f, 12.328749f, 13.69658f));
            cam.setRotation(new Quaternion(-0.09457599f, 0.9038204f, -0.26543108f, -0.32204098f));
            flyCam.setMoveSpeed(10f);
            
            /**
             * 要有光
             */
            rootNode.addLight(new AmbientLight(new ColorRGBA(0.2f, 0.2f, 0.2f, 1f)));
            rootNode.addLight(new DirectionalLight(new Vector3f(-1, -2, -3), new ColorRGBA(0.8f, 0.8f, 0.8f, 1f)));
    
            /**
             * 加载Jaime的模型
             */
            jaime = (Node)assetManager.loadModel("Models/Jaime/Jaime.j3o");
            // 将Jaime放大一点点，这样我们能观察得更清楚。
            jaime.scale(5f);
            rootNode.attachChild(jaime);
            
            // 获得SkeletonControl
            sc = jaime.getControl(SkeletonControl.class);
            
            // 打印骨骼的名称和层次关系
            System.out.println(sc.getSkeleton());
            
            /**
             * 创建一个SkeletonDebugger，用于显示骨骼的形状。
             */
            sd = new SkeletonDebugger("debugger", sc.getSkeleton());
            jaime.attachChild(sd);
            
            // 将SkeletonDebugger的姿态与Jaime的几何体同步。
            Spatial child = jaime.getChild(0);
            sd.setLocalTransform(child.getLocalTransform());
            
            // 创建材质
            Material mat = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");
            mat.setColor("Color", ColorRGBA.Green);
            mat.getAdditionalRenderState().setDepthTest(false);// 禁用深度测试，实现透视效果。
            sd.setMaterial(mat);
    
            /**
             * 绑定附件
             */
            stick = createAttachment();
            
            rightHand= sc.getAttachmentsNode("hand.L");
            rightHand.attachChild(stick);
            
            /**
             * 播放骨骼动画
             */
            AnimControl animControl = jaime.getControl(AnimControl.class);
            animControl.createChannel().setAnim("Walk");
            
            /**
             * 用户输入
             */
            inputManager.addMapping(TOGGLE_SKELETON_DEBUGGER, new KeyTrigger(KeyInput.KEY_F1));
            inputManager.addMapping(TOGGLE_SKELETON_CONTROL, new KeyTrigger(KeyInput.KEY_F2));
            inputManager.addMapping(TOGGLE_ATTACHMENT, new KeyTrigger(KeyInput.KEY_F3));
            inputManager.addListener(actionListener,
                TOGGLE_SKELETON_DEBUGGER, TOGGLE_SKELETON_CONTROL, TOGGLE_ATTACHMENT);
        }
        
        /**
         * 给Jaime做一根棍子当做武器
         * @return
         */
        private Spatial createAttachment() {
            Node node = new Node("Golden Stick");

            // 棍子的中间用黄色的材质
            Material mat = new Material(assetManager, "Common/MatDefs/Light/Lighting.j3md");
            mat.setColor("Diffuse", ColorRGBA.Yellow);
            mat.setColor("Specular", ColorRGBA.White);
            mat.setColor("Ambient", ColorRGBA.Yellow);
            mat.setFloat("Shininess", 2f);
            mat.setBoolean("UseMaterialColors", true);
            
            Geometry body = new Geometry("body", new Cylinder(2, 6, 0.02f, 1.2f, true));
            body.setMaterial(mat);

            // 棍子两端用红色材质。
            mat = mat.clone();// 为了省事，直接克隆材质。
            mat.setColor("Diffuse", ColorRGBA.Red);
            mat.setColor("Ambient", ColorRGBA.Red);
            
            Geometry head1 = new Geometry("head1", new Cylinder(2, 6, 0.02f, 0.4f, true));
            Geometry head2 = new Geometry("head2", new Cylinder(2, 6, 0.02f, 0.4f, true));
            head1.setMaterial(mat);
            head2.setMaterial(mat);
            
            node.attachChild(head1);
            node.attachChild(body);
            node.attachChild(head2);
            head1.move(0, 0, 0.8f);
            head2.move(0, 0, -0.8f);
            
            // 稍微调整一下这根棍子的姿态，使它和Jaime的右手掌心契合。
            node.rotate(0, FastMath.HALF_PI, 0);
            node.move(0.8f, 0.1f, 0.05f);
            return node;
        }
        
        /**
         * 动作监听器
         */
        private ActionListener actionListener = new ActionListener() {
            @Override
            public void onAction(String name, boolean isPressed, float tpf) {
                if (isPressed) {
                    if (TOGGLE_SKELETON_DEBUGGER.equals(name)) {
                        toggleSkeletonDebugger();
                    } else if (TOGGLE_SKELETON_CONTROL.equals(name)) {
                        toggleSkeletonControl();
                    } else if (TOGGLE_ATTACHMENT.equals(name)) {
                        toggleAttachment();
                    }
                }
                
            }
        };
        
        /**
         * 显示或隐藏SkeletonDebugger
         */
        private void toggleSkeletonDebugger() {
            if (sd.getParent() != null) {
                sd.removeFromParent();
            } else {
                jaime.attachChild(sd);
            }
        }
        
        /**
         * 移除或添加SkeletonContorl
         */
        private void toggleSkeletonControl() {
            // sc.setEnabled(!sc.isEnable());

            if (sc.isEnabled()) {
                sc.setEnabled(false);
            } else {
                sc.setEnabled(true);
            }
        }
        
        /**
         * 移除或添加Jaime右手的附件
         */
        private void toggleAttachment() {
            if (stick.getParent() != null) {
                stick.removeFromParent();
            } else {
                rightHand.attachChild(stick);
            }
                
        }
    
        public static void main(String[] args) {
            HelloSkeleton app = new HelloSkeleton();
            app.start();
        }
    
    }

### 骨骼控制器

#### SkeletonControl

动画模型中除了AnimControl以外，还有一个SkeletonControl，这两个控制器共用一套骨骼（Skeleton）。AnimControl的作用是根据动画数据来计算骨骼的姿态，而SkeletonControl的作用是根据“骨骼蒙皮”数据来计算模型的变形。

从模型中获取SkeletonControl非常简单，只需要一行代码：

    private Node jaime;
    private SkeletonContorl sc;

    public void simpleInitApp() {
        ...
        jaime = (Node)assetManager.loadModel("Models/Jaime/Jaime.j3o");
        // 获得SkeletonControl
        sc = jaime.getControl(SkeletonControl.class);
        ...
    }

我们可以通过`Control#setEnable(boolean isEnabled)`方法来启动/禁用控制器。在样例代码中，用户按键F2可以开关SkeletonControl的功能。

    /**
     * 启用/禁用SkeletonContorl
     */
    private void toggleSkeletonControl() {
        // sc.setEnabled(!sc.isEnable());
        
        if (sc.isEnabled()) {
            sc.setEnabled(false);
        } else {
            sc.setEnabled(true);
        }
    }

如果禁用SkeletonControl，AnimControl依然会计算骨骼的姿态，但是模型缺不会再跟随骨骼运动了。

![骨骼错位](/content/images/2017/05/Jaime_Skeleton_disabled.png)

jME3支持硬件蒙皮，即利用GPU来计算模型网格和骨骼的对应关系，这会显著提升游戏画面的刷新率。硬件蒙皮功能默认是开启的，然而有时你可能更倾向于关闭它，例如在硬件不支持这个功能时。

    // 关闭硬件蒙皮
    sc.setHardwareSkinningPreferred(false);

关闭硬件蒙皮的话，骨骼和网格的数据将由CPU来运算。随着模型顶点数和骨骼数量的增加，这个过程会变得相当耗时，以至于在一些中低端的智能手机上根本无法运行。

降低模型面数、降低骨骼复杂度，是游戏性能优化时常用的手段。

#### Skeleton

无论是SkeletonControl还是AnimControl，都提供了查询骨骼的方法，例如：

    Skeleton ske = sc.getSkeleton();

Skeleton内部使用一个数组保存了所有的Bone对象，可以通过Bone的名称查询到骨骼对象或骨骼的索引号。

    Bone bone = ske.getBont("hand.L");
    int index = ske.getBontIndex("hand.L");

每个Bone对象都有一个特定的名字，同时保存着它相对于父节点的位移（Postion）、旋转（Rotaion）、缩放（Scale）等数据。SkeletonControl正是通过这些数据和蒙皮数据来计算模型姿态的。

我们可以在控制台下直接打印Skeleton内部的骨骼名称，以及它们之间的层次关系。

    System.out.println(sc.getSkeleton());

Jaime的骨骼打印出来是这样的：

	Skeleton - 108 bones, 5 roots
	TailCtrl.004 bone
	TailCtrl.003 bone
	TailCtrl.002 bone
	TailCtrl.001 bone
	Root bone
	-IKhand.L bone
	-knee.L bone
	-knee.R bone
	-IKhand.R bone
	-Elbow.L bone
	-Elbow.R bone
	-hips bone
	--pelvis bone
	---tail.001 bone
	----tail.002 bone
	-----tail.003 bone
	------tail.004 bone
	-------tail.005 bone
	--------tail.006 bone
	---------tail.007 bone
	----------tail.008 bone
	-----------tail.009 bone
	------------tail.010 bone
	---thigh.L bone
	----shin.L bone
	-----foot.L bone
	------toe.L bone
	---thigh.R bone
	----shin.R bone
	-----foot.R bone
	------toe.R bone
	--spine bone
	---ribs bone
	----neck bone
	-----head bone
	------IKjawTarget bone
	------jaw bone
	------LipBottom.R bone
	------LipTop.R bone
	------LipSide.R bone
	------eyebrow.01.R bone
	------eyebrow.02.R bone
	------eyebrow.03.R bone
	------Cheek.R bone
	------LipBottom.L bone
	------LipTop.L bone
	------LipSide.L bone
	------eyebrow.01.L bone
	------eyebrow.02.L bone
	------eyebrow.03.L bone
	------Cheek.L bone
	------EyeLidTop.R bone
	------EyeLidBottom.R bone
	------EyeLidTop.L bone
	------EyeLidBottom.L bone
	------SightTarget bone
	-------IKeyeTarget.R bone
	-------IKeyeTarget.L bone
	------eye.L bone
	------eye.R bone
	----shoulder.L bone
	-----upper_arm.L bone
	------forearm.L bone
	-------hand.L bone
	--------palm.01.L bone
	---------finger_index.01.L bone
	----------finger_index.02.L bone
	-----------finger_index.03.L bone
	---------thumb.01.L bone
	----------thumb.02.L bone
	-----------thumb.03.L bone
	--------palm.04.L bone
	---------finger_pinky.01.L bone
	----------finger_pinky.02.L bone
	-----------finger_pinky.03.L bone
	--------palm.02.L bone
	---------finger_middle.01.L bone
	----------finger_middle.02.L bone
	-----------finger_middle.03.L bone
	--------palm.03.L bone
	---------finger_ring.01.L bone
	----------finger_ring.02.L bone
	-----------finger_ring.03.L bone
	----shoulder.R bone
	-----upper_arm.R bone
	------forearm.R bone
	-------hand.R bone
	--------palm.01.R bone
	---------finger_index.01.R bone
	----------finger_index.02.R bone
	-----------finger_index.03.R bone
	---------thumb.01.R bone
	----------thumb.02.R bone
	-----------thumb.03.R bone
	--------palm.04.R bone
	---------finger_pinky.01.R bone
	----------finger_pinky.02.R bone
	-----------finger_pinky.03.R bone
	--------palm.02.R bone
	---------finger_middle.01.R bone
	----------finger_middle.02.R bone
	-----------finger_middle.03.R bone
	--------palm.03.R bone
	---------finger_ring.01.R bone
	----------finger_ring.02.R bone
	-----------finger_ring.03.R bone
	-IKheel.L bone
	-IKheel.R bone

上文搭配此图食用更佳。

![骨骼结构](/content/images/2017/05/Jaime_Skeleton_Bone_Names.png)

### 骨骼调试器

SkeletonDebugger类可用于显示骨骼的形状。当然，前提是这个模型得绑定过骨骼。

首先，在代码顶部加入对SkeletonDebugger和Material的引用：

     import com.jme3.scene.debug.SkeletonDebugger;
     import com.jme3.material.Material;

然后，在simpleInitApp()中添加如下语句，使模型的骨骼可见。

    // 骨骼调试器
    private SkeletonDebugger sd;

    public void simpleInitApp() {
        ...
        /**
         * 创建一个SkeletonDebugger，用于显示骨骼的形状。
         */
        sd = new SkeletonDebugger("debugger", sc.getSkeleton());
        jaime.attachChild(sd);
        
        // 创建材质
        Material mat = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");
        mat.setColor("Color", ColorRGBA.Green);
        mat.getAdditionalRenderState().setDepthTest(false);// 禁用深度测试，实现光透视效果。
        sd.setMaterial(mat);
        ...
    }

对于一般的模型来说，这点代码就够了。不过Jaime的模型有点古怪，作者把它绕Y轴转了180°。直接按上述代码运行的话，会发现骨骼和模型的正面是相反的。

![Jaime的骨骼和正面相反](/content/images/2017/05/Jaime_Skeleton_backtoface.png)

为了使它的骨骼和模型姿态保持一致，需要额外加一点点代码。这个代码不具有通用性，只是为了解决Jaime的问题。
        
        // 将SkeletonDebugger的姿态与Jaime的几何体同步。
        Spatial child = jaime.getChild(0);
        sd.setLocalTransform(child.getLocalTransform());

结果如下：

![把Jaime的骨骼“掰正”](/content/images/2017/05/Jaime_Skeleton_backtoface2.png)

SkeletonDebugger本质上是一个Spatial，因此可以使用attachChild方法把它添加到场景中。在样例代码中，这个SkeletonDebugger对象的父节点是Jaime。

用户按下F1键，可以显示或隐藏SkeletonDebugger。

    /**
     * 显示或隐藏SkeletonDebugger
     */
    private void toggleSkeletonDebugger() {
        if (sd.getParent() != null) {
            sd.removeFromParent();
        } else {
            jaime.attachChild(sd);
        }
    }

### 绑定附件

在很多3D游戏中，都会出现让角色“拿起”道具的情况，例如更换角色手冢的武器。在jME3中，我们可以通过`SkeletonControl#getAttachmentNode(String boneName)`来获取于某个骨头绑定的节点，然后把道具的模型添加到这个节点中，以此实现绑定附件的功能呢。

在样例代码中，我是用红黄两色的圆柱体来给Jaime制作了一根棍子，用这根棍子充当Jaime的道具。

把这根棍子绑定到Jaime右手的代码是这样的。

    // Jaime的右手
    private Node rightHand;
    // Jaime的棍子
    private Spatial stick;

    public void simpleInitApp() {
        /**
         * 绑定附件
         */
        stick = createAttachment();
        
        rightHand = sc.getAttachmentsNode("hand.L");
        rightHand.attachChild(stick);
        ...
    }

注意，SkeletonControl是根据骨骼的名字来查找附件节点的。如果骨骼的命名不规范，会给程序带来很大的麻烦。Jaime这个模型有点小bug，"hand.L"应该是左手骨骼的命名，但实际上它却代表右手。

由于附件模型只是和骨骼的节点绑定，它的运动并不需要蒙皮计算。因此即使禁用了SkeletonControl，附件模型依然会随着Skeleton的运动而运动。

正常运行结果：

![绑定附件](/content/images/2017/05/Jaime_Attachment1.png)

禁用SkeletonControl后，角色静止不动，但是棍子还在自顾自地运动。

![绑定附件2](/content/images/2017/05/Jaime_Attachment2.png)

由于附件节点本质上就是个Node，因此我们可以attach/dettach等方法来操纵场景。在样例代码中，用户按F3可以移除或添加Jaime右手的附件。

    /**
     * 移除或添加Jaime右手的附件
     */
    private void toggleAttachment() {
        if (stick.getParent() != null) {
            stick.removeFromParent();
        } else {
            rightHand.attachChild(stick);
        }
            
    }

### 角色换装

既然已经说到了这里，那么顺便再稍微介绍一下如何实现角色换装。

角色身上的武器、盾牌、戒指、挂件之类的装备不需要按下面的方法干，直接找准骨骼绑定附件即可。

穿戴类装备一般要经过这么几步：

1. 为游戏角色建立一套**[重点]标准身形和标准骨骼**，并绑定骨骼。
2. 根据可换装的部位，把标准身形**[重点]切割成若干个部件**，例如：头发、头、脖子、手臂、躯干、腿、足等。
3. 针对每个部位，制作不同的装备模型。制作时，注意部位之间的**[重点]接缝要对齐**，避免断手断脚的情况。
4. 制作好的装备模型，**[重点]要和标准骨骼重新绑定**。
5. 在程序中，更换装备后要**[重点]替换对应部位的模型**，**[重点]并重新和骨骼绑定**。

总结一下：把角色模型肢解，然后复用同一个SkeletonControl和AnimControl。

更详细教程请阅读：[3D游戏中角色的换装原理-落樱之剑实例图文详细剖析](http://www.huliqing.name/article/articleId=41) 

## 运动路径

运动控制器（`MotionEvent`）可用于遥控物体和摄像机，让它们沿着固定轨迹移动，移动的路径使用MotionPath来描述。`MotionPath`中保存了轨迹上的关键路径点（WayPoints），具体的路线将由算法来生成。路径可以是直线(`SplineType.Linear`)，也可以是平滑的曲线(`SplineType.CatmullRom`)，一切都看你的需求。

![MotionPath](/content/images/2017/05/MotionPath.png)

官方案例：

* 控制物体运动： [TestMotionPath.java
](https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-examples/src/main/java/jme3test/animation/TestMotionPath.java)
* 控制摄像机运动：[TestCameraMotionPath.java](https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-examples/src/main/java/jme3test/animation/TestCameraMotionPath.java)

### 样例代码

按键说明：

    P: 隐藏/显示路线
    I: 切换路线类型（直线/曲线）
    空格：开始/停止移动
    U: 增加曲率（仅曲线模式下有效）
    J: 减少曲率（仅曲线模式下有效）

代码如下：

	package net.jmecn.anim;
	
	import com.jme3.animation.AnimChannel;
	import com.jme3.animation.AnimControl;
	import com.jme3.app.SimpleApplication;
	import com.jme3.cinematic.MotionPath;
	import com.jme3.cinematic.events.MotionEvent;
	import com.jme3.cinematic.events.MotionEvent.Direction;
	import com.jme3.input.KeyInput;
	import com.jme3.input.controls.ActionListener;
	import com.jme3.input.controls.KeyTrigger;
	import com.jme3.light.AmbientLight;
	import com.jme3.light.DirectionalLight;
	import com.jme3.math.ColorRGBA;
	import com.jme3.math.Spline.SplineType;
	import com.jme3.math.Vector3f;
	import com.jme3.scene.Spatial;
	
	/**
	 * 演示运动路径
	 * @author yanmaoyuan
	 *
	 */
	public class TestMotion extends SimpleApplication {
	
		public static void main(String[] args) {
			// 启动程序
			TestMotion app = new TestMotion();
			app.start();
		}
		
		/**
		 * 缩放系数
		 */
		final static float SCALE_FACTOR = 5f;
		
	    private Spatial player;// 玩家
	    private Spatial stage;// 舞台
	    
	    private boolean active = true;
	    private boolean playing = false;
	    
	    private MotionPath motionPath;
	    private MotionEvent motionControl;
	    
		@Override
		public void simpleInitApp() {
	
			flyCam.setMoveSpeed(10);
	
			initLights();
			
			initInputs();
			
			initMotionPath();
			
			stage = assetManager.loadModel("Models/Stage/Stage.j3o");
			stage.scale(SCALE_FACTOR);
			rootNode.attachChild(stage);
			
			player = assetManager.loadModel("Models/Jaime/Jaime.j3o");
			rootNode.attachChild(player);
			
			AnimControl ac = player.getControl(AnimControl.class);
			AnimChannel channel = ac.createChannel();
			channel.setAnim("Run");
			channel.setSpeed(2f);
			
			motionControl = new MotionEvent(player, motionPath);
	        
			// 在行进中，注视某个位置
	//		Vector3f position = new Vector3f(0, 0, 0);
	//		motionControl.setLookAt(position, Vector3f.UNIT_Y);
	//		motionControl.setDirectionType(Direction.LookAt);
			
			// 在行进中，面朝固定方向
	//		Quaternion rotation = new Quaternion().fromAngleNormalAxis(-FastMath.HALF_PI, Vector3f.UNIT_Y);
	//		motionControl.setRotation(rotation);
	//		motionControl.setDirectionType(Direction.Rotation);
	
			// 在行进中，面朝前进方向
			motionControl.setDirectionType(Direction.Path);
			
			// 设置走完全程所需的时间（单位：秒）。
			motionControl.setInitialDuration(10f);
		}
	
		/**
		 * 初始化光源
		 */
		private void initLights() {
			AmbientLight ambient = new AmbientLight(new ColorRGBA(0.4f, 0.4f, 0.4f, 1f));
			
			DirectionalLight sun = new DirectionalLight();
			sun.setDirection(new Vector3f(0.6486864f, -0.72061276f, 0.24479222f));
			sun.setColor(new ColorRGBA(0.8f, 0.8f, 0.8f, 1f));
			
			rootNode.addLight(ambient);
			rootNode.addLight(sun);
		}
		
		/**
		 * 路径点数据
		 */
		final static float[] WayPoints = {
			1.3660254f, -1f, 0.5f,// 0
			0.8660254f, -1f, 0.5f,// 1
			-0.8660254f, 0f, 0.5f,// 2
			-1.3660254f, 0f, 0.5f,// 3
			-1.3660254f, 0f, -0.5f,// 4
			-0.8660254f, 0f, -0.5f,// 5
			0.8660254f, 1f, -0.5f,// 6
			1.3660254f, 1f, -0.5f// 7
		};
		
		/**
		 * 建造路径点
		 */
		private void initMotionPath() {
			
			motionPath = new MotionPath();
	
			// 路径点个数
			int count = WayPoints.length / 3;
			
			for(int i = 0; i < count; i++) {
				// 按比例放大顶点坐标
				int n = i * 3;
				float x = SCALE_FACTOR * WayPoints[n];
				float y = SCALE_FACTOR * WayPoints[n + 1];
				float z = SCALE_FACTOR * WayPoints[n + 2];
				
				motionPath.addWayPoint(new Vector3f(x, y, z));
			}
			
			motionPath.setPathSplineType(SplineType.Linear);
			
			motionPath.enableDebugShape(assetManager, rootNode);
		}
		
		/**
		 * 初始化输入
		 */
		private void initInputs() {
			inputManager.addMapping("display_hidePath", new KeyTrigger(KeyInput.KEY_P));
	        inputManager.addMapping("SwitchPathInterpolation", new KeyTrigger(KeyInput.KEY_I));
	        inputManager.addMapping("tensionUp", new KeyTrigger(KeyInput.KEY_U));
	        inputManager.addMapping("tensionDown", new KeyTrigger(KeyInput.KEY_J));
	        inputManager.addMapping("play_stop", new KeyTrigger(KeyInput.KEY_SPACE));
	        ActionListener acl = new ActionListener() {
	
	            public void onAction(String name, boolean keyPressed, float tpf) {
	                if (name.equals("display_hidePath") && keyPressed) {
	                    if (active) {
	                        active = false;
	                        motionPath.disableDebugShape();
	                    } else {
	                        active = true;
	                        motionPath.enableDebugShape(assetManager, rootNode);
	                    }
	                }
	                
	                if (name.equals("play_stop") && keyPressed) {
	                    if (playing) {
	                        playing = false;
	                        motionControl.stop();
	                    } else {
	                        playing = true;
	                        motionControl.play();
	                    }
	                }
	
	                if (name.equals("SwitchPathInterpolation") && keyPressed) {
	                    if (motionPath.getPathSplineType() == SplineType.CatmullRom){
	                        motionPath.setPathSplineType(SplineType.Linear);
	                    } else {
	                        motionPath.setPathSplineType(SplineType.CatmullRom);
	                    }
	                }
	
	                if (name.equals("tensionUp") && keyPressed) {
	                    motionPath.setCurveTension(motionPath.getCurveTension() + 0.1f);
	                    System.err.println("Tension : " + motionPath.getCurveTension());
	                }
	                if (name.equals("tensionDown") && keyPressed) {
	                    motionPath.setCurveTension(motionPath.getCurveTension() - 0.1f);
	                    System.err.println("Tension : " + motionPath.getCurveTension());
	                }
	
	
	            }
	        };
	
	        inputManager.addListener(acl, "display_hidePath", "play_stop", "SwitchPathInterpolation", "tensionUp", "tensionDown");
		}
	
	}


### 路径点

当拍摄电视/电影时，导演告诉演员要走到哪里（例如在舞台的地板上画标记）。摄影师经常把摄像机安装在轨道上（所谓的轨道车），这样他们可以更容易地跟拍复杂的场景。

![](/content/images/2017/05/dolly_track.jpg)

在jME3中，你可以使用MotionPath来指定物体或摄像机的运动路径。当游戏画面刷新时，MotionPath会自动移动物体的位置，让它从一个点运动到下一个点。

* **路径点(way point)**代表路径上的一个位置。
* **运动路径(motion path)**是一系列路径点的集合。

路径的最终形状，是根据路径点的坐标，通过线性插值或[Catmull-Rom](http://www.mvps.org/directx/articles/catmull/)曲线插值算法生成的。

### 创建路径

实例化一个MotionPath对象，然后将路径点的坐标添加进去。

    /**
     * 路径点数据
     */
    final static float[] WayPoints = {
            1.3660254f, -1f, 0.5f, // 0
            0.8660254f, -1f, 0.5f, // 1
            -0.8660254f, 0f, 0.5f, // 2
            -1.3660254f, 0f, 0.5f, // 3
            -1.3660254f, 0f, -0.5f, // 4
            -0.8660254f, 0f, -0.5f, // 5
            0.8660254f, 1f, -0.5f, // 6
            1.3660254f, 1f, -0.5f// 7
    };

    /**
     * 建造路径点
     */
    private void initMotionPath() {
        motionPath = new MotionPath();

        // 路径点个数
        int count = WayPoints.length / 3;
        for (int i = 0; i < count; i++) {
            int n = i * 3;
            float x = WayPoints[n];
            float y = WayPoints[n + 1];
            float z = WayPoints[n + 2];
            motionPath.addWayPoint(new Vector3f(x, y, z));
        }
        // 设置路径为直线
        motionPath.setPathSplineType(SplineType.Linear);
        // 显示路径
        motionPath.enableDebugShape(assetManager, rootNode);
    }

你可以使用下列方法来配置运动路径。

<table>
  <tr><th>方法</th><th>用途</th></tr>
  <tr>
    <td>path.setCycle(boolean  isCycle)</td>
    <td>这是路径是否形成闭环。参数为true时，将形成闭合路径；为false时，起点和终点将会分离。</td>
  </tr>
  <tr>
    <td>path.addWayPoint(Vector3f vector)</td>
    <td>向路径中添加一个路径点，添加的顺序即路线行进的顺序。</td>
  </tr>
  <tr>
    <td>path.removeWayPoint(Vector3f vector)<br/>
removeWayPoint(int index)</td>
    <td>从路径中移除一个路径点。你可以指定要移除的坐标对象或者索引。</td>
  </tr>
  <tr>
    <td>path.setCurveTension(float curveTension)</td>
    <td>设置曲线的舒张系数（仅对Catmull-Rom曲线有意义）。值为 0.0f 时，形成的将会是直线；值为 1.0f 时，形成的是一条非常圆滑的曲线。</td>
  </tr>
  <tr>
    <td>path.getNbWayPoints()</td>
    <td>返回路径点的数量。</td>
  </tr>
  <tr>
    <td>path.enableDebugShape(AssetManager assetManager, Node rootNode)</td>
    <td>将运动路径显示出来，用于开发调试。</td>
  </tr>
  <tr>
    <td>path.disableDebugShape()</td>
    <td>隐藏路径，一般用于项目发布时。</td>
  </tr>
</table>

### 控制物体运动

MotionPath要和MotionEvent配合使用，才能控制物体运动。MotionEvent负责控制物体沿着MotionPath定义的路径移动。

若已有运动路径 path ，物体 player，欲使player沿着path运动，则需要实例化一个MotionEvent对象。

    MotionPath path = ..
    Spatial player = ..

    MotionEvent motionControl = new MotionEvent(player, path);

如果想移动摄像机，可以先创建一个CameraNode，把摄像机绑定到这个CameraNode上，再用MotionEvent来控制CameraNode沿着path运动。

    MotionPath path = ..
    CameraNode camNode = new CameraNode("Motion cam", cam);
    camNode.setControlDir(ControlDirection.SpatialToCamera);

    MotionEvent motionControl = new MotionEvent(camNode , path);

你可以通过下列方法来配置MotionEvent的工作方式。

<table>
  <tr><th>方法</th><th>用途</th></tr>
  <tr>
    <td>motionControl.setDirectionType(Direction dir)</td>
    <td>设置物体在运动时，正面应该朝向什么方向。例如Direction.Path可以让物体面朝路径的前方。</td>
  </tr>
  <tr>
    <td>motionControl.setRotation(Quaternion rotation);</td>
    <td>若希望物体面朝向固定方向（Direction.Rotation），则需要调用此方法来设置旋转方向。</td>
  </tr>
  <tr>
    <td>motionControl.setLookAt(Vector3f position, Vector3f up)</td>
    <td>若希望物体始终“凝视”某个位置（Direction.LookAt），则需要条用此方法来设置“凝视”的坐标点和摄像机的顶部方向（常用Vector3f.UNIT_Y）。</td>
  </tr>
  <tr>
    <td>motionControl.setInitialDuration(float time)</td>
    <td>若希望物体在固定时间内走完全程，则需要调用此方法来设置所需的时间（单位：秒）。</td>
  </tr>
  <tr>
    <td>motionControl.setLoopMode(LoopMode mode)</td>
    <td>设置动画的循环模式，用法和AnimChannel的LoopMode是一样的。DontLoop表示不循环，Loop表示顺序循环，Cycle表示往复循环。</td>
  </tr>
</table>

### 运动路径监听器

通过MotionPathListener可以监控物体在MotionPath上的运动状态。当物体到达某个路径点时，监听器的`onWayPointReach()`方法就会被触发，并接收到MotionEvent对象和路径点的索引号（wayPointIndex）作为参数。

下面的例子比较简单，每当物体到达一个路径点时，就把这个路径点的索引号打印出来。在实际的游戏中，你可以在特定的路径点触发一些行为，比如播放音乐、开门等。

    path.addListener( new MotionPathListener() {
      public void onWayPointReach(MotionEvent control, int wayPointIndex) {
        if (path.getNbWayPoints() == wayPointIndex + 1) {
          println(control.getSpatial().getName() + " 已到达终点。 ");
        } else {
          println(control.getSpatial().getName() + " 已到达路径点： " + wayPointIndex);
        }
      }
    });

## 剧情动画

电影化（Cinematics）工具可以让你制作剧情动画。你可以遥控3D场景中的多个角色，让他们运动到指定位置、播放音频文件、播放动画、弹出字幕等等。

样例代码：[TestCinematic.java](https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-examples/src/main/java/jme3test/animation/TestCinematic.java)

Cinematics的用法其实并不复杂，重要的是先设计好动画的脚本。由于目前使用Cinematics的人并不多，本章教程也就到此为止，如果你对jME3的电影化工具感兴趣，可以本文后面的评论区留言。

官方教程（英文）：

https://jmonkeyengine.github.io/wiki/jme3/advanced/cinematics.html

https://hub.jmonkeyengine.org/t/cinematics-system-for-jme3/14636

https://hub.jmonkeyengine.org/t/scripting-in-jme3-with-groovy-all-about-it/26516

https://jmonkeyengine.github.io/wiki/jme3/scripting.html

## 更多样例代码

如果你希望详细了解这些各种动画相关的API，jME官方提供了很多关于动画的样例，诸如：

* [jme3test.helloworld.HelloAnimation][HelloAnimation]
* [jme3test.model.anim][jme3test.model.anim]
 * [TestBlenderAnim][TestBlenderAnim] 播放Blender动画
 * [TestBlenderObjectAnim][TestBlenderObjectAnim] 播放Blender动画
 * [TestOgreAnim][TestOgreAnim] 播放Orge动画
 * [TestOgreComplexAnim][TestOgreComplexAnim] 播发Orge复杂动画
 * [TestAnimationFactory][TestAnimationFactory] 通过AnimationFactory来生成动画。
 * [TestSpatialAnim][TestSpatialAnim] 使用AnimationFactory生成动画，控制Spatial运动
 * [TestCustomAnim][TestCustomAnim] 用代码来创建自定义骨骼动画。
 * [TestAttachmentsNode][TestAttachmentsNode] 演示物体与骨骼的绑定，例如让玩家手持武器。
 * [TestAnimBlendBug][TestAnimBlendBug]
 * [TestHWSkinning][TestHWSkinning] 演示硬件蒙皮
 * [TestSkeletonControlRefresh][TestSkeletonControlRefresh]
 * [TestModelExportingCloning][TestModelExportingCloning]
* [jme3test.animation][jme3test.animation]
 * [TestMotionPath][TestMotionPath] 演示使用MotionPath来控制物体的运动。
 * [TestCameraMotionPath][TestCameraMotionPath] 演示使用MotionPath来控制摄像机的移动。
 * [TestCinematic][TestCinematic] 演示“电影化”
 * [SubtitleTrack][SubtitleTrack] 演示自定义电影事件。
 * [TestJaime][TestJaime] 使用“电影化”工具，让Jaime来表演一段动作。

----

[HelloAnimation]:https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-examples/src/main/java/jme3test/helloworld/HelloAnimation.java
[jme3test.model.anim]:https://github.com/jMonkeyEngine/jmonkeyengine/tree/master/jme3-examples/src/main/java/jme3test/model/anim
[TestSpatialAnim]:https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-examples/src/main/java/jme3test/model/anim/TestSpatialAnim.java
[TestBlenderAnim]:https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-examples/src/main/java/jme3test/model/anim/TestBlenderAnim.java
[TestBlenderObjectAnim]:https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-examples/src/main/java/jme3test/model/anim/TestBlenderObjectAnim.java
[TestOgreAnim]:https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-examples/src/main/java/jme3test/model/anim/TestOgreAnim.java
[TestOgreComplexAnim]:https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-examples/src/main/java/jme3test/model/anim/TestOgreComplexAnim.java
[TestCustomAnim]:https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-examples/src/main/java/jme3test/model/anim/TestCustomAnim.java
[TestAnimationFactory]:https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-examples/src/main/java/jme3test/model/anim/TestAnimationFactory.java
[TestAttachmentsNode]:https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-examples/src/main/java/jme3test/model/anim/TestAttachmentsNode.java
[TestAnimBlendBug]:https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-examples/src/main/java/jme3test/model/anim/TestAnimBlendBug.java
[TestHWSkinning]:https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-examples/src/main/java/jme3test/model/anim/TestHWSkinning.java
[TestSkeletonControlRefresh]:https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-examples/src/main/java/jme3test/model/anim/TestSkeletonControlRefresh.java
[TestModelExportingCloning]:https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-examples/src/main/java/jme3test/model/anim/TestModelExportingCloning.java
[jme3test.animation]:https://github.com/jMonkeyEngine/jmonkeyengine/tree/master/jme3-examples/src/main/java/jme3test/animation
[TestJaime]:https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-examples/src/main/java/jme3test/animation/TestJaime.java
[TestMotionPath]:https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-examples/src/main/java/jme3test/animation/TestMotionPath.java
[TestCameraMotionPath]:https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-examples/src/main/java/jme3test/animation/TestCameraMotionPath.java
[SubtitleTrack]:https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-examples/src/main/java/jme3test/animation/SubtitleTrack.java
[TestCinematic]:https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-examples/src/main/java/jme3test/animation/TestCinematic.java
