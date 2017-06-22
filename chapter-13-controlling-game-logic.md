# 第十三章：控制游戏逻辑

## 导读：《入门：游戏主循环》

【转载】[入门：游戏主循环](https://indienova.com/indie-game-development/game-main-loop/)

### 引言

主循环是一款游戏或者框架的核心以及基础，它会让游戏以及动画看起来是在做实时的运行。几乎所有游戏（除了回合制等几种类型以外）都要基于主循环以及精确的时间控制。

下面就是一个最基本的主循环示例代码：


先定义一个简单的游戏引擎接口，声明游戏的基本生命周期。

	package net.jmecn.logic;
	
	/**
	 * 一个简单的游戏引擎接口
	 * 
	 * @author yanmaoyuan
	 *
	 */
	public interface IGameEngine {
		// 初始化
		void init();
		// 主循环
		void update();
		// 渲染
		void render();
		// 清理
		void clear();
	}

然后定义一个抽象的游戏`Engine`，用于驱动游戏主循环。

	package net.jmecn.logic;
	
	/**
	 * 抽象游戏引擎，实现了游戏的主循环。
	 * 
	 * @author yanmaoyuan
	 *
	 */
	public abstract class Engine implements IGameEngine {
	
		protected boolean running = false;
		protected boolean pause = false;
	
		// 开始游戏
		public void start() {
			if (running)
				return;
			
			running = true;
			pause = false;
			
			// 启动主循环
			loop();
		}
		
		// 暂停游戏
		public void pause(boolean pause) {
			this.pause = pause;
		}
		
		// 主循环
		private void loop() {
			init(); // 初始化整个体系，框架、图形、声音等等
			
			while(running) {
				
				// 暂停游戏
				if (pause) {
					return;
				}
				
				update();
				
				render();
			}
			
			clear(); // 清理资源、关闭各种接口等
		}
	
		// 停止游戏
		public void stop() {
			running = false;
		}
	}


主循环每次执行的时候，都会调用指定好的函数来执行相应的工作，比如在上面代码中我们设置 `update()` 来处理游戏逻辑，设置 `render()` 来绘制游戏当前的画面。比如下面这个例子

	package net.jmecn.logic;
	
	/**
	 * 实现游戏逻辑
	 * 
	 * @author yanmaoyuan
	 *
	 */
	public class MyGame extends Engine {
	
		float x = 0;
		
		@Override
		public void init() {}
		
		@Override
		public void update() {
			x += 1;
		}
	
		@Override
		public void render() {
			// TODO 根据x坐标绘制玩家
		}
	
		@Override
		public void clear() {}
	
		public static void main(String[] args) {
			MyGame game = new MyGame();
			game.start();
		}
	
	}


每一次游戏逻辑函数被触发的时候，都会将玩家角色的水平 `x` 位置加 1，这样，经过相应的 `render()` 方法处理，就会看到角色在横向运动。

**但是，上面这个主循环有着明显的问题，那就是：主循环能够被执行的次数是取决于机器配置的，越快的机器，主循环执行的次数越多，那么角色也就运动得越快。**

当然，如果我们知道运行游戏的硬件系统是一致的，比如说都运行在某种主机平台上，那么，还是可以直接使用这样的主循环的。

那么，既然这种主循环不是很合理，我们希望游戏在任何系统上都保持一致的速度，那就需要引入基于时间的主循环了。

### 基于时间的主循环

下面这个主循环例子基于度过的时间，那么，会在不同机器上表现达到一致：

首先，接口中的`update()`方法变成了`update(float deltatime)`

	package net.jmecn.logic;
	
	/**
	 * 一个简单的游戏引擎接口
	 * 
	 * @author yanmaoyuan
	 *
	 */
	public interface IGameEngine {
		// 初始化
		void init();
		/**
		 * 主循环
		 * @param deltatime 间隔时间
		 */
		void update(float deltatime);
		// 渲染
		void render();
		// 清理
		void clear();
	}

其次，主循环中计算了执行`update(float deltatime)`方的的时间间隔（单位：秒）：

	// 主循环
	private void loop() {
		long currentTime = System.nanoTime();
		long lastTime = 0;
		long deltaTime = 0;
		
		init(); // 初始化整个体系，框架、图形、声音等等
		
		while(running) {
			lastTime = currentTime;
			currentTime = System.nanoTime();
			deltaTime = currentTime - lastTime;
		    
			// 暂停游戏
			if (pause) {
				return;
			}
			
			update(deltaTime * 0.0000000001f);
			
			render();
		}
		
		clear(); // 清理资源、关闭各种接口等
	}

在实现类中，不再直接`+1`，而是加上时间差：

	@Override
	public void update(float deltatime) {
		x += deltatime;
	}

我们通过 `System.nanoTime()` 取得程序运行的时间（单位：纳秒），每一次主循环执行的时候我们都取得时间差，然后通过时间差来决定更新距离。

我们用 `lastTime` 来记录上一次的时间，然后通过 `System.nanoTime()` 取得当前的时间，然后减去上一次的时间，就得到了 `deltaTime` ——时间差。

	lastTime = currentTime;
	currentTime = System.nanoTime();
	deltaTime = currentTime - lastTime;

deltaTime的时间单位是纳秒，游戏中一般使用秒作为时间单位，因此需要对齐进行转换`deltaTime * 0.0000000001f`。

### 基于时钟的主循环

有一些语言或者框架提供了按照时钟触发的方式，可以设定固定的触发时间间隔，比如下面的例子：

	package net.jmecn.logic;
	
	import java.util.Timer;
	import java.util.TimerTask;
	
	/**
	 * 抽象游戏引擎，实现了游戏的主循环。
	 * 
	 * @author yanmaoyuan
	 *
	 */
	public abstract class Engine implements IGameEngine {
	
		protected boolean pause = false;
		
		Timer timer;
		TimerTask task;
		
		// 开始游戏
		public void start() {
			init(); // 初始化整个体系，框架、图形、声音等等
	
			// 定时器
			timer = new Timer();
			task = new TimerTask() {
				public void run() {
					// 暂停游戏
					if (pause) {
						return;
					}
					
					update();
					render();
				}
			};
			
			// 设定 1/30 秒触发一次，执行 主循环
			long period = (long) (1000 / 30f);
			timer.scheduleAtFixedRate(task, 0, period);
		}
		
		// 暂停游戏
		public void pause(boolean pause) {
			this.pause = pause;
		}
	
		// 停止游戏
		public void stop() {
			task.cancel();// 退出任务
			task = null;
			
			timer.cancel();
			timer = null;
			
			clear(); // 清理资源、关闭各种接口等
		}
	}

游戏逻辑可以按固定速率执行：

	@Override
	public void update() {
		x += 1;
	}

可见，这种方式可以通过系统时钟来不断的触发主循环 `update()`，可以精确的控制游戏的运行状态。它可以保证游戏在任何机器上都以同样的速度运行。现在很多主流的游戏框架都支持通过时钟来触发事件。

## jME3的主循环

jME3使用的是[基于时间的主循环](#基于时间的主循环)，`jme3-core.jar`中的`com.jme3.system.NanoTimer`类就是是专门用来计算时间的。

### 主循环做了什么？

当你继承SimpleApplication类后，就自动获得了它提供的主循环，利用它我们可以实现自己的游戏逻辑，比如遥控NPC、处理游戏事件、响应用户输入等。

SimpleApplication在后台做了很多事情，我们曾在本教程第二章“jME3基本概念”的[生命周期](http://blog.jmecn.net/chapter-2-basic-concepts/#%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F)章节简单介绍过：

* 执行 `initialize()` 方法，初始化`显示系统`、`场景图`、`摄像机`、`输入系统`、`音效系统`、`资源管理系统`、`应用状态机`等一大堆重要的内容。
 * initialize() 方法只在程序启动时执行一次。
 * 在 initialize() 方法的最后，会执行我们重载的 `simpleInitApp()` 方法。
 * 由于 initialize() 方法已经把该准备的东西都准备好了，所以我们在 simpleInitApp() 方法中才可以直接使用assetManager、inputManager等重要对象。
* 循环执行 `update(float tpf)` 方法
 * 1-响应用户输入，执行所有事件监听器中的代码。见[第九章：用户交互](http://www.jmecn.net/tutorial-for-beginners/chapter-9-user-interaction/)
 * 2-更新游戏状态
  * 2-1 更新全局游戏状态（执行所有 AppState#update() 方法）；
  * 2-2 更新用户自定义游戏逻辑（执行 simpleUpdate() 方法）；
  * 2-3 更新场景图状态（执行所有Spatial中的 Control#update() 方法。）。
 * 3-渲染音频和视频
  * 3-1 渲染全局应用状态中的场景（执行所有 AppState#render() 方法）；
  * 3-2 渲染游戏主场景（执行 renderManager#render() 方法）；
  * 3-3 执行用户自定义渲染代码（执行 simpleRender() 方法）。
 * 重复上述循环。
* 退出游戏。当 stop() 方法被调用后，执行所有 cleanup() 方法和 distory() 方法，然后关闭jME3窗口、终止主循环。

### 主循环的用法

在继承SimpleApplication类之后，我们的游戏逻辑主要通过重载 `simpleUpdate(float tpf)` 方法来实现，游戏主循环会自动调用它。

下面是一个例子：

    package net.jmecn.logic;
    
    import com.jme3.app.SimpleApplication;
    import com.jme3.light.DirectionalLight;
    import com.jme3.material.Material;
    import com.jme3.math.FastMath;
    import com.jme3.math.Quaternion;
    import com.jme3.math.Vector3f;
    import com.jme3.scene.Geometry;
    import com.jme3.scene.Spatial;
    import com.jme3.scene.shape.Box;
    
    /**
     * 主循环
     * 
     * @author yanmaoyuan
     *
     */
    public class HelloLoop extends SimpleApplication {
    
        // 旋转的物体
        private Spatial spatial;
        // 旋转速度：每秒180°
        private float rotateSpeed = FastMath.PI;
    
        @Override
        public void simpleInitApp() {
            cam.setLocation(new Vector3f(3.3435764f, 3.7595856f, 6.611723f));
            cam.setRotation(new Quaternion(-0.05573249f, 0.9440857f, -0.23910178f, -0.22006002f));
    
            // 创建一个方块
            Material mat = new Material(assetManager, "Common/MatDefs/Light/Lighting.j3md");
            spatial = new Geometry("Box", new Box(1, 1, 1));
            spatial.setMaterial(mat);
    
            rootNode.attachChild(spatial);
    
            // 添加光源
            rootNode.addLight(new DirectionalLight(new Vector3f(-1, -2, -3)));
        }
    
        @Override
        public void simpleUpdate(float tpf) {
            // 绕Y轴以固定速率旋转
            spatial.rotate(0, tpf * rotateSpeed, 0);
        }
    
        public static void main(String[] args) {
            // 启动
            HelloLoop app = new HelloLoop();
            app.start();
        }
    
    }

这个程序的作用，是创建一个方块，并让它以每秒180°的速度绕Y轴旋转。

### 扩展主循环

一般来说，我们会在 `simpleInitApp()` 方法中初始化程序所需的资源，然后在 `simpleUpdate(float tpf)` 方法中编写代码逻辑。直到本章为止，教程中几乎所有的例子都是这样写的。

这种写法并不好。对于一个稍微复杂一点的游戏来说，这都将导致单一类中的代码超级长，难以阅读，而且难以维护。我们并不需要在游戏启动时加载所有资源，也不需要在主循环中执行所有逻辑代码。

最好的做法是将游戏模块化，把 simpleInitApp() 和 simpleUpdate(float tpf) 方法中的代码分解到不同的Java类中。jME3提供了两个接口（AppState和Control），利用它们可以把系统的功能模块分解成若干个子模块。

* `AppState` 用于将**全局游戏机制**模块化。
* `Control` 用于将**单个游戏对象**的行为模块化。

诸如天气、光照、音效、物理等模块的代码，都可以搬到AppState中。如果游戏中有几个不同的地图，可以分解成多个子场景，并在不同的AppState中并分别初始化。当玩家进入某个地图就加载某个AppState，离开时再把它移除。

如果要控制单个游戏对象的行为，就可以用Control来做。如果程序设计得合理，就可以得到很多可复用的小组件。比如让一个物体旋转、让NPC寻找到达目的地的最佳路线、让摄像机始终跟随玩家、让一个物体始终正面朝向玩家。

下面我们分别介绍 AppState 和 Control 的用法。

## AppState

### 介绍

AppState是jME3的一个重要接口，主要用于处理全局的游戏机制。合理应用AppState，可以让你的程序结构清晰、灵活，而且代码可复用性更高。当 AppState 和 Control 结合使用时，还可以实现更加复杂的功能。

#### 应用场景

举几个在游戏开发中经常遇到的用例：

* 当用户处于不同的界面时，键盘和鼠标的输入需要有不同的处理。比如在开始界面、在创建角色界面、在游戏场景内...不同的场景下，输入的处理方式是不一样的。能否根据不同的场景，分别激活或者禁用不同的输入处理代码？

* 我的程序包含“开始菜单”、“游玩模式”、“角色编辑模式”、“展览模式”，我能否在这些场景之间自由切换？

* 我在 simpleUpdate() 方法中有一段超级复杂的逻辑代码（例如：人工智能），能不能把这段代码独立出去，单独做成一个功能模块？

这些都可以通过AppState完成。每个AppState都是应用程序的一个子集（或扩展），可以访问到jME3中的全局对象，诸如：

* AssetManager
* InputManager
* AppStateManager
* ViewPort
* Camera
* RootNode
* GuiNode
* 等等..

有了这些全局对象，AppState 就能做到原本你在 SimpleApplication 中做的任何事情。

#### 生命周期

AppState的生命周期有三个主要阶段：初始化（initialize）、主循环（update）、清理（cleanup）。

![AppState life cycle](/content/images/2017/06/AppState-Lift-Cycle.svg)

`AppStateManager` 用于管理所有的 AppState 实例，通过 下面两行代码，可以把一个 AppState 实例添加到系统的处理队列中，或者移除一个 AppState对象。

    stateManager.attach(AppState appState);// 添加AppState
    stateManager.detach(AppState appState);// 移除AppState

AppState的生命历程是这样的：

* **初始化**：当一个 AppState 实例被添加到 AppStateManager 之后，它的生命周期就开始了，随后 initialize() 方法将会执行一次。`initialze()` 方法类似于 `simpleInitApp()`，用于初始化游戏资源、输入处理、GUI等内容。
* **主循环**：每个 AppState 都有自己的 `update(float tpf)` 方法，类似于 `simpleUpdate(float tpf)`。AppState 的主循环会被 SimpleApplication 的主循环驱动运行。
 * **激活**：调用 `setEnabled(true)` 将激活 AppState 的主循环。我们应该在 AppState 激活的同时把子场景添加到主场景中，这样运行时才能看到子场景。
 * **运行**：当 `isEnabled()` 的返回值为 `true` 时，AppState 的 `update(float tpf)` 方法会循环运行。此时就可以更新游戏状态、修改场景图、处理用户输入了。
 * **暂停**：调用 `setEnabled(false)` 将暂停 AppState 的主循环。此时应该吧子场景从主场景中移除，直到 AppState 再次被激活。
* **清理**：当一个 AppState 实例被从 AppStateManager 中移除后，它的生命周期即将走到终点。此时 `cleanup()` 方法将会执行一次，用于清理 AppState 中的各种资源，比如删除子场景，移除输入监听器，关闭GUI等等。

#### 用法

对于一个已经定义好的 AppState，只需要把 AppState 对象交给 AppStateManager 即可。例如，jME3中集成了Bullet物理引擎，使用方法是这样的：

    @Override
    public void simpleInitApp() {
        stateManager.attach(new BulletAppState());
        ...
    }


自定义 AppState 通常遵循下列步骤：

* 创建一个AppState的实现类。
* 实现 AppState 中的抽象方法。
 * 在initialize()方法中进行初始化，在update()方法中实现游戏逻辑，在cleanup()方法中进行清理工作。
 * 你可以使用构造方法来给这个AppState传参。
 * 分别定义当 isEnabled() 为 true 和 false 时的行为，比如开灯/关灯。
* 创建一个 AppState 对象，将其添加到AppStateManager中（`stateManager.attach(appState);`）。
* 在你需要的时候，激活/暂停 AppState 的运行状态（`appState.setEnabled(true);`）。
* 移除一个AppState对象（`stateManager.attach(appState);`），并进行清理。

关于AppState的具体使用，有这么几点需要注意的地方：

* 当你的程序中有多个 AppState 时，它们运行的顺序与添加到AppStateManager中的顺序一致。
* AppState 的设计应遵循**高内聚、低耦合**原则，两个 AppState 之间最好不要存在依赖关系。否则当你移除一个AppState时，当心导致其他AppState发生异常。
* 永远不要忘记清理 AppState 中的资源。

### 实例：子场景管理

![](/content/images/2017/06/HelloAppState.png)

上图中的场景我们已经看过很多次了，场景中是一个红色的方块，使用Lighting.j3md材质。为了让你能够看清它，场景中还加入了光源。

现在我要使用AppState来实现这个程序。

* VisualAppState 管理子场景，控制方块的显示和隐藏。
* LightAppState 管理光照，控制开灯、关灯功能。
* InputAppState 管理用户输入。
 * 按下<kbd>空格</kbd> 键，调用 LightAppState 来开关灯；
 * 按下<kbd>Tab</kbd> 键，调用 VisualAppState 来显示/隐藏场景。

#### HelloAppState

HelloAppState.java 是我们的主类，现在它的作用仅仅是启动程序，并把上述3个 AppState 交给 AppStateManager 管理。

    package net.jmecn;
    
    import com.jme3.app.SimpleApplication;
    
    import net.jmecn.logic.InputAppState;
    import net.jmecn.logic.LightAppState;
    import net.jmecn.logic.VisualAppState;
    
    /**
     * 演示AppState的作用
     * 
     * @author yanmaoyuan
     *
     */
    public class HelloAppState extends SimpleApplication {
    
        public static void main(String[] args) {
            HelloAppState app = new HelloAppState();
            app.start();
        }
        
        @Override
        public void simpleInitApp() {
            stateManager.attach(new LightAppState());
            stateManager.attach(new VisualAppState());
            stateManager.attach(new InputAppState());

            // 初始化摄像机
            cam.setLocation(new Vector3f(2.4611378f, 2.8119917f, 9.150583f));
            cam.setRotation(new Quaternion(-0.020502187f, 0.97873497f, -0.16252096f, -0.1234684f));
        }
    
    }

比起我们以前写的代码，是不是整洁多了？

除了在 simpleInitApp() 方法中初始化AppState以外，还有另一种更优雅的写法：调用父类构造方法，设置所需的 AppState。

    package net.jmecn;
    
    import com.jme3.app.DebugKeysAppState;
    import com.jme3.app.FlyCamAppState;
    import com.jme3.app.SimpleApplication;
    import com.jme3.app.StatsAppState;
    import com.jme3.audio.AudioListenerState;
    import com.jme3.math.Quaternion;
    import com.jme3.math.Vector3f;
    
    import net.jmecn.logic.InputAppState;
    import net.jmecn.logic.LightAppState;
    import net.jmecn.logic.VisualAppState;
    
    /**
     * SimpleApplication的最佳形式
     * 
     * @author yanmaoyuan
     *
     */
    public class HelloAppState2 extends SimpleApplication {
    
        public static void main(String[] args) {
            HelloAppState2 app = new HelloAppState2();
            app.start();
        }
    
        /**
         * 在构造方法中初始化AppState
         */
        public HelloAppState2() {
            super(new StatsAppState(), new FlyCamAppState(), new AudioListenerState(), new DebugKeysAppState(),
                    new LightAppState(), new VisualAppState(), new InputAppState());
        }
    
        @Override
        public void simpleInitApp() {
            // 初始化摄像机
            cam.setLocation(new Vector3f(2.4611378f, 2.8119917f, 9.150583f));
            cam.setRotation(new Quaternion(-0.020502187f, 0.97873497f, -0.16252096f, -0.1234684f));
        }
    
    }

这里比上面多了4个AppState，它们是jME3系统自带的AppState：

* StatsAppState 它的功能就是显示屏幕左下角的状态信息。
* FlyCamAppState 它的功能就是jME3自带的第一人称摄像机。
* AudioListenerState 它的作用是渲染3D音效。
* DebugKeysAppState 它的作用就是按 <kbd>C</kbd> 显示摄像机位置，按 <kbd>M</kbd> 显示内存使用情况。

#### VisualAppState

下面是 VisualAppState.java 的代码。initialize() 方法用于初始化场景，update(float tpf) 方法用于控制方块旋转。

注意，在 VisualAppState 中，我没有直接使用 SimpleApplication 中的 rootNode，而是又定义了一个 sceneNode。子场景中的所有物体都被添加到 sceneNode中，这样只需要一行代码就可以把整个子场景添加到 rootNode 中，同样也可以用一行代码就把整个子场景移除掉。

    package net.jmecn.logic;
    
    import com.jme3.app.Application;
    import com.jme3.app.SimpleApplication;
    import com.jme3.app.state.AppState;
    import com.jme3.app.state.AppStateManager;
    import com.jme3.asset.AssetManager;
    import com.jme3.material.Material;
    import com.jme3.math.ColorRGBA;
    import com.jme3.math.FastMath;
    import com.jme3.renderer.RenderManager;
    import com.jme3.scene.Geometry;
    import com.jme3.scene.Mesh;
    import com.jme3.scene.Node;
    import com.jme3.scene.shape.Box;
    
    /**
     * 管理自场景的AppState
     * 
     * @author yanmaoyuan
     *
     */
    public class VisualAppState implements AppState {
        
        private boolean initialized = false;
        private boolean enabled = true;
    
        /**
         * 创建一个独立的根节点，便于管理子场景。
         */
        private Node sceneNode = new Node("MyScene");
        
        private Geometry cube = null;
        
        /**
         * 对于那些我们用得上的系统对象，保存一份对象的引用。
         */
        private SimpleApplication simpleApp;
        private AssetManager assetManager;
        
        @Override
        public void stateAttached(AppStateManager stateManager) {}
        
        @Override
        public void initialize(AppStateManager stateManager, Application app) {
            this.simpleApp = (SimpleApplication) app;
            this.assetManager = app.getAssetManager();
            
            // 创建一个方块
            Material mat = new Material(assetManager, "Common/MatDefs/Light/Lighting.j3md");
            mat.setColor("Diffuse", ColorRGBA.Red);
            mat.setColor("Ambient", ColorRGBA.Red);
            mat.setColor("Specular", ColorRGBA.Black);
            mat.setFloat("Shininess", 1);
            mat.setBoolean("UseMaterialColors", true);
            
            Mesh mesh = new Box(1, 1, 1);
            
            cube = new Geometry("Cube", mesh);
            cube.setMaterial(mat);
            
            // 将方块添加到我们这个场景中。
            sceneNode.attachChild(cube);
            
            // 初始化完毕
            initialized = true;
            
            if (enabled)
                simpleApp.getRootNode().attachChild(sceneNode);
        }
    
        @Override
        public boolean isInitialized() {
            return initialized;
        }
    
        @Override
        public void setEnabled(boolean active) {
            if ( this.enabled == active )
                return;
            this.enabled = active;
            
            if (!initialized)
                return;
            
            if (enabled) {
                simpleApp.getRootNode().attachChild(sceneNode);
            } else {
                sceneNode.removeFromParent();
            }
        }
    
        @Override
        public boolean isEnabled() {
            return enabled;
        }
        
        @Override
        public void update(float tpf) {
            cube.rotate(0, tpf * FastMath.PI, 0);
        }
        
        @Override
        public void render(RenderManager rm) {}
        
        @Override
        public void postRender() {}
    
        @Override
        public void stateDetached(AppStateManager stateManager) {}
    
        @Override
        public void cleanup() {
            if (enabled)
                sceneNode.removeFromParent();
            
            initialized = false;
        }
    
    }

#### LightAppState

下面是 LightAppState.java 的代码。仅仅是在 initialize() 中初始化了光源，没有什么特别的。

    package net.jmecn.logic;
    
    import com.jme3.app.Application;
    import com.jme3.app.SimpleApplication;
    import com.jme3.app.state.AppState;
    import com.jme3.app.state.AppStateManager;
    import com.jme3.light.AmbientLight;
    import com.jme3.light.PointLight;
    import com.jme3.math.ColorRGBA;
    import com.jme3.math.Vector3f;
    import com.jme3.renderer.RenderManager;
    import com.jme3.scene.Node;
    
    public class LightAppState implements AppState {
        
        private boolean initialized = false;
        private boolean enabled = true;
        
        // 点光源
        private Vector3f lightPos;
        private ColorRGBA pointLightColor;
        private PointLight pointLight;
        
        // 环境光
        private ColorRGBA ambientLightColor = new ColorRGBA(0.2f, 0.2f, 0.2f, 1f);
        private AmbientLight ambientLight;
        
        // 背景色
        private ColorRGBA bgColor = new ColorRGBA(0.7f, 0.8f, 0.85f, 1f);
        
        /**
         * 灯光应该引用到整个场景中，所以需要保存SimpleApplication中的根节点。
         */
        private Node rootNode;
        
        @Override
        public void initialize(AppStateManager stateManager, Application app) {
            SimpleApplication simpleApp = (SimpleApplication) app;
            this.rootNode = simpleApp.getRootNode();
            
            // 创建光源
            lightPos = new Vector3f(1, 2, 3);
            pointLightColor = new ColorRGBA(0.8f, 0.8f, 0.0f, 1f);
            pointLight = new PointLight(lightPos, pointLightColor);
            
            ambientLightColor = new ColorRGBA(0.2f, 0.2f, 0.2f, 1f);
            ambientLight = new AmbientLight(ambientLightColor);
    
            // 设置背景色，在关灯之后你依然能看到场景中漆黑的物体。
            app.getViewPort().setBackgroundColor(bgColor);
            
            // 初始化完毕
            initialized = true;
            
            if (enabled)
                turnOn();
        }
    
        // 开灯
        public void turnOn() {
            rootNode.addLight(pointLight);
            rootNode.addLight(ambientLight);
        }
        
        // 关灯
        public void turnOff() {
            rootNode.removeLight(pointLight);
            rootNode.removeLight(ambientLight);
        }
        
        @Override
        public boolean isInitialized() {
            return initialized;
        }
    
        @Override
        public void setEnabled(boolean active) {
            if ( this.enabled == active )
                return;
            this.enabled = active;
            
            if (!initialized)
                return;
            
            if (enabled) {
                turnOn();
            } else {
                turnOff();
            }
        }
    
        @Override
        public boolean isEnabled() {
            return enabled;
        }
    
        @Override
        public void stateAttached(AppStateManager stateManager) {}
    
        @Override
        public void stateDetached(AppStateManager stateManager) {}
    
        @Override
        public void update(float tpf) {}
    
        @Override
        public void render(RenderManager rm) {}
    
        @Override
        public void postRender() {}
    
        @Override
        public void cleanup() {
            if (enabled)
                turnOff();
            
            initialized = false;
        }
    
    }

#### InputAppState

下面是 InputAppState.java 的代码。

    package net.jmecn.logic;
    
    import com.jme3.app.Application;
    import com.jme3.app.state.AppState;
    import com.jme3.app.state.AppStateManager;
    import com.jme3.input.InputManager;
    import com.jme3.input.KeyInput;
    import com.jme3.input.controls.ActionListener;
    import com.jme3.input.controls.KeyTrigger;
    import com.jme3.input.controls.Trigger;
    import com.jme3.renderer.RenderManager;
    
    /**
     * 输入模块
     * @author yanmoayuan
     *
     */
    public class InputAppState implements AppState, ActionListener {
    
        // 电灯开关
        public final static String SWITCH_LIGHT = "switch_light";
        public final static Trigger TRIGGER_KEY_SPACE = new KeyTrigger(KeyInput.KEY_SPACE);
        
        // 显示/隐藏子场景
        public final static String TOGGLE_SUBSCENE = "toggle_subscene";
        public final static Trigger TRIGGER_KEY_TAB = new KeyTrigger(KeyInput.KEY_TAB);
        
        private boolean initialized = false;
        private boolean enabled = true;
        
        /**
         * 保存我们所需要的系统对象
         */
        private InputManager inputManager;
        private AppStateManager stateManager;
        
        @Override
        public void initialize(AppStateManager stateManager, Application app) {
            this.stateManager = stateManager;
            this.inputManager = app.getInputManager();
            
            initialized = true;
            
            if (enabled)
                addInputs();
        }
    
        /**
         * 添加输入
         */
        public void addInputs() {
            inputManager.addMapping(SWITCH_LIGHT, TRIGGER_KEY_SPACE);
            inputManager.addMapping(TOGGLE_SUBSCENE, TRIGGER_KEY_TAB);
            
            inputManager.addListener(this, SWITCH_LIGHT, TOGGLE_SUBSCENE);
        }
        
        /**
         * 移除输入
         */
        public void removeInputs() {
            inputManager.deleteTrigger(SWITCH_LIGHT, TRIGGER_KEY_SPACE);
            inputManager.deleteTrigger(TOGGLE_SUBSCENE, TRIGGER_KEY_TAB);
            
            inputManager.removeListener(this); 
        }
    
        @Override
        public void onAction(String name, boolean isPressed, float tpf) {
            if (isPressed) {
                if (SWITCH_LIGHT.equals(name)) {
                    
                    // 开关灯
                    LightAppState light = stateManager.getState(LightAppState.class);
                    if (light != null)
                        light.setEnabled(!light.isEnabled());
                    
                } else if (TOGGLE_SUBSCENE.equals(name)) {
                    // 显示/隐藏场景
                    VisualAppState visual = stateManager.getState(VisualAppState.class);
                    if (visual != null)
                        visual.setEnabled(!visual.isEnabled());
                }
            }
        }
        
        @Override
        public boolean isInitialized() {
            return initialized;
        }
    
        @Override
        public void setEnabled(boolean active) {
            if ( this.enabled == active )
                return;
            this.enabled = active;
            
            if (!initialized)
                return;
            
            if (enabled) {
                addInputs();
            } else {
                removeInputs();
            }
        }
    
        @Override
        public boolean isEnabled() {
            return enabled;
        }
    
        @Override
        public void stateAttached(AppStateManager stateManager) {}
    
        @Override
        public void stateDetached(AppStateManager stateManager) {}
    
        @Override
        public void update(float tpf) {}
    
        @Override
        public void render(RenderManager rm) {}
    
        @Override
        public void postRender() {}
    
        @Override
        public void cleanup() {
            if (enabled)
                removeInputs();
    
            initialized = false;
        }
    
    }

### 最佳实践

#### 使用抽象类

你并不需要总是去直接实现AppState接口，这个接口中需要重载的方法太多，而且每次都去重新写一遍几乎相同的 `setEnabled()` 方法也太无聊了。

jME3提供了2个抽象类，它们各自有一些基本实现。如果你懒得把 AppState 接口中的方法都实现一遍，可以尝试继承下面2个类。

* [com.jme3.app.state.BaseAppState](https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-core/src/main/java/com/jme3/app/state/BaseAppState.java)
* [com.jme3.app.state.AbstractAppState](https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-core/src/main/java/com/jme3/app/state/AbstractAppState.java)

BaseAppState是最好用的一个，而AbstractAppState更加简单粗暴。我不解释为什么，你应该在实际使用后根据自己的理解来决定怎么做。

下面是我常用的一个AxisAppState，基于BaseAppState实现。它的作用是在场景的中央显示一个参考坐标系，按<kbd>F4</kbd> 可以显示/隐藏坐标系。

    package net.jmecn.state;
    
    import com.jme3.app.Application;
    import com.jme3.app.SimpleApplication;
    import com.jme3.app.state.BaseAppState;
    import com.jme3.asset.AssetManager;
    import com.jme3.input.InputManager;
    import com.jme3.input.KeyInput;
    import com.jme3.input.controls.ActionListener;
    import com.jme3.input.controls.KeyTrigger;
    import com.jme3.material.Material;
    import com.jme3.math.ColorRGBA;
    import com.jme3.math.Vector3f;
    import com.jme3.renderer.queue.RenderQueue.ShadowMode;
    import com.jme3.scene.Geometry;
    import com.jme3.scene.Node;
    import com.jme3.scene.debug.Arrow;
    import com.jme3.scene.debug.Grid;
    
    /**
     * 坐标系
     * 
     * @author yanmaoyuan
     *
     */
    public class AxisAppState extends BaseAppState implements ActionListener {
    
        public final static String TOGGLE_AXIS = "toggle_axis";
    
        private AssetManager assetManager;
    
        private Node rootNode = new Node("AxisRoot");
    
        @Override
        protected void initialize(Application app) {
            assetManager = app.getAssetManager();
    
            // 网格
            Geometry grid = new Geometry("Grid", new Grid(21, 21, 1));
            Material mat = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");
            mat.setColor("Color", ColorRGBA.DarkGray);
            grid.setMaterial(mat);
            grid.center().move(0, 0, 0);
            grid.setShadowMode(ShadowMode.Off);
    
            rootNode.attachChild(grid);
    
            // 坐标
            createArrow("X", Vector3f.UNIT_X.mult(10), ColorRGBA.Red);
            createArrow("Y", Vector3f.UNIT_Y.mult(10), ColorRGBA.Green);
            createArrow("Z", Vector3f.UNIT_Z.mult(10), ColorRGBA.Blue);
    
            toggleAxis();
        }
    
        @Override
        protected void cleanup(Application app) {
        }
    
        @Override
        protected void onEnable() {
            SimpleApplication simpleApp = (SimpleApplication) getApplication();
            simpleApp.getRootNode().attachChild(rootNode);
    
            // 注册按键
            InputManager inputManager = getApplication().getInputManager();
            inputManager.addMapping(TOGGLE_AXIS, new KeyTrigger(KeyInput.KEY_F4));
            inputManager.addListener(this, TOGGLE_AXIS);
        }
    
        @Override
        protected void onDisable() {
            rootNode.removeFromParent();
    
            // 移除按键
            InputManager inputManager = getApplication().getInputManager();
            inputManager.removeListener(this);
            inputManager.deleteMapping(TOGGLE_AXIS);
    
        }
    
        @Override
        public void onAction(String name, boolean keyPressed, float tpf) {
            if (name.equals(TOGGLE_AXIS) && keyPressed) {
                toggleAxis();
            }
        }
    
        /**
         * 坐标轴开/关
         * 
         * @return
         */
        public boolean toggleAxis() {
            SimpleApplication simpleApp = (SimpleApplication) getApplication();
            if (simpleApp.getRootNode().hasChild(rootNode)) {
                simpleApp.getRootNode().detachChild(rootNode);
                return false;
            } else {
                simpleApp.getRootNode().attachChild(rootNode);
                return true;
            }
        }
    
        /**
         * 创建一个箭头
         * 
         * @param vec3
         *            箭头向量
         * @param color
         *            箭头颜色
         */
        private void createArrow(String name, Vector3f vec3, ColorRGBA color) {
            // 创建材质，设定箭头的颜色
            Material mat = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");
            mat.setColor("Color", color);
            mat.getAdditionalRenderState().setLineWidth(3f);
            mat.getAdditionalRenderState().setWireframe(true);
    
            // 创建几何物体，应用箭头网格。
            Geometry geom = new Geometry("Axis_" + name, new Arrow(vec3));
            geom.setMaterial(mat);
            geom.setShadowMode(ShadowMode.Off);
    
            // 添加到场景中
            rootNode.attachChild(geom);
        }
    }

效果：

![](/content/images/2017/06/AxisAppState.png)

#### 保存对全局对象的引用

你可以在AppState中保存一份对全局对象的引用，这样一来编码比较方便，另外编码的习惯会与在SimpleApplication中一致。

    public class MyAppState extends AbstractAppState {

        private SimpleApplication simpleApp;
        private AssetManager      assetManager;
        private AppStateManager   stateManager;
        private InputManager      inputManager;
        private ViewPort          viewPort;
        private Camera            camera;

        @Override
        public void initialize(AppStateManager stateManager, Application app) {
            super.initialize(stateManager, app);
            this.simpleApp    = (SimpleApplication) app;
            this.assetManager = app.getAssetManager();
            this.stateManager = app.getStateManager();
            this.inputManager = app.getInputManager();
            this.viewPort     = app.getViewPort();
            this.camera       = app.getCamera();
        }
    }

#### AppState之间的通信

虽说我们希望 AppState 的设计要符合**高内聚低耦合**原则，做开发时总是避免不了要与其他AppState交互。比较推荐的方式，是通过 AppStateManager 进行通信。

例如在InputAppState中，需要调用LightAppState中的方法，是这样做的：

    // 开关灯
    LightAppState light = stateManager.getState(LightAppState.class);
    if (light != null) {
        light.setEnabled(!light.isEnabled());
    }

AppStateManager的 `T getState(Class<T> stateClass)` 方法利用了Java的泛型机制，通过 AppState 的类型来查找实例。如果存在相同类型的的 AppState，就会返回这个对象，没有的话就会返回 null。

如果不顾虑返回值为空的情况，你甚至可以直接这么做：

    // 关灯
    stateManager.getState(LightAppState.class).turnOff();

**由于这个机制，AppStateManager 中相同类型的 AppState 对象只能有一个**。

如果想判断AppStateManager中是否有某个 AppState 对象，可以这么做：

    stateManager.has(myAppState);

#### 界面或场景切换

经常有人问这个问题：我该怎么切换主界面和游戏界面？

首先把主界面和游戏界面分别写到两个不同的AppState中，各自管理自己的子场景、输入处理等内容。例如：

* MainMenuAppState
* InGameAppState

然后，在主类中实例化这些 AppState，把主界面添加到AppStateManager中，作为游戏开始时的界面。

    public MyGame extends SimpleApplication {
        MainMenuAppState mainMenu;
        InGameAppState inGame;

        public void simpleInitApp() {
            mainMenu = new MainMenuAppState();
            inGame = new InGameAppState();

            stateManager.attach(mainMenu);
        }
    }

你想切换到游戏界面？写一个方法进行切换就好了。

        // 切换到游戏界面
        public void switchToInGame() {
            stateManager.detach(mainMenu);
            stateManager.attach(inGame);
        }

如果你不希望因为用户频繁切换界面导致 AppState 总是重复初始化，也可以一次性把2个 AppState都添加到 AppStateManager 中，然后把不需要显示的 AppState 先禁用掉。

        public void simpleInitApp() {
            stateManager.attachAll(new MainMenuAppState(),
                    new InGameAppState());
            stateManager.getState(InGameAppState.class).setEnabled(false);
        }

切换界面时使用 setEnable() 方法来代替 attach/detach。

        // 切换到游戏界面
        public void switchToInGame() {
            stateManager.getState(MainMenuAppState.class).setEnabled(false);
            stateManager.getState(InGameAppState.class).setEnabled(true);
        }

### jME3中的那些AppState

有4个AppState前面我们已经介绍过了，它们是：

* StatsAppState
* FlyCamAppState
* AudioListenerState
* DebugKeysAppState

除了这4个自动加载的 AppState 以外，jME3还提供了一些具有特殊功能的 AppState。

* `com.jme3.app.state.ScreenshotAppState` 它的作用是按 <kbd>Print Screen</kbd> 键截屏
* `com.jme3.app.BasicProfilerState` 它的作用是按 <kbd>F6</kbd> 键，以图形方式显示渲染效率。
* `com.jme3.app.ChaseCameraAppState` 它是一个第三人称摄像机，激活时将主动禁用FlyCamAppState。
* `com.jme3.cinematic.Cinematic` 它的作用是根据脚本播放剧情动画。
* `com.jme3.app.state.VideoRecorderAppState` 录制游戏画面，保存为avi文件。位于jme3-desktop模块中，**渣机慎用**。
* `com.jme3.bullet.BulletAppState` 集成Bullet物理引擎。位于jme3-bullet模块中。

## Control

`com.jme3.scene.control.Control`可以操纵单个游戏实体（Spatial）的行为，例如AnimControl就是用来控制模型动画的，MotionEvent可以遥控模型沿着固定轨迹移动。

每个Spatial可以绑定多个Control实例，每个Control控制一种特定的行为。通过一行代码，就可以把Control对象和Spatial绑定。

    spatial.addControl(Control control);

举个例子来说 ，我们可以根据NPC的不同行为，分别实现多个Control，并把它们绑定到一个NPC身上：

* PhysicalControl 配合物理引擎使用，控制NPC的运动；
* AiControl 负责控制NPC的智能行为，例如3D寻路等；
* AnimationContorl 负责在NPC处于不同状态时，播放对应的3D动画。
* TalkControl 负责控制NPC如何说话，根据说话的内容播放音频。
* FollowControl 让NPC能够跟随某个目标一起运动。
* 等等...

### 实例：物体自旋

下面是一个简单的自定义控件，它可以让模型绕Y轴以固定速率旋转。注意`update(float tpf)`方法，这是该控件内部的主循环。

    package net.jmecn.logic;
    
    import java.io.IOException;
    
    import com.jme3.export.JmeExporter;
    import com.jme3.export.JmeImporter;
    import com.jme3.math.FastMath;
    import com.jme3.renderer.RenderManager;
    import com.jme3.renderer.ViewPort;
    import com.jme3.scene.Spatial;
    import com.jme3.scene.control.Control;
    
    /**
     * 让模型绕Y轴以固定速率旋转
     * 
     * @author yanmaoyuan
     *
     */
    public class RotateControl implements Control {
    
        private Spatial spatial;
    
        // 旋转速度：每秒180°
        private float rotateSpeed = FastMath.PI;
    
        public RotateControl() {
            this.rotateSpeed = FastMath.PI;
        }
    
        public RotateControl(float rotateSpeed) {
            this.rotateSpeed = rotateSpeed;
        }
    
        @Override
        public void setSpatial(Spatial spatial) {
            this.spatial = spatial;
        }
    
        @Override
        public void update(float tpf) {
            spatial.rotate(0, tpf * rotateSpeed, 0);
        }
    
        @Override
        public void render(RenderManager rm, ViewPort vp) {
        }
    
        @Override
        public void write(JmeExporter ex) throws IOException {
            throw new IOException("暂不支持");
        }
    
        @Override
        public void read(JmeImporter im) throws IOException {
            throw new IOException("暂不支持");
        }
    
        @Override
        public Control cloneForSpatial(Spatial spatial) {
            RotateControl c = new RotateControl(rotateSpeed);
            c.setSpatial(spatial);
            return c;
        }
    }

下面这个代码，RotateControl让方块绕Y轴旋转。

    package net.jmecn.logic;
    
    import com.jme3.app.SimpleApplication;
    import com.jme3.light.DirectionalLight;
    import com.jme3.material.Material;
    import com.jme3.math.FastMath;
    import com.jme3.math.Quaternion;
    import com.jme3.math.Vector3f;
    import com.jme3.scene.Geometry;
    import com.jme3.scene.shape.Box;
    
    /**
     * 主循环
     * 
     * @author yanmaoyuan
     *
     */
    public class HelloControl extends SimpleApplication {
    
        @Override
        public void simpleInitApp() {
            cam.setLocation(new Vector3f(3.3435764f, 3.7595856f, 6.611723f));
            cam.setRotation(new Quaternion(-0.05573249f, 0.9440857f, -0.23910178f, -0.22006002f));
    
            // 创建一个方块
            Material mat = new Material(assetManager, "Common/MatDefs/Light/Lighting.j3md");
            Geometry spatial = new Geometry("Box", new Box(1, 1, 1));
            spatial.setMaterial(mat);
    
            // 添加控件
            spatial.addControl(new RotateControl(FastMath.PI));
    
            rootNode.attachChild(spatial);
    
            // 添加光源
            rootNode.addLight(new DirectionalLight(new Vector3f(-1, -2, -3)));
        }
    
        public static void main(String[] args) {
            // 启动
            HelloControl app = new HelloControl();
            app.start();
        }
    
    }

### AbstractControl

与 AppState 一样，你也没有必要直接实现 Control 接口。我们可以通过继承 AbstractControl 来节省代码。

[com.jme3.scene.control.AbstractControl](https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-core/src/main/java/com/jme3/scene/control/AbstractControl.java)

AbstractControl 中做了很多基础工作，比如保存被绑定的Spatial引用，增加 setEnabled() 方法让你可以激活/禁用某个Control的功能。

比较特别是的，继承AbstractControl的话，我们的主循环就不能写在 update() 方法中了，而是改为写在 `controlUpdate()` 方法中。

#### 实例：让物体上下浮动

下例的FloatControl基于AbstractControl，作用是让物体做上下往返运动。

    package net.jmecn.logic;
    
    import com.jme3.renderer.RenderManager;
    import com.jme3.renderer.ViewPort;
    import com.jme3.scene.control.AbstractControl;
    
    /**
     * 一个让目标上下浮动的控件
     * 
     * @author yanmaoyuan
     *
     */
    public class FloatControl extends AbstractControl {
    
        float tmp = 0;
        boolean raise = true;
    
        float dist;// 上下浮动的距离
        float speed;// 浮动的速度
    
        public FloatControl() {
            this.dist = 0.5f;
            this.speed = 1f;
        }
    
        public FloatControl(float dist, float speed) {
            this.dist = dist;
            this.speed = speed;
        }
    
        @Override
        protected void controlUpdate(float tpf) {
            if (speed == 0)
                return;
    
            float delta = tpf * speed;
    
            if (tmp < dist && raise) {
                tmp += delta;
                spatial.move(0, delta, 0);
            } else {
                raise = false;
            }
    
            if (tmp > -dist && !raise) {
                tmp -= delta;
                spatial.move(0, -delta, 0);
            } else {
                raise = true;
            }
        }
    
        @Override
        protected void controlRender(RenderManager rm, ViewPort vp) {
        }
    };

代码量是不是比直接实现Control接口少多了？

#### 实例：让物体朝目标移动

下例的MotionControl也不算很复杂，它的作用是让物体朝目标点做直线运动。

    package net.jmecn.game;
    
    import com.jme3.math.Vector3f;
    import com.jme3.renderer.RenderManager;
    import com.jme3.renderer.ViewPort;
    import com.jme3.scene.Spatial;
    import com.jme3.scene.control.AbstractControl;
    
    /**
     * 这是一个运动控件，其作用是让模型朝目标点直线运动。
     * 
     * @author yanmaoyuan
     *
     */
    public class MotionControl extends AbstractControl {
        
        // 运动速度
        private float walkSpeed = 1.0f;
        private float speedFactor = 1.0f;
        
        // 运动的方向向量
        private Vector3f walkDir;
        // 运动一步的向量
        private Vector3f step;
        
        // 当前位置
        private Vector3f loc;
        // 目标位置
        private Vector3f target;
    
        // 观察者
        private Observer observer;
    
        public MotionControl() {
            this(1.0f);
        }
        
        public MotionControl(float walkSpeed) {
            this.walkSpeed = walkSpeed;
            walkDir = null;
            target = null;
            loc = new Vector3f();
            step = new Vector3f();
        }
        
        /**
         * 设置运动速度
         * @param walkSpeed
         */
        public void setWalkSpeed(float walkSpeed) {
            this.speedFactor = walkSpeed;
        }
        
        /**
         * 设置观察者
         * @param observer
         */
        public void setObserver(Observer observer) {
            this.observer = observer;
        }
    
        /**
         * 设置目标点
         * 
         * @param target
         */
        public void setTarget(Vector3f target) {
            this.target = target;
    
            if (target == null) {
            	walkDir = null;
            	return;
            }
            
            // 当模型面朝目标点
            this.spatial.lookAt(target, Vector3f.UNIT_Y);
    
            // 计算运动方向
            walkDir = target.subtract(loc);
            walkDir.normalizeLocal();
            
        }
        
        @Override
        public void setSpatial(Spatial spatial) {
            super.setSpatial(spatial);
            // 初始化位置
            loc = new Vector3f(spatial.getLocalTranslation());
        }
    
        /**
         * 重写主循环，让这个模型向目标点移动。
         */
        @Override
        protected void controlUpdate(float tpf) {
            if (walkDir != null) {
    
                // 计算下一步的步长
                float stepDist = walkSpeed * tpf * speedFactor;
                
                if (stepDist == 0f) {
                	return;
                }
    
                // 计算离目标点的距离
                float dist = loc.distance(target);
    
                if (stepDist < dist) {
                    // 计算位移
                    walkDir.mult(stepDist, step);
                    loc.addLocal(step);
                    
                    spatial.setLocalTranslation(loc);
                    
                } else {
                    // 可以到达目标点
                    walkDir = null;
                    
                    spatial.setLocalTranslation(target);
                    target = null;
    
                    // 通知观察者，已经抵达目标点了。
                    if (observer != null) {
                    	observer.onReachTarget();
                    }
                }
    
            }
        }
    
        @Override
        protected void controlRender(RenderManager rm, ViewPort vp) {
        }
    
    }

### 与Control通信

Control是被绑定到Spatial上的，通过Spatial的 `T getControl(Class<T> class)` 方法可以查询Spatial对象身上的Control对象。

    AnimContorl animControl = spatial.getControl(AnimContorl .class);

与 AppStateManager 中的 `T getState(Class<T> class)` 方法类似，getContorl（）方法同样使用了泛型机制。一旦查询到Spatial中绑定了相同类型的Control，就会直接返回Control的实例。

在不考虑返回值为null的情况下（并不是什么好习惯），我们可以直接调用Control的方法。基于这种特性，我们就可以在Control访问绑定到同一个Spatial的其他Control，并实现Control之间的通信。

    spatial.getControl(MotionContorl.class).setTarget(new Vector3f(10f, 0f, 12f));

### jME3中的Control

jME3有一些定义好的Control，各有用途。

* `com.jme3.scene.control.BillboardControl` 这个Control能够让物体的正面始终对准摄像机。
* `com.jme3.scene.control.CameraControl` 让摄像机跟随某个物体，或者让物体跟随摄像机。
* `com.jme3.scene.control.LightControl` 让光源跟随某个物体，或者让物体跟随光源。
* `com.jme3.scene.control.LodControl` 根据视野距离自动调整模型的层次细节（Level Of Detail），用于性能优化。

## 多线程优化

### 为什么要用多线程

首先，从操作系统的角度来看，**多线程并不比单线程运行得更快**。因为CPU除了进行计算以外，还要额外负责线程的调度工作。

然而，CPU并不仅仅是在做计算。程序运行时，CPU绝大多数时间是在**等待要计算的数据**。打个比方，我（CPU）吃完一锅莲藕排骨汤只需要十几分钟，但是把汤熬好却要准备大半天。

多线程的目的是为了最大限度的利用CPU资源。这就像在等待排骨汤熬好的这段时间，我还可以煮个饭、炒个菜。

### 优化思路

1. 图形渲染线程和逻辑线程并行

2. I/O线程与其他线程并行

3. 逻辑线程并行（难）


### jME3的多线程模型

### 线程同步
