# 第十章：图形用户界面

## 基本概念

界面设计人员经常把UI、GUI等词汇挂载嘴边，那么到底什么是GUI？另外你可能还听说过HUD，HUD又是什么？

首先上结论：`UI > GUI > HUD`

### UI

UI是User Interface的缩写，也是User Interaction的缩写。UI涵盖一切用户和机器交互的内容，如果把计算机程序抽象为“输入-处理-输出”这个过程，那么UI负责的就是“输入”和“输出”。

用户交互包含了用户体验、用户接口、交互设计等多个方面。在游戏开发前，UI设计就要想好游戏的流程，玩家要如何玩这个游戏，游戏过程中又会得到什么反馈。对于一个典型的格斗游戏来说，它的UI设计可能是这样的：

* 玩家通过手柄来玩游戏，程序获得输入后，就可以控制游戏角色产生对应的招式；
* 屏幕的上方分别显示玩家和BOSS的血条，血条会在角色受到伤害时减少；
* 玩家角色攻击BOSS时，BOSS头上会浮现伤害数字，以及“格挡”、“暴击”、“躲闪”等文字；
* 当玩家的攻击被BOSS格挡时，程序驱动手柄产生震颤，让玩家获得一个强烈的反馈；
* ……

除了计算机程序以外，UI设计还应用于其他领域，例如电视的遥控器；轿车的方向盘、油门、刹车、各种仪表等。有的UI甚至不是为人类用户设计的，比如数据库管理系统的SQL解释器，它的UI是提供给客户端程序使用的。

UI是一个广义的概念，当它应用于游戏UI设计时，主要指的就是各种游戏控制器以及图形界面的设计。

### GUI

GUI是Graphics User Interface的缩写，意为图形用户接口。GUI是UI的子集，它的作用是以图形、画面的方式进行用户交互。游戏中的血条、地图、背包等都属于GUI的范畴。

事实上，网站设计师、桌面应用设计师、App设计师等工作主要都是GUI设计。所以很多时候设计师们在谈到UI的时候，其实指的是GUI。

GUI设计通常包含画面效果和交互功能两个方面。GUI的画面效果主要是平面设计工作，成果一般是文字、原画、效果图等静态内容，有时还会包含一些动画特效；GUI的交互功能则与程序逻辑息息相关，比如游戏菜单、在小地图上标注关键地点等，一般需要通过编写代码来实现。

### HUD

HUD是Head Up Display的缩写，指的是抬头平视显示设计，简称HUD设计。游戏的HUD设计，是从军事术语中衍生而来的，即为作战设备上的“平视显示仪及其子系统”，其前身是光学观瞄仪。

抬头显示有什么作用？举一例子，战斗机驾驶员的主要注意力都应该集中在搜索敌机和瞄准上，如果驾驶员想看看飞行高度、水平方向、雷达、燃料量、载弹量等参数，就必须低头看仪表。如果能够直接在头盔屏幕上显示这些数值，驾驶员就不需要低头了。

下图为游戏“皇牌空战6”的HUD。

![皇牌空战6](/content/images/2017/05/hud.jpg)

HUD是游戏的辅助系统，它的设计重点是信息整合和合理摆放，以此来满足玩家最直接的需求。玩家通过HUD可以随时注意到游戏中最重要的信息（诸如血量、时间、比分等），而且不需要暂停游戏去查看别的窗口。

总的来说，HUD应该算是GUI的子集，但HUD和GUI设计的关注点并不一样。GUI关注的是整个图形界面的交互设计，HUD关注的则是通过合理的信息整合来辅助用户游戏。

## jME3的GUI

jME3是一个3D游戏引擎，3D环境下的GUI和2D有很大的不同。2D游戏可以通过图形API直接在窗口上绘图，而3D游戏不可以。只有2D环境下才有矩形、圆形、图片等概念，3D环境多了一个维度，变成了方块、球体、圆柱等多边形网格。

在jME3中，我们无法直接在屏幕上绘制一张图片，但是却可以通过一些方法来欺骗玩家的眼睛。最常用的方式是：先创建一个四边形(Quad)几何体，然后把要显示的图片作为纹理贴图应用到这个四边形上。


作为例子，我随手画了一张界面白板图，将其存放在项目的resource目录中，路径为`Interface/Gui/pic.png`。

![界面白板图](/content/images/2017/05/pic.png)

下面的代码演示了如何在jME3中显示这张“图片”。

    package net.jmecn.gui;
    
    import com.jme3.app.SimpleApplication;
    import com.jme3.material.Material;
    import com.jme3.math.ColorRGBA;
    import com.jme3.math.Vector3f;
    import com.jme3.scene.Geometry;
    import com.jme3.scene.Spatial;
    import com.jme3.scene.debug.Arrow;
    import com.jme3.scene.shape.Quad;
    import com.jme3.texture.Texture;
    
    /**
     * 伪装图片
     * @author yanmaoyuan
     *
     */
    public class FakePicture extends SimpleApplication {
    
        public static void main(String[] args) {
            // 启动游戏
            FakePicture app = new FakePicture();
            app.start();
        }
    
        @Override
        public void simpleInitApp() {
    
            // 创建X、Y、Z方向的箭头，作为参考坐标系。
            createArrow(new Vector3f(5, 0, 0), ColorRGBA.Green);
            createArrow(new Vector3f(0, 5, 0), ColorRGBA.Red);
            createArrow(new Vector3f(0, 0, 5), ColorRGBA.Blue);
    
            // 加载“图片”
            Spatial pic = getPicture();
    
            // 将“图片”添加到场景图中
            rootNode.attachChild(pic);
        }
    
        /**
         * 创建一个“图片”
         * 
         * @return
         */
        private Spatial getPicture() {
            // 创建一个四边形
            Quad quad = new Quad(4, 3);
            Geometry geom = new Geometry("Picture", quad);
    
            // 加载图片
            Texture tex = assetManager.loadTexture("Interface/Gui/pic.png");
    
            Material mat = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");
            mat.setTexture("ColorMap", tex);
    
            // 应用这个材质
            geom.setMaterial(mat);
    
            return geom;
        }
    
        /**
         * 创建一个箭头
         * 
         * @param vec3 箭头向量
         * @param color 箭头颜色
         */
        private void createArrow(Vector3f vec3, ColorRGBA color) {
            // 创建材质，设定箭头的颜色
            Material mat = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");
            mat.setColor("Color", color);
    
            // 创建几何物体，应用箭头网格。
            Geometry geom = new Geometry("arrow", new Arrow(vec3));
            geom.setMaterial(mat);
    
            // 添加到场景中
            rootNode.attachChild(geom);
        }
    
    }

作为一张“图片”，这个四边形不需要光照也应该能够正常显示，因此这里选用Unshaded.j3md材质。另外，为了突显3D环境，场景中还添加了3个箭头作为坐标系的参造物，其中绿色为X轴，红色为Y轴，蓝色为Z轴。

运行程序，效果如下。

![显示“图片”](/content/images/2017/05/FakePicture.png)

