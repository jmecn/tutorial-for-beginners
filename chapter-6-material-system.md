# 第六章：材质系统

> 作者：这一章内容不属于jME3入门必须弄懂的知识，如果你对jME3的材质系统工作原理感兴趣可以继续往下阅读，直接跳过下方内容也不影响开发。
> 
> 但我建议至少读完“材质实例：j3m文件”，因为这会帮助你理解jME3 SDK中的材质编辑器的作用，并且在一定程度上减少编写Java代码的工作量，

jME3的材质系统定义了2种文件格式：j3md和j3m。j3md来关联材质所使用的着色器程序，充当Java代码与Shader代码中间的粘合剂。j3m来保存一个具体的材质参数。

如果你对源代码感兴趣，可以去查看这2个组件中的文件：

* `jme3-core`
 * 其中内置了一些最常用的`j3md`文件以及对应的着色器代码，Unshaded.j3md和Lighting.j3md正是其中的一部分。
* `jme3-effects`
 * 包含了SSAO、卡通边缘、水面反射、粒子等多种特效文件。

## 材质系统

### jME3与Shader

着色器(Shader)是运行在GPU上的程序，它可以让图形引擎获得GPU硬件加速渲染能力。现代3D图形引擎基本上都支持使用着色器语言，允许开发者编程实现各种奇妙的图形渲染效果。

根据语言来划分，有：

* 低级语言（汇编风格）
 * LLSL(Low Level Shading Language)
 * AGAL(Adobe Graphics Assembly Language)
* 高级语言（C语言风格）
 * CG Nvidia显卡
 * GLSL OpenGL
 * HLSL DirectX

根据用途来划分，有：

* VertexShader 顶点着色器
 * **顶点着色器**将执行于网格中的每一个顶点，用途是计算**顶点在屏幕中的位置**。不过实际上只会计算用户在屏幕上能看见的顶点，看不见的都被剔除了。
* FragmentShader/PixelShader 片元着色器/象素着色器
 * **片元着色器**将执行于屏幕上的每一个像素点，用途是计算**像素的颜色**。
* Tessellation Shader 细分着色器
 * **细分着色器**可以把现有模型的三角形拆分得更细小、更细致，从而使得渲染对象的表面和边缘更平滑、更精细。
* GeometryShader 几何着色器
 * **几何着色器**可以增加/减少/编辑模型中的顶点数量，能够改变模型的形态，比如让一个光滑的表面“长毛”。

jME3基于OpenGL，使用GLSL着色器语言来实现3D图形渲染。在jME3中，不同的着色器程序文件采用不同的后缀名。

* 顶点着色器 .vert
* 几何着色器 .geom
* 片元着色器 .frag

在j3md文件中引用它们非常简单。

    MaterialDef SimpleGeom {
        MaterialParameters {
        }
        Technique {
            VertexShader GLSL330:   Materials/Geom/SimpleGeom.vert
            GeometryShader GLSL330: Materials/Geom/SimpleGeom.geom
            FragmentShader GLSL330: Materials/Geom/SimpleGeom.frag

            WorldParameters {
                WorldViewProjectionMatrix
            }
        }
    }

考虑到用户显卡的功能千奇百怪，因此顶点着色器和片元着色器是jME3应用最广泛的两种着色器。几何着色器是OpenGL 3.2后才有的功能，而细分着色器则是OpenGL 4.0以后才有的功能，使用这2种着色器一定要在Technique中声明GLSL的最低版本。

着色器在jME3中执行的过程是这样的：

1. jME3主程序将Mesh数据提交给GPU
2. 顶点着色器计算出顶点在屏幕中的位置
3. 片元着色器计算出屏幕上每个像素的颜色
4. 显示到屏幕上(或者渲染到纹理)

着色器编程是一门专业学科，属于计算机图形学范畴，本文不再细述。如果你对GLSL感兴趣，推荐一本书：

    书名：图形着色器——理论与实践(第2版)
    作者：Mike Bailey, Steve Cunnungham
    翻译：刘鹏
    出版社：清华大学出版社
    ISBN：978-7-302-31599-5

### 模板与实例

