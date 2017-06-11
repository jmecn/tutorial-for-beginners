# 第九章：用户交互

## 游戏控制器

游戏控制器是玩家于游戏交互的主要设备，大部分游戏都是通过控制器来互动的，比如：

* 街机
 * 游戏板、操纵杆、跳舞毯、方向盘
* 主机（XBOX、PlayStation等）
 * 手柄、专业方向盘、Kinect体感传感器等
* 个人电脑（PC）
 * 键盘、鼠标
* 手机、平板电脑
 * 触屏、各种传感器（重力传感器、加速度传感器、陀螺仪等）
* 穿戴式设备（VR）
 * 手持控制器、各种传感器（重力传感器、加速度传感器、陀螺仪等）

不同的游戏有不同的玩法，但绝大多数时刻开发者都会允许用户使用不同的控制器来进行交互。比如赛车类游戏最适合使用专业方向盘来玩，但是在PC上还可以通过键盘+鼠标来控制赛车。

![方向盘](/content/images/2017/05/wheel.png)

jME3默认支持的游戏控制器有如下4种：

* com.jme3.input.MouseInput（鼠标）
* com.jme3.input.KeyInput（键盘）
* com.jme3.input.JoyInput（游戏操纵杆或者手柄）
* com.jme3.input.TouchInput（智能手机的触摸屏）

在SimpleApplication中，可以通过判断是否为null的方式来检测游戏当前的运行环境是否有这些输入设备，例如：

	package net.jmecn;
	
	import com.jme3.app.SimpleApplication;
	
	/**
	 * 用户交互
	 * @author yanmaoyuan
	 *
	 */
	public class HelloInput extends SimpleApplication {
	
		@Override
		public void simpleInitApp() {
			// 检测输入设备
			System.out.printf("Mouse: %b\nKeyboard: %b\nJoystick: %b\nTouch: %b\n",
					mouseInput != null, keyInput != null, joyInput != null, touchInput != null);
		}
	
		public static void main(String[] args) {
			HelloInput app = new HelloInput();
			app.start();
		}
	
	}

在PC上运行的结果如下，意思是存在鼠标和键盘，但是没有游戏手柄和触摸屏。

    Mouse: true
    Keyboard: true
    Joystick: false
    Touch: false

## 事件处理机制

jME3通过`InputManager`来管理各种游戏控制器的输入，InputManager的工作原理基于**消息/事件机制**。

事件处理机制是一种应用范围非常广泛的用户交互模型，在很多编程语言中都有它的身影，它是**观察者模式**的一种具体应用。一般的事件处理机制中会有3个主要参与者：

* 事件源
* 事件(Event)
* 事件监听器(EventListener)

事件源是输入的来源，而事件(Event)则是由事件源发出的输入信号，事件监听器是负责处理事件的对象。一般来说，事件源和事件是由系统决定的，而事件监听器则是由开发者编写的。

事件(Event)对象一般都会携带着事件发生时产生的各种数据，而事件监听器就可以根据这些数据来决定如何响应。例如用户敲击键盘后，键盘事件(KeyEvent)会记录用户敲击的是哪个键，以及键盘的`按下/弹起`状态。开发者可以根据这些数据干很多事情，比如判断按下的是不是空格键，如果是的话就让玩家控制的角色做出"跳跃"动作。

然而，在有些系统中，事件并不携带数据，或者说事件被区分为“事件消息”和“事件数据”两部分。例如在C++的MFC框架中，窗口中的某个按钮只需要用一个宏(整数)来表示。因为很多时候开发者只关心这个按钮是否被按下了，不在乎用户点击按钮时鼠标的坐标。

在jME3中，事件机制和消息机制都有所应用，在本文的后续部分我们会进一步介绍。

### HTML

下面是一段HTML代码，其中超链接`<a>`是事件源，当用户点击这个超链接时，就会产生一个`onclick`事件，由函数`function clickMe()`处理，`clickMe`就是事件监听器。

    <script>
    function clickMe()
    {
        alert("Hi!")
    }
    </script>
    <a href="#" onclick="clickMe();">点我</a>

### Android

