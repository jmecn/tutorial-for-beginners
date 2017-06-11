# 第十一章：3D音效

## 3D音效

本章将介绍如何在jME3中实现3D音效，了解音源（Audio Source）、音频侦听器（Audio Listener）、混响（Reverb）等概念及其应用。

jME3支持2种音频资源格式：WAV和OGG。使用OGG格式需要在项目中添加jme3-jogg类库的依赖：

    dependencies {
        compile 'org.jmonkeyengine:jme3-jogg:3.1.0-stable'
    }

先看一段样例代码，在场景中创建两个音源，其中一个会在玩家点击鼠标左键时发出枪声，另一个会循环播放海潮声。

> 说明：本章所使用的所有音频文件都来自于jME3 SDK自带的`jme3-testdata.jar`。

> 假设你使用jME3 SDK，首先右键单击项目名，在弹出菜单中选择“Properties”，然后打开“Libraries”选项卡，点击“Add Library”后选择“jme3-test-data”即可将其添加到项目中。

> 若你使用其他IDE，但安装了jME3 SDK，可以在SDK安装目录的`./jmonkeyplatform/jmonkeyplatform/libs`文件夹中找到这个jar文件。

> 此外，你还可以使用git克隆jMonkeyEngine的源代码，然后在`jmonkeyengine/jme3-testdata/src/main/resources`目录下找到这些资源文件。

### 样例代码

	package net.jmecn;
	
	import com.jme3.app.SimpleApplication;
	import com.jme3.audio.AudioData.DataType;
	import com.jme3.audio.AudioNode;
	import com.jme3.input.MouseInput;
	import com.jme3.input.controls.ActionListener;
	import com.jme3.input.controls.MouseButtonTrigger;
	import com.jme3.material.Material;
	import com.jme3.math.ColorRGBA;
	import com.jme3.scene.Geometry;
	import com.jme3.scene.shape.Box;
	
	/**
	 * 3D音效
	 * 
	 * @author yanmaoyuan
	 *
	 */
	public class HelloAudio extends SimpleApplication {
	
		public static void main(String[] args) {
			// 启动程序
			HelloAudio app = new HelloAudio();
			app.start();
		}
		
		/**
		 * 枪声
		 */
		private AudioNode audioGun;
		/**
		 * 环境音效
		 */
		private AudioNode audioNature;
	
		/**
		 * 开枪
		 */
		private final static String SHOOT = "Shoot";
	
		@Override
		public void simpleInitApp() {
			flyCam.setMoveSpeed(40);
	
			/**
			 * 制作一个蓝色的小方块，用来表示音源的位置。
			 */
			Geometry geom = new Geometry("Player", new Box(1, 1, 1));
			Material mat = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");
			mat.setColor("Color", ColorRGBA.Blue);
			geom.setMaterial(mat);
			rootNode.attachChild(geom);
	
			/**
			 * 初始化用户输入
			 */
			initKeys();
			
			/**
			 * 初始化音效
			 */
			initAudio();
	
		}
	
		/**
		 * 保持音频监听器（Audio Listener）和摄像机（Camera）同步运动。
		 */
		@Override
		public void simpleUpdate(float tpf) {
			listener.setLocation(cam.getLocation());
			listener.setRotation(cam.getRotation());
		}
	
		/**
		 * 创建两个AudioNode作为音源，并添加到场景中。
		 */
		private void initAudio() {
			/**
			 * 创建一个“枪声”音源，用户点击鼠标时会发出枪声。
			 */
			audioGun = new AudioNode(assetManager, "Sound/Effects/Gun.wav", DataType.Buffer);
			audioGun.setLooping(false);// 禁用循环播放
			audioGun.setPositional(false);// 设置为非定位音源，玩家无法通过耳机辨别音源的位置。常用于背景音乐。
			audioGun.setVolume(2);
			// 将音源添加到场景中
			rootNode.attachChild(audioGun);
	
			/**
			 * 创建一个自然音效（海潮声），这个音源会一直循环播放。
			 */
			audioNature = new AudioNode(assetManager, "Sound/Environment/Ocean Waves.ogg", DataType.Stream);
			audioNature.setLooping(true); // 循环播放
			audioNature.setPositional(true);// 设置为定位音源，这将产生3D音效，玩家能够通过耳机来辨别音源的位置。
			audioNature.setVolume(3);// 音量
			// 将音源添加到场景中
			rootNode.attachChild(audioNature);
			
			audioNature.play(); // 持续播放
		}
	
		/**
		 * 定义“开枪(SHOOT)”动作，用户点击鼠标左键时触发此动作。
		 */
		private void initKeys() {
			inputManager.addMapping(SHOOT, new MouseButtonTrigger(MouseInput.BUTTON_LEFT));
			inputManager.addListener(actionListener, SHOOT);
		}
	
		/**
		 * 定义一个监听器，用来处理开枪动作，当玩家点击鼠标左键时发出枪声。
		 */
		private ActionListener actionListener = new ActionListener() {
			@Override
			public void onAction(String name, boolean isPressed, float tpf) {
				if (SHOOT.equals(name) && isPressed) {
					audioGun.playInstance(); // 只播放一次
				}
			}
		};
	
	}