材质是一个模板，通过实例化某个材质并设置不同的参数和贴图，就可以让物体表现出不同的显示效果。j3md文件定义的是材质模板，j3m文件保存的则是实例数据。

例如Lighting.j3md就是一个模板，我们使用它来实例化多个Material对象。改变每个实例的参数，就能得到不同的效果。

![模板和实例](/content/images/2017/04/Materials_shininess.png)

### 多Technique

多Technique是指一个材质中，应该包含不只一个实现方案。这样当我们进行材质更替或者进行高中低端机适配的时候，就不会那么麻烦，同时在数据管理上也显得更为规范。

例如Android手机上使用的GLES，默认是不支持光影的，想显示影子就不能用和OpenGL 4.0一样的方式来实现。

**高中低端机适配**

高中低端机适配是一个很重要的特性，因为玩家的机型不可能是一样的。高档显卡应该能够绘制出更精彩的效果，中低端显卡则可以适当减少不必要的特效。

在需要保证效率的情况下，很多时候需要降低物体渲染的复杂度。具体实现则有两种方案：一种是通过宏定义，一种是动态切换Technique。

**方法一：宏定义**

下例是Unshaded.j3md中的一个`Technique`，`Defines`块中定义了很多宏，它们的作用就像开关一样，Shader代码将根据这些宏是否存在来决定怎么绘制图像。

    Technique {
        VertexShader GLSL100 GLSL150:   Common/MatDefs/Misc/Unshaded.vert
        FragmentShader GLSL100 GLSL150: Common/MatDefs/Misc/Unshaded.frag

        WorldParameters {
            WorldViewProjectionMatrix
            ViewProjectionMatrix
            ViewMatrix
        }

        Defines {
            INSTANCING : UseInstancing
            SEPARATE_TEXCOORD : SeparateTexCoord
            HAS_COLORMAP : ColorMap
            HAS_LIGHTMAP : LightMap
            HAS_VERTEXCOLOR : VertexColor
            HAS_COLOR : Color
            NUM_BONES : NumberOfBones
            DISCARD_ALPHA : AlphaDiscardThreshold
        }
    }

对应的顶点着色器中使用这些宏来调整代码结构。

    #if defined(HAS_GLOWMAP) || defined(HAS_COLORMAP) || (defined(HAS_LIGHTMAP) && !defined(SEPARATE_TEXCOORD))
        #define NEED_TEXCOORD1
    #endif

    #if defined(DISCARD_ALPHA)
        uniform float m_AlphaDiscardThreshold;
    #endif
    // ...

**方法二：动态切换**

下方为jME3自带的Gui材质，它将根据用户显卡支持的GLSL版本来决定Technique是否生效。

    MaterialDef Default GUI {
        MaterialParameters {
            Texture2D Texture
            Color Color (Color)
            Boolean VertexColor (UseVertexColor)
        }
        Technique {
            VertexShader GLSL150:   Common/MatDefs/Gui/Gui.vert
            FragmentShader GLSL150: Common/MatDefs/Gui/Gui.frag
            // 略..
        }
        Technique {
            VertexShader GLSL100:   Common/MatDefs/Gui/Gui.vert
            FragmentShader GLSL100: Common/MatDefs/Gui/Gui.frag
            // 略..
        }
    }

### 多Pass

有时为了实现一种绘制效果，靠单次绘制是无法实现的，这就要求系统能够多次向GPU提交材质来绘制一个物体。例如阴影，首先要完成3D场景的渲染，然后再根据光源和物体之间的位置关系绘制阴影。