在Android中，事件源通常是各种View控件。当用户触摸屏幕后，会触发OnTouch事件，Android将会执行开发者编写的`OnTouchListener`代码。

    private Button btn;
    public void onCreate(Bundle bundle) {
        setContentView(R.layout.main);
        btn = (Button) findViewById(R.id.btn);
        btn.setOnTouchListener(new OnTouchListener() {
            @Override
            public boolean onTouch(View v, MotionEvent e) {
                Log.d("Debug", "Hi!");
                return false;
            }
        });
    }

## jME3中的事件

### 事件源

在jME3中，事件源指的就是各种游戏控制器，比如键盘、鼠标等，jME3把这些事件源称为“触发器（`com.jme3.input.controls.Trigger`）”。jME3一共有6种触发器，对应4类输入设备。

* 键盘
 * KeyTrigger，通过键盘按键触发。
* 鼠标
 * MouseAxisTrigger，通过划动鼠标触发。
 * MouseButtonTrigger，通过点击鼠标左键/右键/滚轮触发。
* 游戏手柄或操纵杆
 * JoyAxisTrigger，通过拨动操纵杆触发。
 * JoyButtonTrigger，通过手柄上的按键触发。
* 触摸屏
 * TouchTrigger，通过触摸手机屏幕触发。

注：jME3暂不支持智能手机的传感器(Sensor)，如果想把手机传感器作为事件源，需要开发者自行扩展功能。VR设备的输入设备同样没有jME3的原生支持，需要用过jme3-vr组件进行扩展。

### 事件对象

玩家操作这些设备时，会产生输入事件(`com.jme3.input.event.InputEvent`)对象，这些对象携带有事件发生时的一些数据。jME3中有6种事件，对应6种触发器。

* 键盘事件
 * KeyInputEvent 记录键盘的按键值、按下状态等；
* 鼠标事件
 * MouseMotionEvent 记录鼠标的坐标、运动方向、滚轮运动方向等；
 * MouseButtonEvent 记录鼠标的按键值(左键/右键/滚轮)、坐标、按下状态等；
* 游戏手柄和操纵杆事件
 * JoyAxisEvent 记录操纵杆的坐标、运动方向等；
 * JoyButtonEvent 记录手柄的按键值、按钮状态等；
* 触摸屏事件
 * TouchEvent 记录触摸的坐标、触摸的状态、压力大小等；

### 事件监听器

jME3支持消息模型和事件模型，分别采用不同的输入监听器来实现。

* 消息监听器
 * ActionListener，数字信号监听器
 * AnalogListener，模拟信号监听器
 * TouchListener，触摸屏监听器
* 事件监听器
 * RawInputListener，原始输入监听器

## jME3中的事件处理

jME3通过InputManager来管理用户的输入，触发器、消息、监听器都要在InputManager中注册后才能生效。

对于消息监听器，开发者需要在InputManager中通过`addMapping`和`addListener`来绑定触发器、消息和监听器的关系。

对于事件监听器，只需要通过InputManager中的`addRawInputListener`来注册即可。

### 消息模型

假设我们在开发一款射击游戏，玩家控制一辆坦克。无论玩家点击"鼠标左键"或按下键盘上的"空格键"，都会让坦克朝目标开火。

具体步骤是这样的：

1. 定义消息
2. 绑定消息和触发器
3. 定义监听器
4. 绑定消息和监听器

**1 定义消息**

jME3中的消息是Stirng类型，我们可以将程序中的各种消息定义为字符串常量，例如：

    /**
     * 开火消息
     */
    public final static String FIRE = "Fire";

**2 绑定消息和触发器**

在simpleInitApp中，可以通过inputManager的addMapping方法来绑定消息和触发器。**一个消息可以绑定多个触发器。**

InputManager会监控玩家的输入，点击鼠标左键或者按下键盘上的空格键时，就会触发这个消息。

	/**
	 * 开火消息
	 */
	public final static String FIRE = "Fire";
		
	@Override
	public void simpleInitApp() {
		// 绑定消息和触发器
		inputManager.addMapping(FIRE, 
				new KeyTrigger(KeyInput.KEY_SPACE), 
				new MouseButtonTrigger(MouseInput.BUTTON_LEFT));
	}