运行此程序，屏幕上中会出现一个蓝色方块，它在场景中的位置和两个AudioNode是重叠的。你会听到一个类似海潮声的环境音效循环播放；点击鼠标左键会听到枪声。

### AudioNode

在jME3中，AudioNode代表场景中的音源，可用于播放音频资源。

AudioNode既可以表示定位（Positional）音源，也可以表示环境（Ambient）音源，默认为定位音源。当你把一个AudioNode添加到3D场景中后，它的位置和运动速度会影响玩家听到的声音，如距离衰减、多普勒效应等。

在**定位音源**模式下，AudioNode的音频资源必须是单声道（mono）的，左右耳机中的声音会根据音频侦听器（Audio Listener）和音源的相对位置来计算，玩家可以听音辨位。

在**环境音源**模式下，AudioNode可以支持立体声（stero）音效（即音频文件本身中包含多个声道，玩家左右耳会听到不一样的声音）。此时音源的位置、运动速度等参数都不再生效，玩家将无法分辨声音的来源，这种情况更适合播放背景音乐。

通过AudioNode#setPositional(boolean)方法可以设置音源的定位模式。

> 注意：在定位音源模式下，如果音频文件不是单声道（mono）的，jME3将会抛出异常，并提示"Only mono audio is supported for positional audio nodes"。

### 创建音源

本章的样例代码中定义了两个AudioNode，分别作为枪声和环境声的音源。

    /**
     * 枪声
     */
    private AudioNode audioGun;
    /**
     * 环境音效
     */
    private AudioNode audioNature;

在jME3中，每个音源都需要创建一个AudioNode，而且每个AudioNode只能关联一个音频资源文件。

这两个音源在`initAudio()`方法中初始化，有效代码一共2行。

        /**
         * 创建一个“枪声”音源，用户点击鼠标时会发出枪声。
         */
        audioGun = new AudioNode(assetManager, "Sound/Effects/Gun.wav", DataType.Buffer);
        ...
        audioNature = new AudioNode(assetManager, "Sound/Environment/Ocean Waves.ogg", DataType.Stream);

这里创建了2个AudioNode对象，并通过AssetManager来加载各自的音频文件。DataType.Buffer表示将音频数据解码后缓存到内存中，缓存完毕后才可以播放声音；DataType.Stream表示以“流”模式打开音频文件，这对体积比较大的音乐文件非常有意义。

随后的代码设置了音源的一些参数。

<table>
  <tr>
    <th>方法</th><th>说明</th>
  </tr>
  <tr>
    <td>setLooping(boolean)</td><td>设置循环模式：true 循环播放；false 单曲播放</td>
  </tr>
  <tr>
    <td>setVolume(float)</td><td>设置音量，0.0代表最小值，1.0代表最大值。</td>
  </tr>
  <tr>
    <td>setPitch(float)</td><td>设置音源数据的频率（采样率）倍数，取值范围是0.5至2.0。大于1.0倍，播放会加速；小于1.0，播放时间会被拉长。</td>
  </tr>
  <tr>
    <td>setTimeOffset(float)</td><td>设置播放时间的偏移量（单位：秒）。例：从音乐的第10秒处开始播放，audioNode.setTimeOffset(10);。<br>
注意，这个方法在ogg文件+流模式下无法工作，因为ogg格式不支持播放时间。</td>
  </tr>
</table>

### 流和缓存

1、流（DataType.Stream）
播放音乐时，AudioNode将以流的形式音频文件，并在播放的同时解码音频数据。这种做法的好处是可以省内存，而且在音频时间很长或者音频文件很大的情况下比较好用。 

2、缓存（DataType.Buffer）
在播放音乐之前，jme3将会对音频文件进行解码，并把解码后的数据加载到缓存中，缓存完毕后才可以播放。

一般来说，短促的音效适合以缓存模式的打开，时间较长的音乐适合以流模式打开。

### play和playInstance

AudioNode有2个播放方法，play()和playInstance()。playInstance()方法只对以缓存（DataType.Buffer）模式加载的音源有效，play()对两种模式都有效。

每调用一次`playInstance()`方法，jME3都会单独播放这个音效，典型应用场景就是一声枪响还没结束，马上又响起了另一声枪响，这样可以制造出枪声大作的效果。

调用`play()`方法来播放音乐，在播放结束之前调用play()方法是没有作用的。你可以通过AudioNode的`getStatus()`方法来查询播放状态，其返回结果是一个枚举类型：

调用`pause()`方法可以暂停播放，调用`stop`方法可以停止播放。

    /**
     * <code>Status</code> 表示音源的播放状态。
     */
    public enum Status {
        /**
         * 该音源正在播放中，调用{@link AudioSource#play()}会设置为此状态。
         */
        Playing,
        
        /**
         * 该音源已被暂停播放。
         */
        Paused,
        
        /**
         * 该音源播放结束。
         * 调用{@link AudioSource#stop()}或音频文件播放到末尾时会设置为此状态。
         */
        Stopped,
    }