在jME3的材质支持多Technique，每个Technique描述一种绘制物体的不同方式（等于是一个Pass）。下面是`Unshaded.j3md`中的部分代码，可以看到这个材质一共有5个Pass，有些Technique就是专门用来绘制阴影的，有些则是用来绘制发光效果的。

    MaterialDef Unshaded {
        MaterialParameters {
            // 参数..
        }
        // 绘制颜色、纹理
        Technique {
            VertexShader GLSL100 GLSL150:   Common/MatDefs/Misc/Unshaded.vert
            FragmentShader GLSL100 GLSL150: Common/MatDefs/Misc/Unshaded.frag
            // 略..
        }
        // 计算法线
        Technique PreNormalPass {
            VertexShader GLSL100 GLSL150 :   Common/MatDefs/SSAO/normal.vert
            FragmentShader GLSL100 GLSL150 : Common/MatDefs/SSAO/normal.frag
            // 略..
        }
        // 预处理阴影
        Technique PreShadow {
            VertexShader GLSL100 GLSL150 :   Common/MatDefs/Shadow/PreShadow.vert
            FragmentShader GLSL100 GLSL150 : Common/MatDefs/Shadow/PreShadow.frag
            // 略..
        }
        // 绘制阴影
        Technique PostShadow {
            VertexShader GLSL100 GLSL150:   Common/MatDefs/Shadow/PostShadow.vert
            FragmentShader GLSL100 GLSL150: Common/MatDefs/Shadow/PostShadow.frag
            // 略..
        }
        // 绘制发光特效
        Technique Glow {
            VertexShader GLSL100 GLSL150:   Common/MatDefs/Misc/Unshaded.vert
            FragmentShader GLSL100 GLSL150: Common/MatDefs/Light/Glow.frag
            // 略..
        }
    }

## 材质实例：j3m文件

j3m名为材质实例文件(Material Instance file)，它最根本的作用是节省代码。例如在程序中通过Java代码来给Lighting.j3md材质设置参数需要很多代码，这些参数可以保存在一个j3m文件中。

### j3m文件语法

jME3 SDK中内置了j3m材质编辑器，可以通过可视化方式来编辑/保存j3m文件。不过就算是手写也难不倒哪里去。

j3m文件只有`Material`、`MaterialParameters`、`AdditionalRenderState`这少数几个关键字，其他都是以键值对方式来定义的属性。

#### 引用材质模板

每个j3m文件都是以`Material`关键字开头，接着是你对这个j3m文件的描述信息(用于debug和异常提示)，再接着是所引用j3md文件的绝对路径。

例:

    Material brick wall : Common/MatDefs/Light/Lighting.j3md {
        // TODO
    }

#### 设置材质参数MaterialParameters

j3m文件的主要内容就是定义材质的参数，以`MaterialParameters`开头，然后根据j3md文件所需的参数来设值，左边是参数名，右边是值。

参数的赋值格式与j3md文件中定义的数据类型有关系。

    数据类型        | 说明          | 取值范例
    ---------------|---------------|--------------------
    Int            | 整数          | 123
    Float          | 浮点数        | 1.0 （必须带小数点）
    Boolean        | 布尔型        | true或false
    Vector2        | 2个Float      | 0.1 5.6
    Vector3        | 3个float      | 0.1 5.6 2.99
    Vector4        | 4个float      | 0.1 5.6 2.99 3
    Color          | 4个float      | 0.7 0.8 0.9 1.0
    Texture2D      | 纹理图片的路径 | Textures/MyTex.jpg
    TextureCubeMap | 纹理图片的路径 | Textures/MyTex.jpg

例：

    MaterialParameters {
        DiffuseMap : Textures/Terrain/BrickWall/BrickWall.jpg
        NormalMap  : Textures/Terrain/BrickWall/BrickWall_normal.jpg
        ParallaxMap: Textures/Terrain/BrickWall/BrickWall_normal.jpg
        UseMaterialColors : true
        Ambient  : 0.0 0.0 0.0 1.0
        Diffuse  : 1.0 1.0 1.0 1.0
        Specular : 0.0 0.0 0.0 1.0
        Shininess: 1.0
    }

**Flip和Repeat**

对于Texture，可以额外指定2个参数：`Flip`和`Repeat`。`Flip`指将图片按Y方向翻转，`Repeat`则是对于超出[0, 1]范围的uv坐标采用重复渲染。如果同时需要`Flip`和`Repeat`，`Flip`要写在`Repeat`前面。

下面3种写法都是正确的。

    ColorMap : Flip Textures/Terrain/BrickWall/BrickWall.jpg
    ColorMap : Repeat Textures/Terrain/BrickWall/BrickWall.jpg
    ColorMap : Flip Repeat Textures/Terrain/BrickWall/BrickWall.jpg

