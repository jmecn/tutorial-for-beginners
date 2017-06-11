# 第五章：材质，障眼法

## 五色令人目盲

佛经将我们身处的世界称为“色界”，为什么呢？一则因为此界众生色欲重故；二则是此界众生主要靠眼睛看到的各种光影颜色来区分万物。

玩家想象中的3D世界其实根本就不存在，人们所看到的只是屏幕上的像素而已。程序员是专门欺骗玩家眼睛的大骗子，他们利用计算机辅助计算出屏幕上每一个像素的颜色，然后填充到屏幕上，让你产生错觉。

你看到了什么，一个三维方块？

![一个三维方块？](http://ww2.sinaimg.cn/bmiddle/005yeTZ6jw1eplpjobgrpg309p06xkjl.gif)

或者是一头瞪着你的龙？

![一只头瞪着你的龙？](http://ww4.sinaimg.cn/bmiddle/005yeTZ6jw1eplpjp4mepg307s053kjl.gif)

这些家具到底谁大谁小？

![近大远小？](http://ww4.sinaimg.cn/bmiddle/005yeTZ6jw1eplpjo6sxug308c064kjd.gif)

这就是本章的主题，障眼法。

## jME3的材质

在jME3中，一个3D模型可以是单个几何物体(Geometry)，也可能是由多个Geometry组成。Geometry是能够被渲染的最小的单元，网格(Mesh)定义了Geometry的形状，材质(Material)则决定了它的外表。

材质包含了用于描述物体表面的一切信息，诸如：

* 色彩
* 纹理(贴图)
* 反光度/光滑度
* 透明度
* 其他..

场景中的每一个Geometry都必须有一个Material对象来描述它的颜色或者纹理，否则引擎就不知道该怎么绘制这个物体。忘了给Geometry设置材质的下场就是收获下面这个异常：
`java.lang.IllegalStateException: No material is set for Geometry`

Material对象基于材质定义文件(.j3md，jME3 Material Definition)，例如Lighting.j3md和Unshaded.j3md。jME3的的材质系统基于着色器语言(GLSL)，支持各种材质库(shader library)。它的设计是用户友好的，只需要记住一些常用的材质和API就可以正常使用，并不会因为不懂GLSL而寸步难行。比如：

* 不需要光源照时，使用`Common/MatDefs/Misc/Unshaded.j3md`材质；
* 需要计算光照，使用`Common/MatDefs/Light/Lighting.j3md`材质；

jME3的材质系统相当灵活，而且扩展性很强，可以通过GLSL编程来实现所需要的任何特效。例如下图为应用了卡通边缘材质的滤镜效果。

![卡通边缘](/content/images/2017/04/PostCartoonEdge.png)

又比如PBR材质，只需要把PBR着色器的代码添加到assets中，再为其创建一个j3md文件来声明输入参数，就可以在jME3中产生这种效果。

![PBR光照](/content/images/2017/04/PBR.png)

libGdx、Unity3D等引擎底层也是基于相同的技术，这些引擎除了给Shader传递参数的方式不同，其他部分与jME3没任何差别。我在学习GLSL时经常逛[www.shadertoy.com](http://www.shadertoy.com)，遇到一些有趣的Shader就会在jME3中尝试一下。

### 实例：不同材质的效果

jME3使用`com.jme3.material.Material`来表示材质，我们先通过一段代码来看看Material是如何使用的。

    package net.jmecn;
    
    import com.jme3.app.SimpleApplication;
    import com.jme3.light.AmbientLight;
    import com.jme3.light.DirectionalLight;
    import com.jme3.material.Material;
    import com.jme3.math.ColorRGBA;
    import com.jme3.math.Quaternion;
    import com.jme3.math.Vector3f;
    import com.jme3.scene.Geometry;
    import com.jme3.scene.shape.Box;
    import com.jme3.scene.shape.Sphere;
    import com.jme3.texture.Texture;
    
    /**
     * 材质
     * @author yanmaoyuan
     *
     */
    public class HelloMaterial extends SimpleApplication {
    
        public static void main(String[] args) {
            // 启动程序
            HelloMaterial app = new HelloMaterial();
            app.start();
        }
        
        @Override
        public void simpleInitApp() {
            
            // 初始化摄像机位置
            cam.setLocation(new Vector3f(-3.06295f, 3.1202009f, 6.756448f));
            cam.setRotation(new Quaternion(0.036418974f, 0.94834185f, -0.11822353f, 0.29213792f));
            
            flyCam.setMoveSpeed(10);
            
            // 添加物体
            addUnshadedBox();
            addLightingBox();
            
            addUnshadedSphere();
            addLightingSphere();
            
            // 添加光源
            addLight();
            
            // 把窗口背景改成淡蓝色
            viewPort.setBackgroundColor(new ColorRGBA(0.6f, 0.7f, 0.9f, 1));
        }
        
        /**
         * 创造一个红色的小球，应用无光材质。
         * @return
         */
        private void addUnshadedSphere() {
            // #1 加载一个无光材质
            Material mat = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");
            
            // #2 设置参数
            mat.setColor("Color", ColorRGBA.Red);// 小球的颜色。
            
            // #3 创造1个球体，应用此材质。
            Geometry geom = new Geometry("普通球体", new Sphere(20, 40, 1));
            geom.setMaterial(mat);
            
            geom.move(4, 3, 0);
            rootNode.attachChild(geom);
        }
        
        /**
         * 创造一个红色的小球，应用受光材质。
         * @return
         */
        private void addLightingSphere() {
            // #1 加载一个受光材质
            Material mat = new Material(assetManager, "Common/MatDefs/Light/Lighting.j3md");
            
            // #2 设置参数
            mat.setColor("Diffuse", ColorRGBA.Red);// 在漫射光照射下反射的颜色。
            mat.setColor("Ambient", ColorRGBA.Red);// 在环境光照射下，反射的颜色。
            mat.setColor("Specular", ColorRGBA.White);// 镜面反射时，高光的颜色。
            
            // 反光度越低，光斑越大，亮度越低。
            mat.setFloat("Shininess", 32);// 反光度
            
            // 使用上面设置的Diffuse、Ambient、Specular等颜色
            mat.setBoolean("UseMaterialColors", true);
            
            // #3 创造1个球体，应用此材质。
            Geometry geom = new Geometry("文艺小球", new Sphere(20, 40, 1));
            geom.setMaterial(mat);
            
            geom.move(0, 3, 0);
            rootNode.attachChild(geom);
        }
        
        /**
         * 创造一个方块，应用无光材质。
         * @return
         */
        private void addUnshadedBox() {
            // #1 创建一个无光材质
            Material mat = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");
    
            // #2 加载一个纹理贴图，设置给这个材质。
            Texture tex = assetManager.loadTexture("Textures/Terrain/BrickWall/BrickWall.jpg");
            mat.setTexture("ColorMap", tex);// 设置贴图
            
            // #3 创造1个方块，应用此材质。
            Geometry geom = new Geometry("普通方块", new Box(1, 1, 1));
            geom.setMaterial(mat);
            
            geom.move(4, 0, 0);
            rootNode.attachChild(geom);
        }
        
        
        /**
         * 创造一个方块，应用受光材质。
         * @return
         */
        private void addLightingBox() {
            // #1 创建一个无光材质
            Material mat = new Material(assetManager, "Common/MatDefs/Light/Lighting.j3md");
    
            // #2 设置纹理贴图
            // 漫反射贴图
            Texture tex = assetManager.loadTexture("Textures/Terrain/BrickWall/BrickWall.jpg");
            mat.setTexture("DiffuseMap", tex);
            
            // 法线贴图
            tex = assetManager.loadTexture("Textures/Terrain/BrickWall/BrickWall_normal.jpg");
            mat.setTexture("NormalMap", tex);
            
            // 视差贴图
            tex = assetManager.loadTexture("Textures/Terrain/BrickWall/BrickWall_height.jpg");
            mat.setTexture("ParallaxMap", tex);
    
            // 设置反光度
            mat.setFloat("Shininess", 2.0f);
            
            // #3 创造1个方块，应用此材质。
            Geometry geom = new Geometry("文艺方块", new Box(1, 1, 1));
            geom.setMaterial(mat);
            
            rootNode.attachChild(geom);
        }
        
        /**
         * 添加光源
         */
        private void addLight() {
            // 定向光
            DirectionalLight sun = new DirectionalLight();
            sun.setDirection(new Vector3f(-1, -2, -3));
    
            // 环境光
            AmbientLight ambient = new AmbientLight();
    
            // 调整光照亮度
            ColorRGBA lightColor = new ColorRGBA();
            sun.setColor(lightColor.mult(0.8f));
            ambient.setColor(lightColor.mult(0.2f));
            
            // #3 将模型和光源添加到场景图中
            rootNode.addLight(sun);
            rootNode.addLight(ambient);
        }
    }

运行结果如下：

![不同材质的渲染效果](/content/images/2017/04/Materials.png)

*补充说明：有些同学找不到上面代码中的BrickWall.jpg在哪里。代码中用的几张砖块的贴图，来自jME3的测试数据，在官方github仓库的[jme3-testdata目录](https://github.com/jMonkeyEngine/jmonkeyengine/tree/master/jme3-testdata/src/main/resources/Textures/Terrain/BrickWall)下可以找到这些文件。如果你下载了jME3.1-stable或者jME3 SDK，就可以直接在jME3的lib文件夹下搜索到jme3-testdata.jar。*

## 加载j3md材质

jME3中的Material对象代表了一个着色器(Shader)程序，我们不能像实例化一个普通的Java对象一样直接new出来，而是需要通过assetManager来加载材质定义文件。

    Material mat = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");

从上面的代码可以看出，实例化一个Material对象需要2个参数：

 * assetManager对象，用于加载材质资源。
 * `Common/MatDefs/Misc/Unshaded.j3md`，材质定义文件`.j3md`的路径。

创建材质对象后，使用Geometry对象的setMaterial方法来应用这个材质。

    geom.setMaterial(mat);

## 改变材质参数

由于Material实质上代表着一个Shader程序，而所有的参数都是由Shader在Java程序外部定义的，不同的着色器可能有不同的参数。因此Material并没有setDiffuse或setAmbient之类的固定方法。Material提供了一组API，让我们通过这些方法来给Shader传参。

    public void setFloat(String name, float value)
    public void setBoolean(String name, boolean value)
    public void setColor(String name, ColorRGBA value)
    public void setTexture(String name, Texture value)
    public void setVector2(String name, Vector2f value)
    public void setVector3(String name, Vector3f value)
    public void setVector4(String name, Vector4f value)
    public void setMatrix4(String name, Matrix4f value)
    public void setParam(String name, VarType type, Object value) 

如果想查询材质中的参数，同样也必须根据Shader定义的参数名字来查询。

    public MatParam getParam(String name)

在`Common/MatDefs/Misc/Unshaded.j3md`中，通常使用下面的参数：

* Color : ColorRGBA，表示模型颜色
* ColorMap : Texture，表示模型表面的纹理贴图
* LightMap : Texture，表示烘焙的亮度贴图
* UseSperatedTex

在`Common/MatDefs/Light/Lighting.j3md`中，通常使用下面这些参数：

* Ambient : ColorRGBA，环境光色
* Diffuse : ColorRGBA，漫反射颜色
* Specular : ColorRGBA，高光颜色
* UseMaterialColors : Boolean，是否使用材质中定义的颜色。
* Shininess : Float，反光度。1.0最低，128最高。
* DiffuseMap : Texture2D，漫反射贴图
* NormalMap : Texture2D，法线贴图
* SpecularMap : Texture2D，高光贴图

由于Lighting材质要计算光照，因此参数要比Unshaded复杂许多。通过上文的例子可以看出，同样是设置颜色，addLightingSphere()要比addUnshadedSphere()复杂。

Lighting材质

    // #1 加载一个受光材质
    Material mat = new Material(assetManager, "Common/MatDefs/Light/Lighting.j3md");
    // #2 设置参数
    mat.setColor("Diffuse", ColorRGBA.Red);// 在漫射光照射下反射的颜色。
    mat.setColor("Ambient", ColorRGBA.Red);// 在环境光照射下，反射的颜色。
    mat.setColor("Specular", ColorRGBA.White);// 镜面反射时，高光的颜色。
    // 反光度越低，光斑越大，亮度越低。
    mat.setFloat("Shininess", 32);// 反光度
    // 使用上面设置的Diffuse、Ambient、Specular等颜色
    mat.setBoolean("UseMaterialColors", true);

Unshaded材质

    // #1 加载一个无光材质
    Material mat = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");
    // #2 设置参数
    mat.setColor("Color", ColorRGBA.Red);// 小球的颜色。

### 色彩

在jME3中，我们使用ColorRGBA来描述色彩，通过4个float变量来构造一个ColorRGBA对象，每个浮点数的取值范围都是0~1。即红(Red)、绿(Green)、蓝(Blue)三元色加一个Alpha通道。RGB定义一种具体的颜色，A用于表示透明度或者亮度。

在很多系统中，比如网页样式CSS和Java的AWT组件，经常使用三个字节来表示三元色，每个字节可以有256种状态(0~255)，以此来描述24位真彩色。例如红色可以写成：`(255, 0, 0)`或`#FF0000`；白色为`(255, 255, 255)`或`#FFFFFF`；黑色为`(0, 0, 0)`或`#000000`。

为了满足人们对色彩无尽的追求，在着色器程序中每种颜色只有256种状态是不够用的。GPU在计算颜色的过程中会使用浮点数运算，这样色彩会更加丰富。而且由于计算机性能的问题，float类型既能满足人眼对颜色精确度的要求，运算速度又比double类型快很多，因此3D图形引擎通常使用float类型的浮点数来表示颜色。

ColorRGBA中定义了一些常用的颜色，通过这些颜色的定义可以快速了解ColorRGBA。

    /**
     * 黑色 (0,0,0).
     */
    public static final ColorRGBA Black = new ColorRGBA(0f, 0f, 0f, 1f);
    /**
     * 白色 (1,1,1).
     */
    public static final ColorRGBA White = new ColorRGBA(1f, 1f, 1f, 1f);
    /**
     * 深灰色 (.2,.2,.2).
     */
    public static final ColorRGBA DarkGray = new ColorRGBA(0.2f, 0.2f, 0.2f, 1.0f);
    /**
     * 灰色 (.5,.5,.5).
     */
    public static final ColorRGBA Gray = new ColorRGBA(0.5f, 0.5f, 0.5f, 1.0f);
    /**
     * 淡灰色 (.8,.8,.8).
     */
    public static final ColorRGBA LightGray = new ColorRGBA(0.8f, 0.8f, 0.8f, 1.0f);
    /**
     * 红色 (1,0,0).
     */
    public static final ColorRGBA Red = new ColorRGBA(1f, 0f, 0f, 1f);
    /**
     * 绿色 (0,1,0).
     */
    public static final ColorRGBA Green = new ColorRGBA(0f, 1f, 0f, 1f);
    /**
     * 蓝色 (0,0,1).
     */
    public static final ColorRGBA Blue = new ColorRGBA(0f, 0f, 1f, 1f);
    /**
     * 黄色 (1,1,0).
     */
    public static final ColorRGBA Yellow = new ColorRGBA(1f, 1f, 0f, 1f);
    /**
     *紫色 (1,0,1).
     */
    public static final ColorRGBA Magenta = new ColorRGBA(1f, 0f, 1f, 1f);
    /**
     * 青色 (0,1,1).
     */
    public static final ColorRGBA Cyan = new ColorRGBA(0f, 1f, 1f, 1f);
    /**
     * 橘黄色 (251/255, 130/255,0).
     */
    public static final ColorRGBA Orange = new ColorRGBA(251f / 255f, 130f / 255f, 0f, 1f);
    /**
     * 棕色 (65/255, 40/255, 25/255).
     */
    public static final ColorRGBA Brown = new ColorRGBA(65f / 255f, 40f / 255f, 25f / 255f, 1f);
    /**
     * 粉色 (1, 0.68, 0.68).
     */
    public static final ColorRGBA Pink = new ColorRGBA(1f, 0.68f, 0.68f, 1f);
    /**
     * 纯黑无光 (0, 0, 0, 0).
     */
    public static final ColorRGBA BlackNoAlpha = new ColorRGBA(0f, 0f, 0f, 0f);


### 纹理映射

把图片直接贴在模型的表面是另一种给模型上色的方式，称为**纹理映射**。jME3使用`com.jme3.texture.Texture`对象来表示纹理，通过assetManager可以把jpg/png/bmp/tga/dds等多种格式的图片加载为纹理。

在前文实例代码的`addUnshadedBox`方法中，我们先加载*BrickWall.jpg*文件作为纹理，然后通过*ColorMap*变量设置给了Unshaded材质。

    // #1 创建一个无光材质
    Material mat = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");
    // #2 加载一个纹理贴图，设置给这个材质。
    Texture tex = assetManager.loadTexture("Textures/Terrain/BrickWall/BrickWall.jpg");
    mat.setTexture("ColorMap", tex);// 设置纹理贴图

### 表面法线

表面法线可以增加模型表面的细节，在jME3中有2种方式来为模型增加表面法线。

* 计算顶点法线，然后将其保存到Mesh对象中，对应的类型为Type.Normal。
* 制作法线贴图，然后将法线贴图应用到材质中。

如果使用第一种方式，而且模型本身没有法线，可以使用jme3的工具类`com.jme3.util.TangentBinormalGenerator`来计算顶点法线`TangentBinormalGenerator.generate(mesh);`。

法线贴图通常称为NormalMap或BumpMap。[wiki百科](https://en.wikipedia.org/wiki/Bump_mapping)

![](/content/images/2017/04/400px-Bump-map-demo-full.png)

使用第二种方式的话，可以使用`CrazyBump`等工具制作法线贴图，然后将其应用到Lighting.j3md材质中。

Lighting.j3md的`NormalMap`就是用来定义法线贴图的参数的。

    Texture normalMap = assetManager.loadTexture("Textures/Terrain/BrickWall/BrickWall_normal.jpg");
    mat.setTexture("NormalMap", normalMap );

### 光泽度

Lighting.j3md材质可以表现模型的光泽度，Unshaded.j3md是没有光泽的。

想要使用这种效果，首先要设定材质的`Shininess`参数，取值范围是1到128：1表示模型表面很粗糙，没有光泽；128表示模型表面和镜子一样光滑。

其次，需要设置模型的颜色(Diffuse)和光斑的颜色(Specular)，并且将`UseMaterialColors`设为true。

    mat.setFloat("Shininess", 5f);
    mat.setColor("Diffuse",ColorRGBA.Red);
    mat.setColor("Specular",ColorRGBA.White);
    mat.setBoolean("UseMaterialColors",true);

光斑颜色(Specular)应该和光源的颜色一致，白色(ColorRGBA.White)是最常使用的颜色。

如果不想让模型表面有光斑，你需要把高光颜色(Specular)设成纯黑(ColorRGBA.Black)，而不仅仅是把光泽度(Shininess)参数设为0。

    mat.setColor("Specular",ColorRGBA.Black);

下面是一个具体的例子。

    package net.jmecn.material;
    
    import com.jme3.app.SimpleApplication;
    import com.jme3.light.AmbientLight;
    import com.jme3.light.DirectionalLight;
    import com.jme3.material.Material;
    import com.jme3.math.ColorRGBA;
    import com.jme3.math.Vector3f;
    import com.jme3.scene.Geometry;
    import com.jme3.scene.shape.Sphere;
    
    /**
     * 测试Lighting.j3md的Shininess参数
     * 
     * @author yanmaoyuan
     *
     */
    public class TestShininess extends SimpleApplication {
    
        @Override
        public void simpleInitApp() {
            flyCam.setMoveSpeed(10);
            
            addLight();
    
            for(float shininess = 1, x= 0; shininess <= 128; shininess += 32f, x += 2.5f) {
                Geometry geom = createSphere(shininess);
                rootNode.attachChild(geom);
                geom.move(x, 0, 0);
            }
        }
        
        private Geometry createSphere(float shininess) {
            // 加载一个受光材质
            Material mat = new Material(assetManager, "Common/MatDefs/Light/Lighting.j3md");
            mat.setColor("Diffuse", ColorRGBA.Red);// 在漫射光照射下反射的颜色。
            mat.setColor("Ambient", ColorRGBA.Red);// 在环境光照射下，反射的颜色。
            
            mat.setColor("Specular", ColorRGBA.White);// 镜面反射时，高光的颜色。
            mat.setFloat("Shininess", shininess);// 光泽度，取值范围1~128。
            
            // 使用上面设置的Diffuse、Ambient、Specular等颜色
            mat.setBoolean("UseMaterialColors", true);
            
            // 创造1个球体，应用此材质。
            Geometry geom = new Geometry("小球", new Sphere(40, 36, 1));
            geom.setMaterial(mat);
            
            return geom;
        }
    
        /**
         * 添加光源
         */
        private void addLight() {
            // 定向光
            DirectionalLight sun = new DirectionalLight();
            sun.setDirection(new Vector3f(-1, -2, -3));
    
            // 环境光
            AmbientLight ambient = new AmbientLight();
    
            // 调整光照亮度
            ColorRGBA lightColor = new ColorRGBA();
            sun.setColor(lightColor.mult(0.8f));
            ambient.setColor(lightColor.mult(0.2f));
            
            rootNode.addLight(sun);
            rootNode.addLight(ambient);
        }
        
        public static void main(String[] args) {
            TestShininess app = new TestShininess();
            app.start();
        }
    
    }

![光泽度](/content/images/2017/04/Materials_shininess.png)

**(可选) 高光贴图**

高光贴图(SpecularMap)是另一种手段，使用它可以指定模型表面哪些部位光泽度高（白色或浅灰色），哪些部位光泽度低（黑色或深灰色）。不使用SpecularMap的话，模型整个表面看起来都是反光的。

    mat.setTexture("SpecularMap", assetManager.loadTexture("Textures/metal_spec.png")); // Lighting.j3md


### 发光

Lighting.j3md和Unshaded.j3md都可以让模型看起来在发光，做法是这样的：

首先，在摄像机的视口(viewPort)中插入一块发光滤镜([BloomFilter PostProcessor](https://jmonkeyengine.github.io/wiki/jme3/advanced/bloom_and_glow.html))，这将对视野中的所有发光物体生效。

**注意：BloomFilter只需要添加一次，不要重复添加！**

    FilterPostProcessor fpp=new FilterPostProcessor(assetManager);
    BloomFilter bloom = new BloomFilter(BloomFilter.GlowMode.Objects);
    fpp.addFilter(bloom);
    viewPort.addProcessor(fpp);

然后，改变材质中的`GlowColor`参数，使用ColorRGBA来设定物体的发光颜色。你可以根据自己的喜好来设定冷暖色调，一般来说白色看起来比较自然。

    mat.setColor("GlowColor",ColorRGBA.White);

不想让物体发光的话可以把GlowColor设为纯黑(ColorRGBA.Black)。

    mat.setColor("GlowColor", ColorRGBA.Black);

**(可选) 使用发光贴图(GlowMap)。**

这种贴图将制定模型表面哪些部位发光，黑色区域表示不发光，其他区域将根据图像颜色来发光。如果不用GlowMap的话，整个模型看起来都会发光。

    mat.setTexture("GlowMap", assetManager.loadTexture("Textures/alien_glow.png"));

漫反射贴图 DiffuseMap
![漫反射贴图](/content/images/2017/04/tank_diffuse_ss.png)

发光贴图 GlowMap
![发光贴图](/content/images/2017/04/tank_glow_map_ss.png)

最终效果
![最终效果](/content/images/2017/04/tanlglow1.png)

Learn more about [Bloom and Glow](https://jmonkeyengine.github.io/wiki/jme3/advanced/bloom_and_glow.html)

### 透明物体

想让场景中的物体看起来是半透明或者完全透明，做法是这样的。

(1)改变ColorRGBA的第4个值：`Alpha`

对于Unshaded.j3md材质，通过`Color`变量来改变透明度。

    mat.setColor("Color", new ColorRGBA(1, 1, 1, 0.5f));

对于Lighting.j3md材质，通过`Diffuse`变量来改变透明度。

    mat.setColor("Diffuse", new ColorRGBA(1, 1, 1, 0.5f));

Alpha的取值范围是0~1，默认值为1。

* Alpha = 0，表示完全透明(transparent)
* Alpha = 1，表示不透明(opaque)
* Alpha的值在0~1之间，会让物体看起来半透明(translucent)。

(2)将材质的混色模式设置为`BlendMode.Alpha`。

    mat.getAdditionalRenderState().setBlendMode(BlendMode.Alpha);

(3)更改Geometry的渲染序列，根据Alpha的值将其设为`Bucket.Translucent`或者`Bucket.Transparent`。

    geom.setQueueBucket(Bucket.Transparent);

* `Bucket.Translucent`模式：不会被SceneProcessor影响，经常用于粒子特效。
* `Bucket.Transparent`模式：会被SceneProcessor影响，比如可以产生影子。

下面是一个简单的例子：

    package net.jmecn.material;
    
    import com.jme3.app.SimpleApplication;
    import com.jme3.light.AmbientLight;
    import com.jme3.light.DirectionalLight;
    import com.jme3.material.Material;
    import com.jme3.material.RenderState.BlendMode;
    import com.jme3.math.ColorRGBA;
    import com.jme3.math.Vector3f;
    import com.jme3.renderer.queue.RenderQueue.Bucket;
    import com.jme3.scene.Geometry;
    import com.jme3.scene.shape.Quad;
    import com.jme3.scene.shape.Sphere;
    
    /**
     * 测试透明物体
     * 
     * @author yanmaoyuan
     *
     */
    public class TestAlpha extends SimpleApplication {
    
        @Override
        public void simpleInitApp() {
            flyCam.setMoveSpeed(10);
    
            createQpaqueSphere();
            createTranslucentSphere();
            createTranslucentQuad();
            
            addLight();
        }
        
        /**
         * 创造一个不透明的红色小球
         * @return
         */
        private Geometry createQpaqueSphere() {
            // 加载一个受光材质
            Material mat = new Material(assetManager, "Common/MatDefs/Light/Lighting.j3md");
            mat.setColor("Diffuse", ColorRGBA.Red);
            mat.setColor("Ambient", ColorRGBA.Red);
            mat.setColor("Specular", ColorRGBA.White);
            mat.setFloat("Shininess", 16f);// 光泽度，取值范围1~128。
            mat.setBoolean("UseMaterialColors", true);
            
            // 应用材质
            Geometry geom = new Geometry("不透明的红色小球", new Sphere(40, 36, 1));
            geom.setMaterial(mat);
            
            rootNode.attachChild(geom);
            return geom;
        }
    
        /**
         * 创造一个半透明的青色小球
         * @return
         */
        private void createTranslucentSphere() {
            // 加载一个受光材质
            Material mat = new Material(assetManager, "Common/MatDefs/Light/Lighting.j3md");
            mat.setColor("Diffuse", new ColorRGBA(0, 1, 1, 0.5f));
            mat.setColor("Ambient", ColorRGBA.Cyan);
            mat.setColor("Specular", ColorRGBA.White);
            mat.setFloat("Shininess", 16f);
            mat.setBoolean("UseMaterialColors", true);
            
            // 创造1个球体，应用此材质。
            Geometry geom = new Geometry("半透明的青色小球", new Sphere(40, 36, 1));
            geom.setMaterial(mat);
            
            // 使小球看起来透明
            mat.getAdditionalRenderState().setBlendMode(BlendMode.Alpha);
            geom.setQueueBucket(Bucket.Transparent);
            
            geom.move(0, 0, 3);
            rootNode.attachChild(geom);
        }
        
        /**
         * 创造一个半透明的白色正方形
         */
        private void createTranslucentQuad() {
            // 加载一个无光材质
            Material mat = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");
            mat.setColor("Color", new ColorRGBA(1f, 1f, 1f, 0.5f));// 镜面反射时，高光的颜色。
            
            // 应用材质。
            Geometry geom = new Geometry("一个半透明的正方形", new Quad(1, 1));
            geom.setMaterial(mat);
            
            // 使物体看起来透明
            mat.getAdditionalRenderState().setBlendMode(BlendMode.Alpha);
            geom.setQueueBucket(Bucket.Transparent);
            
            geom.move(-1, -1, 5);
            rootNode.attachChild(geom);
        }
    
        /**
         * 添加光源
         */
        private void addLight() {
            // 定向光
            DirectionalLight sun = new DirectionalLight();
            sun.setDirection(new Vector3f(-1, -2, -3));
    
            // 环境光
            AmbientLight ambient = new AmbientLight();
    
            // 调整光照亮度
            ColorRGBA lightColor = new ColorRGBA();
            sun.setColor(lightColor.mult(0.8f));
            ambient.setColor(lightColor.mult(0.2f));
            
            rootNode.addLight(sun);
            rootNode.addLight(ambient);
        }
        
        public static void main(String[] args) {
            TestAlpha app = new TestAlpha();
            app.start();
        }
    
    }

效果如下。

![透明度](/content/images/2017/04/Materials_alpha.png)

**(可选)AlphaMap**

使用AlphaMap可以指定物体表面各个部位的透明度，否则整体都是透明的。AlphaMap经常用于伪造不规则形状，比如血条、装备栏之类的。

    Texture tex = assetManager.loadTexture("Textures/align_alpha.png");
    mat.setTexture("AlphaMap", tex);

### 透明纹理

有些纹理贴图(DiffuseMap)本身就是带有透明度通道的，比如jme3-testdata中有一个Monkey.png文件，它比一般的jpg文件多了alpha通道。这个图片中有些部分是不透明的，有些是半透明的，有些是完全透明的。为了正确显示纹理中的透明度信息，需要进行如下操作：

1. 告诉材质你要开启Alpha通道`mat.setBoolean("UseAlpha",true);`。
2. 将材质的混色模式设置为BlendMode.Alpha。`mat.getAdditionalRenderState().setBlendMode(BlendMode.Alpha);`
3. 将Geometry添加到透明物体队列，以确保正确的绘制循序。`geom.setQueueBucket(Bucket.Transparent);`

下面是一个实例代码，创建一个方块，把Monkey.png设置成它的纹理贴图。

    /**
     * 添加一个半透明纹理
     */
    private void addTransparentQuad() {
        // 加载无光材质。
        Material mat = new Material(assetManager,
                "Common/MatDefs/Misc/Unshaded.j3md");
        // 加载一个带透明度通道的纹理
        mat.setTexture("ColorMap",
                assetManager.loadTexture("Textures/ColoredTex/Monkey.png"));
        
        Geometry geom = new Geometry("window frame", new Quad(4, 4));
        geom.setMaterial(mat);
        
        geom.move(0, 0, 4);
        rootNode.attachChild(geom);
        
        // 将材质的混色模式设置为：BlendMode.Alpha
        mat.getAdditionalRenderState().setBlendMode(BlendMode.Alpha);// 重要!
        // 将Geometry的渲染序列设置为Transparent，这将使它在其他不透明物体绘制后再绘制。
        geom.setQueueBucket(Bucket.Transparent);                     // 重要!
        
    }

运行结果如下，透过猴子的脸可以看到后面的物体。

![透明材质](/content/images/2017/04/Materials_transparent.png)

### 线框模式

通过如下方式可以让材质以wireframe方式绘制模型。

    mat.getAdditionalRenderState().setWireframe(true);

这种模式可以直接观察到网格的真实形态，通常用于开发调试。

![渲染模式](/content/images/2017/04/Material_no_wireframe.png)

![线框模式](/content/images/2017/04/Material_wireframe.png)

## 扩展阅读：UV坐标

**纹理映射是怎么工作的？**

3D模型使用纹理坐标(TexCoords)来将网格中的每个点与图片上的点对应起来，以此完成贴图。纹理坐标又称为UV坐标，指的是u,v纹理贴图坐标的简称(它和空间模型的X, Y, Z轴是类似的)。

虽然艾希肯定不希望你看到她这副样子，但是为了让你能够迅速理解纹理映射，我决定牺牲她。

![艾希的纹理](/content/images/2017/04/b_ash.jpg)

这幅图跟我们之前看到的艾希有很大的区别。

![艾希的模型](/content/images/2017/04/Ashe_texture.png)

打开艾希的网格数据文件`b_Ashe.obj`，可以找到类似下面的数据，这就是纹理坐标。

    vt 0.975111 0.009289
    vt 0.975112 0.421428
    vt 0.915536 0.076851
    vt 0.983578 0.009289
    vt 0.983578 0.009289
    vt 0.992046 0.009289
    vt 0.966643 0.009289

纹理坐标采用UV坐标系，u、v的值是按比例计算的，使用2个float类型的浮点数来表示。无论图片本身的分辨率是多大，u、v的标准取值范围都是0.0~1.0。

*具体关于uv坐标的概念，请自行百度，下文不再详细叙述。*

**在jME3中，我们使用Type.TexCoord在Mesh中存储纹理坐标，然后通过给材质设置Texture来完成纹理映射。**

下面我们对上一章的六边形网格增加纹理坐标，通过这个例子来理解纹理坐标的作用。

    package net.jmecn;
    
    import com.jme3.app.SimpleApplication;
    import com.jme3.material.Material;
    import com.jme3.math.ColorRGBA;
    import com.jme3.math.Quaternion;
    import com.jme3.math.Vector3f;
    import com.jme3.scene.Geometry;
    import com.jme3.scene.Mesh;
    import com.jme3.scene.VertexBuffer.Type;
    import com.jme3.scene.debug.Arrow;
    import com.jme3.texture.Texture;
    import com.jme3.util.BufferUtils;
    
    /**
     * 自定义网格，制作一个六边形。
     * 
     * @author yanmaoyuan
     *
     */
    public class HelloMesh extends SimpleApplication {
    
        @Override
        public void simpleInitApp() {
            cam.setLocation(new Vector3f(4.893791f, 4.5420675f, 9.626116f));
            cam.setRotation(new Quaternion(-0.031222044f, 0.9664778f, -0.14307737f, -0.21089031f));
    
            flyCam.setMoveSpeed(10);
    
            // 创建六边形
            createHex();
    
            // 创建X、Y、Z方向的箭头，作为参考坐标系。
            createArrow(new Vector3f(5, 0, 0), ColorRGBA.Green);
            createArrow(new Vector3f(0, 5, 0), ColorRGBA.Red);
            createArrow(new Vector3f(0, 0, 5), ColorRGBA.Blue);
            
            viewPort.setBackgroundColor(ColorRGBA.LightGray);
        }
    
        /**
         * 创建一个六边形
         */
        private void createHex() {
            // 六个顶点
            float[] vertex = {
                    2.5f, 4f, 0f, // 零
                    1f, 3.26f, 0f,// 壹
                    1f, 1.74f, 0f,// 贰
                    2.5f, 1f, 0f, // 叁
                    4f, 1.74f, 0f,// 肆
                    4f, 3.26f, 0f // 伍
            };
    
            // 纹理坐标
            float[] texCoords = new float[] {
                   0.5f, 0.75f,  // 零
                   0.25f, 0.625f,// 壹
                   0.25f, 0.375f,// 贰
                   0.5f, 0.25f,  // 叁
                   0.75f, 0.375f,// 肆
                   0.75f, 0.625f // 伍
            };
            
            // 四个三角形
            int[] indices = new int[] {
                    0, 1, 2, // 三角形0
                    2, 3, 4, // 三角形1
                    4, 5, 0, // 三角形2
                    0, 2, 4 // 三角形3
            };
    
            // 创建网格
            Mesh mesh = new Mesh();
            // 保存顶点位置和顶点索引
            mesh.setBuffer(Type.Position, 3, BufferUtils.createFloatBuffer(vertex));
            mesh.setBuffer(Type.TexCoord, 2, BufferUtils.createFloatBuffer(texCoords));
            mesh.setBuffer(Type.Index, 1, BufferUtils.createIntBuffer(indices));
    
            mesh.updateBound();
            mesh.setStatic();
    
            // 创建材质，使我们可以看见这个六边形
            Material mat = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");
            // mat.getAdditionalRenderState().setWireframe(true);
    
            // 设置纹理贴图
            Texture tex = assetManager.loadTexture("Models/Hexagon/hex.png");
            mat.setTexture("ColorMap", tex);
            
            // 使用网格和材质创建一个物体
            Geometry geom = new Geometry("六边形");
            geom.setMesh(mesh);
            geom.setMaterial(mat);
            geom.center();
    
            // 将物体添加到场景图中
            rootNode.attachChild(geom);
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
    
            // 添加到场景中
            rootNode.attachChild(geom);
        }
    
        public static void main(String[] args) {
            // 启动程序
            HelloMesh app = new HelloMesh();
            app.start();
        }
    }

效果如下。

![](/content/images/2017/04/TexCoord.png)
