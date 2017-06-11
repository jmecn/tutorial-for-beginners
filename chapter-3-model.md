# 第三章：模型

## 理解3D模型

> 东风夜放花千树，更吹落，星如雨。宝马雕车香满路，凤萧声动，壶光转，一夜鱼龙舞。
> 《青玉案 元夕》 （宋）辛弃疾 

这首词描述的是元宵节夜晚的灯会，人们逛灯市所见到各式各样的花灯，火树银花、宝马雕车、鱼龙共舞。

3D模型就是三维的、立体的模型，D是英文单词“维度”(Dimensions)的缩写。各种形态的花灯，其实就是我们在日常生活中最常见的一种3D模型。

下图来源：[百度经验](http://jingyan.baidu.com/article/a378c960723876b329283073.html)

先制作网格(Mesh)，定义形态。
![](/content/images/2017/03/r1.png)
再选择材质(Material)，贴图(Texture)上色。
![](/content/images/2017/03/r2.png)

## 模型的来源

模型有很多种来源。一些简单的形状（方块、球体）可以直接用代码来生成，复杂的模型（地形、人物）则需要使用专业工具来建模。

* 程序生成
 * 基本形状：平面、方块、球体、圆柱体等
 * 算法生成地形：山脉、河流、地牢、迷宫等。[例如这个多边形地形生成算法](http://www-cs-students.stanford.edu/~amitp/game-programming/polygon-map-generation/)
* 3D建模
 * 免费建模工具：Blender、Google SketchUp
 * 商业建模工具：Auto CAD、3DS Max、Maya、ZBrush等
* 3D扫描
 * 人像扫描
 * 点云扫描
 * 地形测绘
* 素材网站/商店
 * http://opengamearg.org/
 * http://tf3dm.com/
 * https://sketchfab.com/
 * http://www.cadnav.com/
 * https://www.assetstore.unity3d.com/

模型的精细度越高，看起来越真实，对计算机硬件的要求也越高。现在业界最精细的模型基本上都是使用ZBrush制作出来的，一些细腻的人物模型面数可达到数百万以上。

![Diablo](/content/images/2017/03/diablo.jpg)

这种级别的高模一般是用于CG电影或者动画，做成游戏则需要玩家主机的显卡配置足够高。

由于游戏开发需要能够实时渲染3D模型，模型面数太高会显著降低画面的刷新率。因此实际开发时，对于一些数量较多的实体（玩家、怪物），通常更倾向于使用面数比较低的模型，通常只有几千、几百面，也称为**低模**(low poly model)。

![low poly wolf](/content/images/2017/03/l38467-low-poly-wolf-4601.png)

为了保证效率和质量，3D游戏开发中经常要像做生意一样讨价还价，用尽量少的面数展现出尽量真实的效果。甚至，很多时候程序员都会施展障眼法来迷惑玩家的眼睛，这个问题我们在第五章讨论。

## 实例：寒冰射手-艾希

下面我们来实际观察一个3D模型。

Sketchfab.com是个好网站，它让我们能够在现代浏览器中直接观赏3D模型。我在sketchfab中找到了其他人上传的"LOL英雄-寒冰射手艾希"模型。

**提示：以下内容建议在电脑中观看，如果无法显示这个模型，请更换支持WebGL的现代浏览器。**

<div class="sketchfab-embed-wrapper"><iframe width="640" height="480" src="https://sketchfab.com/models/8bc512e972aa4a608e7ad38368c353d8/embed" frameborder="0" allowvr allowfullscreen mozallowfullscreen="true" webkitallowfullscreen="true" onmousewheel=""></iframe>

<p style="font-size: 13px; font-weight: normal; margin: 5px; color: #4A4A4A;">
    <a href="https://sketchfab.com/models/8bc512e972aa4a608e7ad38368c353d8?utm_medium=embed&utm_source=website&utm_campain=share-popup" target="_blank" style="font-weight: bold; color: #1CAAD9;">SFM: League of Legends - Ashe</a>
    by <a href="https://sketchfab.com/cmasden?utm_medium=embed&utm_source=website&utm_campain=share-popup" target="_blank" style="font-weight: bold; color: #1CAAD9;">Clint Masden</a>
    on <a href="https://sketchfab.com?utm_medium=embed&utm_source=website&utm_campain=share-popup" target="_blank" style="font-weight: bold; color: #1CAAD9;">Sketchfab</a>
</p>
</div>

模型加载完毕后，我们就能在浏览器中直接用鼠标来操作艾希了。

 * 拖拽左键，可以调整摄像机的方向
 * 拖拽右键，可以调整摄像机焦点的位置
 * 滚动中键滚轮，可以放大、缩小摄像机到焦点的距离。

点击画面右下角的齿轮图标，在弹出菜单中选择Rendering。然后在Wireframe下方的色板中选择黄色。

![Ashe_settings](/content/images/2017/03/Ashe_settings.png)

现在可以看清楚，艾希的模型形状其实是由很多个形态各异的三角形拼接而成的。

![Ashe_wireframe](/content/images/2017/03/Ashe_wireframe.png)

## 实例：加载3D模型

###  下载模型文件

首先从Stetchfab下载艾希的模型，这需要注册一个Stetchfab账号。艾希模型的地址是：

https://sketchfab.com/models/8bc512e972aa4a608e7ad38368c353d8

并不是所有的Stetchfab作者都允许用户下载模型，允许下载的模型也经常缺少必要文件，我找了很久才找到这么一个能直接被jME3导入的模型，因此大家别忘了给这位原作者点个赞。

下载之后，我得到的是一个名为`sfm-league-of-legends-ashe.zip`的压缩包，解压后在`source`文件夹里面找到了另一个压缩包。再解压，得到下列文件。

    b_ash.tga
    b_ashe_b.mtl
    b_ashe_b.obj

哦，这是一个OBJ模型，这种模型不包含动画数据。`.obj`文件一般记录了模型的网格数据，`.mtl`中定义了模型的材质，其他的图片文件则是模型的`纹理贴图(Texture)`。

`.obj`和`.mtl`都是纯文本文件，可以用记事本或者其他编辑器打开，感兴趣的话可以观察一下它的数据结构。

### 将模型文件添加到项目中

**jME3 SDK使用者**

在jME3 SDK的`Project Assets/Models`文件夹下创建`Ashe`文件夹，然后把模型文件统统放到同一个文件夹中。

    Models/Ashe/b_ash.tga
    Models/Ashe/b_ashe_b.mtl
    Models/Ashe/b_ashe_b.obj

呃，你说你用的不是jME3 SDK？

**Eclipse开发者**

在工程中创建一个名为assets的源码文件夹(Source Folder)，然后在里面创建一个名为Models.Ashe的包(pacakge)，再把文件丢到这个目录下。

![Eclipse](/content/images/2017/03/Eclipse.png)

不过你也可以直接在src下建一个Models.Ashe包，作用是一样的，因为jME3默认会从classpath下加载资源。但是单独建一个assets目录，更方便管理游戏资源。

**Android Studio开发者**

先把项目视图切换为Project，然后看看你的工程目录下是否有assets文件夹。如果没有就在src/main目录下创建一个assets文件夹。

![Android Studio中](/content/images/2017/03/Android-Studio.png)

然后在assets文件夹中创建Models/Ashe目录，再把艾希的模型文件都丢进去。

**Intellij IDEA开发者**

你可以直接选择在src/main/resources文件夹下创建Models/Ashe目录。理由和在Eclipse建立assets文件夹一样，把模型放在resources文件夹内比较方便管理。

如果工程中没有出现这个目录的话，就创建一个，然后在build.gradle中把resources设置srcDir。

    sourceSets {
        main {
            resources {
                srcDir 'src/main/resources'
            }
        }
    }

Gradle和Maven的目录结构是一样的。

**Vim/Sublime/EMacs开发者..?**

大神，随你喜好建文件夹吧，把Models/Ashe所在的目录添加到classpath就行了。

我相信你！

### AssetManager

使用jME来加载模型很简单，`SimpleApplication`中有一个资源管理器`AssetManager`，在simpleInitApp方法中通过`assetManager.loadModel(String path)`就可以加载模型了。

除了3D模型以外，assetManager还可以用于加载纹理、材质、音乐等多种资源。关于它的用法我们以后再仔细讨论，下面是它最简单的用法：

    @Override
    public void simpleInitApp() {
        // 加载模型
        Spatial model = assetManager.loadModel("Models/Ashe/b_ashe_b.obj");
        // 将模型添加到场景图中
        rootNode.attachChild(model);
    }

需要注意的是，AssetManager加载资源时**对路径的大小写敏感**，且无关操作系统。Windows平台的开发者经常会忽略这个问题，导致抛出`AssetNotFoundException`。

### 编写代码，加载模型

创建一个`HelloModel`类来加载、显示这个模型。

	package net.jmecn;
	
	import com.jme3.app.SimpleApplication;
	import com.jme3.light.AmbientLight;
	import com.jme3.light.DirectionalLight;
	import com.jme3.math.ColorRGBA;
	import com.jme3.math.Quaternion;
	import com.jme3.math.Vector3f;
	import com.jme3.scene.Spatial;
	
	/**
	 * 加载模型
	 * @author yanmaoyuan
	 *
	 */
	public class HelloModel extends SimpleApplication {
	
		@Override
		public void simpleInitApp() {
			cam.setLocation(new Vector3f(0.41600543f, 3.2057908f, 6.6927643f));
			cam.setRotation(new Quaternion(-0.00414816f, 0.9817784f, -0.18875499f, -0.021575727f));
			
			flyCam.setMoveSpeed(10);
			viewPort.setBackgroundColor(ColorRGBA.LightGray);
			
			// #1 导入模型
			Spatial model = assetManager.loadModel("Models/Ashe/b_ashe_b.obj");
			model.scale(0.03f);// 按比例缩小
			model.center();// 将模型的中心移到原点
			
			// #2 创造光源
			
			// 定向光
			DirectionalLight sun = new DirectionalLight();
			sun.setDirection(new Vector3f(-1, -2, -3));
			
			// 环境光
			AmbientLight ambient = new AmbientLight();
			
			// 调整光照亮度
			ColorRGBA lightColor = new ColorRGBA();
			sun.setColor(lightColor.mult(0.6f));
			ambient.setColor(lightColor.mult(0.4f));
			
			// #3 将模型和光源添加到场景图中
			rootNode.attachChild(model);
			rootNode.addLight(sun);
			rootNode.addLight(ambient);
		}
	
		public static void main(String[] args) {
			// 启动程序
			HelloModel app = new HelloModel();
			app.start();
		}
	
	}

运行程序，结果如下。

![Ashe_black](/content/images/2017/03/Ashe_black.png)

### 结果修正

结果跟网上展示的效果不太一样，估计这个艾希的材质需要做一点修正。

OBJ模型的材质一般是定义在`mtl`文件中的，用任意文本编辑器打开`Models/Ashe/b_ashe_b.mtl`文件，看看出了什么问题。

    newmtl ashe_base_2011_MD_lambert2SG1_SG
    illum 4
    Kd 0.00 0.00 0.00
    Ka 0.00 0.00 0.00
    Tf 1.00 1.00 1.00
    map_Kd b_ash.tga
    Ni 1.00
    Ks 0.50 0.50 0.50

`Kd`表示漫射光(diffuse)颜色，`Ka`表示环境光(ambient)颜色，此文件中2种颜色都是纯黑：`0.00 0.00 0.00`，难怪看起来是黑的。

将`Kd`、`Ka`从纯黑改成纯白`1.00 1.00 1.00`，使其能够反射所有颜色。

    newmtl ashe_base_2011_MD_lambert2SG1_SG
    illum 4
    Kd 1.00 1.00 1.00
    Ka 1.00 1.00 1.00
    Tf 1.00 1.00 1.00
    map_Kd b_ash.tga
    Ni 1.00
    Ks 0.50 0.50 0.50

F5刷新一下工程文件，让我们的修改生效。然后再次运行程序，效果如下。

![Ashe](/content/images/2017/03/Ashe.png)

感觉没有原来的模型清晰，图像边缘有锯齿。我默认是不开抗锯齿的，现在开启4倍抗锯齿(`Anti-Aliasing:4x`)再看看。

![Ashe_AA](/content/images/2017/03/Ashe_AA_4x.png)

看起来好多了！