#### 附加渲染状态AdditionalRenderState

在Material对象中，我们通过getAditionalRenderState()来为材质增加一些额外的渲染状态。在j3m文件中，可以增加`AdditionalRenderState`块来定义附加渲染状态。

    参数名             数据类型       用途
    -------------------------------------------------------------
    Wireframe         Boolean       开启线框渲染
    FaceCull          FaceCullMode  设置面剔除模式
    DepthWrite        Boolean       允许写深度缓冲区(Depth Buffer)
    DepthTest         Boolean       开启深度测试
    Blend             BlendMode     设置混色模式
    AlphaTestFalloff  Float         开启Alpha测试，并设置衰减值。
    PolyOffset        Float, Float  设定多边形补偿的系数和单位
    ColorWrite        Boolean       允许写颜色缓冲区(Color Buffer)
    PointSprite       Boolean       开启粒子绘制(点精灵)，适用于点网格。
    LineWidth         Float         绘制线框网格时，线的宽度（单位：像素）。

例：

        AdditionalRenderState {
            Blend Alpha
            FaceCull Off
        }

注1：FaceCull的值为`com.jme3.material.RenderState.FaceCullMode`的枚举值。Off表示关闭剔除背面。

注2：Blend的值为`com.jme3.material.RenderState.BlendMode`的枚举值。Alpha表示使用透明度通道进行混色。

注3：在`AdditionalRenderState`块中，Boolean类型的值使用 On/Off来表示，而不是true/false。

### 完整例子

下面我们通过一个具体的例子，看看j3m是如何工作的。

首先是一段Java代码，展示了在jME3中如何加载j3md材质，并给材质的实例赋值。

    // 加载一个受光材质
    Material mat = new Material(assetManager, "Common/MatDefs/Light/Lighting.j3md");
    // 漫反射贴图
    Texture tex = assetManager.loadTexture("Textures/Terrain/BrickWall/BrickWall.jpg");
    mat.setTexture("DiffuseMap", tex);
    // 法线贴图
    tex = assetManager.loadTexture("Textures/Terrain/BrickWall/BrickWall_normal.jpg");
    mat.setTexture("NormalMap", tex);
    // 视差贴图
    tex = assetManager.loadTexture("Textures/Terrain/BrickWall/BrickWall_height.jpg");
    mat.setTexture("ParallaxMap", tex);

    mat.setBoolean("UseMaterialColors", true);
    mat.setColor("Diffuse", new ColorRGBA(1, 1, 1, 1));
    mat.setColor("Ambient", new ColorRGBA(0, 0, 0, 1));
    mat.setColor("Specular", new ColorRGBA(0, 0, 0, 1));
    mat.setFloat("Shininess", 2.0f);

    mat.getAdditionalRenderState().setBlendMode(BlendMode.Alpha);
    mat.getAdditionalRenderState().setFaceCullMode(FaceCullMode.Off);

创建一个`Materials/BrickWall.j3m`文件，用于描述上面的材质。

    Material brick wall : Common/MatDefs/Light/Lighting.j3md {
        MaterialParameters {
            DiffuseMap : Textures/Terrain/BrickWall/BrickWall.jpg
            NormalMap  : Textures/Terrain/BrickWall/BrickWall_normal.jpg
            ParallaxMap: Textures/Terrain/BrickWall/BrickWall_normal.jpg
            UseMaterialColors : true
            Ambient  : 0.0 0.0 0.0 1.0
            Diffuse  : 1.0 1.0 1.0 1.0
            Specular : 0.0 0.0 0.0 1.0
            Shininess: 1.0
        }
        AdditionalRenderState {
            Blend Alpha
            FaceCull Off
        }
    }