**3 定义监听器**

事件监听器会在消息被触发后执行，jME3定义了3种监听器接口，应用于不同的场合：

`ActionListener`，适用于敲击键盘、点击鼠标、按下按钮等一次性事件，又称为数字信号监听器。

`AnalogListener`，适用于监听鼠标滑动轨迹、按下按钮不释放等持续事件，又称为模拟信号监听器。

`TouchListener`，适用于触屏事件。

定义一个ActionListener，当玩家触发FIRE消息后，在控制台下输出"bang!"。

	// 监听器
	class MyActionListener implements ActionListener {
		@Override
		public void onAction(String name, boolean isPressed, float tpf) {
			if (FIRE.equals(name) && isPressed) {
				System.out.println("bang!");
			}
		}
	}

**4 绑定消息和监听器**

通过InputManager的addListener方法可以绑定消息和监听器，**一个监听器可以绑定多个消息**。

代码如下：

	package net.jmecn;
	
	import com.jme3.app.SimpleApplication;
	import com.jme3.input.KeyInput;
	import com.jme3.input.MouseInput;
	import com.jme3.input.controls.ActionListener;
	import com.jme3.input.controls.KeyTrigger;
	import com.jme3.input.controls.MouseButtonTrigger;
	
	/**
	 * 用户交互
	 * @author yanmaoyuan
	 *
	 */
	public class HelloInput extends SimpleApplication {
	
		public static void main(String[] args) {
			HelloInput app = new HelloInput();
			app.start();
		}
	
		/**
		 * 开火消息
		 */
		public final static String FIRE = "Fire";
		
		@Override
		public void simpleInitApp() {
			// 绑定消息和触发器
			inputManager.addMapping(FIRE, 
					new KeyTrigger(KeyInput.KEY_SPACE), 
					new MouseButtonTrigger(MouseInput.BUTTON_LEFT));
			
			// 绑定消息和监听器
			inputManager.addListener(new MyActionListener(), FIRE);
		}
		
		// 监听器
		class MyActionListener implements ActionListener {
			@Override
			public void onAction(String name, boolean isPressed, float tpf) {
				if (FIRE.equals(name) && isPressed) {
					System.out.println("bang!");
				}
			}
		}
	
	}

运行这个程序，无论玩家按下鼠标左键或是键盘上的空格键，控制台下都会输出"bang!"。

### 事件模型

jME3通过RawInputListener来直接处理事件对象。

"Raw"的意思是“原始”，“Raw Input Listener”的意思是原始输入监听器。这意味着着InputManager不会对玩家的输入做任何额外封装，事件对象将会直接交给这个监听器。

具体使用时，首先实现RawInputListener接口，然后通过InputManager的`addRawInputListener`方法来使其生效。

	@Override
	public void simpleInitApp() {
		// 原始输入监听器
		inputManager.addRawInputListener(new MyRawInputListener());
	}

	// 原始输入监听器
	class MyRawInputListener implements RawInputListener {

		/**
		 * 键盘输入事件
		 */
		@Override
		public void onKeyEvent(KeyInputEvent evt) {
			int keyCode = evt.getKeyCode();
			boolean isPressed = evt.isPressed();
			
			//当玩家按下Y键时，输出"Yes!"
			if (isPressed) {
				switch (keyCode) {
				case KeyInput.KEY_Y: {
					System.out.println("Yes!");
					break;
				}
				}
			}
		}
		
		/**
		 * 鼠标输入事件
		 */
		@Override
		public void onMouseMotionEvent(MouseMotionEvent evt) {
			int x = evt.getX();
			int y = evt.getY();
			// 打印鼠标的坐标
			System.out.println("x=" + x + " y="+y);
		}

		@Override public void onMouseButtonEvent(MouseButtonEvent evt) {}
		
		@Override public void beginInput() {}

		@Override public void endInput() {}

		@Override public void onJoyAxisEvent(JoyAxisEvent evt) {}

		@Override public void onJoyButtonEvent(JoyButtonEvent evt) {}

		@Override public void onTouchEvent(TouchEvent evt) {}
	}

这个监听器做了2件事：