### 音频侦听器，你的耳朵

在jME3应用程序中，只有一个音频侦听器（Audio Listener）用来代表听众的耳朵。一般来说，这个侦听器有左右两个“耳朵”，应该贴合在主摄像机的两侧。玩家操作摄像机运动时，侦听器也跟着摄像机一起运动。在本章的代码中，通过主循环的2行代码完成了这个效果。

    /**
     * 保持Listener和Camera同步移动，这样才能感受到3D音效。
     */
    @Override
    public void simpleUpdate(float tpf) {
        listener.setLocation(cam.getLocation());
        listener.setRotation(cam.getRotation());
    }

注意：代码中出现的listener对象代表的是音频侦听器（`com.jme3.audio.Listener`），而不是用于处理用户输入的监听器（`com.jme3.input.controls.InputListener`）！它是在SimpleApplication的父类中定义的，可以通过`getListener()`方法获得。

请大家注意，由于这个listener成员的存在，以后在定义输入监听器对象时，尽量避免将对象命名为listener，防止与父类中的listener成员重名。

### 定位音效

启用定位音效，玩家左右耳获得的声音将根据音频侦听器（Audio Listener）和音源（Audio Source）的相对位置来计算，随着听者和声源之间距离的增加，玩家听到的音量会逐渐衰减。

为了计算侦听器与音源之间的距离，AudioNode必须添加到场景图中，定位音效才能正常工作。AudioNode是Node的子类，音源的位置可以通过`setLocalTranslation(Vector3f)`方法来设置。

定位音源相关参数如下，其中大部分参数在positional为false时没有意义。

<table>
  <tr>
    <th>方法</th><th>说明</th>
  </tr>
  <tr>
    <td>setPositional(boolean)</td><td>设置定位模式：true 定位音源；false 环境音源</td>
  </tr>
  <tr>
    <td>setMaxDistance(float)</td><td>设置音源能被听到的最远距离。</td>
  </tr>
  <tr>
    <td>setRefDistance(float)</td><td>设置音量衰减为原音量一半的距离。</td>
  </tr>
  <tr>
    <td>setVelocity(Vector3f)</td><td>设置音源速度向量，用于模拟多普勒效应。</td>
  </tr>
</table>

### 定向音效

定向音源，其效果类似于广播喇叭。在声音传播方向上的听众听到的声音比较清晰，不在这个方向上的听众可能听不清。

定向音源相关参数如下，其中大部分参数在directional为false时没有意义。

<table>
  <tr>
    <th>方法</th><th>说明</th>
  </tr>
  <tr>
    <td>setDirectional(boolean)</td><td>设置定向模式：true 定向音源；false 非定向音源。</td>
  </tr>
  <tr>
    <td>setDirection(Vector3f)</td><td>设置定向声源的方向向量。</td>
  </tr>
  <tr>
    <td>setInnerAngle(float)</td><td>设置定向声波的内角角度，在这个角度内听到的声音最清晰。</td>
  </tr>
  <tr>
    <td>setOuterAngle(float)</td><td>设置定向音源的外角角度，超过设个角度的听众就听不见声音了。</td>
  </tr>
</table>

### 混响效果

混响（reverberation）是一种声学特性，一般我们亦称其为“回声”。声音在介质中以声波的形式传播，声波遇到障碍会反射，这就造成了混响。

想要在游戏中造成混响效果，首先要为音源开启混响模式，其次还要设置混响环境。注意：当positional参数为false时，音源无法开启混响效果。

<table>
  <tr>
    <td>方法</td><td>说明</td>
  </tr>
  <tr>
    <td>AudioNode#setReverbEnabled(boolean)</td><td>设置混响模式，true 开启混响效果；false 关闭混响效果。</td>
  </tr>
  <tr>
    <td>AudioRenderer#setEnvironment(Environment)</td><td>设置混响环境，通过给Environment对象设置不同的参数，能够模拟地牢、洞穴、房间等不同环境下的混响效果。</td>
  </tr>
</table>

AudioRenderer使用`com.jme3.audio.Environment`对象来描述混响环境的各种参数，音效设计人员可以通过调节Environment中的参数来创造不同的混响环境。对于不太精通音效的开发人员来说，Environment类中预定义了一些常用的环境：

* Garage 车库
* Dungeon 地牢
* Cavern 洞穴
* AcousticLab 声学实验室
* Closet 密室

稍微修改一下本章例子中的`initAudio()`方法，在方法的末尾增加两行语句，为音源audioNature开启混响模式，并把混响环境设置为“洞穴”。

    /**
     * 创建两个AudioNode作为音源，并添加到场景中。。
     */
    private void initAudio() {

        前略...
        
        /**
         * 开启混响模式
         */
        audioNature.setReverbEnabled(true);
    	/**
    	 * 设置混响环境：洞穴
    	 */
        audioRenderer.setEnvironment(Environment.Dungeon);
    }

重新运行程序，你会发现原来的海潮声发生了一些变化，感觉仿佛置身于一座巨大的溶洞中。