将j3m文件放在资源目录中，然后就可以在程序中使用它。

    package net.jmecn;
    
    import com.jme3.app.SimpleApplication;
    import com.jme3.light.DirectionalLight;
    import com.jme3.material.Material;
    import com.jme3.math.Vector3f;
    import com.jme3.scene.Geometry;
    import com.jme3.scene.shape.Quad;
    
    /**
     * 用j3m文件来保存/加载材质
     * @author yanmaoyuan
     *
     */
    public class HelloJ3M extends SimpleApplication {
    
        @Override
        public void simpleInitApp() {
            // 加载j3m材质，应用于一个方块表面。
            Material mat = assetManager.loadMaterial("Materials/BrickWall.j3m");
            
            Geometry geom = new Geometry("BrickWall", new Quad(8, 8));
            geom.setMaterial(mat);
            
            geom.center();
            rootNode.attachChild(geom);
           
            // 添加一个定向光
            DirectionalLight sun = new DirectionalLight(new Vector3f(0, 0, -1));
            rootNode.addLight(sun);
        }
        
        public static void main(String[] args) {
            HelloJ3M app = new HelloJ3M();
            app.start();
        }
    
    }

结果如下：

![加载j3m材质](/content/images/2017/04/Materials_j3m.png)

超级方便，不是吗？

## 材质模板：j3md文件

j3md名为材质定义文件(Material Definition file)，它定义了材质绘制物体的逻辑。通常Shader程序会定义一系列参数，使用者根据需要来调整配置，以期得到不同的效果。j3md文件将这些参数的定义抽取出来，让开发者能够通过Java代码来控制Shader。

j3md文件内部对Shader的支持非常精妙。首先，Shader可以引用着色程序库(shader libraries)，类似于Java的`import`或者C++的`include`，Shader libraries也可以这样引用其他的shader libraries。利用这点就能在Shader中使用很多的函数，来完成更加高级的效果。例如，如果某个Shader想要使用硬件加速功能，只需要引入对应的库文件就可以了，不需要额外写任何代码。

其次，jME3允许运行时修改材质。在程序运行时，通过修改Material对象的参数就可以改变Shader的参数。**需要注意的是，每次修改材质参数后，jME3都会把材质提交给GPU，重新进行解析和编译。因此，并不推荐在程序中频繁改动Material的参数。**

### j3md文件语法

#### 材质定义MaterialDef

j3md是纯文本文件，可以用任意文本编辑器书写。每个j3md文件都以`MaterialDef`开头，这也是整个文件的根节点。`MaterialDef`后面跟随的是这个材质的名字，整个名字不需要用引号，多个单词间用空格隔开。

`MaterialDef`的命名没有什么特别规则，这个名字仅用于调试和异常提示。

例：

    MaterialDef Test Material 123 {
        // TODO
    }

在`MaterialDef`块内部，可以包含**唯一**一个`MaterialParameters`块，以及一个或多个`Technique`块。你可以给`Technique`命名，也可以不写，不写的话名字默认为“Default”。

例：

    MaterialDef Test Material 123 {
        MaterialParameters { }
        Technique { }
        Technique NamedTech { }
    }

#### 材质参数MaterialParameters

`MaterialParameters`块用于定义材质所需的参数，定义方式和Java很像。左边是变量的类型，右边是变量名。

例：

    MaterialParameters {
        Texture2D TexParam
        Color     ColorParam
        Vector3   VectorParam
        Boolean   BoolParam
        // ...
    }

在定义参数时，还可以设置参数的默认值。若用户实例化材质对象时没有给这个参数赋值，Shader就会使用j3md中定义的默认值。

    MaterialParameters {
        Float MyParam : 1.0
        Color MyColor : 1.0 1.0 1.0 1.0
        // ...
    }

j3md文件支持的常用变量类型如下：

* `Int`
* `Boolean`
* `Float`
* `Vector2`
* `Vector3`
* `Vector4`
* `Color`
* `Texture2D`
* `TextureCubeMap`

*想了解更多数据类型，可以查看[`com.jme3.shader.VarType`](https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-core/src/main/java/com/jme3/shader/VarType.java)。*

在j3md文件中定义材质参数后，就可以在Java代码中通过Material对象的setFloat、setColor等方法来给材质传递参数。当然也可以使用j3m文件来设置材质的参数，定义一个材质实例。

#### 技术方案Technique

