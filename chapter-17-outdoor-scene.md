# 户外场景

本章我们将会学习如何在3D游戏中制作户外场景，包括天空、海洋、地形等。

## 天空

3D游戏中的天空有很多实现方法，总的来说，**皆是障眼法**。最简单的手法不过是改变画面的背景色，让玩家“感觉”到天色的变化。

如果在玩家头顶上放置一个平面，再把云朵、太阳、星星、月亮等图片“贴”上去，就可以混合成类似下面的效果。

![Sky](/content/images/2017/06/Sky-1.png)

这种技术称为**“天空面（SkyPlane）”**。

显然，这种方法是很容易露馅的。当玩家的视野足够远时，就会发现天空的“边缘”。

![Sky's edge](/content/images/2017/06/SkyEdge.png)

稍微改进一下这个障眼法，可以试图让“天空面”始终遮挡在摄像机的前方，或者在远处用高山挡住玩家的视线，这样玩家就没有机会看到边缘。

![](/content/images/2017/06/sky_scroll_1.png)

但是，也不过是另一种骗局。

![](/content/images/2017/06/sky_scroll_2.png)

### 环境贴图

比天空面更精妙一些的障眼法，当属**环境贴图**：用一个立方体或球体把场景包裹起来，然后使用360°无死角的贴图，让场景中的玩家看不到边缘。

使用立方体环境贴图时，这种技术也被称为“**天空盒（SkyBox）**”。使用球体环境贴图时，则叫**“天空穹（SkyDome）”**。

![SkyDome](/content/images/2017/06/sphere.png)

这种障眼法的破绽依然很明显。第一，环境贴图是静止的，这于我们的常识不符；其二，一切看起来都是扁平的，而云应该有体积。

对于第一个破绽，可以使用多层环境贴图。保持背景图层静止，再设法让云层、太阳、月亮运动起来。在夜晚，可以让整个星空贴图缓慢转动，用来模拟时间流逝。

对于第二个破绽，有很多方法可以让云/雾看起来更加真实。可以为云层增加法线贴图，可以用粒子特效，可以用3D云模型，也可以利用特殊的着色器。

下图是网游“天涯明月刀”的画面截图。

![](/content/images/2017/06/hangzhou-6.jpg)

游戏开发者对画面真实性的追求是无止境的，而机器的性能是有限的。你尽可以在游戏中使用动态天气，但这意味着能用到烘焙阴影的地方会大量减少。如何在机器性能与画面真实性之间进行取舍，如何压榨出设备的每一分性能，是对开发者最大的考验。

### 用jME3制作天空

前文介绍了天空的多种制作方法，并不局限于某种引擎，是3D游戏中通用的手段。这些方法在jME3中都可以实现。

调节画面背景色是最简单的办法，调用 `viewPort.setBackgroundColor(new ColorRGBA(0.6f, 0.7f, 0.9f, 1));` 即可。天空面（SkyPlane）也很容易，用一个纸片加载贴图即可。

jME3提供了 `com.jme3.util.SkyFactory` 类，使开发者可以对**环境贴图**提供了完整的支持。你可以通过SkyFactory来加载环境贴图，生成天空盒或天空穹。

至于动态云层，无非是在场景中添加多个Spatial，或者使用特殊的着色器来控制云层和光照。诸如体积云、体积光、光晕、彩虹等画面效果，实则是通过着色器完成的，与jME3无关。如果你希望实现这方面的效果，需要另外学习着色器编程。

### SkyFactory

在jME3中，SkyFactory是使用最频繁的天空工具类。它支持三种不同的环境贴图：

1. 球体贴图（Sphere Map）
2. 立方体贴图（Cube Map）
3. 等距矩形贴图（Equirectangle Map）

对于这三种不同类型的环境贴图，SkyFactory使用枚举类型`SkyFactory.EnvMapType` 来表示：

    public enum EnvMapType{
        CubeMap,
        SphereMap,
        EquirectMap
    }

举个例子，假设要使用立方体贴图来创建天空盒，用法是这样的：

    @Override
    public void simpleInitApp() {
        Spatial sky = SkyFactory.createSky(assetManager,
                "Textures/Sky/Bright/BrightSky.dds", // 贴图路径
                SkyFactory.EvnMapType.CubeMap);// 贴图类型
        rootNode.attachChild(sky);
    }

调用 `createSky()` 方法时，SkyFactory 会完成下列工作

* 根据纹理贴图的路径，使用 assetManager 去加载纹理；
* 根据 EvnMapType ，加载对应类型的材质，使着色器可以正确渲染环境贴图。
* 生成网格和几何体，返回一个 Spatial 对象。

这个Spatial就是我们所需的天空盒，把它添加到场景图中就可以看到天空。

在SkyFactory的内部，它还调用了下列方法：

* `sky.setQueueBucket(Bucket.Sky);` 设置天空盒的渲染顺序，确保它比场景中的所有物体都先绘制，这样它就会出现在画面的最底层。
* `sky.setCullHint(Spatial.CullHint.Never);` 确保天空盒在渲染时不会被剔除。天空盒本身也是场景图中的一个模型。对于一般的3D物体来说，如果没有出现在摄像机的视锥中，就不会被绘制到画面上。但是对于天空而言，它应该始终都存在于画面中。
* SkyFactory 内部使用的是 `Common/MatDefs/Misc/Sky.j3md` 材质。它是专门用于天空的着色器，可以对3种不同环境贴图进行渲染。

SkyFactory中提供了多个重载的 `createSky()` 方法，其中之一是使用加载好的 Texture 来代替图片路径。

    createSky(AssetManager, Texture, EnvMapType);

有时我们在程序中加载未知来源的环境贴图，可能会出现上下颠倒、左右颠倒等情况。为此，SkyFactory中提供了createSky的重载方法，可以使用一个 Vector3f 变量来对 X/Y/Z轴进行翻转。

    createSky(AssetManager, Texture, Vectorf, EnvMapType);

例如我想把天空盒的贴图上下颠倒，即按Y轴对称翻转，就可以这么做：

    Spatial sky = SkyFactory.createSky(assetManager,
            assetManager.loadTexture("Textures/Sky/St Peters/StPeters.hdr"),
            new Vectorf(1, -1, 1),// 按Y轴对称翻转
            EnvMapType.SphereMap);

**如何制作环境贴图？**

上文介绍了如何使用环境贴图来制作天空，但是如何制作环境贴图呢？

* 环境贴图应该使用专业工具来制作，比如Terragen、Bycle等3D景观生成器，或者Blender、3DS Max等3D建模工具。
* 环境贴图的尺寸不是很重要，因为实际在游戏中，天空会被渲染到无穷远处。贴图的分辨率越高，天空看起来越清晰，游戏运行越慢。
* 实际在开发中，可以使用Node来容纳多个不同的天空图层，用来制作更加复杂的效果。

我想你可能会对这个超连接感兴趣：[免费高清天空图下载](https://blenderartists.org/forum/showthread.php?24038-Free-high-res-skymaps-%2528Massive-07-update!%2529)

#### SphereMap

**球体映射（Sphere Mapping）**是基于这样一个事实：将一个理想高反射的球体置于场景中央，从一个角度无穷远处拍摄此球体，将得到一张全景图。例如：

圣彼得大教堂

![StPeters](/content/images/2017/06/StPeters.jpg)

平原

![](/content/images/2017/06/SphereMap.jpg)

通常在场景建模中，朝向z轴正方向，利用正交投影模拟无穷远处进行渲染，就可以得到这个纹理图。


    public void simpleInitApp() {
        // 天球，圣彼得大教堂
        Texture texture = assetManager.loadTexture("Textures/Sky/St Peters/StPeters.hdr");

        Spatial sky = SkyFactory.createSky(assetManager, texture,
                new Vector3f(1, -1, 1), // 图片上下颠倒，故改变Y方向的法线
                SkyFactory.EnvMapType.SphereMap);

        rootNode.attachChild(sky);
    }

效果：

![](/content/images/2017/06/stpeters_result.png)

![](/content/images/2017/06/sphere_result.png)

SphereMap出现的时间比较早，它有一个很大的破绽，就在摄像机的背后。对于那些制作得不够精细的贴图，边缘汇聚在一起的痕迹会非常重。

![](/content/images/2017/06/sphere_result_bug.png)

#### CubeMap

**立方体贴图**的做法比较简单：把摄像机置于场景中央，朝着x，-x，y，-y，z，-z方向将场景渲染出6张纹理。然后用6张纹理组成一个立方体的6个面。这样一个真正的全景图组成了。

![Lagoon](/content/images/2017/06/Lagoon.jpg)

对于那些制作得不够好的CubeMap，破绽在于面与面的接缝处。

这6个面的排列顺序是一个很有趣的问题，在OpenGL和Dirext3D中，加载同样的天空盒会出现上下颠倒的情况。你可以从这篇文章了解更多内容：[OpenGL和D3D中Cubemap的图象方向问题 ](http://blog.csdn.net/nhsoft/article/details/1398630)

![](/content/images/2017/06/Cube_map.png)

实际加载CubeMap时，有两种截然不同的方式。不过这两种方式只是图片格式不同，效果并没有什么区别。

![](/content/images/2017/06/cube_map_result.png)

**Texture * 6**

比较常见的情况是，CubeMap并不是一张贴图，而是由6张贴图组成。有时甚至只有5个面，因为可能不需要底部的贴图。


    public void simpleInitApp() {
        Texture west = assetManager.loadTexture("Textures/Sky/Lagoon/lagoon_west.jpg");
        Texture east = assetManager.loadTexture("Textures/Sky/Lagoon/lagoon_east.jpg");
        Texture north = assetManager.loadTexture("Textures/Sky/Lagoon/lagoon_north.jpg");
        Texture south = assetManager.loadTexture("Textures/Sky/Lagoon/lagoon_south.jpg");
        Texture up = assetManager.loadTexture("Textures/Sky/Lagoon/lagoon_up.jpg");
        Texture down = assetManager.loadTexture("Textures/Sky/Lagoon/lagoon_down.jpg");

        Spatial sky = SkyFactory.createSky(assetManager, west, east, north, south, up, down);
        rootNode.attachChild(sky);
    }

**TextureCubeMap**

另一种情况是使用特殊格式，在一个文件里同时保存了6个图层，用一个贴图文件来代表整个立方体贴图，常见于微软的dds格式。

    public void simpleInitApp() {
        TextureKey key = new TextureKey("Textures/Sky/Bright/BrightSky.dds", true);
        key.setGenerateMips(false);
        key.setTextureTypeHint(Texture.Type.CubeMap);
        Texture tex = assetManager.loadTexture(key);

        // 天空盒
        Spatial sky = SkyFactory.createSky(assetManager, tex, SkyFactory.EnvMapType.CubeMap);
        rootNode.attachChild(sky);
    }

#### EquirectMap

**等距矩形投影（Equirectangle Projection）**，又称球面投影、方格投影、等距柱状投影等。假想球面与圆筒面相切于赤道，赤道为没有变形的线。经纬线网格，同一般正轴圆柱投影，经纬线投影成两组相互垂直的平行直线。

其特性是：保持经距和纬距相等，经纬线成正方形网格；沿经线方向无长度变形；角度和面积等变形线与纬线平行，变形值由赤道向高纬逐渐增大。该投影适合于低纬地区制图 。

![](/content/images/2017/06/cylindrical.gif)

实际应用中，最常见的就是地图。

![](/content/images/2017/06/earth.jpg)

还有全景实景展示：

![](/content/images/2017/06/path.jpg)

![](/content/images/2017/06/SkyEquirectMap.jpg)

代码：

    public void simpleInitApp() {
        // 天空盒
        Spatial sky = SkyFactory.createSky(assetManager, "Textures/Sky/SkyEquirectMap.jpg", SkyFactory.EnvMapType.EquirectMap);
    }

效果：

![](/content/images/2017/06/earth_result.png)

![](/content/images/2017/06/path_result-1.png)

这种贴图的破绽在于头顶和脚底，即纬度最高处。那些制作得不够精细的贴图，在这两个点会有明显的“汇聚感”。

![](/content/images/2017/06/equirect_result_bug.png)

## 水面

### 水面纹理

水面的制作方式也有很多种，最简单的莫过于采用纹理贴图。

![波纹](/content/images/2017/06/op.jpg)

如果嫌弃静态的水面，可以准备多张纹理贴图，使用帧动画让水面看起来是流动的。

![](/content/images/2017/06/many_water.png)

当然，也可以使用着色器技术，通过俗称“滚UV”的方式让一张图片看起来是流动的。

![](/content/images/2017/06/water.gif)

### 水面反射

更复杂一些的做法，是制作一个平面来代表水面，然后用着色器再其表面绘制场景的反射贴图。另外，还可以利用算法让平面波动起来，这样显得更加真实。

![](/content/images/2017/06/simplewater.png)

jME3的 [SimpleWaterProcessor](https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-effects/src/main/java/com/jme3/water/SimpleWaterProcessor.java) 就是基于这个原理实现的。

官方教程：[水面](http://jmonkeyengine.github.io/wiki/jme3/advanced/water.html)

官方例子：

* [TestSceneWater.java](https://github.com/jMonkeyEngine/jmonkeyengine/blob/445f7ed010199d30c484fe75bacef4b87f2eb38e/jme3-examples/src/main/java/jme3test/water/TestSceneWater.java)
* [TestSimpleWater.java](https://github.com/jMonkeyEngine/jmonkeyengine/blob/445f7ed010199d30c484fe75bacef4b87f2eb38e/jme3-examples/src/main/java/jme3test/water/TestSimpleWater.java)

### 后期水体特效

反色技术使得水面看起来更加真实，但说白了依然是一层纸。如果想体验水下场景，就需要额外的渲染技术了。

在jME3中，[WaterFilter](https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-effects/src/main/java/com/jme3/water/WaterFilter.java) 用于模拟真实的水体，它是一种后期特效。其核心算法与 SimpleWaterFilter 类似，也是实时计算水面反射。除此之外，当玩家把摄像机移到水面以下时，还能够实现水下的特效。

![水下](/content/images/2017/06/under_water.png)

WaterFilter 须配合 FilterPostProcessor 一起使用。关于它的用法，我们在“第十四章：特效”中有所介绍，这里就不再赘述了。

官方教程：[后期水体特效](http://jmonkeyengine.github.io/wiki/jme3/advanced/post-processor_water.html)

官方例子：

* [TestPostWater.java](https://github.com/jMonkeyEngine/jmonkeyengine/tree/master/jme3-examples/src/main/java/jme3test/water/TestPostWater.java)
* [TestPostWaterLake.java](https://github.com/jMonkeyEngine/jmonkeyengine/tree/master/jme3-examples/src/main/java/jme3test/water/TestPostWaterLake.java)
* [TestMultiPostWater.java](https://github.com/jMonkeyEngine/jmonkeyengine/tree/master/jme3-examples/src/main/java/jme3test/water/TestMultiPostWater.java)

## 地形

制作3D游戏时，可以使用建模工具来雕刻大型的游戏地图。这样做能够获得非常精细的地图模型，而且创作自由度也非常高，缺陷是渲染速度较慢。

![3D terrain model](/content/images/2017/06/terrain_model.png)

在实际开发中，经常使用**高度图（Height map）**来创建能够快速渲染的地形。再结合着色器实现“抛雪球算法（Texture Splatting）”算法，能够使用少量纹理贴图，绘制出效果不错的地形。

### 高度图

3D游戏中的地形生成有许多方法，其中应用最广泛的就是高度图。在地理学科中，一幅地图的不同海拨用不同的颜色表示，即等高线表示法。高度图基于同样的原理，只不过这里的高度值表现为图像中的亮度值。

下面两幅图分别为等高线图与高度图：

![等高线图](/content/images/2017/06/isoheight.png)

在高度图中，图像的每个象素存储了对应的高度值，取值范围为0~255。

![高度图](/content/images/2017/06/heightmap.png)

根据图像每个象素的(x, y)坐标，以及高度值height，就可以获得3D空间中的顶点。把这些顶点连接成网格，就可以生成3D模型。

    Vector3f position = new Vector3f(x, height, -y);

下面3个截图就是用过一副高度图生成的3D地形。这个算法的实现并不复杂，如果你感兴趣，可以读读这个测试类：[TestHeightmap.java](https://github.com/jmecn/jME3Tutorials/blob/master/src/main/java/net/jmecn/outscene/TestHeightmap.java)。

![地形，点云模式](/content/images/2017/06/point_clouds.png)

![地形，线框模式](/content/images/2017/06/terrain_tri_mesh.png)

![地形，着色模式](/content/images/2017/06/terrain_shade_model.png)

#### 制作高度图

地形编辑器是用于制作高度图的工具。jMonkeyEngine SDK、Unity3D之类的3D游戏引擎有内置的地形编辑器，但是我更推荐专业工具。诸如：

* [EarthSculptor](http://www.earthsculptor.com/index.htm) 地形雕刻器
* [World Machine](http://www.world-machine.com/) 创世机
* [Terragen](http://planetside.co.uk) 地形生成器
* [L3DT](http://www.bundysoft.com/L3DT/) 大型3D地形生成器
* [VUE xStream](http://www.e-onsoftware.com/products/vue/vue_2016_xstream/) 自然景观生成器
* [LCS](http://www.lilchips.com/index.htm) LCS游戏开发工具箱
* [http://terrain.party](http://terrain.party/) 一个可以把真实地图导出成高度图的网站。

目前我最喜欢 EarthSculptor。它的界面和功能都不复杂，很容易上手。免费版只能生成257*257分辨率的高度图，但也足够新手使用了。

下面是它的主界面：

![](/content/images/2017/06/terra_mesh.png)

![](/content/images/2017/06/terra_color.png)

本文使用的高度图均由 EarthSculptor 生成。下载 EarthSculptor 后，你可以在Maps目录、Textures目录中找到一些默认的资源图片。我已经把高度图添加到了工程的资源目录中：[Secenes/Maps/DefaultMap](https://github.com/jmecn/jME3Tutorials/tree/master/src/main/resources/Scenes/Maps/DefaultMap)。

#### 使用高度图

**依赖**

jME3内置了对高度图的支持，并提供了很多优化功能。想使用这些功能，需要在项目中添加对 `jme3-terrain` 模块的依赖。

Gradle

    dependencies {
        // 添加jme3地形模块的依赖库
        compile 'org.jmonkeyengine:jme3-terrain:3.1.0-stable'
    }

在jMonkeyEngine SDK中，需要把 `jme3-terrain.jar` 添加到项目依赖。

**限制**

使用高度图生成3D地形时，图片的格式和分辨率是有限制的。

* 由于四叉树优化的需要。图像的长和宽必须相等，并且分辨率最好是2的n次方+1。例如129（128+1）、257（256+1）、513（512+1）、1025（1024+1）等。
* 使用256度的灰度图。就算你使用彩色图片，程序也会使用加权平均法把它变成灰度图，可能会导致一些诡异的结果。
* 使用jpg或png格式的图片。

**功能介绍**

`jme3-terrain` 模块有很多功能，包括但不限于：

* **根据高度数据生成3D地形**。通过 `AbstractHeightMap` 来定义统一的高度图接口，既可以使用 `ImageBasedHeightMap` 来加载图像数据，也可以通过一些算法来随机生成高度数据。
* **基于GeoMipMapping算法的层次细节（LOD）技术。**这种技术可以根据顶点到摄像机的距离来动态改变层次细节。离摄像机越近，细节越清晰；离摄像机越远，看起来越简化。

![](/content/images/2017/06/terrain-lod-high-medium-low.png)

* **四叉树（Quad Tree）网格优化**。整个地形的网格由多个地形区块（TerrainPatch）组成，并归于地形四叉树（TerrainQuad）统一管理。这些区块存储了实际的网格数据，可以支持层次细节、加速视锥裁剪等优化功能。
* **Texture Splatting渲染**。这是一种基于着色器的多重纹理渲染技术，jME3最大支持16张不同的纹理。
* **实时编辑地形数据**。TerrainQuad中的地形数据是可以实时编辑的，jMonkeyEngine SDK基于这个功能提供了内置的地形编辑器。

jME3官方教程中提供了很多使用Terrain的例子，诸如：

* [TerrainTest.java](https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-examples/src/main/java/jme3test/terrain/TerrainTest.java)
* [TerrainTestAdvanced.java](https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-examples/src/main/java/jme3test/terrain/TerrainTestAdvanced.java)
* [TerrainTestCollision.java](https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-examples/src/main/java/jme3test/terrain/TerrainTestCollision.java)
* [TerrainTestModifyHeight.java](https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-examples/src/main/java/jme3test/terrain/TerrainTestModifyHeight.java)
* [TerrainTestReadWrite.java](https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-examples/src/main/java/jme3test/terrain/TerrainTestReadWrite.java)

**使用方法**

**第一步：使用 `AssetManager` 加载高度图。**

        // 加载高度图
        Texture heightMapImage = assetManager.loadTexture("Scenes/Maps/DefaultMap/default.png");

**第二步：根据图像生成高度数据。**

`ImageBasedHeightMap` 的功能是解析图像数据，根据每个像素的灰度值来计算高度值。依次调用 `load()` 方法和 `getHeightMap()` 方法，可以得到一个 float[] 数组，其中存储了地图高度数据。

        // 根据图像内容，生成高度数据
        ImageBasedHeightMap heightMap = new ImageBasedHeightMap(heightMapImage.getImage(), 1f);
        heightMap.load();
        float[] heightData = heightMap.getHeightMap();

**第三步：使用 `TerrainQuad`，根据高度数据生成3D地形。**

`TerrainQuad` 内部使用`四叉树`算法来优化网格结构。它的内部由多个 `TerrainPatch` 组成，每个 `TerrainPatch` 代表网格中的一个区块。

在创建 `TerrainQuad` 时，需要设置4个参数，这些参数的含义如下：

* String name, 地形的名称。
* int patchSize, 区块的大小。若区块大小为 64x64，则取值为 65。
* int totalSize, 高度图的分辨率。对于分辨率为 257x257 的高度图，取值为 257。
* float[] heightMap, 地形的高度数据。数组的长度应该为 totalSize x totalSize。

注意，TerrainQuad 是 Spatial 的子类，需要添加到场景图中方可显示。

        // 根据高度图生成3D地形。
        // 该地形被分解成边长65(64*64)的矩形区块，用于优化网格。
        // 高度图的边长为 257，分辨率 256*256。
        TerrainQuad terrain = new TerrainQuad("heightmap", 65, 257, heightData);
        rootNode.attachChild(terrain);

**第四步：设置材质，用于渲染地形。**

TerrainQuad 是 Spatial 的子类，可以根据需要来给它设置材质，哪怕是 Unshaded.j3md都可以。

        // 加载材质
        Material material = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");
        material.getAdditionalRenderState().setWireframe(true);
        terrain.setMaterial(material);

实际开发时，地形一般使用多重纹理渲染，这样起来比较美观。`jme3-terrain` 模块中包含了一些地形专用材质，诸如：

* `Common/MatDefs/Terrain/Terrain.j3md`
* `Common/MatDefs/Terrain/TerrainLighting.j3md`
* `Common/MatDefs/Terrain/HeightBasedTerrain.j3md`

对于这些材质，我们将在**地形渲染**章节再详细介绍。

**第五步：层次细节（LOD）优化。**

`TerrainLodControl` 的作用是控制地形网格的层次细节（LOD）。创建 TerrainLodControl 时需要指定 Terrain 和 Camera 对象，因为它需要根据摄像机到地形的的距离来控制LOD。改变LOD的距离由 `LodCalculator` 来计算，默认的LOD计算器为 `DistanceLodCalculator`。

        // 层次细节（LOD）优化
        TerrainLodControl lodControl = new TerrainLodControl(terrain, cam);
        // LOD计算器，一个参数代表区块大小，第二个参数代表距离系数。
        // size = 65, multiplier = 2.7f, distance = 65 * 2.7f
        lodControl.setLodCalculator(new DistanceLodCalculator(65, 2.7f));

        terrain.addControl(lodControl);


**完整代码**如下：

    package net.jmecn.outscene;

    import com.jme3.app.SimpleApplication;
    import com.jme3.material.Material;
    import com.jme3.terrain.geomipmap.TerrainLodControl;
    import com.jme3.terrain.geomipmap.TerrainQuad;
    import com.jme3.terrain.geomipmap.lodcalc.DistanceLodCalculator;
    import com.jme3.terrain.heightmap.ImageBasedHeightMap;
    import com.jme3.texture.Texture;

    /**
     * 演示加载高度图
     *
     * @author yanmaoyuan
     *
     */
    public class TestTerrain extends SimpleApplication {

        @Override
        public void simpleInitApp() {
            flyCam.setMoveSpeed(100);

            // 加载高度图
            Texture heightMapImage = assetManager.loadTexture("Scenes/Maps/DefaultMap/default.png");

            // 根据图像内容，生成高度数据
            ImageBasedHeightMap heightMap = new ImageBasedHeightMap(heightMapImage.getImage(), 1f);
            heightMap.load();
            float[] heightData = heightMap.getHeightMap();

            // 根据高度图生成3D地形。
            // 该地形被分解成边长65(64*64)的矩形区块，用于优化网格。
            // 高度图的边长为 257，分辨率 256*256。
            TerrainQuad terrain = new TerrainQuad("heightmap", 65, 257, heightData);
            rootNode.attachChild(terrain);
            terrain.center();

            // 加载材质
            Material material = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");
            material.getAdditionalRenderState().setWireframe(true);
            terrain.setMaterial(material);

            // 层次细节（LOD）优化
            TerrainLodControl lodControl = new TerrainLodControl(terrain, cam);
            // LOD计算器，一个参数代表区块大小，第二个参数代表距离系数。
            // size = 65, multiplier = 2.7f, distance = 65 * 2.7f
            lodControl.setLodCalculator(new DistanceLodCalculator(65, 2.7f));

            terrain.addControl(lodControl);
        }

        public static void main(String[] args) {
            TestTerrain app = new TestTerrain();
            app.start();
        }

    }

运行结果是这样的：

![地形，线框模式](/content/images/2017/06/terrain_tri_mesh.png)

### 地形渲染

使用高度图生成3D地形，主要优势是渲染速度比普通3D模型快。常用的渲染方法有很多，例如下面几种：

* 使用纹理贴图
* 基于等高线渲染
* 使用抛雪球算法

#### 使用纹理贴图

通过高度图生成的3D地形，本质上依然是一个3D物体。因此，直接使用普通的纹理贴图就可以渲染地形了。

**ColorMap**

相比于一般的3D模型来说，使用高度图生成的地形跟容易着色。
因为地形一定是矩形的，只要画好对应的彩色贴图（ColorMap）就可以了。

例如，使用 `Unshaded.j3md`设置下面的 ColorMap：

![default_c](/content/images/2017/06/default_c.png)

代码：

        // 加载材质
        Material material = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");

        Texture colorMap = assetManager.loadTexture("Scenes/Maps/DefaultMap/default_c.png");
        material.setTexture("ColorMap", colorMap);

        terrain.setMaterial(material);

结果：

![ColorMap](/content/images/2017/06/terrain_unshaded_color_map.png)

**LightMap**

还可以把光影烘焙成亮度图（LightMap），这样能够节省计算光影的开销，更适合手游。

![default_l](/content/images/2017/06/default_l.png)

代码：

        // 加载材质
        Material material = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");

        Texture colorMap = assetManager.loadTexture("Scenes/Maps/DefaultMap/default_c.png");
        material.setTexture("ColorMap", colorMap);

        Texture lightMap = assetManager.loadTexture("Scenes/Maps/DefaultMap/default_l.png");
        material.setTexture("LightMap", lightMap);

        terrain.setMaterial(material);

结果：

![LightMap](/content/images/2017/06/terrain_unshaded_light_map.png)

**其他？**

想让地图看起来更加真实、细腻，还可以继续为材质加上法线贴图（NormalMap）、细节贴图（DetailMao）、发光贴图（GlowMap）等。不过 `Unshaded.j3md` 材质并不支持法线，需要使用 `Lighting.j3md` 材质了。

总的来说，这种渲染方式与普通的3D模型并没有什么区别。

**default_unshaded.j3m**

相比于在Java代码中加载材质、设置参数，我更喜欢使用j3m文件来记录材质参数。创建 `Scenes/Maps/DefaultMap/default_unshaded.j3m` 文件，写入下列内容：

    Material unshaded material : Common/MatDefs/Misc/Unshaded.j3md {
        MaterialParameters {
            ColorMap : Scenes/Maps/DefaultMap/default_c.png
            LightMap : Flip Scenes/Maps/DefaultMap/default_l.png
        }
    }

在Java代码中，只需要一行语句就可以加载这个材质了。

        // 加载材质
        Material material = assetManager.loadMaterial("Scenes/Maps/DefaultMap/default_unshaded.j3m");
        terrain.setMaterial(material);


#### 基于等高线算法

这种渲染方法的思路与等高线图一脉相承。根据高度值，把地形划分为不同的区域，每个区域使用不同的贴图。例如：

* 山谷，高度值0~50
* 平原，高度值50~200
* 高原，高度值200~255

![](/content/images/2017/06/terrain-from-heightmap.png)

`jme3-terrain` 模块提供了一个 `Common/MatDefs/Terrain/HeightBasedTerrain.j3md` 材质，我们可以使用它来实现基于等高线的地形渲染。材质定义的内容是这样的：

    MaterialDef Terrain {

        MaterialParameters {
            Texture2D region1ColorMap
            Texture2D region2ColorMap
            Texture2D region3ColorMap
            Texture2D region4ColorMap
            Texture2D slopeColorMap
            Float slopeTileFactor
            Float terrainSize
            Vector3 region1
            Vector3 region2
            Vector3 region3
            Vector3 region4
        }

        Technique {
            VertexShader GLSL100:   Common/MatDefs/Terrain/HeightBasedTerrain.vert
            FragmentShader GLSL100: Common/MatDefs/Terrain/HeightBasedTerrain.frag

            WorldParameters {
                WorldViewProjectionMatrix
                WorldMatrix
                NormalMatrix
            }
        }
    }

在这个材质中，最多可以设置4个高度区域的纹理。第`X`个区域的贴图用`regionXColorMap`表示，高度范围用`regionX`表示。

高度范围是一个Vector3f类型的参数，regionX.x表示高度的起点，regionX.y表示高度的终点，regionX.z表示贴图的缩放系数。

slopeColorMap 贴图用于绘制悬崖，slopeTileFactor 表示斜坡的缩放系数。当斜率较大时，将绘制成悬崖峭壁。

terrainSize 表示地形的大小，即高度图的分辨率。

创建 `Scenes/Maps/DefaultMap/default_height_based.j3m` 文件，描述材质对象。

    Material height based : Common/MatDefs/Terrain/HeightBasedTerrain.j3md {
        MaterialParameters {

            terrainSize : 257

            //slopeColorMap : Repeat Scenes/Maps/DefaultMap/Textures/bigRockFace.png
            //slopeTileFactor : 10

            region1ColorMap : Repeat Scenes/Maps/DefaultMap/Textures/hardDirt.png
            region2ColorMap : Repeat Scenes/Maps/DefaultMap/Textures/shortGrass.png
            region3ColorMap : Repeat Scenes/Maps/DefaultMap/Textures/grayRock.png

            region1 : 0.0 60.0 20.0
            region2 : 60.0 120.0 20.0
            region3 : 120.0 255.0 20.0
        }
    }

加载材质：

        // 加载材质
        Material material = assetManager.loadMaterial("Scenes/Maps/DefaultMap/default_height_based.j3m");

        terrain.setMaterial(material);

效果：

![](/content/images/2017/06/terrain_height_based.png)

#### 抛雪球算法

Texture Splatting，中文翻译为“抛雪球”算法，也叫作“足迹法”。**它是一种使用alphamap 将纹理融合到表面的技术。**

一个纹理中通常有多个通道：红、绿、蓝、或者亮度。在Texture Splatting技术中，alphamap 用于控制纹理在当前位置显示颜色的**强度**。通过简单的乘法，很容易就能够调整纹理的颜色值：alphamap * texture（texture指代当前位置纹理的颜色值）。如果某像素的alphamap是1，则纹理显示全值，如果某像素的alphamap是0，则该纹理完全不显示。

下图是[wikipedia](https://en.wikipedia.org/wiki/Texture_splatting)上对texture splatting技术的演示。在这个例子中，一共有2个texture和1个alphamap。alphamap中使用黑白二色表示了2个texture各自的颜色强度，经过混合后得到了右下的纹理。

![](/content/images/2017/06/texture_splatting-1.png)

这种技术允许我们使用多种不同的纹理在地形的表面作画。通过着色器实现texture splatting算法，就可以混合出丰富的颜色。

一般来说，每个alphamap中最多有4个通道可以使用。例如 EarthSculptor（未注册版）的画刷功能，提供的就是4种纹理，恰好可以用1张alphamap来表示。

![](/content/images/2017/06/terra_color.png)

最终生成的alphamap看起来很怪异，仿佛是随意涂鸦而成。

![](/content/images/2017/06/color_map.png)

实际上，alphamap中的每个通道都对应着一种纹理，例如下面4个。

![](/content/images/2017/06/texture_splatting.png)

当它们混色之后，就可以得到下面的实际纹理。

![](/content/images/2017/06/splatting.png)

`jme3-terrain` 提供了2个材质，都实现了 texture-splatting 算法。

* `Common/MatDefs/Terrain/Terrain.j3md` 支持1个alphamap和3个纹理，并且不能处理光照。
* `Common/MatDefs/Terrain/TerrainLighting.j3md` 支持最大3张alphamap 和12张纹理，还支持法线贴图、高光贴图、发光贴图等纹理，并且可以处理光照。

**Terrain.j3md**

官方范例：[TerrainTest.java](https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-examples/src/main/java/jme3test/terrain/TerrainTest.java)

在 [Terrain.j3md](https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-terrain/src/main/resources/Common/MatDefs/Terrain/Terrain.j3md) 的所定义的材质中，参数 `Alpha` 表示 alphamap；`Tex1`、`Tex2`、`Tex3`分别对应 alphamap 中红、绿、蓝三个通道的纹理；`Tex1Scale`、`Tex2Scale`、`Tex3Scale`对应表示每一种纹理的缩放系数。

`useTriPlanarMapping` 参数表示是否开启三维映射。开启后，着色器将对地形中拉伸变形的部位予以修正。

`useTriPlanarMapping = false`

![](/content/images/2017/06/triPlanar-regularTerrain.jpg)

`useTriPlanarMapping = true`

![](/content/images/2017/06/triPlanar-Terrain.jpg)

**TerrainLighting.j3md**

官方范例：[TerrainTestAdvanced.java](https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-examples/src/main/java/jme3test/terrain/TerrainTestAdvanced.java)

[TerrainLighting.j3md](https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-terrain/src/main/resources/Common/MatDefs/Terrain/TerrainLighting.j3md) 可以使用比 Terrain.j3md 更多的纹理，包括：

1~3个alphamap：

* `AlphaMap`
* `AlphaMap_1`
* `AlphaMap_2`

每个alphamap可以控制4个纹理，合计最多12个。每个纹理的定义，可以包含1个DuffuseMap、1个NormalMap以及1个缩放系数。

* `DiffuseMap, DiffuseMap_0_scale, NormalMap`
* `DiffuseMap_1, DiffuseMap_1_scale, NormalMap_1`
* `DiffuseMap_2, DiffuseMap_2_scale, NormalMap_2`
* `DiffuseMap_3, DiffuseMap_3_scale, NormalMap_3`
* `DiffuseMap_4, DiffuseMap_4_scale, NormalMap_4`
* ...
* `DiffuseMap_11, DiffuseMap_11_scale, NormalMap_11`

两种光照特效贴图：

* `GlowMap`
* `SpecularMap`

由于在OpenGL中，每个着色器最多只能使用16个纹理，所以你并不能同时使用上面那么多贴图。使用 `TerrainLighting.j3md` 时，有如下限制：

* 1-12 DiffuseMap。 至少得有1个DiffuseMap。
* 1-3 AlphaMap。每使用4个DiffuseMap，就需要多用1个AlphaMap！
* 0-6 NormalMap。DiffuseMap和NormalMap总是成对使用！
* 0-1 GlowMap。
* 0-1 SpecularMap。
* 纹理的总数不能超过16个！

下图是使用 `TerrainLighting.j3md` 材质渲染出来的地形。

![](/content/images/2017/06/DefaultMap.png)

### 地形的碰撞检测

根据前面我们在“物理引擎”章节中讲解过的知识，对地形进行碰撞检测是很容易的。首先为Terrain增加一个质量为0的刚体控制器（RigidBodyControl），然后把它添加到Bullet的物理空间（PhysicsSpace）即可。

    terrain.addControl(new RigidBodyControl(0));
    bulletAppState.getPhysicsSpace().add(terrain);

官方范例：[TerrainTestCollision.java](https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-examples/src/main/java/jme3test/terrain/TerrainTestCollision.java)

### 平滑化

高度图中每个象素高度的取值范围为0~255的整数，在生成3D地形网格时，斜坡给人的感觉更像是“阶梯”。

使用高斯模糊算法，可以“抚平”这些棱角，让地形看起来更加平滑。根据网上对高斯模糊算法的介绍，我编写了这个类：[GaussianBlur.java](https://github.com/jmecn/jME3Tutorials/blob/master/src/main/java/net/jmecn/outscene/GaussianBlur.java)

高斯模糊前：

![](/content/images/2017/06/BeforeGaussianBlur.png)

高随模糊后：

![](/content/images/2017/06/AfterGaussianBlur.png)

使用方法：

        // 加载地形的高度图
        Texture heightMapImage = assetManager.loadTexture("Scenes/Maps/DefaultMap/default.png");

        // 根据图像内容，生成高度图
        ImageBasedHeightMap heightmap = new ImageBasedHeightMap(heightMapImage.getImage(), 1f);
        heightmap.load();

        // 高斯平滑
        GaussianBlur gaussianBlur = new GaussianBlur();

        float[] heightData = heightmap.getHeightMap();
        int width = heightMapImage.getImage().getWidth();
        int height = heightMapImage.getImage().getHeight();

        heightData = gaussianBlur.filter(heightData, width, height);

        /*
         * 根据高度图生成实际的地形。该地形被分解成边长65(64*64)的矩形区块，用于优化网格。高度图的边长为 257，分辨率 256*256。
         */
        TerrainQuad terrain = new TerrainQuad("terrain", 65, 257, heightmap.getHeightMap());

## 实例：户外场景

现在，你已经学会了如何在jME3中制作天空、海洋和大地。下面以一个完整的例子来结束本章的内容。

    package net.jmecn.outscene;

    import com.jme3.app.SimpleApplication;
    import com.jme3.light.AmbientLight;
    import com.jme3.light.DirectionalLight;
    import com.jme3.math.ColorRGBA;
    import com.jme3.math.Vector3f;
    import com.jme3.post.FilterPostProcessor;
    import com.jme3.scene.Spatial;
    import com.jme3.terrain.geomipmap.TerrainLodControl;
    import com.jme3.terrain.geomipmap.TerrainQuad;
    import com.jme3.terrain.geomipmap.lodcalc.DistanceLodCalculator;
    import com.jme3.terrain.heightmap.ImageBasedHeightMap;
    import com.jme3.texture.Texture;
    import com.jme3.util.SkyFactory;
    import com.jme3.water.WaterFilter;

    /**
     * 通过高度图加载地形。
     *
     * @author yanmaoyuan
     *
     */
    public class HelloTerrain extends SimpleApplication {

        @Override
        public void simpleInitApp() {

            cam.setLocation(new Vector3f(-100, 80, 50));

            flyCam.setMoveSpeed(20f);

            initLight();

            initSky();

            initWater();

            initTerrain();
        }

        /**
         * 初始化灯光
         */
        private void initLight() {
            AmbientLight ambient = new AmbientLight();
            ambient.setColor(new ColorRGBA(0.298f, 0.2392f, 0.2745f, 1f));
            rootNode.addLight(ambient);

            DirectionalLight light = new DirectionalLight();
            light.setDirection((new Vector3f(0.097551f, -0.733139f, -0.673046f)).normalize());
            light.setColor(new ColorRGBA(1, 1, 1, 1));
            rootNode.addLight(light);
        }

        /**
         * 初始化天空
         */
        private void initSky() {
            Spatial sky = SkyFactory.createSky(assetManager, "Textures/Sky/SkySphereMap.jpg",
                    SkyFactory.EnvMapType.SphereMap);
            rootNode.attachChild(sky);
        }

        /**
         * 初始化水面
         */
        private void initWater() {
            FilterPostProcessor fpp = new FilterPostProcessor(assetManager);
            viewPort.addProcessor(fpp);

            // 水
            WaterFilter waterFilter = new WaterFilter();
            waterFilter.setWaterHeight(50f);// 水面高度
            waterFilter.setWaterTransparency(0.2f);// 透明度
            waterFilter.setWaterColor(new ColorRGBA(0.4314f, 0.9373f, 0.8431f, 1f));// 水面颜色

            fpp.addFilter(waterFilter);
        }

        /**
         * 初始化地形
         */
        private void initTerrain() {

            // 加载地形的高度图
            Texture heightMapImage = assetManager.loadTexture("Scenes/Maps/DefaultMap/default.png");

            // 根据图像内容，生成高度图
            ImageBasedHeightMap heightmap = new ImageBasedHeightMap(heightMapImage.getImage(), 1f);
            heightmap.load();

            // 高斯平滑
            GaussianBlur gaussianBlur = new GaussianBlur();

            float[] heightData = heightmap.getHeightMap();
            int width = heightMapImage.getImage().getWidth();
            int height = heightMapImage.getImage().getHeight();

            heightData = gaussianBlur.filter(heightData, width, height);

            /*
             * 根据高度图生成实际的地形。该地形被分解成边长65(64*64)的矩形区块，用于优化网格。高度图的边长为 257，分辨率 256*256。
             */
            TerrainQuad terrain = new TerrainQuad("terrain", 65, 257, heightmap.getHeightMap());

            // 层次细节
            TerrainLodControl control = new TerrainLodControl(terrain, getCamera());
            control.setLodCalculator(new DistanceLodCalculator(65, 2.7f));
            terrain.addControl(control);

            // 地形材质
            terrain.setMaterial(assetManager.loadMaterial("Scenes/Maps/DefaultMap/default.j3m"));

            terrain.setLocalTranslation(0, -100, 0);
            rootNode.attachChild(terrain);
        }

        public static void main(String[] args) {
            HelloTerrain app = new HelloTerrain();
            app.start();
        }

    }

效果图：

![outscene](/content/images/2017/06/outscene.png)

## 扩展阅读

* [谈谈游戏中“水”效果的进化](https://tieba.baidu.com/p/4805119096)
* [拒绝被忽悠 游戏画面效果知识大扫盲](http://xielei-1026.blog.sohu.com/196161497.html)
* [游戏图像秘密大起底！一帧 3D 游戏画面是如何诞生的？](http://www.ipc.me/gta-v-graphics-study.html)
* [NVIDIA视觉特效](https://developer.nvidia.com/gameworks-visualfx-overview)
* [[英雄联盟]工程师讲述做场景渲染的全过程](http://www.gamelook.com.cn/2017/01/280474)
* [在顶级游戏开发的过程中需要怎样的编程实力？](https://www.zhihu.com/question/57582995)
* [好的游戏制作人需要对人性有哪些理解？](https://www.zhihu.com/question/46465078/answer/101566563)
* [《阴阳师》手游：为肝而肝](https://zhuanlan.zhihu.com/p/22435275)