更多关于混响效果的知识，建议感兴趣的读者通过维基百科、google、baidu等途径自行查找资料，本节后面的内容是从[百度百科：混响](http://baike.baidu.com/link?url=-w0AkTh2tvqoZA_6fTaeU3jrZgzxHfjnWzUqdertn5VhoHmsRsfm4R7VxQqOKFOS7vyYOva_XHPM_aMhGFqO6ieLi4BkUpHdpUy6d3LxJXi)词条中摘录的一部分。

#### 混响的要求

声波在室内传播时，要被墙壁、天花板、地板等障碍物反射，每反射一次都要被障碍物吸收一些。这样，当声源停止发声后，声波在室内要经过多次反射和吸收，最后才消失，我们就感觉到声源停止发声后还有若干个声波混合持续一段时间。这种现象叫做混响，这段时间叫做混响时间。

对讲演厅来说，混响时间不能太长．我们平时讲话，每秒钟大约发出2～3个单字，假定发出两个单字“物理”，设想混响时间是3秒，那么，在发出“物”字的声音之后，虽然声强逐渐减弱，但还要持续一段时间(3秒)，在发出“理”字的声音的时刻，“物”字的声强还相当大。因而两个单字的声音混在一起，什么也听不清楚了。但是，混响时间也不能太短，太短则响度不够，也听不清楚。因此需要选择一个最佳混响时间．北京科学会堂有一个学术报告厅，混响时间为1秒。

不同用途的厅堂，最佳混响时间也不相同，一般来说，音乐厅和剧场的最佳混响时间比讲演厅要长些，而且因情况不同而不同。轻音乐要求节奏鲜明，混响时间要短些，交响乐的混响时间可以长些。难于听懂的剧种如昆曲之类，混响时间一长，就更难于听懂.节奏较慢而偏于抒情的剧种，混响时间则可以长些。总之，要有一定的、恰当的混响时间，才能把演奏和演唱的感情色彩表现出来，收到应有的艺术效果。北京“首都剧场”的混响时间，坐满观众时为1.36秒，空的时候是3.3秒。这是因为满座时，吸收声音的物体多了，所以混响时间缩短，上面所说的最佳混响时间是指满座时的混响时间。高级的音乐厅或剧场，为了满足不同的要求，需要人工调节混响时间．其中一种办法是改变厅堂的吸声情况。在厅堂内安装一组可以转动的圆柱体，柱面的一半是反射面，反射强、吸收少；另一半是吸声面，反射弱、吸收多．把反射面转到厅堂的内表面，混响时间就变长；反之，把吸收面转到厅堂的内表面，混响时间就变短。

高水平的音乐会都不使用扩音设备，为的是使听众直接听到舞台上的声音.为了让全场听众都能听到较强的声音，音乐厅的天花板上挂着许多反射板，这些反射板的大小、形状、安放位置和角度都经过精确设计，以便把舞台上的声音反射到音乐厅的各个角落。
处理好不同建筑物的声响效果，取得好的音质，这是一门很重要的学问，叫做建筑声学。上面介绍的混响只是其中的一个方面，希望能引起同学们对声学的兴趣，钻研这门与我们生活关系密切的科学。

#### 混响特征和各种参数

为了研究的方便，声学上把混响分为几个部份，规定了一些习惯用语。混响的第一个声音也就是直达声（Directsound），也就是源声音，在效果器里叫做 dry out （干声输出），随后的几个明显的相隔比较开的声音叫做“早期反射声”（Earlyreflectedsounds），它们都是只经过几次反射就到达了的声音，声音比较大，比较明显，它们特别能够反映空间中的源声音、耳朵及墙壁之间的距离关系。后面的一堆连绵不绝的声音叫做 reverberation。

大多数的混响效果器会有一些参数选项给你调节，接下来讲讲这些参数具体是什么意思。

（一）衰减时间（Decay time）

也就是整个混响的总长度。不同的环境会有不同的长度，有以下几个特点：

空间越大，decay 越长，反之越短。空间越空旷，decay 越长，反之越短。空间中家具或别的物体（比如柱子之类）越少，decay 越长；反之越短。空间表面越光滑平整，decay 越长，反之越短。

因此，大厅的混响比办公室的混响长；无家具的房间的混响比有家具的房间长；荒山山谷的混响比森林山谷的混响长；水泥墙壁的空间的混响比布制墙壁的空间的混响长 ……

一般很多人喜欢把混响时间设得很长。其实真正的一些剧院、音乐厅的混响时间并没有我们想象得那么长。例如波士顿音乐厅的混响时间是 1.8 秒，纽约卡内基音乐厅是 1.7 秒，维也纳音乐厅是 2.05 秒。

这里给一个混响时间计算公式，大家可以用来算算某房间的混响时间 打开页面

（二）前反射的延迟时间（Predelay）

就是直达声与前反射声的时间距离。有以下几个特点：
空间越大，Predelay 越长；反之越短 空间越宽广，Predelay 越长；反之越短。

因此，大厅的 Predelay 比办公室的长；而隧道的空间虽然大，但是它很窄，所以 Predelay 就很短。

想要表现很宽大空旷的空间，就把 Predelay 设大一点。

（三）wet out

也就是混响效果声的大小。有以下几个特点：

wetout 与空间大小无关，而只与空间内杂物的多少以及墙壁及物体的材质有关 墙壁及室内物体的表面材质越松软，wet out 越小；反之越大空间内物体越多，wet out 越小；反之越大 墙壁越不光滑，wet out 越小，反之越大 墙壁上越多坑坑凹凹，wet out越小，反之越大。

因此，挤满了人的车厢的混响就比空车要小得多；放满了家具的房间的混响就比空房间要小；有地毯的房间的混响比无地毯的小；森林山谷的混响比荒山山谷的混响要小。

（四）高低频截止（low cut / high cut）

这个参数在有些效果器里是以 EQ 的形式来表现的，例如 Waves 的 RVerb。

这项内容实际上跟现实情况没有太直接的联系，它只是为了我们做混响处理时声音好听而设计的。不过它也能表现高频声音在传播中损失比较厉害的现象。后面我们有具体的解释。

一般在做处理的时候，为了混响声的清晰和温暖，都会把低频和高频去掉一部份。只有在表现一些诸如“宇宙声”等科幻环境时，才把高低频保留。

另外有些效果器也把这个叫做“color”（色彩）。例如 TC 的效果器就是 color。color也就是“冷”和“暖”的感觉，高频就是冷，低频就是暖。所以这些效果器用颜色来表示高低频截止，暖色（红）表示混响声偏向低频，冷色（蓝）表示混响声偏向高频。上面给大家看的 Waves 的 RVerb 的 EQ ，它分别用橙色和蓝绿色来做那两个点，也是出于此目的。

补充：

高低频截止实际上在现实中是不存在的，现实中的普遍现象是：低频声音的混响无论是声音大小还是衰减时间，都要比高频声音大。这是因为不同频率的声音由于波长不同，因此绕过障碍的能力不同，高频声音波长短，不容易绕过障碍，低频声音波长长，容易绕过障碍。加上它们在空气中传播时的衰弱程度不同（频率越高越容易衰弱），被墙壁吸收的程度不同（频率越高越容易损失），所以不同频率的声音的混响时间和大小是不相同的。在真实世界中，在大多数中小空间里，越低的声音具有越长的混响时间，越高的声音具有越短的混响时间，而不可能做到反过来。如何做到降低低频混响是任何一个录音棚头疼的难题。唯独有一种情况，是低频混响小于高频混响的，那就是很大的空间，并且里面布满了由硬质材料制成的障碍和表面，比如采用硬塑料凳子和水泥墙壁地板的室内体育馆。
我们从某音乐厅的真实 IR 的频谱中可以很清楚地看到这个规律。

因此，有的混响效果器还会有不同频率的声音的衰弱程度的设置项目。但是也有很多效果器却没有这项内容。

（五）不同频率的不同衰弱程度（Damp）

接着上面说。这个项目在有些混响效果器里没有提供。另外在采样混响器里也基本上不提供这个项目，因为采样混响的不同频率的不同衰减程度的特性已经包含在 IR 里面了。例如 Waves RVerb 提供了这个项目。另外有的效果器只有一个参数设置，就是“damp”或者“damping”，就是让高频更快地衰减。8zo Uw$iF&shy;一般来说混响中的高频是很容易大幅度衰减的。空间越大，空间内物体越多，物体和墙壁表面越不光滑，高频的衰减就越厉害。只有在中小空间中，并且空间表面比较光滑的情况下，高频的衰减才与低频接近。但我们做音乐混音的时候，有时为了声音的好听，也并不一定要遵循高频更容易衰弱的自然规律。

（六）不同频率的不同的混响时间

有的效果器也提供了不同的衰减时间给你调节，英文是 High-frequency decay and low-frequency decay ，或者别的叫法，例如 Ultrafunk Reverb 就可以设置不同的衰减时间。这个特性与前面的 damp 基本一致。一般来说混响中的高频的持续时间肯定比低频要短。空间越大，空间内物体越多，物体和墙壁表面越不光滑，高频的持续时间就短，与低频的差距就越大。只有在中小空间中，并且空间表面比较光滑的情况下，高频的时间才与低频接近。以上的三个与频率有关的参数，并不是所有的效果器都提供，有的全部提供，有的提供了其中两个甚至一个。如果没有全部提供的话，你可以用其他参数之一来代替没有提供的参数，因为它们之间的特性比较接近。

（七）散射度（diffusion）

传统上是叫做 Early reflections diffusion（早反射的散射度）。我们知道早反射就是一组比较明显的反射声。这些反射声的相互接近程度，就是 diffusion。墙壁越不光滑（例如铺上了地毯的），声音的散射度就越大，反射声越多，相互之间越接近，混响是连声一片的，声音很温和；墙壁越光滑（例如玻璃），声音的散射度就越小，反射声越少，相互之间隔得越开，混响声听起来就比较接近回声了，声音很清晰。因此，对于一些延音类的声音，比如 organ ，合成弦乐，可以使用较小的 diffusion ，声音就比较漂亮清楚；对于脉冲类的声音，比如打击乐、木琴等，可以使用较大的 diffusion ，混响就比较 smooth。有些效果器里也有 diffusion 这个参数，但是具体的定义不太一样。在某些效果器里，diffusion 是指反射声的无规律程度，空间的形状越不规则（例如山洞、教堂里），墙壁越不光滑，反射声音的出现越没有规律，diffusion 越大；空间的形状越规则（例如无家具的住宅、空的教室），墙壁越光滑，反射声的出现越有规律，diffusion 越小。

（八）混响密度（Reverb density）

这个参数的意思跟 diffusion 差不多，只是针对早反射之后的混响部份的。很多效果器并不提供 density ，而是用 diffusion 来控制整个混响。

（九）空间大小（Room size）

这个应该很好理解，空间可以体现出声场的宽度和纵深度。不过不同的效果器在这个上面会有不同的算法。另外，采样混响器不会提供这个参数，因为空间大小已经体现在 IR 中了。

（十）早反射音量（Early reflections level）

也就是早反射的声音大小。很多效果器可以让你独立调节早反射和后面的混响的声音大小。

（十一）立体声宽度（Width）

有的混响效果器有这样的参数，如果把这个值设大，那么效果器会做手脚使 IR 的左右差异变得很大，立体声感觉就出来了。

## 音效系统分析

### OpenAL
jME3的3D音效基于LWGJL，而LWJGL基于OpenAL。如果想了解3D音效的工作原理，花点时间学习OpenAL是必要的。

OpenAL的官方网站为，[www.openal.org](http://www.openal.org/)。官网提供了2份参考文档：

* [OpenAL 1.1 规格说明书](http://www.openal.org/documentation/openal-1.1-specification.pdf)
* [OpenAL编程指南](http://www.openal.org/documentation/OpenAL_Programmers_Guide.pdf)

OpenAL编程指南这个文档介绍了OpenAL的体系结构：

![openal](/content/images/2017/05/openal.png)

OpenAL从本质上来讲，是一套管理**音频场景图**的库（audio scene graph library）。它描述对象之间的一系列关系，其中重要概念有：

* 设备（device）
* 渲染上下文环境的描述表（context）
* 听众（listener）
* 音源（source）
* 缓存（buffer）

一般而言，对象之间有如下关系：

设备（device）是最终输出PCM（Pulse Code Modulation，脉冲编码调制）数据的硬件。无论任何格式的音频文件（mp3、wav、ogg、aac），最终都要被解码成PCM数据才能被硬件播放。

缓存（buffer）中存储的是原始PCM样本数据，不能直接播放。只有把缓存和音源关联起来，并播放该声音，声音才能被渲染出来。

音源（source）表示音频在场景中的位置，每个场景（context）中可以包含多个音源。音源必须和缓存关联才能获取需要播放的数据，缓存中的数据是共享的。

一个听众（listener）属于且仅属于一个context，而每个context也刚好只能有一个listener。通常，每个场景中有一个listener，有对应的位置和其他应用程序用户属性。

### OpenAL的应用

开发人员在使用OpenAL时，打交道最多的是3个对象：缓存（Buffer）、音源（Source）、侦听器（Listener）。

这并不是说设备（Device）和音频上下文环境（Context）不重要，只是这两个对象通常都是由引擎或框架的核心部分来管理的，一般不需要开发者直接编码来操作它们。

在绝大多数使用OpenAL的软件中，OpenAL充当的都是**播放器**的角色，负责播放音频数据。由于音频文件存在各种不同的压缩/编码格式，程序中一般还需要各种**解码器**来配合OpenAL使用。

各种**解码器**的作用，是把不同格式的音频数据解析为原始的PCM数据，这样OpenAL才能正确播放它。

例如开发一款mp3播放器软件，可能就需要3种技术结合起来实现：openal播放器+`mp123`解码器+GUI框架。

* `mp123`解码器负责把mp3音乐文件解析成PCM数据，然后保存为OpenAL的缓存（Buffer）；
* OpenAL负责定义音源（Source）、侦听器（Listener）等对象，音源和缓存关联之后，侦听器才能听到播放的音效；
* GUI框架则用于制作图形界面，然用户通过鼠标、按键等方式来操纵播放器。

如果想要让音乐播放器能够识别多种不同的音乐格式，那么就需要在程序中增加多种不同的解码器。

### jME3和OpenAL

我们已经在前文中介绍了如何在jME3中播放3D音效，现在站在OpenAL的角度重新审视那些对象。

<table>
  <tr><th>OpenAL对象</th><th>jME3对象</th><th>作用</th></tr>
  <tr><td>设备（Device）</td><td>无</td><td>表示播放音乐的硬件设备，如扬声器、耳机等</td></tr>
  <tr><td>音频上下文环境（Context）</td><td>com.jme3.audio.AudioContext<br>com.jme3.audio.AudioRenderer</td><td>表示音频上下文环境</td></tr>
  <tr><td>缓冲（Buffer）</td><td>com.jme3.audio.AudioData<br>com.jme3.audio.AudioBuffer<br>com.jme3.audio.AudioStream</td><td>保存音频数据</td></tr>
  <tr><td>音源（Source）</td><td>com.jme3.audio.AudioSource<br>com.jme3.audio.AudioNode</td><td>定义音源的位置、姿态，关联缓冲数据。</td></tr>
  <tr><td>侦听器（Listener）</td><td>com.jme3.audio.Listener</td><td>表示听众的位置、姿态。</td></tr>
</table>

jME3音频模块的对象和OpenAL中的对象是对应的，唯独缺少了音频设备对象。这并不是jME3的遗漏，而是一种高层的封装，因为jME3中不太需要直接操作硬件设备。

在jME3应用程序启动时，控制台中通常都会显示音频设备和OpenAL版本等信息。

    五月 16, 2017 4:16:12 下午 com.jme3.audio.openal.ALAudioRenderer initOpenAL
    信息: Audio Renderer Information
     * Device: OpenAL Soft
     * Vendor: OpenAL Community
     * Renderer: OpenAL Soft
     * Version: 1.1 ALSOFT 1.15.1
     * Supported channels: 64
     * ALC extensions: ALC_ENUMERATE_ALL_EXT ALC_ENUMERATION_EXT ALC_EXT_CAPTURE ALC_EXT_DEDICATED ALC_EXT_disconnect ALC_EXT_EFX ALC_EXT_thread_local_context ALC_SOFT_loopback
     * AL extensions: AL_EXT_ALAW AL_EXT_DOUBLE AL_EXT_EXPONENT_DISTANCE AL_EXT_FLOAT32 AL_EXT_IMA4 AL_EXT_LINEAR_DISTANCE AL_EXT_MCFORMATS AL_EXT_MULAW AL_EXT_MULAW_MCFORMATS AL_EXT_OFFSET AL_EXT_source_distance_model AL_LOKI_quadriphonic AL_SOFT_buffer_samples AL_SOFT_buffer_sub_data AL_SOFTX_deferred_updates AL_SOFT_direct_channels AL_SOFT_loop_points AL_SOFT_source_latency
    五月 16, 2017 4:16:12 下午 com.jme3.audio.openal.ALAudioRenderer initOpenAL
    警告: Pausing audio device not supported.
    五月 16, 2017 4:16:12 下午 com.jme3.audio.openal.ALAudioRenderer initOpenAL
    信息: Audio effect extension version: 1.0
    五月 16, 2017 4:16:12 下午 com.jme3.audio.openal.ALAudioRenderer initOpenAL
    信息: Audio max auxiliary sends: 4

#### 音频数据

jME3支持WAV和OGG格式的音频资源，我们可以使用AssetManager来加载这两种类型的音频文件，加载将得到一个AudioData对象。

##### 音频解码器

根据我们前面的分析，jME3中应该有这两种音频格式的**解码器**才对。那么这两个解码器在哪里？

<table>
  <tr><th>音频格式</th><th>解码器</th><th>所处jar包</th></tr>
  <tr><td>WAV</td><td><a href="https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-core/src/plugins/java/com/jme3/audio/plugins/WAVLoader.java">com.jme3.audio.plugins.WAVLoader</a></td><td>jme3-core-版本号.jar</td></tr>
  <tr><td>OGG</td><td><a href="https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-jogg/src/main/java/com/jme3/audio/plugins/OGGLoader.java">com.jme3.audio.plugins.OGGLoader</a></td><td>jme3-jogg-版本号.jar</td></tr>
</table>

jME3程序在初始化时，这两个解码器已经被AssetManager自动识别了。当然，如果你没有把`jme3-jogg-版本号.jar`添加到项目的依赖中，那么OGG解析器肯定就没法用了。而且控制台还会出现提示信息：

    五月 16, 2017 5:20:10 下午 com.jme3.asset.AssetConfig loadText
    警告: Cannot find loader com.jme3.audio.plugins.OGGLoader

##### 加载音频数据

回顾一下本章最开始的内容，当时我们直接使用AudioNode构造方法创建音源。

        /**
         * 创建一个“枪声”音源，用户点击鼠标时会发出枪声。
         */
        audioGun = new AudioNode(assetManager, "Sound/Effects/Gun.wav", DataType.Buffer);
        ...
        audioNature = new AudioNode(assetManager, "Sound/Environment/Ocean Waves.ogg", DataType.Stream);

这个方法需要的三个参数，分别是：AssetManager对象、音频文件路径、音频加载模式（流或缓存）。

实际上AudioNode构造方法在获得这3个参数之后，干了两件事情：1、加载音频数据；2、将数据和音源关联。

下面我们把这个过程还原。

以DataType.Buffer模式加载“枪声”，并和一个音源关联：

    // 1、加载并解析音频文件，保存为缓存对象。
    AudioKey keyGun = new AudioKey("Sound/Effects/Gun.wav", false);
    AudioData dataGun = assetManager.loadAudio(keyGun);

    // 2、关联音源和缓存
    AudioNode audioGun = new AudioNode(dataGun, keyGun);

以DataType.Stream模式加载“海潮声”，并和一个音源滚阿联：

    // 1、加载并解析音频文件，保存为缓存对象。
    AudioKey keyNature = new AudioKey("Sound/Environment/Ocean Waves.ogg", true, true);
    AudioData dataNature = assetManager.loadAudio(keyNature);

    // 2、关联音源和缓存
    AudioNode audioNature = new AudioNode(dataNature, keyNature);

##### AudioKey

要知道，音频文件在存储与传播时，通常都是压缩过的。未经压缩的PCM数据体积可能有几十MB，经过OGG或者AAC格式压缩编码后，文件也许就只有几百KB了。**解码器**的作用就是把这几百KB的数据还原为几十MB的数据，再交给**播放器**处理。

AudioKey对象定义了加载音频文件的路径和方式。它有很多重载构造方法，最完整的构造方法有三个参数，用于表示3种加载音频数据的模式。

    public AudioKey(String name, boolean stream, boolean streamCache)

第一个参数表示音频文件的路径，如"Sound/Environment/Ocean Waves.ogg"。

第二个参数表示音频数据是否以流模式加载，默认为false。为false时，表示禁用“流模式”，解码后的音频数据将会完全保存在内存中；为true时，表示开启“流模式”，会在播放音频数据时同步解码。

第三个参数只在第二个参数为true时有效，它表示是否需要把未解码的音频数据缓存到内存中，默认为false。为true时将把数据保存到内存中，为false时将保留在磁盘上。

例1：将音频数据解码，把解码后的数据缓存到内存中。这种方式可以节省解码的时间和I/O操作的时间，但会增加内存开销。原本几百KB的压缩音乐文件，解码后占用内存可能高达几十MB。不过由于数据就在内存中，播放器可以实现快进等功能。

    AudioKey key = new AudioKey("Sound/Environment/Ocean Waves.ogg", false);

例2：不解码，直接把未经解码的数据缓存到内存中。原文件多大，占用的内存也差不多大。播放音乐时，需要一边解码一边播放，也就是所谓的“流模式”。由于音频数据就在内存中，有些格式的音频文件可以实现快进功能（要视编码格式而定，OGG编码就不支持快进功能）。

    AudioKey key = new AudioKey("Sound/Environment/Ocean Waves.ogg", true, true);

例3：不解码，也不做任何缓存。音频文件依然保留在磁盘中，当需要播放音乐时，首先要从磁盘上读取文件数据，然后同步进行解码，最后再播放。这种模式最省内存，但是相对耗时。

    AudioKey key = new AudioKey("Sound/Environment/Ocean Waves.ogg", true, false);

##### AudioData

通过`assetManager.loadAudio(AudioKey)`可以加载音频数据，这个方法的返回值是一个AudioData对象，它存储了音频数据。

事实上AudioData是一个抽象类，它的两个派生类分别为：AudioBuffer、AudioStream。AudioBuffer表示经过解码后的数据，AudioStream表示未经解码的数据。

#### 音源

AudioNode表示音源，它的父类是Node，因此可以通过Node中的属性和方法来代表音源的位置、旋转、缩放等参数。

AudioNode同时也实现了AudioSource接口，与音乐播放有关的方法（例如`play()`、`setPositional(boolean)`）都是定义在AudioSource接口中的。

它的完整定义为：

    public class AudioNode extends Node implements AudioSource {
        ...
    }

#### AudioRenderer

AudioRenderer代表了OpenAL中的上下文环境（context）。在OpenAL中，每个context中只能有一个侦听器，这个侦听器可以通过setListener方法来设置。

    AudioRenderer#setListener(Listener listener)

Context中可以有多个音源，而且Context负责播放这些音源。AudioNode本身其实并不能播放音乐，音源中的数据都是由AudioRenderer负责播放的，例如：

    AudioRenderer#playSourceInstance(AudioSource)
    AudioRenderer#playSource(AudioSource)
    AudioRenderer#pauseSource(AudioSource)
    AudioRenderer#stopSource(AudioSource)

### 实例：Lmeur音乐播放器

本章的内容稍微有一些抽象，我也不想再继续深入讨论下去了，我们看一个实例。

下面是我利用“jME3中的音频接口+jME3自带OGG/WAV解码器+LemurGUI”制作的一个简易音乐播放器。

![Lemur音乐播放器](/content/images/2017/05/LemurMusicPlayer.png)

源代码：
[net.jmecn.lemur.MusicPlayer.java](https://github.com/jmecn/jME3Tutorials/blob/master/jME3Tutorials/src/main/java/net/jmecn/lemur/MusicPlayer.java)

对比我做的这个渣渣，jME3官方社区的用户@Tryder直接把我秒了。
![CD_MusicPlayer_Dev](/content/images/2017/05/3e26652f4a_690x388.png)

原帖地址：https://hub.jmonkeyengine.org/t/may-2015-montly-wip-screenshot-thread/32463/3

不过这个项目没有源码 :P