`Technique`块比`MaterialParameters`块更高级、更复杂，它的内部可以嵌套`WorldParameters`块、`RenderState`块、`Defines`块以及很多其他语句。

`Technique`块中有两条最重要的语句，分别定义了顶点着色器(VertexShader)和片元着色器(FragmentShader)，说明了这个`Technique`绘制模型的方式。

通常来说这两个语句中还会指定着色器适用的OpenGL版本，例如`GLSL100`用于`OpenGL Shading Language version 1.00`。其后是着色器代码在资源文件夹中的绝对路径，指向`.glsl`、`.frag`、`.vert`等文件。

例：

        Technique {
            VertexShader GLSL100:   Common/MatDefs/Misc/Unshaded.vert
            FragmentShader GLSL100: Common/MatDefs/Misc/Unshaded.frag
            // 略..
        }

当这个材质被应用于Geoemtry之后，根据`MaterialParameters`中定义的参数，Shader就会接收到所需的uniform变量。Shader中对应的uniform变量名，即`MaterialParameters`中的变量名加上前缀`m_`。

举个例子，假设在`MaterialParameters`块中定义了`Shininess`变量。

    MaterialParameters {
        Float Shininess
    }

它的值将会传递给Shader中的同名uniform变量，前缀`m_`。

    uniform float m_Shininess;

前缀小写字母`m`的意思是材质(material)。

##### 全局变量WorldParameters

Shader中有些常用的变量，比如总时间(Time)、世界-视口投影矩阵(WorldViewMatrix)等。jME3引擎已经自动为我们计算好了这些变量的值，只需要在`WorldParameters`块中声明使用就可以了。

`WorldParameters`是另一个与Shader有关的重要数据结构，它的作用和`MaterialParameters`类似，都是将多个变量暴露给Shader，但是工作的机制不尽相同。`MaterialParameters`的值是用户设置的，而`WorldParameters`的值则是由引擎本身指定的。此外，`WorldParameters`块嵌套在`Technique`块内部，因为它的值跟具体使用的Shader有关。

例如，全局变量“Time”，它代表了从jME3引擎启动开始到现在的总时间(单位：秒)。在`WorldParameters`块中，我们这样来描述它：

    WorldParameters {
        Time
        // ...
    }

Shader代码中通过一个同名的uniform变量来获得它的值，变量名要加上前缀`g_`：

    uniform float g_Time;

小写字母g代表了全局(global)变量。

想了解jME3内置了哪些全局变量？这个类中有一份清单：[`com.jme3.shader.UniformBinding`](https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-core/src/main/java/com/jme3/shader/UniformBinding.java)。

##### 光照模式LightMode

jME3定义了一些uniform变量用来计算光照：

* `g_LightDirection`(vec4) ：光照方向
 * 对于聚光灯`SpotLight`：`x,y,z` 表示聚光灯在世界坐标系中的方向向量，`w` 表示聚光灯角度的余弦值 `cos(angle)`。
* `g_LightColor` (vec4)：表示光的颜色
* `g_LightPosition` (vec4)：表示光源位置
 * 对于聚光灯`SpotLight`：`x,y,z` 表示光源在世界坐标系中的位置，`w` 表示灯光辐射距离的倒数 `1/lightRange`。
 * 对于点光源`PointLight`：`x,y,z` 表示光源在世界坐标系中的位置，`w` 表示光球半径的倒数 `1/lightRadius`。
 * 对于定向光`DirectionalLight`：这里有点奇怪，`x,y,z` 表示定向光的**方向**，`w` 的值为-1，用来让Shader知道这是一个定向光。
* `g_AmbientLightColor` (vec4)：表示环境光的颜色.

使用上述这些uniform变量并不需要在`WorldParameters`中进行声明，只需要在`Technique`块中加上一句`LightMode MultiPass`。

例：

        Technique {
            LightMode MultiPass
            VertexShader GLSL100 GLSL150:   Common/MatDefs/Light/Lighting.vert
            FragmentShader GLSL100 GLSL150: Common/MatDefs/Light/Lighting.frag
            // ..
        }

##### 渲染状态RenderState