这个技巧在很多3D游戏中都有所应用，只是一种障眼法。使用这种技巧的游戏又被称为2.5D游戏。很多早期游戏中都使用了这种“纸片人”的技巧，[知乎：开发游戏时，有哪些欺骗人眼睛的技巧？](https://www.zhihu.com/question/41720683)

![](/content/images/2017/05/2_5_d_car.png)

![](/content/images/2017/05/2_5_d_car_pic.png)

### GuiNode

上文中的代码演示了如何在3D引擎中显示一张“图片”，然而这与我们想要制作的Gui还是有些不同。Gui通常都是固定在屏幕上不动的，而上面的“图片”会随着玩家晃动摄像机而“改变位置”。

在现实世界中我们要做到这些其实挺简单的，只要把“图片”直接贴在镜片上就好了！

![glass](/content/images/2017/05/glass.png)

VR头盔、Google Glass等设备虽然非常复杂，但是其本意都是直接在人眼处做文章。

![google project glass](/content/images/2017/05/google_glass.jpg)

说穿了，在现实世界中我们欺骗人的眼睛，在3D引擎中我们欺骗虚拟“摄像机”。为了让一个图片看起来更像是屏幕上的Gui，我们需要一些额外的障眼法，例如：

* “图片”应该始终正面面对摄像机，否则玩家一旦看到纸片的侧面就穿帮了。
* “图片”应该随摄像机同步运动，这样玩家就会觉得这个图片是（相对）静止的。
* “图片”在整个场景中应该是最后渲染的，而且不能被其他物体遮挡。

在jME3中很容易就可以做到这些。SimpleApplication提供了一个guiNode对象，它和rootNode一样都是场景的根节点，不同的是guiNode代表的是以摄像机为核心的场景。把几何体添加到guiNode中，就可以满足上面的3个条件。

修改一下上一章的代码，把rootNode改为guiNode，看看会发生什么。

    package net.jmecn.gui;
    
    import com.jme3.app.SimpleApplication;
    import com.jme3.material.Material;
    import com.jme3.math.ColorRGBA;
    import com.jme3.math.Vector3f;
    import com.jme3.scene.Geometry;
    import com.jme3.scene.Spatial;
    import com.jme3.scene.debug.Arrow;
    import com.jme3.scene.shape.Quad;
    import com.jme3.texture.Texture;
    
    /**
     * GuiNode
     * @author yanmaoyuan
     *
     */
    public class HelloGUI extends SimpleApplication {
    
        public static void main(String[] args) {
            // 启动程序
            HelloGUI app = new HelloGUI();
            app.start();
        }
    
        @Override
        public void simpleInitApp() {
            
            // 创建X、Y、Z方向的箭头，作为参考坐标系。
            createArrow(new Vector3f(5, 0, 0), ColorRGBA.Green);
            createArrow(new Vector3f(0, 5, 0), ColorRGBA.Red);
            createArrow(new Vector3f(0, 0, 5), ColorRGBA.Blue);
    
            // 加载“图片”
            Spatial pic = getPicture();
            
            // 将“图片”添加到GUI场景中
            guiNode.attachChild(pic);
        }
    
        /**
         * 创建一个“图片”
         * @return
         */
        private Spatial getPicture() {
            // 创建一个四边形
            Quad quad = new Quad(4, 3);
            Geometry geom = new Geometry("Picture", quad);
    
            // 加载图片
            Texture tex = assetManager.loadTexture("Interface/Gui/pic.png");
    
            Material mat = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");
            mat.setTexture("ColorMap", tex);
            
            // 应用这个材质
            geom.setMaterial(mat);
    
            return geom;
        }
        
        /**
         * 创建一个箭头
         * 
         * @param vec3  箭头向量
         * @param color 箭头颜色
         */
        private void createArrow(Vector3f vec3, ColorRGBA color) {
            // 创建材质，设定箭头的颜色
            Material mat = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");
            mat.setColor("Color", color);
    
            // 创建几何物体，应用箭头网格。
            Geometry geom = new Geometry("arrow", new Arrow(vec3));
            geom.setMaterial(mat);
    
            // 添加到GUI场景中
            guiNode.attachChild(geom);
        }
    }

运行程序，观察一下效果。

![guiNode](/content/images/2017/05/guiNode.png)

奇怪了，为什么什么都没了？坐标轴呢？图片呢？

![黑人问号](/content/images/2017/05/why.jpg)

先按下F5，关闭左下角的状态界面。然后把你的全部注意力都集中到画面的左下角，仔细观察那里有什么？

#### 屏幕坐标系

为了保护你的视力，我把左下角的画面放大8倍，然后再观察。

![small picture](/content/images/2017/05/small_pic.png)

是的，这个小玩意就是我们上文中出现过的坐标轴和图片。为什么它在guiNode中看起来比在rootNode小那么多？

这是因为guiNode和rootNode中的单位不一样。

rootNode中的数值并没有单位。通过一个透视摄像机去观察场景，看到的画面遵循近大远小的原则。“图片”的大小为4*3个单位，摄像机距离它不超过10个单位距离，因此看起来是比较大的。如果我们把摄像机退远之后再观察，这个物体看起来就会变小。下图是退后200个单位距离后看到的结果。

![远距离观察物体](/content/images/2017/05/far_away.png)

guiNode中的物体会被绘制到屏幕上，屏幕的单位是像素（pixel）。getPicture()方法中定义的“图片”宽和高分别是4和3，因此通过摄像机观察它时，它只有4x3像素这么大。而坐标轴的长度只有5，也就是5个像素，看起来也很短。

下面稍微修改一下代码，把图片的大小改为400*300，坐标轴的长度改为500。

![放大100倍后](/content/images/2017/05/100size.png)

通过上图可以得出结论：**在jME3中，屏幕采用XOY坐标系，单位为像素。坐标系原点O位于屏幕左下角，向右为X轴正方向，向上为Y轴正方向。**

#### 屏幕分辨率

由于guiNode实质上是在欺骗摄像机，因此我们可以通过摄像机来获得屏幕的一些参数，例如通过`cam.getWidth()`和`cam.getHeight()`就可以用来获得屏幕宽和高。

下面我们调整一下代码，让图片填充整个屏幕。

    /**
     * 创建一个“图片”
     * @return
     */
    private Spatial getPicture() {
        
        // 获得屏幕分辨率
        float width = cam.getWidth();
        float height = cam.getHeight();
        
        // 创建一个四边形
        Quad quad = new Quad(width, height);
        Geometry geom = new Geometry("Picture", quad);

        // 加载图片
        Texture tex = assetManager.loadTexture("Interface/Gui/pic.png");

        Material mat = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");
        mat.setTexture("ColorMap", tex);
        
        // 应用这个材质
        geom.setMaterial(mat);

        return geom;
    }

运行程序，效果如下：

![全屏显示](/content/images/2017/05/full_screen.png)

#### 图像居中

有了屏幕的分辨率和图像的分辨率，就可以计算坐标图像居中时的x、y坐标，然后通过`setLocalTranslation(float x, float y, float z)`或`move(float x, float y, float z)`方法来改变图片的相对位置。

    /**
     * 创建一个“图片”
     * @return
     */
    private Spatial getPicture() {
        
        // 获得屏幕分辨率
        float width = 400;
        float height = 300;
        
        // 创建一个四边形
        Quad quad = new Quad(width, height);
        Geometry geom = new Geometry("Picture", quad);

        // 计算图像居中的坐标
        float x = 0.5f * (cam.getWidth() - width);
        float y = 0.5f * (cam.getHeight() - height);
        geom.setLocalTranslation(x, y, 0);// 改变geom在guiNode中的相对位置
        
        // 加载图片
        Texture tex = assetManager.loadTexture("Interface/Gui/pic.png");

        Material mat = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");
        mat.setTexture("ColorMap", tex);
        
        // 应用这个材质
        geom.setMaterial(mat);

        return geom;
    }

效果如下：

![图像居中](/content/images/2017/05/center_screen.png)


#### Z坐标

还有别忘了jME3是一个3D引擎，它是有Z轴的。在屏幕中看不到Z轴，因为Z轴是垂直于摄像机的正面。Z坐标在屏幕坐标系中表示“深度”，它的正方向指向用户。当多个物体出现在guiNode中时，Z坐标大的会遮挡住Z坐标小的几何体。

通过`setLocalTranslation(float x, float y, float z)`或`move(float x, float y, float z)`方法的第3个参数就可以设置图片的Z坐标。下面我们把图片全屏显示，并把它的Z坐标改为-1，看看会发生什么。

    /**
     * 创建一个“图片”
     * @return
     */
    private Spatial getPicture() {
        
        // 获得屏幕分辨率
        float width = cam.getWidth();
        float height = cam.getHeight();
        
        // 创建一个四边形
        Quad quad = new Quad(width, height);
        Geometry geom = new Geometry("Picture", quad);
        
        // 将Z坐标设为-1
        geom.setLocalTranslation(0, 0, -1);

        // 加载图片
        Texture tex = assetManager.loadTexture("Interface/Gui/pic.png");

        Material mat = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");
        mat.setTexture("ColorMap", tex);
        
        // 应用这个材质
        geom.setMaterial(mat);

        return geom;
    }

效果如下：

![改变Z坐标](/content/images/2017/05/z_axis.png)

可以看到，当图片的Z坐标变成-1后，原本被图片挡住的坐标轴和左下角的状态界面又出现了。

实际上，jME3窗口左下角的状态界面本身就是guiNode中的物体，它的Z坐标是0。当图片的Z坐标为0时，它与坐标轴、状态界面的Z坐标是重合的。图片的Z坐标改为-1之后，状态界面就遮挡在图片前方了。

#### 正交投影

理论上来讲，guiNode中的物体z坐标值越大，表示这个物体离摄像机越近，所以才能遮挡住远处的物体。根据我们的日常经验，物体看起来是近大远小的。通过cam去观察rootNode中的物体时，也能证明这种经验。那么guiNode中会不会出现近大远小的情况？

事实上，无论将物体的Z坐标改为多少，它在屏幕上的大小都是一样的。这是因为引擎在绘制guiNode时采用了正交视图（也称为平行视图），这种模式是不遵循近大远小原则的。

将3D场景渲染到2D屏幕上的过程称为投影（Projection），常见的投影模式有2种，分别是透视投影（Perspective projection）和正交投影（Orthographic projection）。

透视投影的原理来源于小孔成像，与人眼的工作原理类似。jME3的摄像机默认采用的就是透视投影，因此场景中的物体看起来才更真实。

![透视投影](/content/images/2017/05/perspective.jpg)

正交投影又称为平行投影（Parallel projection），无论摄像机距离物体多远，物体的大小都不会改变。guiNode中的场景渲染采用的是正交投影，因此改变Z坐标并不会改变物体的大小。

![正交投影](/content/images/2017/05/ortho.png)

在jME3中，摄像机默认采用透视投影，可以通过`cam.setParallelProjection(true)`来开启摄像机的平行投影功能。

通过`cam.setFrustum(float near, float far, float left, float right, float top, float bottom)`可以改变摄像机的视锥参数。`near`和`far`这两个参数决定了摄像机能观察到的物体Z坐标范围，其他4个参数决定了视口的大小。

    @Override
    public void simpleInitApp() {
        flyCam.setMoveSpeed(50);
        cam.setLocation(new Vector3f(5.076195f, 5.100953f, 10.473327f));
        cam.setRotation(new Quaternion(-0.03069693f, 0.96919596f, -0.16531673f, -0.17996438f));
        
        // 改变视锥大小
        cam.setFrustum(-999, 999, 4, -4, 3, -3);
        // 开启平行投影
        cam.setParallelProjection(true);

        // 创建X、Y、Z方向的箭头，作为参考坐标系。
        createArrow(new Vector3f(5, 0, 0), ColorRGBA.Green);
        createArrow(new Vector3f(0, 5, 0), ColorRGBA.Red);
        createArrow(new Vector3f(0, 0, 5), ColorRGBA.Blue);

        // 加载“图片”
        Spatial pic = getPicture();

        // 将“图片”添加到场景图中
        rootNode.attachChild(pic);
    }

通过平行投影模式去观察rootNode中的物体，效果如下：

![正交投影](/content/images/2017/05/ParallelProjection.png)

透视投影下是这样的：

![透视投影](/content/images/2017/05/FakePicture.png)

写到这里，我想应该可以回答新手经常问的一个问题了：jME3如何做2D游戏？

答案很简单：把摄像机改为正交投影，然后用纸片来代替3D模型即可。

总的来说，一切都是障眼法。

### Picture

用“纸片”来表示图像是3D引擎中一种非常常用的障眼法，jME3甚至专门提供了一个Picture类。Picture是Geometry的子类，其内部直接引用了一个Quad网格。

它的作用和我们前面写的getPicture方法差不多，不过使用起来更加方便，只需要我们设置其纹理即可。如果图片包含alpha通道，则可以通过`setImage`或`setTexture`方法的第三个参数来开启透明模式。

我用gimp（一款开源的图形编辑工具）删除了pic.png图像中大部分的纯白色区域，为其添加了alpha通道，重新保存为`pic_with_alpha.png`文件。下面的代码演示了如何使用Picture类来显示图片。

    package net.jmecn.gui;
    
    import com.jme3.app.SimpleApplication;
    import com.jme3.light.AmbientLight;
    import com.jme3.light.DirectionalLight;
    import com.jme3.material.Material;
    import com.jme3.math.ColorRGBA;
    import com.jme3.math.FastMath;
    import com.jme3.math.Quaternion;
    import com.jme3.math.Vector3f;
    import com.jme3.renderer.queue.RenderQueue.ShadowMode;
    import com.jme3.scene.Geometry;
    import com.jme3.scene.shape.Box;
    import com.jme3.scene.shape.Quad;
    import com.jme3.ui.Picture;
    
    /**
     * Picture的用法。
     * 
     * @author yanmaoyuan
     *
     */
    public class HelloPicture extends SimpleApplication {
    
        public static void main(String[] args) {
            // 启动程序
            HelloPicture app = new HelloPicture();
            app.start();
        }
    
        @Override
        public void simpleInitApp() {
            // 初始化摄像机位置
            cam.setLocation(new Vector3f(9.443982f, 13.542627f, 8.93058f));
            cam.setRotation(new Quaternion(-0.015316938f, 0.9377411f, -0.34448296f, -0.041695934f));
    
            flyCam.setMoveSpeed(10);
    
            // 添加物体
            addObjects();
    
            // 添加光源
            addLights();
    
            // 添加图片
            addPicture();
    
            viewPort.setBackgroundColor(ColorRGBA.LightGray);
        }
    
        /**
         * 创建一个场景
         * 
         * @return
         */
        private void addObjects() {
            Material mat = new Material(assetManager, "Common/MatDefs/Light/Lighting.j3md");
    
            // 创建一个平面，把它作为地板，用来承载光影
            Geometry geom = new Geometry("Floor", new Quad(17, 17));
            geom.setMaterial(mat);
            geom.setShadowMode(ShadowMode.Receive);// 承载阴影
    
            geom.rotate(-FastMath.HALF_PI, 0, 0);
            rootNode.attachChild(geom);
    
            // 创造多个方块
            for (int y = 0; y < 10; y += 3) {
                for (int x = 0; x < 10; x += 3) {
                    geom = new Geometry("Cube", new Box(0.5f, 0.5f, 0.5f));
                    geom.setMaterial(mat);
                    geom.setShadowMode(ShadowMode.Cast);// 产生阴影
                    geom.move(x + 4, 0.5f, -y - 4);
                    rootNode.attachChild(geom);
                }
            }
        }
    
        /**
         * 添加光源
         */
        private void addLights() {
    
            // 定向光
            DirectionalLight sunLight = new DirectionalLight();
            sunLight.setDirection(new Vector3f(-1, -2, -3));
            sunLight.setColor(new ColorRGBA(0.2f, 0.2f, 0.2f, 1f));
    
            // 环境光
            AmbientLight ambientLight = new AmbientLight();
            ambientLight.setColor(new ColorRGBA(0.2f, 0.2f, 0.2f, 1f));
    
            // 将模型和光源添加到场景图中
            rootNode.addLight(sunLight);
            rootNode.addLight(ambientLight);
        }
    
        /**
         * 加载“图片”
         * 
         * @return
         */
        private void addPicture() {
            Picture pic = new Picture("picture");
    
            // 设置图片
            pic.setImage(assetManager, "Interface/Gui/pic_with_alpha.png", true);
    
            // 设置图片全屏显示
            pic.setWidth(cam.getWidth());
            pic.setHeight(cam.getHeight());
    
            // 将图片后移一个单位，避免遮住状态界面。
            pic.setLocalTranslation(0, 0, -1);
    
            guiNode.attachChild(pic);
        }
    }

运行结果如下：

![Picture](/content/images/2017/05/Picture.png)

需要注意的是，Picture类使用的材质并不是Unshaded.j3md，而是Gui.j3md。如果想要改变Picture的纹理，参数名并不是`ColorMap`而是`Texture`。

    // 加载纹理
    Texture tex = assetManager.loadTexture(...);
    // 改变Picture的贴图
    pic.getMaterial.setTexture("Texture", tex);

### 显示文字

通过前面的讨论，相信你已经对3D引擎中的GUI有了一定的概念。3D引擎中的一切都是3D几何体，文字也不例外。在GUI中显示文字，目前有三种方法。

1. 使用图片
2. 使用BitmapFont
3. 使用TTF字体

#### 使用图片

这种方法是最容易实现的，只要把文字直接画到到图片上就可以了。事实上本章代码中使用的pic.png图片里就有“头像”、“小地图”、“准星”等字样。

这种方法的优点是简单，缺点是无法改变文字的内容。在制作GUI时，通常会把窗口标题、按钮提示文字等制作成图片。

#### 使用BitmapFont

[Bitmap Font](http://www.angelcode.com/products/bmfont/)是AngleCode公司开发的一种技术，用于在3D场景中显示2D的文字。这种方法的原理是把要使用的文字做成一副图片，然后由程序根据文字的内容来动态选择图块，再绘制成一副新的图片。

用法：
首先你需要用一些工具来制作BitmapFont字体，将你需要的文字整合到一个png图片上，并生成对应的fnt文件。比如：

* Bitmap Font Generator

![Bitmap Font Generator](/content/images/2017/05/BMFontGenerator.png)

* Hiero

![Hiero](/content/images/2017/05/Hiero.png)

然后，你会得到一个.fnt文件和一个.png文件(如果字符量大，可能会有多个png文件)，这就是你得到的字体。这种字体不仅适用于JME3应用，cocos2d-x、libgdx等游戏引擎也可以使用这种字体。

得到BMFont字体文件后，接下来要在程序中加载这种字体，然后使用它们。JME 3.0实现了对Bitmap Font字体的支持，而且jme3-core.jar这个lib中已经内置了2套字体（可惜是都是英文字符，没有中文。）

![jME3自带字体](/content/images/2017/05/DefaultFonts.png)

当你每次启动JME3应用程序的时候，左下角显示的参数其实就是用`Interface/Fonts/Console.fnt`和`Interface/Fonts/Default.fnt`这2个字体。

![StatsAppState](/content/images/2017/05/StatsAppState.png)

JME3也提供了一个样例代码，教你怎么使用BitmapFont。

[jme3test.gui.TestBitmapFont](https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-examples/src/main/java/jme3test/gui/TestBitmapFont.java)

    package jme3test.gui;
    
    import com.jme3.app.SimpleApplication;
    import com.jme3.font.BitmapFont;
    import com.jme3.font.BitmapText;
    import com.jme3.font.LineWrapMode;
    import com.jme3.font.Rectangle;
    import com.jme3.input.KeyInput;
    import com.jme3.input.RawInputListener;
    import com.jme3.input.controls.ActionListener;
    import com.jme3.input.controls.KeyTrigger;
    import com.jme3.input.event.*;
    
    public class TestBitmapFont extends SimpleApplication {
    
        private String txtB =
        "ABCDEFGHIKLMNOPQRSTUVWXYZ1234567 890`~!@#$%^&*()-=_+[]\\;',./{}|:<>?";
    
        private BitmapText txt;
        private BitmapText txt2;
        private BitmapText txt3;
    
        public static void main(String[] args){
            TestBitmapFont app = new TestBitmapFont();
            app.start();
        }
    
        @Override
        public void simpleInitApp() {
            inputManager.addMapping("WordWrap", new KeyTrigger(KeyInput.KEY_TAB));
            inputManager.addListener(keyListener, "WordWrap");
            inputManager.addRawInputListener(textListener);
    
            BitmapFont fnt = assetManager.loadFont("Interface/Fonts/Default.fnt");
            txt = new BitmapText(fnt, false);
            txt.setBox(new Rectangle(0, 0, settings.getWidth(), settings.getHeight()));
            txt.setSize(fnt.getPreferredSize() * 2f);
            txt.setText(txtB);
            txt.setLocalTranslation(0, txt.getHeight(), 0);
            guiNode.attachChild(txt);
    
            txt2 = new BitmapText(fnt, false);
            txt2.setSize(fnt.getPreferredSize() * 1.2f);
            txt2.setText("Text without restriction. \nText without restriction. Text without restriction. Text without restriction");
            txt2.setLocalTranslation(0, txt2.getHeight(), 0);
            guiNode.attachChild(txt2);
    
            txt3 = new BitmapText(fnt, false);
            txt3.setBox(new Rectangle(0, 0, settings.getWidth(), 0));
            txt3.setText("Press Tab to toggle word-wrap. type text and enter to input text");
            txt3.setLocalTranslation(0, settings.getHeight()/2, 0);
            guiNode.attachChild(txt3);
        }
    
        private ActionListener keyListener = new ActionListener() {
            @Override
            public void onAction(String name, boolean isPressed, float tpf) {
                if (name.equals("WordWrap") && !isPressed) {
                    txt.setLineWrapMode( txt.getLineWrapMode() == LineWrapMode.Word ?
                                            LineWrapMode.NoWrap : LineWrapMode.Word );
                }
            }
        };
    
        private RawInputListener textListener = new RawInputListener() {
            private StringBuilder str = new StringBuilder();
    
            @Override
            public void onMouseMotionEvent(MouseMotionEvent evt) { }
    
            @Override
            public void onMouseButtonEvent(MouseButtonEvent evt) { }
    
            @Override
            public void onKeyEvent(KeyInputEvent evt) {
                if (evt.isReleased())
                    return;
                if (evt.getKeyChar() == '\n' || evt.getKeyChar() == '\r') {
                    txt3.setText(str.toString());
                    str.setLength(0);
                } else {
                    str.append(evt.getKeyChar());
                }
            }
    
            @Override
            public void onJoyButtonEvent(JoyButtonEvent evt) { }
    
            @Override
            public void onJoyAxisEvent(JoyAxisEvent evt) { }
    
            @Override
            public void endInput() { }
    
            @Override
            public void beginInput() { }
    
            @Override
            public void onTouchEvent(TouchEvent evt) { }
    
        };
    
    }

效果如下：

![BitmapFont](/content/images/2017/05/BitmapFont.png)

BitmapText作为一个几何物体，当也可以被添加到rootNode中。如果需要在3D场景中显示文字（例如在怪物头顶显示名称），它同样可以生效。

[jme3test.gui.TestBitmapText3D](https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-examples/src/main/java/jme3test/gui/TestBitmapText3D.java)

    package jme3test.gui;
    
    import com.jme3.app.SimpleApplication;
    import com.jme3.font.BitmapFont;
    import com.jme3.font.BitmapText;
    import com.jme3.font.Rectangle;
    import com.jme3.renderer.queue.RenderQueue.Bucket;
    import com.jme3.scene.Geometry;
    import com.jme3.scene.shape.Quad;
    
    public class TestBitmapText3D extends SimpleApplication {
    
        private String txtB =
        "ABCDEFGHIKLMNOPQRSTUVWXYZ1234567890`~!@#$%^&*()-=_+[]\\;',./{}|:<>?";
    
        public static void main(String[] args){
            TestBitmapText3D app = new TestBitmapText3D();
            app.start();
        }
    
        @Override
        public void simpleInitApp() {
            Quad q = new Quad(6, 3);
            Geometry g = new Geometry("quad", q);
            g.setLocalTranslation(0, -3, -0.0001f);
            g.setMaterial(assetManager.loadMaterial("Common/Materials/RedColor.j3m"));
            rootNode.attachChild(g);
    
            BitmapFont fnt = assetManager.loadFont("Interface/Fonts/Default.fnt");
            BitmapText txt = new BitmapText(fnt, false);
            txt.setBox(new Rectangle(0, 0, 6, 3));
            txt.setQueueBucket(Bucket.Transparent);
            txt.setSize( 0.5f );
            txt.setText(txtB);
            rootNode.attachChild(txt);
        }
    
    }

效果如下：

![BitmapFont3D](/content/images/2017/05/BitmapFont3D.png)

**总结：**

BitmapFont实质上是“纸片”模式的变种，它的主要优点是可以在程序中动态生成文本。由于BitmapFont使用的是矩形网格，多边形的面数相当少，其渲染的效率也比较高。

BitmapFont的缺点也很明显。首先BitmapFont需要专门制作，而且无法改变字体。如果需要不同字体的文字，就需要制作多套BitmapFont文件。其次，BitmapFont中的文字数量是有限的。这对于英语来说不存在任何问题，但是对中文就不太友好了。

如果你制作的是网络游戏，想要支持玩家在线聊天，那就需要制作一套庞大的BitmapFnt文件，尽量包含玩家可能会用到的各种字符。在这种应用场景下，BitmapFont并不是一个很好的选择。

#### 使用TTF字体

![jME True Type Font](/content/images/2017/05/jme3_ttf.jpg)

TTF（TrueTypeFont）是Apple公司和Microsoft公司共同推出的字体文件格式，是各种操作系统中最常用的一种字体文件表示方式。如果你使用的是Windows操作系统，可以直接在自己的`C:/Windows/Fonts/`文件夹下找到很多内置的字体；也可以在网上下载到诸多个性化的字体。

JME3原本是不支持TTF字体的，不过好在2016年3月份有人为JME3社区贡献了一个TTF字体渲染库：[jME-TrueTypeFont Rendering Library](https://hub.jmonkeyengine.org/t/jme-truetypefont-rendering-library/35395) 

[项目地址](http://1337atr.weebly.com/jttf.html)

这个库的用法及其简单，首先把TTF字体添加到你的项目中，然后把jME-TrueTypeFont这个jar也添加到你的项目中。

![工程结构](/content/images/2017/05/ttf_libs.png)

点击查看项目源码：[True TypeFont Test](https://github.com/jmecn/learnJME3/tree/master/True%20TypeFont%20Test)

接着就可以直接在代码里面用了。

	package net.jmecn.gui;
	
	import com.jme3.app.SimpleApplication;
	import com.jme3.math.ColorRGBA;
	import com.jme3.scene.Geometry;
	
	import truetypefont.TrueTypeFont;
	import truetypefont.TrueTypeKey;
	import truetypefont.TrueTypeLoader;
	
	/**
	 * 测试TTF字体
	 * 
	 * @author yanmaoyuan
	 *
	 */
	public class HelloTTF extends SimpleApplication {
	
		public static void main(String[] args) {
			// 启动程序
			HelloTTF app = new HelloTTF();
			app.start();
		}
	
		// 字形
		public final static int PLAIN = 0;// 普通
		public final static int BOLD = 1;// 粗体
		public final static int ITALIC = 2;// 斜体
		
		// 字号
		public final static int FONT_SIZE = 64;
		
		@Override
		public void simpleInitApp() {
			// 注册ttf字体资源加载器
			assetManager.registerLoader(TrueTypeLoader.class, "ttf");
	
			// 创建字体 (例如：楷书)
			TrueTypeKey ttk = new TrueTypeKey("Interface/Fonts/SIMKAI.TTF", // 字体
					PLAIN, // 字形：0 普通、1 粗体、2 斜体
					FONT_SIZE);// 字号
	
			TrueTypeFont font = (TrueTypeFont) assetManager.loadAsset(ttk);
	
			
			// 在屏幕中央显示一首五言绝句。
			String[] poem = { "空山新雨后", "天气晚来秋", "明月松间照", "清泉石上流" };
	
			// 计算坐标
			float x = 0.5f * (cam.getWidth() - FONT_SIZE * 5);
			float y = 0.5f * (cam.getHeight() + FONT_SIZE * 2);
			
			for (int i = 0; i < poem.length; i++) {
				// 创建文字
				Geometry text = font.getBitmapGeom(poem[i], 0, ColorRGBA.White);
				text.setLocalTranslation(x, y - i * FONT_SIZE, 0);
				guiNode.attachChild(text);
			}
	
		}
	
	}

运行效果如下：

![测试TTF字体](/content/images/2017/05/TTF.png)

优点：字体丰富；理论上支持所有语言；字体大小缩放不影响清晰度。

缺点：这个库对每个字符都生成了一个单独的网格，占用内存比Bitmap
Font大，而且文字量大的时候会增加多边形的面数。

### 鼠标

鼠标是GUI中的重要组成部分，jME3通过InputManager来管理鼠标。在上一章中，我们讨论了如何获得用户输入，其中就包括鼠标。下面继续介绍在GUI中与鼠标有关的内容。

#### 显示鼠标

通过InputManager的`isCursorVisible()`方法可以查询鼠标的显示状态，通过`setCursorVisible(boolean visible)`方法可以更改其显示状态。

    inputManager.setCursorVisible(true);
    inputManager.isCursorVisible();

jME3默认是不显示鼠标的，主要原因是jME3提供了一个第一人称摄像机（com.jme3.input.FlyByCamera）。在第一人称射击（FPS）游戏中，鼠标是多余的。当玩家晃动鼠标时，FlyByCamera会敏锐地捕捉到鼠标的输入，并实时改变镜头的朝向。

FlyByCamera在初始化时将鼠标的显示模式设为了false，显示鼠标最简单的方式是通过`flyCam.setEnable(false)`来禁用flyCam，此时flyCam会把鼠标的显示模式改回来。

下方是FlyByCamera中的`setEnable`方法的源码。

    /**
     * @param enable If false, the camera will ignore input.
     */
    public void setEnabled(boolean enable){
        if (enabled && !enable){
            if (inputManager!= null && (!dragToRotate || (dragToRotate && canRotate))){
                inputManager.setCursorVisible(true);
            }
        }
        enabled = enable;
    }

如果不想完全禁用FlyByCamera，则可以通过`flyCam.setDragToRotate(true)`方法将镜头的摆动模式设置为拖拽（drag）。在这种模式下，改变镜头方向需要先按住鼠标的左键或右键，然后再晃动鼠标。当鼠标的按键在释放（release）状态时，镜头是不会自动随鼠标晃动的。

下方是FlyByCamera中的`setDragToRotate`方法的源码。

    /**
     * Set if drag to rotate mode is enabled.
     * 
     * When true, the user must hold the mouse button
     * and drag over the screen to rotate the camera, and the cursor is
     * visible until dragged. Otherwise, the cursor is invisible at all times
     * and holding the mouse button is not needed to rotate the camera.
     * This feature is disabled by default.
     * 
     * @param dragToRotate True if drag to rotate mode is enabled.
     */
    public void setDragToRotate(boolean dragToRotate) {
        this.dragToRotate = dragToRotate;
        if (inputManager != null) {
            inputManager.setCursorVisible(dragToRotate);
        }
    }

#### 改变图标

jME3支持3种格式的图标：cur、ico、ani。如果想要更改鼠标的图标，首先要通过AssetManager加载图标资源，再通过`inputManager.setMouseCursor(JmeCurosr)`方法来应用图标。

    JmeCursor cur = (JmeCursor) assetManager.loadAsset("Textures/Cursors/meme.cur")
    inputManager.setMouseCursor(cur);

制作cur、ico、ani等图标文件有些麻烦，如果想直接使用图片作为图标行不行呢？用系统的API当然不可以，但是别忘了我们有障眼法。

最常见的做法是这样的：

第一步：制作一个完全透明的图标（invisible.cur），将它设置给InputManager，这样原来的鼠标就被“隐藏”起来了。

第二步：创建一个纸片，把我们准备的鼠标图像作为纹理设置给它。考虑到鼠标的特性，它的Z坐标应该设置一个比较大的值，好保证鼠标始终置于所有GUI物体之上。

第三步：创建一个事件监听器，通过它来保持这个“纸片”的位置和鼠标的位置同步，并在InputManager中注册这个监听器。

完整代码如下：

    package net.jmecn.gui;
    
    import com.jme3.app.SimpleApplication;
    import com.jme3.cursors.plugins.JmeCursor;
    import com.jme3.input.RawInputListener;
    import com.jme3.input.event.JoyAxisEvent;
    import com.jme3.input.event.JoyButtonEvent;
    import com.jme3.input.event.KeyInputEvent;
    import com.jme3.input.event.MouseButtonEvent;
    import com.jme3.input.event.MouseMotionEvent;
    import com.jme3.input.event.TouchEvent;
    import com.jme3.math.FastMath;
    import com.jme3.math.Vector3f;
    import com.jme3.ui.Picture;
    
    /**
     * 障眼法：伪装鼠标
     * 
     * @author yanmaoyuan
     *
     */
    public class FakeCursor extends SimpleApplication {
    
        public static void main(String[] args) {
            // 启动程序
            FakeCursor app = new FakeCursor();
            app.start();
        }
    
        private Picture cursor;// 伪装鼠标
        private Vector3f position = new Vector3f();// 鼠标位置
        private boolean isPressed = false;// 鼠标按键状态
    
        // 屏幕分辨率
        private float width;
        private float height;
    
        @Override
        public void simpleInitApp() {
            width = cam.getWidth();
            height = cam.getHeight();
    
            // 改变摄像机摆动方式，显示鼠标。
            flyCam.setDragToRotate(true);
    
            // 隐藏鼠标图标
            hideCursor();
    
            // 创造伪装鼠标
            cursor = fakeCursor();
            guiNode.attachChild(cursor);
    
            // 注册监听器
            inputManager.addRawInputListener(inputListener);
        }
    
        /**
         * 隐藏鼠标图标
         */
        private void hideCursor() {
            // 初始化鼠标
            // 把鼠标的图片搞成透明的，这样玩家就看不见鼠标了！
            JmeCursor jmeCursor = (JmeCursor) assetManager.loadAsset("Interface/Gui/Cursor/invisible.cur");
            inputManager.setMouseCursor(jmeCursor);
        }
    
        /**
         * 创建一个纸片，伪装成鼠标
         * 
         * @return
         */
        private Picture fakeCursor() {
            Picture cursor = new Picture("cur");
            cursor.setWidth(32);
            cursor.setHeight(32);
            cursor.setImage(assetManager, "Interface/Gui/Cursor/MyCursor.tga", true);
    
            // 设置鼠标居中
            position = new Vector3f(width / 2, height / 2 - 32, Float.MAX_VALUE);
            cursor.setLocalTranslation(position);
            return cursor;
        }
    
        /**
         * 光标移动监听器
         */
        private RawInputListener inputListener = new RawInputListener() {
            private float x;
            private float y;
    
            public void onMouseMotionEvent(MouseMotionEvent evt) {
                if (isPressed)
                    return;
                // 获得当前鼠标的坐标
                x = evt.getX();
                y = evt.getY();
    
                // 处理屏幕边缘
                x = FastMath.clamp(x, 0, cam.getWidth());
                y = FastMath.clamp(y, 0, cam.getHeight());
    
                position.x = x;
                position.y = y - 32;
    
                // 设置光标位置
                cursor.setLocalTranslation(position);
            }
    
            public void beginInput() {
            }
    
            public void endInput() {
            }
    
            public void onJoyAxisEvent(JoyAxisEvent evt) {
            }
    
            public void onJoyButtonEvent(JoyButtonEvent evt) {
            }
    
            public void onMouseButtonEvent(MouseButtonEvent evt) {
                isPressed = evt.isPressed();
    
                if (isPressed) {
                    // 显示鼠标位置
                    System.out.printf("MousePosition:(%.0f, %.0f)\n", position.x, position.y + 32f);
                }
            }
    
            public void onKeyEvent(KeyInputEvent evt) {
            }
    
            public void onTouchEvent(TouchEvent evt) {
            }
        };
    
    }

效果如下：

![FakeCursor](/content/images/2017/05/FakeCursor.png)

#### 获得鼠标位置

一般情况下，鼠标的位置应该通过InputManager的`getCursorPosition()`方法获取。只要存在鼠标设备，无论鼠标的图标是否显示，都可以通过这个方法得到鼠标在屏幕上的坐标。

    Vector2f pos = inputManager.getCursorPosition();

如果你使用前面提到的伪装鼠标，就可以直接使用position这个变量。它的x、y坐标代表了鼠标在屏幕上的位置。

    // 显示鼠标位置
    System.out.printf("MousePosition:(%.0f, %.0f)\n", position.x, position.y + 32f);

## Lemur框架

在前文中，我们主要讨论了3D游戏中的GUI原理，以及在jME3中制作GUI的方法。这些原理基本上适用于任何3D引擎。不过作为一名开发者来说，可能更想知道具体如何制作一个小地图、装备背包之类的东西。

jME3目前没有提供任何传统的GUI控件（例如窗口、按钮、标签、文本框、下拉框、表格等）。作为一款开源3D引擎，jME3更倾向于使用已有的开源产品，它的选择是Nifty-gui。

我不喜欢Nifty-gui，我推荐Lemur框架。理由如下：

* Nifty-gui是一个通用框架，Lemur是专门为jME3开发的。
* Lemur的作者是Paul Speed，而Paul Speed是jME3官方团队的核心成员。Paul在官方社区非常活跃，有问题可以随时向他请教。
* Lemur的设计吸收了Swing、JavaFX等框架中的精髓，学过Swing或JavaFX框架的Java程序员会很容易上手，而这两个框架我都学过。
* Lemur的类库体积很小，核心jar文件才340K，扩充控件包不到100K。
* 除了GUI控件之外，Lemur框架中还包含了InputMapper、Referrenced Object等非常有用的工具。
* Lemur有很丰富的（英文）文档和源码教程，易于学习。

基于这些理由，我倾向于使用Lemur框架。

### Lemur框架介绍

![Lemur](http://i.imgur.com/2Pur3pG.png)

Lemur（狐猴）是一个为jME3开发的UI工具包，它支持标准的2D用户界面以及3D用户界面。

![Mythruna](http://i.imgur.com/xlhbDgL.png)
![Mythruna](http://i.imgur.com/5wFF4YY.png)
![Arboreal](http://i.imgur.com/2O0Ivmq.png)
![Ethereal](http://i.imgur.com/zrYgDgI.png)

Lemur内部使用模块化设计，它实质上是一些模块的集合，这些模块可以用于创造个性化的GUI库。Lemur允许应用程序根据实际需要来使用它的部分或者全部模块，开发者甚至可以基于Lemur来开发一套全新的GUI库。

Lemur使用这些模块定义了一套标准GUI控件，包括窗口、按钮、标签、文本框等。你可以直接使用这些控件，也可以把它们当做教程来定义自己的GUI控件。

![Module Overview](http://i.imgur.com/Q7OYlBn.png)

* [项目地址](https://github.com/jMonkeyEngine-Contributions/Lemur)
* [Getting Started](https://github.com/jMonkeyEngine-Contributions/Lemur/wiki/Getting-Started)
* [Documentation](https://github.com/jMonkeyEngine-Contributions/Lemur/wiki/Documentation)

#### 基于jME3的Spatial

Lemur提供的所有GUI元素都不是2D平面，它们都是jME3中的3D物体，可以直接作为场景图中的元素而存在。

#### 简洁的API

Lemur的GUI API采用仿Swing API设计，它的开发者拥有15年的Swing开发经验。

#### 高级自定义样式

Lemur支持类似CSS结构的样式定义。GUI元素的样式可以通过Java代码或者groovy脚本来定义。通过使用注解（Annotation），开发者还可以利用Lemur的样式系统来定义新的GUI元素或属性。

#### 模块化设计

Lemur底层采用模块化设计，包含输入映射（InputMapper）、样式（Styles）、触屏/鼠标支持等多个模块。每个模块都可以独立使用，而不依赖其他模块。

#### 个性化设计

Lemur的核心模块用于创建个性化的GUI控件。

这是最有意思的部分，Lemur设计了一种用于创造GUI元素的规则，它提供的默认GUI控件本身也是基于这套规则来创造的。如果你不满意Lemur提供的默认控件，完全可以通过Lemur的核心模块来创造属于自己的GUI库。

### 众筹（Patreon）

如果你觉得Lemur有用，可以在 [Patreon](https://www.patreon.com/pspeed42?ty=h) 网站上参与原作者的众筹。

### 许可证（License）

Lemur使用与jMonkeyEngine相同的许可证，详细可查看许可证文件。

[LICENSE](https://github.com/jMonkeyEngine-Contributions/Lemur/blob/master/LICENSE)

### Lemur起步

1. 下载 Lemur.jar
2. 下载 Lemur-proto.jar
3. 下载 Lemur的依赖jar文件
4. 将上述jar文件添加到工程中
5. 初始化GUI
6. 初始化样式
7. 制作GUI

#### 下载

Lemur的核心jar文件为`lemur.jar`，其中包含了Lemur框架的所有核心模块。使用Lemur框架必须下载`lemur.jar`。

推荐同时下载`lemur-proto.jar`，这个包中提供了一些有用的组件。作者把一些具有试验性质的GUI组件放在`lemur-proto.jar`中，待其稳定后，最终会把这些组件移动到`lemur.jar`中。

Lemur的最新版本的发布地址为：[releases](https://github.com/jMonkeyEngine-Contributions/Lemur/releases)

javadoc地址：

[Lemur API javadoc](http://jmonkeyengine-contributions.github.io/Lemur/javadoc/Lemur/)

[LemurProto API javadoc](http://jmonkeyengine-contributions.github.io/Lemur/javadoc/LemurProto/)

#### 依赖说明

Lemur依赖下列类库：

* [Guava](https://github.com/google/guava) 12 或更高版本。
* [slf4j](http://www.slf4j.org/download.html) 1.7.5 或更高版本。除此之外，你可能还需要一个slf4j到其他日志框架的适配器。
* jME 3.1 或更高版本。
* [Groovy](http://www.groovy-lang.org/download.html) 2.1.9 或更高版本。(可选但强烈推荐) Lemur支持使用groovy脚本来定义GUI样式，因此需要groovy类库支持。例如：groovy-all-2.1.9.jar

#### 添加Lemur依赖

**直接添加jar文件**

这没有什么好说的，下载Lemur及其依赖的jar文件，然后添加到你的工程中。

**Gradle**

你可以使用Gradle或Maven等项目管理工具来管理Lemur依赖，通过JCenter仓库可以获取Lemur类库。

下面是使用Gradle来添加lemur和lemur-proto依赖的代码：

    dependencies {
        compile "com.simsilica:lemur:1.10.1"
        compile "com.simsilica:lemur-proto:1.9.1"
    }

这里有一份使用build.gradle来管理Lemur依赖的官方样例：[SimEthereal Basic Example](https://github.com/Simsilica/Examples/tree/master/sim-eth-basic)

这是本教程项目的脚本文件，其中也添加了Lemur依赖：[build.gradle](https://github.com/jmecn/jME3Tutorials/blob/master/build.gradle)

#### 初始化Lemur

Lemur只需要极少量的初始化工作来设置一些内部的默认值、注册监听器以及创建默认的AppState。完成这些工作只需要一行代码：`GuiGlobals.initialize(app)`

初始化要在jME3程序启动后尽早完成，一般来说这行代码应该写在`simpleInitApp()`方法中。

    @Override
    public void simpleInitApp() {

        // 初始化Lemur GUI
        GuiGlobals.initialize(this);
    ....

#### 初始化样式

Lemur的GUI元素默认是没有任何样式的，按钮（Button）连边框都没有。不过Lemur定义了一套默认的样式，我们可以非常方便地使用它。（或者你也可以参考样式文件的结构来自定义样式）。

>注意： Lemur的默认样式是使用groovy脚本定义的，如果要使用这套样式，你需要在项目中包含Groovy类库（即`groovy-all-版本号.jar`）。

Lemur提供的默认样式名为"glass"，它的外观风格看起来像是墨绿色的玻璃。下方的代码演示了如何加载"glass"样式。

    // 加载 'glass' 样式
    BaseStyles.loadGlassStyle();

下面这行代码会把"glass"样式设置为所有GUI元素的默认样式。

    // 将'glass'设置为GUI默认样式
    GuiGlobals.getInstance().getStyles().setDefaultStyle("glass");

#### 制作GUI

现在，你终于可以使用Lemur来制作GUI了。下面是一个简单的例子：

		// 创建一个Container作为窗口中其他GUI元素的容器
		Container myWindow = new Container();
		guiNode.attachChild(myWindow);

		// 设置窗口在屏幕上的坐标
		// 注意：Lemur的GUI元素是以控件左上角为原点，向右、向下生成的。
		// 然而，作为一个Spatial，它在GuiNode中的坐标原点依然是屏幕的左下角。
		myWindow.setLocalTranslation(300, 300, 0);

		// 添加一个Label控件
		myWindow.addChild(new Label("Hello, World."));
		
		// 添加一个Button控件
		Button clickMe = myWindow.addChild(new Button("Click Me"));
		clickMe.addClickCommands(new Command<Button>() {
			@Override
			public void execute(Button source) {
				System.out.println("The world is yours.");
			}
		});

运行程序，效果如下：

![Hello Lemur](/content/images/2017/05/HelloLemur.png)

完整代码如下：

	package net.jmecn.lemur;
	
	import com.jme3.app.SimpleApplication;
	import com.simsilica.lemur.Button;
	import com.simsilica.lemur.Command;
	import com.simsilica.lemur.Container;
	import com.simsilica.lemur.GuiGlobals;
	import com.simsilica.lemur.Label;
	import com.simsilica.lemur.style.BaseStyles;
	
	/**
	 * Lemur GUI
	 * @author yanmaoyuan
	 *
	 */
	public class HelloLemur extends SimpleApplication {
	
		public static void main(String[] args) {
			// 启动程序
			HelloLemur app = new HelloLemur();
			app.start();
		}
	
		@Override
		public void simpleInitApp() {
	
			// 初始化Lemur GUI
			GuiGlobals.initialize(this);
	
			// 加载 'glass' 样式
			BaseStyles.loadGlassStyle();
	
			// 将'glass'设置为GUI默认样式
			GuiGlobals.getInstance().getStyles().setDefaultStyle("glass");
	
			// 创建一个Container作为窗口中其他GUI元素的容器
			Container myWindow = new Container();
			guiNode.attachChild(myWindow);
	
			// 设置窗口在屏幕上的坐标
			// 注意：Lemur的GUI元素是以控件左上角为原点，向右、向下生成的。
			// 然而，作为一个Spatial，它在GuiNode中的坐标原点依然是屏幕的左下角。
			myWindow.setLocalTranslation(300, 300, 0);
	
			// 添加一个Label控件
			myWindow.addChild(new Label("Hello, World."));
			
			// 添加一个Button控件
			Button clickMe = myWindow.addChild(new Button("Click Me"));
			clickMe.addClickCommands(new Command<Button>() {
				@Override
				public void execute(Button source) {
					System.out.println("The world is yours.");
				}
			});
		}
	
	}

### Lemur应用

#### 术语

Lemur框架中有一些常用的术语，下面列出部分术语，并解释它们的含义。

* **GUI Element**：GUI元素包括面板（Panel）、按钮（Button）、标签（Label）、滑块（Slider）等，指的是用户眼中的最小物体。它在其他GUI框架中通常被叫做“组件（Component）”或“控件（Control）”。由于在Lemur中“组件（Component）”另有所指，为避免歧义，故称其为GUI元素。有时文档中会简称其为“元素（Element）”。
* **Component**：组件包括图标（Icon）、文本（Text）、背景（Background）、布局（Layout）、边距（Insets）等，GUI元素通常由多种“组件（Component）”组合而成。


#### 官方文档

Lemur的官方文档相当丰富，而且图文并茂，很容易读懂。虽然是英文的，但我依然推荐读者尝试阅读官方文档。

[GUI组件](https://github.com/jMonkeyEngine-Contributions/Lemur/wiki/GUI-Components)

[基本GUI元素](https://github.com/jMonkeyEngine-Contributions/Lemur/wiki/Base-GUI-Elements)

[复合GUI元素](https://github.com/jMonkeyEngine-Contributions/Lemur/wiki/Composite-GUI-Elements)

[容器和布局](https://github.com/jMonkeyEngine-Contributions/Lemur/wiki/Containers-and-Layouts)

[特效与动画](https://github.com/jMonkeyEngine-Contributions/Lemur/wiki/Effects-and-Animation)

[样式](https://github.com/jMonkeyEngine-Contributions/Lemur/wiki/Styling)

[Lemur核心模块](https://github.com/jMonkeyEngine-Contributions/Lemur/wiki/Modules)

#### 样例代码

Lemur还非常贴心地提供了大量样例代码供开发者学习。

* [com.simsilica.lemur.demo.BasicDemo](https://github.com/jMonkeyEngine-Contributions/Lemur/blob/master/src/main/java/com/simsilica/lemur/demo/BasicDemo.java) 这是`lemur.jar`中自带的Demo，演示了Lemur的核心功能。
* [com.simsilica.lemur.demo.ProtoDemo](https://github.com/jMonkeyEngine-Contributions/Lemur/blob/master/extensions/LemurProto/src/main/java/com/simsilica/lemur/demo/ProtoDemo.java) 这是`lemur-proto.jar`中自带的Demo，演示了一些试验GUI元素的功能。
* [Demos](https://github.com/jMonkeyEngine-Contributions/Lemur/tree/master/examples/demos/src/main/java/demo) 这个Demo项目演示了一些常用GUI元素的用法。（
   MainMenuState 主菜单、
   OptionPanelState 对话框、
   PopupPanelDemoState 弹出菜单、
   TextEntryDemoState 文本框、
   FormattedTextEntryDemoState 格式化文本框、
   WordWarpDemoState 文字自动换行、
   DragAndDropDemoState 拖拽、
   ListBoxDemoState 列表框）
* [LemurGems](https://github.com/jMonkeyEngine-Contributions/Lemur/blob/master/examples/LemurGems) 这个Demo项目演示了一些更加复杂的GUI，例如在3D场景中的物体“头上”添加一个“血条”。

#### FAQ

**Q1. 如何在Lemur中显示中文？**

Lemur使用的是BitmapFont字体，你得把需要在GUI上显示的文字做成BitmapFont，然后再设置给Lemur的样式。

    @Override
    public void simpleInitApp() {
	
        // 初始化Lemur GUI
        GuiGlobals.initialize(this);
	
        // 加载 'glass' 样式
        BaseStyles.loadGlassStyle();
	
        // 将'glass'设置为GUI默认样式
        GuiGlobals.getInstance().getStyles().setDefaultStyle("glass");

        // 加载BitmapFont字体
        BitmapFnt font = GuiGlobals.loadFont("你的字体文件.fnt");
        // 将这个字体设置为样式中默认字体
        GuiGlobals.getInstance().getStyles().setDefault(font);
    ...

**Q2. Lemur能用TTF字体吗？**

不能。不过你可以尝试修改Lemur的源码，把所有使用BitmapFont的地方换成TrueTypeFont。这是一个开源项目，随便玩吧，不要有什么心理负担。

**Q3. 为什么我的Android项目在使用Lemur后编译不通过报错？**

这很有可能是同名文件冲突造成的。`lemur.jar`和`lemur-proto.jar`中各包含一个名为`com/simsilica/lemur/style/base/glass-styles.groovy`的文件，这会导致Android在编译dex文件时发生异常。

解决方法是在build.gradle中声明不包含这个文件。

    android {
        packagingOptions {
        	exclude 'com/simsilica/lemur/style/base/glass-styles.groovy'
        }
    }

如果你确实需要使用这个样式文件，可以手动删除`lemur.jar`中的这个文件，因为`lemur-proto.jar`中的样式比`lemur.jar`中的更加丰富。

### 本章源码

* [net.jmecn.gui.FakePicture](https://github.com/jmecn/jME3Tutorials/blob/master/src/main/java/net/jmecn/gui/FakePicture.java)
* [net.jmecn.gui.HelloGUI](https://github.com/jmecn/jME3Tutorials/blob/master/src/main/java/net/jmecn/gui/HelloGUI.java)
* [net.jmecn.gui.HelloPicture](https://github.com/jmecn/jME3Tutorials/blob/master/src/main/java/net/jmecn/gui/HelloPicture.java)
* [net.jmecn.gui.HelloTTF](https://github.com/jmecn/jME3Tutorials/blob/master/src/main/java/net/jmecn/gui/HelloTTF.java)
* [net.jmecn.gui.FakeCursor](https://github.com/jmecn/jME3Tutorials/blob/master/src/main/java/net/jmecn/gui/FakeCursor.java)
* [net.jmecn.lemur.HelloLemur](https://github.com/jmecn/jME3Tutorials/blob/master/src/main/java/net/jmecn/lemur/HelloLemur.java)