1. 当玩家按下Y键时，在控制台下输出"Yes!"
2. 当玩家滑动鼠标时，在控制台下实时输出鼠标当前的XY坐标。

这个监听器可以通过InputManager获得玩家输入的所有信息，功能最为强大，代价是代码的复杂度会迅速增加。主要原因是jME3没有为每一种事件定义一种监听器，而是通过一个RawInputListener来接收了所有事件。

## 耗时操作

jME3的事件处理是在主线程中进行的，这意味着可以直接在监听器代码中操作场景图，另外也意味着耗时操作将会导致程序卡顿。

例如我们定义一个LOAD消息，当玩家按下L键后加载寒冰射手艾希的模型。

    /**
     * 开火消息
     */
    public final static String FIRE = "Fire";
    /**
     * 加载模型
     */
    public final static String LOAD = "Load";

    @Override
    public void simpleInitApp() {
        // 初始化输入
        initInput();
    }

    /**
     * 初始化输入
     */
    private void initInput() {
        // 绑定消息和触发器
        inputManager.addMapping(FIRE, new KeyTrigger(KeyInput.KEY_SPACE),
                new MouseButtonTrigger(MouseInput.BUTTON_LEFT));

        inputManager.addMapping(LOAD, new KeyTrigger(KeyInput.KEY_L));

        // 绑定消息和监听器
        inputManager.addListener(new MyActionListener(), FIRE, LOAD);
    }

    // 事件监听器
    class MyActionListener implements ActionListener {
        @Override
        public void onAction(String name, boolean isPressed, float tpf) {
            if (isPressed) {
                if (FIRE.equals(name)) {
                    System.out.println("bang!");
                } else if (LOAD.equals(name)) {
                    loadModel();
                }
            }
        }
    }

    /**
     * 加载模型
     */
    private void loadModel() {
        // 导入模型
        final Spatial model = assetManager.loadModel("Models/Ashe/b_ashe_b.obj");
        model.scale(0.03f);// 按比例缩小
        model.center();// 将模型的中心移到原点
                
        // 将模型添加到场景图中。
        rootNode.attachChild(model);
    }

当玩家按下L键之后，需要数秒的时间才能加载完这个模型。在这段时间，程序的画面是卡顿的。

类似加载模型这种耗时操作，最佳方式是在事件发生后开启一个子线程，并在线程的工作完成后，再通知主线程将模型添加到场景中。

    /**
     * 加载模型
     */
    private void loadModel() {
        // 开启一个子线程
        new Thread () {
            public void run() {
                // 在子线程中导入模型
                final Spatial model = assetManager.loadModel("Models/Ashe/b_ashe_b.obj");
                model.scale(0.03f);// 按比例缩小
                model.center();// 将模型的中心移到原点
                
                // 通知主线程，将模型添加到场景图中。
                enqueue(new Runnable() {
                    public void run() {
                        rootNode.attachChild(model);
                    }
                });
            }
        }.start();
    }

需要注意的是，**子线程不可以改变游戏主场景图！**不可以在子线程中为rootNode添加/移除Spatial。主线程要负责场景渲染，如果在渲染的过程中改变场景，就可能会因为场景和画面不同步而导致异常。

为了保证场景和画面同步，模型只能在主线程渲染画面结束后才能更改主场景。jME3内部提供了一些与线程同步有关的API，`enqueue`方法正是其中之一，它的作用是把任务提交给主线程，这些任务会在画面渲染结束后执行。

在上面这段修改过的`loadModel`方法被分成了两部分，加载模型的部分在子线程中进行，更新场景图的部分则通过`enqueue`交给主线程来执行。这样做既可以避免画面卡顿，又可以保证线程同步。

关于jME3中线程同步的问题，本章就先介绍到这里。

## 本章源码

本章中出现的全部代码，可以直接通过github阅读，地址：[net.jmecn.HelloInput](https://github.com/jmecn/jME3Tutorials/blob/master/jME3Tutorials/src/main/java/net/jmecn/HelloInput.java)

希望通过阅读本章内容之后，你能同样看懂官方教程的代码：[jme3test.helloworld.HelloInput](https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-examples/src/main/java/jme3test/helloworld/HelloInput.java)