`RenderState`块位于`Technique`块内部，它用于描述材质中那些无法被Shader控制的渲染功能，比如深度测试(depth testing)、Alpha混色(alpha blending)等，最常用的渲染状态是Alpha混色。想要设置一种渲染状态，需要在`RenderState`块中添加一条一句，例如：

    RenderState {
        Blend Alpha
    }

RenderState的具体功能与OpenGL有关，jME3所支持的RenderState可以查看[`com.jme3.material.RenderState`](https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-core/src/main/java/com/jme3/material/RenderState.java)类。下面是我简单整理的常用功能

    RenderStates         | 取值
    ---------------------|----------------------------
    Wireframe            | On/Off
    FaceCull             | Enum: FaceCullMode
    DepthWrite           | On/Off
    DepthTest            | On/Off
    DepthFunc            | Enum: TestFunction
    ColorWrite           | On/Off
    BlendEquation        | Enum: BlendEquation
    BlendEquationAlpha   | Enum: BlendEquationAlpha
    Blend                | Enum: BlendMode
    PolyOffset           | Float, Float
    LineWidth            | Float

除了`RenderState`外，j3md中还可以定义`ForceRenderState`块，它们的语法是一样的，只不过后者将会强行改变渲染状态。

##### 宏定义Defines

`Defines`块的作用是根据材质实例中的参数来声明Shader所需的宏。若宏对应的参数存在，则宏存在；若用户没有传递该参数，则宏不存在。

假设在`MaterialParameters`中声明了一个`Texture2D MyTex`变量，片元着色器(FragmentShader)打算使用它来进行纹理映射，而用户在实例化这个材质的时候却并没有传入这个参数，这将导致shader出现错误。

为了防止这种情况，就可以使用Defines来定义一个宏`HAS_TEXTURE`，然后在FragmentShader中进行判断。

    MaterialDef Test Material 123 {
        MaterialParameters {
            Float Shininess
            Texture2D MyTex
        }
        Technique {
            VertexShader GLSL100 : Common/MatDefs/Misc/MyShader.vert
            FragmentShader GLSL100 : Common/MatDefs/Misc/MyShader.frag
        }
        Defines {
            HAS_TEXTURE : MyTex
        }
    }

如果用户没有给MyTex赋值，那么HAS_TEXTURE这个宏就不存在。在MyShader.frag可以这么判断：

    #ifdef HAS_TEXTURE
        uniform sample2D m_MyTex;
    #endif

`Defines`块在jME3中应用十分广泛，例如在Lighgint.j3md材质中，法线贴图(NormalMap)、发光贴(GlowMap)图等参数都有对应的宏定义，所以即使你不设置这些参数，着色器也能正常工作。

### 完整例子

根据本节介绍的知识，你应该能看懂下面这个j3md文件做了些什么了。

    MaterialDef Scroll {
    
        MaterialParameters {
            Texture2D ColorMap
            Texture2D LightMap
            Color Color (Color)
            Boolean VertexColor (UseVertexColor)
            Boolean SeparateTexCoord
       
            // Alpha threshold for fragment discarding
            Float AlphaDiscardThreshold (AlphaTestFallOff)
    
            // ScrollSpeed
            Float Speed : 1.0
        }
    
        Technique {

            LightMode MultiPass

            VertexShader GLSL100:   Shader/Misc/Scroll.vert
            FragmentShader GLSL100: Shader/Misc/Scroll.frag
    
            WorldParameters {
                WorldViewProjectionMatrix
                ViewProjectionMatrix
                ViewMatrix
                Time
            }
    
            Defines {
                SEPARATE_TEXCOORD : SeparateTexCoord
                HAS_COLORMAP : ColorMap
                HAS_LIGHTMAP : LightMap
                HAS_VERTEXCOLOR : VertexColor
                HAS_COLOR : Color
                DISCARD_ALPHA : AlphaDiscardThreshold
            }
        }
    }

## 附录

### 附录：jME3与Unity3D的对比

