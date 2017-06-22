# 第二章：jME3基本概念

*在正文开始之前，如果你不太了解3D开发的基本概念，建议先简单阅读一下[3D游戏开发术语](http://blog.jmecn.net/3d-game-terminology/)。不必全部搞懂，简单过一遍即可。*

## SimpleApplication

`com.jme3.app.SimpleApplication`是jME3游戏的基类，直接继承这个类就可以创建3D游戏。等你以后对`SimpleApplication`足够熟悉，还可以通过继承这个类来自定义自己的应用。

### 第1次尝试

怎么创建一个Java/Android项目我们就不讨论了。一个最小的jME3程序看起来是这样的：

	package net.jmecn;
	
	import com.jme3.app.SimpleApplication;
	
	/**
	 * 你的第一个jME3程序
	 * @author yanmaoyuan
	 */
	public class HelloJME3 extends SimpleApplication {

		@Override
		public void simpleInitApp() {
			// TODO 初始化场景
		}
	
		public static void main(String[] args) {
			// 启动jME3程序
			HelloJME3 app = new HelloJME3();
			app.start();
		}
	
	}

运行这个程序，在桌面上将会弹出一个对话框，让我们选择一些启动参数。

![启动参数](/content/images/2017/03/settings.png)

* Fullscreen?: 是否全屏
* Screen Resolution: 窗口分辨率
* Refresh Rate: 刷新率
* Vsync?: 是否开启垂直同步
* Gamma correction: 伽玛校正
* Color Depth: 色深
* Anti-Aliasing: 抗锯齿

点击`Continue`即可启动游戏，然后你将看到**无尽虚空(一片漆黑)**，按`ESC`可以退出程序。

窗口的左下角显示当前画面的刷新速度，按`F5`可以隐藏/显示这些文字。

![启动界面](/content/images/2017/03/HelloJME3.png)

### Android的第一次？

Android项目和Java项目有一点点区别，你没有办法从main方法启动程序，需要使用`com.jme3.app.AndroidHarness`这个类。让MainActivity继承AndroidHarness，然后在构造方法中将`appClass`的值改为我们刚才写的那个类，**注意要写类的全名**。

    import com.jme3.app.AndroidHarness;

    public class MainActivity extends AndrdoidHarness {
        public MainActivity() {
            this.appClass = "net.jmecn.HelloJME3";
        }
    }

如果你不希望jME3的画面占据整个Activity窗口，还可以使用`com.jme3.app.AndroidHarnessFragment`，这与`AndroidHarness`的使用并没有太大区别。

    import com.jme3.app.AndroidHarnessFragment;

    public class MyFragment extends AndroidHarnessFragment {
        public MyFragment() {
            this.appClass = "net.jmecn.HelloJME3";
        }
    }

`AndroidHarness`和`AndroidHarnessFragment`中还有一些参数可以设置，但这不是我们现在讨论的重点，下面就不再细讲了。

### 第2次尝试

空无一物的世界太过无趣，我们让它再复杂一点点。

1. 创造一个方块形状的网格(Mesh)；
2. 加载一个能够感光的材质(Material)；
3. 创造一个几何体(Geometry)，应用刚才和网格和材质；
4. 创造一束定向光(DirectionalLight)，并让它斜向下照射，好使我们能够看清那个方块；
5. 将方块和光源都添加到场景图(rootNode)中。

现在代码看起来是这样了。

	package net.jmecn;
	
	import com.jme3.app.SimpleApplication;
	import com.jme3.light.DirectionalLight;
	import com.jme3.material.Material;
	import com.jme3.math.Vector3f;
	import com.jme3.scene.Geometry;
	import com.jme3.scene.Mesh;
	import com.jme3.scene.shape.Box;
	
	/**
	 * 你的第一个jME3程序
	 * @author yanmaoyuan
	 */
	public class HelloJME3 extends SimpleApplication {
	
		/**
		 * 初始化3D场景，显示一个方块。
		 */
		@Override
		public void simpleInitApp() {
			
			// #1 创建一个方块形状的网格
	        Mesh box = new Box(1, 1, 1);
	
	        // #2 加载一个感光材质
	        Material mat = new Material(assetManager, "Common/MatDefs/Light/Lighting.j3md");

	        // #3 创建一个几何体，应用刚才和网格和材质。
	        Geometry geom = new Geometry("Box");
	        geom.setMesh(box);
	        geom.setMaterial(mat);

	        // #4 创建一束定向光，并让它斜向下照射，好使我们能够看清那个方块。
	        DirectionalLight sun = new DirectionalLight();
	        sun.setDirection(new Vector3f(-1, -2, -3));
	        
	        // #5 将方块和光源都添加到场景图中
	        rootNode.attachChild(geom);
	        rootNode.addLight(sun);
		}
    
		public static void main(String[] args) {
			// 启动jME3程序
			HelloJME3 app = new HelloJME3();
			app.start();
		}
	
	}

运行这个程序，我们会看到屏幕上多了一个...正方形？？这怎么看都是2D的吧？！说好的3D方块呢？！

![正方形](/content/images/2017/03/FirstShot.png)

耐心点，你用手机拍过照吗？`SimpleApplication`中自带一个第一人称摄像机(flyCam)。画面之所以看上去是个正方形，是因为摄像机恰好对准了方块的其中一个面。

* 按`WSAD`键可以让摄像机前后左右运动
* 按`QZ`键可以上下移动
* `摆动鼠标`可以调整摄像机的角度
* 滑动`鼠标滚轮`可以调整摄像机的焦距

通过这些操作，我们可以调整窗口中的画面。先按`Q`让摄像机上升一些，然后按住`D`让摄像机右移，再用摆动鼠标让镜头对准那个方块。

如果你乐意，还可以按`W`让摄像机离方块近一点，这将使画面中的方块变大；按`S`则会离目标远一些，让它看起来更小。

![FlyCam](/content/images/2017/03/FlyCam.png)

### Debug

`SimpleApplication`还有一个最基本的调试功能。按`C`键可以显示摄像机当前的位置和旋转姿态；按`M`键可以查看内存的使用情况。这些调试数据并不会在窗口上显示，而是在控制台中打印出来。

按C键之后看到的结果：

    Camera Position: (4.4114223, 3.3620508, 7.5415998)
    Camera Rotation: (-0.046265673, 0.9518722, -0.1815604, -0.2425582)
    Camera Direction: (-0.44496882, -0.36808884, -0.81640255)
    cam.setLocation(new Vector3f(4.4114223f, 3.3620508f, 7.5415998f));
    cam.setRotation(new Quaternion(-0.046265673f, 0.9518722f, -0.1815604f, -0.2425582f));

把上面最后2句代码添加到simpleInitApp()方法中，就可以在初始化时设定摄像机的位置和视角。

按M键之后看到的结果：

    Total   heap memory held: 15566kb
    Only heap memory available, if you want to monitor direct memory use BufferUtils.setTrackDirectMemoryEnabled(true) during initialization.

堆内存已使用15566kb，差不多15.2MB。

在程序启动时设置`BufferUtils.setTrackDirectMemoryEnabled(true)`，还可以查看通过底层分配的直接内存。

    Existing buffers: 64
    (b: 10  f: 38  i: 7  s: 9  d: 0)
    Total   heap memory held: 17351kb
    Total direct memory held: 596kb
    (b: 553kb  f: 36kb  i: 0kb  s: 5kb  d: 0kb)

## 生命周期

刚接触jME3就来讲生命周期是不是太早了？好在我并不打算讲得太复杂，只需要知道主要方法的执行顺序就可以了。

jME3 Application的**简化版**生命周期是这样的：

1. 启动 start
2. 初始化 initialize
3. 主循环 update
4. 停止 stop
5. 销毁 destroy

*神马？居然有人问复杂版？你可以看看[这幅图](http://bbs.jmecn.net/t/topic/37)。*

### 启动

当你写好了一个jME3程序，通过调用start()方法就可以启动程序，启动后程序就会进行初始化。

    public static void main(String[] args) {
        // 启动jME3程序
        HelloJME3 app = new HelloJME3();
        app.start();
    }

在启动之前，其实我们还可以调整一些参数，比如阻止`配置窗口`的出现。

    public static void main(String[] args) {
        // 启动jME3程序
        HelloJME3 app = new HelloJME3();
        app.setShowSettings(false);
        app.start();
    }

`com.jme3.system.AppSettings`可以让我们设置更多的参数。

	public static void main(String[] args) {
		// 配置参数
		AppSettings settings = new AppSettings(true);
		settings.setTitle("一个方块");// 标题
		settings.setResolution(480, 720);// 分辨率
		
		// 启动jME3程序
		HelloJME3 app = new HelloJME3();
		app.setSettings(settings);// 应用参数
		app.setShowSettings(false);
		app.start();
	}

![AppSettings](/content/images/2017/03/AppSettings.png)

### 初始化

jME3始化时会先把`场景图`、`GUI`、`摄像机`、`渲染器`、`资源管理器`、`输入系统`、`声音系统`等一大堆东西准备好，最后执行`simpleInitApp`方法。

我们可以在`simpleInitApp`方法中编写初始化游戏场景的代码。

	/**
	 * 初始化3D场景，显示一个方块。
	 */
	@Override
	public void simpleInitApp() {
		
	   // #1 创建一个方块形状的网格
	    Mesh box = new Box(1, 1, 1);
	
	   // #2 加载一个感光材质
	   Material mat = new Material(assetManager, "Common/MatDefs/Light/Lighting.j3md");

	   // #3 创建一个几何体，应用刚才和网格和材质。
	   Geometry geom = new Geometry("Box");
	   geom.setMesh(box);
	   geom.setMaterial(mat);

	   // #4 创建一束定向光，并让它斜向下照射，好使我们能够看清那个方块。
	   DirectionalLight sun = new DirectionalLight();
	   sun.setDirection(new Vector3f(-1, -2, -3));
	        
	   // #5 将方块和光源都添加到场景图中
	   rootNode.attachChild(geom);
	   rootNode.addLight(sun);
	}

jME3是一个基于`场景图 Scenegraph`的3D引擎，SimpleApplication中整个场景的原点就是`rootNode`，3D模型和光源都要添加到`rootNode`中才会生效，否则是看不到的。

### 主循环

jME3就像一台汽车发动机，点火成功之后发动机就会飞速运转，驱动汽车行驶。SimpleApplication初始化完毕后，主循环就会开始执行，驱动整个游戏的逻辑。我们可以通过重写SimpleApplication中的`public void simpleUpdate(float tpf)`方法来实现自己的游戏逻辑。

    /**
     * 主循环
     */
    @Override
    public void simpleUpdate(float tpf) {
        // TODO
    }

写在这个方法中的代码将会随着游戏状态更新而重复执行。其参数`tpf`是`Time per frame`的简写，代表画面两次刷新的时间间隔，单位是秒。它的倒数`1/tpf`就是`FPS(Frame per second)`，即每秒帧数。

比起`tpf`这个变量名，我更倾向于把它叫做`deltaTime`，即物理学常用的变量`Δt`，这样更能代表它的现实意义。

现在我想让方块绕Y轴旋转，角速度是每秒360°，也就是`2π/s`，那么在主循环中每一帧的旋转角度应该是：`Δr = 2π/s * Δt`。

用代码来表示：

    /**
     * 主循环
     */
    @Override
    public void simpleUpdate(float deltaTime) {
        float speed = FastMath.TWO_PI;
        float rad = speed * deltaTime;
    }

*注：`com.jme3.math.FastMath`是jME3提供的一个数学工具类。*

下面我们把之前在`simpleInitApp`中创建的Geometry对象变成HelloJME3类的一个成员，然后在`simpleUpdate`方法中调用`rotate`方法使其旋转。

#### 实例：让方块旋转

	package net.jmecn;
	
	import com.jme3.app.SimpleApplication;
	import com.jme3.light.DirectionalLight;
	import com.jme3.material.Material;
	import com.jme3.math.FastMath;
	import com.jme3.math.Vector3f;
	import com.jme3.scene.Geometry;
	import com.jme3.scene.Mesh;
	import com.jme3.scene.shape.Box;
	import com.jme3.system.AppSettings;
	
	/**
	 * 你的第一个jME3程序
	 * @author yanmaoyuan
	 */
	public class HelloJME3 extends SimpleApplication {
		
		private Geometry geom;
	
		/**
		 * 初始化3D场景，显示一个方块。
		 */
		@Override
		public void simpleInitApp() {
			
			// #1 创建一个方块形状的网格
	        Mesh box = new Box(1, 1, 1);
	
	        // #2 加载一个感光材质
	        Material mat = new Material(assetManager, "Common/MatDefs/Light/Lighting.j3md");
	
	        // #3 创建一个几何体，应用刚才和网格和材质。
	        geom = new Geometry("Box");
	        geom.setMesh(box);
	        geom.setMaterial(mat);
	
	        // #4 创建一束阳光，并让它斜向下照射，好使我们能够看清那个方块。
	        DirectionalLight sun = new DirectionalLight();
	        sun.setDirection(new Vector3f(-1, -2, -3));
	        
	        // #5 将方块和都添加到场景图中
	        rootNode.attachChild(geom);
	        rootNode.addLight(sun);
	        
		}
		
		/**
		 * 主循环
		 */
		@Override
		public void simpleUpdate(float deltaTime) {
			// 旋转速度：每秒360°
			float speed = FastMath.TWO_PI;
			// 让方块匀速旋转
			geom.rotate(0, deltaTime * speed, 0);
		}
	
		public static void main(String[] args) {
			// 配置参数
			AppSettings settings = new AppSettings(true);
			settings.setTitle("一个方块");
			settings.setResolution(480, 720);
			
			// 启动jME3程序
			HelloJME3 app = new HelloJME3();
			app.setSettings(settings);// 应用参数
			app.setShowSettings(false);
			app.start();
		}
	
	}

![旋转](/content/images/2017/03/rotate2.gif)

关于主循环还有太多东西可以说，但现在只需要简单理解一下它的作用就足够了。

### 停止和销毁

这一部分工作通常和开发者无关，当你按下`ESC`键时，就会通过输入系统触发stop()方法，然后执行destory()。

有时候我们希望禁用ESC键，避免玩家不小心退出游戏，那就得设计别的方式来主动调用stop()方法。

## 其他API

除了前面介绍的，SimpleApplication中还定义了很多有用的API。看不懂没关系，先列出来了解一下。

**1. 场景图管理**

* Node rootNode 3D场景根节点
* Node guiNode GUI根节点
* getRootNode()
* getGuiNode() 

**2. 视口**

* ViewPort viewPort
* ViewPort guiViewPort
* getViewPort()
* getGuiViewPort()

**3. 摄像机**

* Camera cam
* FlyCamera flyCam
* getCamera()

**4. 渲染管理器**

* RendererManager renderManager
* Renderer renderer
* getRenderManager()
* getRenderer()

**5. 资源管理器**

* AssetManager assetManager
* getAssetManager()

**6. 输入管理**

* InputManager inputManager
* getInputManager();
* MouseInput mouseInput 鼠标输入
* KeyInput keyInput 键盘输入
* JoyInput joyInput 手柄输入
* TouchInput touchInput 触屏输入

**7.状态机管理**

* AppStateManager stateManager
* getStateManager()

**8. 声音系统**

* AudioRenderer audioRenderer
* Listener listener
  getAudioRenderer()
  getListener()