本文中的Shader来源：[猫都能学会的Unity3D Shader入门指南（一）](https://onevcat.com/2013/07/shader-tutorial-1/)

首先来看一段Unity3D中的Shader：

    Shader "Custom/Diffuse Texture" {
        Properties {
            _MainTex ("Base (RGB)", 2D) = "white" {}
        }
        SubShader {
            Tags { "RenderType"="Opaque" }
            LOD 200
		
            CGPROGRAM
            #pragma surface surf Lambert

            sampler2D _MainTex;

            struct Input {
                float2 uv_MainTex;
            };

            void surf (Input IN, inout SurfaceOutput o) {
                half4 c = tex2D (_MainTex, IN.uv_MainTex);
                o.Albedo = c.rgb;
                o.Alpha = c.a;
            }
            ENDCG
        } 
        FallBack "Diffuse"
    }

#### 根节点

`Shader`块是整个着色器的根节点，定义了这个着色器的名字。

jME3中的`MaterialDef`块与其相似。

#### 参数

`Properties`块定义了这个着色器的属性，允许用户设置着色器实例的参数，可以定义默认值。

    Properties {  
        _LightColor ( "Light color", Color) = (1.0, 1.0, 1.0, 1.0)
        _MainTex ("Texture", 2D)
    }  

jME3中的`MaterialParameters`块用于定义参数，允许用户设置材质实例的参数，可以定义默认值。

    MaterialParameters {
        Color LightColor : 1.0 1.0 1.0 1.0
        Texture2D MainTex
    }

不同之处：Unity3D的着色器属性有2个名字，写在前面的是着色器程序内部使用的变量名，写在括号里的是在材质编辑器中显示给开发者看的名字。

#### 多Technique

Unity3D的`Shader`中可以包含多个`SubShader`，每个`SubShader`就相当于一个`Technique`。

jME3的`MaterialDef`中可以包含多个`Technique`。

#### 多Pass

Unity3D的每个`SubShader`只执行一次，但是每个`SubShader`中可以包含多个`Pass`，以此实现多Pass。

jME3的j3md文件中没有单独的`Pass`，但是每个`Technique`都可以依序执行，以此实现多Pass。

#### 动态改变Technique

Unity3D通过给游戏对象设置不同的标签`Tags`，以及变更网格的层次细节(`LOD 100`)等条件，得以动态改变使用的`SubShader`。

jME3根据GLSL的版本来动态改变`Technique`。

不同：Unity3D考虑更多的是游戏逻辑和性能，jME3考虑的是硬件兼容性和跨平台。

#### Shader语言

Unity3D使用CG/HLSL语言编写Shader代码，源码直接写在`SubShader`中。

jME3使用GLSL编写Shader代码，源码保存为单独的文件，并在j3md中引用。

不同：jME3的j3md比Unity3D的Shader更灵活，可复用性比较高。

### 附录：JME3与OpenGL的兼容性

GLSL 1.0 到 1.2 有一些内建的attribute和uniform变量，例如`gl_Vertex`、`gl_ModelViewMatrix`等。
这些内建变量在GLSL 1.3（OpenGL 3.0）以后被废弃了，下面是被废弃的GLSL变量以及JME3中的等价变量。

    GLSL 1.2 attributes          |  JME3 等价变量
    -----------------------------|--------------------------------
    gl_Vertex                    |  inPosition
    gl_Normal                    |  inNormal
    gl_Color                     |  inColor
    gl_MultiTexCoord0            |  inTexCoord
    gl_ModelViewMatrix           |  g_WorldViewMatrix
    gl_ProjectionMatrix          |  g_ProjectionMatrix
    gl_ModelViewProjectionMatrix |  g_WorldViewProjectionMatrix
    gl_NormalMatrix              |  g_NormalMatrix

### 附录：扩展阅读

https://www.shadertoy.com/
http://vga.zol.com.cn/223/2234814.html
http://imgtec.eetrend.com/blog/1948
http://www.cppblog.com/Leaf/archive/2013/02/22/198015.aspx
http://www.cnblogs.com/freeblues/p/5707030.html
http://www.cnblogs.com/geniusalex/p/5321527.html
http://blog.csdn.net/aganlengzi/article/details/50479605
http://blog.csdn.net/kongbu0622/article/details/5315309
