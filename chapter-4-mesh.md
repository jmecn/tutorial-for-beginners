# 第四章：网格

声明：本章理论内容大量参考《DirectX 9.0 3D游戏开发编程基础》，素材都是我手绘的。

## 定义模型的形状

一个场景是由多个物体或模型组成的。一个物体可以用三角形网格（triangle mesh）来近似表示。使用网格来建立一个物体的过程，称为3D建模。3D世界中最基本的图元就是**三角形**，但是我们也会用到点、线、多边形等图元。

下图为在Blender中制作一个苹果模型的界面。

![modeling](/content/images/2017/03/modeling.png)

一个多边形的两边相交的点叫做**顶点**。为了描述一个三角形，我们通常指定三个点的位置(Position)来对应三角形的三个顶点，这样我们就能够很明确的表示出这个三角形了。

![triangle](/content/images/2017/03/triangle.png)

### 顶点格式

下图为XOY平面上的六个顶点，定义了4个三角形。本章节接下来的内容就以这个六边形为例。

![hex](/content/images/2017/03/hex-1.png)

在jME3中，我们使用3个float类型的变量来表示一个顶点数据。

            float[] vertex = {
                    2.5f, 4f, 0f,// 零
                    1f, 3.26f, 0f,// 壹
                    1f, 1.74f, 0f,// 贰
                    2.5f, 1f, 0f,// 叁
                    4f, 1.74f, 0f,// 肆
                    4f, 3.26f, 0f// 伍
            };

也可以直接用`com.jme3.math.Vector3f`来表示顶点位置。

            // 六个顶点
            Vector3f[] v = new Vector3f[6];
            v[0] = new Vector3f(2.5f, 4f, 0f);
            v[1] = new Vector3f(1f, 3.26f, 0f);
            v[2] = new Vector3f(1f, 1.74f, 0f);
            v[3] = new Vector3f(2.5f, 1f, 0f);
            v[4] = new Vector3f(4f, 1.74f, 0f);
            v[5] = new Vector3f(4f, 3.26f, 0f);

### 顶点索引

三角形是构建3D 物体的基本图形。为了构造物体，我们创建了三角形列表（triangle list）来描述物体的形状和轮廓。三角形列包含了我们将要画的每一个三角形的数据信息。

例如为了构造一个上面的六边形，我们把它分成四个三角形，最后指定每个三角形的顶点。

    Vector3f[] hex = new Vector3f[] {
        v[0], v[1], v[2], // 三角形0
        v[2], v[3], v[4], // 三角形1
        v[4], v[5], v[0], // 三角形2
        v[0], v[2], v[4]  // 三角形3
    }; 

**注意：指定三角形顶点的顺序是很重要的，应该按照一定顺序环绕排列。在OpenGL中，逆时针排列成的三角形是正面，顺时针排列的三角形是背面。**

3D 物体中的三角形经常会有许多共用顶点。例如在这六边形中，零、贰、肆这三个点都同时被3个不同的三角形复用。虽然现在仅有三个点被重复使用，但是当要表现一个更精细更复杂的模型的时候，重复的顶点数将会变得很大。

为了解决这个问题，我们引入索引（indices）这个概念。它的工作方式是：我们创建一个顶点列表和一个索引列表（index list）。顶点列表包含所有不重复的顶点，索引列中则用顶点列中定义的值来表示每一个三角形的构造方式。回到那个六边形的示例上来，它的顶点列表的构造方式如下：

        float[] vertex = {
            2.5f, 4f, 0f,// 零
            1f, 3.26f, 0f,// 壹
            1f, 1.74f, 0f,// 贰
            2.5f, 1f, 0f,// 叁
            4f, 1.74f, 0f,// 肆
            4f, 3.26f, 0f// 伍
        };

索引列表则定义顶点列中的顶点是如何构造这四个三角形的：

        int[] indices = new int[] {
            0, 1, 2, // 三角形0
            2, 3, 4, // 三角形1
            4, 5, 0, // 三角形2
            0, 2, 4  // 三角形3
        };

## 实例：自定义网格

### 顶点数据类型

在3D游戏引擎中，顶点所包含的的信息可不止是位置(Position)，通常还有顶点法线(Normal)、纹理坐标(TexCoords)等信息。

jME3使用`com.jme3.scene.Mesh`对象来存储物体的网格数据，不同类型的顶点数据通过`com.jme3.scene.VertexBuffer.Type`来指定。下面是jME3中比较常用的类型。

    Type       |  Components   |  Usage
    -----------|---------------|-------------------------
    Position   |  3 floats     |  顶点位置
    Normal     |  3 floats     |  顶点法线(用于计算光照)
    Tangent    |  4 floats     |  顶点切线(用于计算光照)
    Binormal   |  3 floats     |  副法线  (用于计算光照)
    TexCoord   |  2 floats     |  纹理坐标(用于贴图)
    Color      |  4 floats     |  顶点颜色
    Size       |  1 float      |  顶点大小
    Index      |  1 uint       |  顶点索引
    BoneIndex  |  4 ubytes     |  骨骼索引(用于骨骼蒙皮动画)
    BoneWeight |  4 floats     |  顶点权重(用于骨骼蒙皮动画)

### 编写测试类

下面我们创建一个HelloMesh类，利用上面的知识在jME3中显示本章前面出现的六边形。

    package net.jmecn;
    
    import com.jme3.app.SimpleApplication;
    import com.jme3.material.Material;
    import com.jme3.scene.Geometry;
    import com.jme3.scene.Mesh;
    import com.jme3.scene.VertexBuffer.Type;
    import com.jme3.util.BufferUtils;
    
    /**
     * 网格
     * 
     * @author yanmaoyuan
     *
     */
    public class HelloMesh extends SimpleApplication {
        
        @Override
        public void simpleInitApp() {
            // 六个顶点
            float[] vertex = {
                    2.5f, 4f, 0f, // 零
                    1f, 3.26f, 0f, // 壹
                    1f, 1.74f, 0f, // 贰
                    2.5f, 1f, 0f, // 叁
                    4f, 1.74f, 0f, // 肆
                    4f, 3.26f, 0f// 伍
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
            mesh.setBuffer(Type.Index, 1, BufferUtils.createIntBuffer(indices));
            
            mesh.updateBound();// 更新边界
            mesh.setStatic();// 设为静态模型
            
            // 创建材质，使我们可以看见这个六边形
            Material mat = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");
            //mat.getAdditionalRenderState().setWireframe(true);
            
            // 使用网格和材质创建一个物体
            Geometry geom = new Geometry("六边形");
            geom.setMesh(mesh);
            geom.setMaterial(mat);
    
            // 将物体添加到场景图中
            rootNode.attachChild(geom);
    
        }
        
        public static void main(String[] args) {
            // 启动程序
            HelloMesh app = new HelloMesh();
            app.start();
        }
    }

### 不太满意的结果

![hex_white](/content/images/2017/03/hex_white.png)

结果跟我设想的差不多，我很满意，但是有些强迫症的同学就不太满意了。

**为什么这个六边形在屏幕的右上角，不能居中吗？**

### 修正模型位置

在本教程的第二章中我们介绍过jME3的一些基本概念，其中包括`摄像机`，现在我们需要更加了解它。在本程序刚启动时，按`C`查看摄像机的状态。

    Camera Position: (0.0, 0.0, 10.0)
    Camera Rotation: (0.0, 1.0, 0.0, 0.0)
    Camera Direction: (0.0, 0.0, -1.0)
    cam.setLocation(new Vector3f(0.0f, 0.0f, 10.0f));
    cam.setRotation(new Quaternion(0.0f, 1.0f, 0.0f, 0.0f));

解读上面的信息可知：

* 摄像机的初始坐标是(0.0, 0.0, 10.0)，位于Z轴的正轴上。
* 它朝向Z轴的负方向(0.0, 0.0, -1.0)，正盯着整个场景的原点。

因此现在画面正中心应该就是原点的位置。我用作图工具在运行截图上添加了2个辅助线，能够看得更清楚一些。

![hex_white_axis](/content/images/2017/03/hex_white_axis.png)

知道原因就好办了，让六边形出现在画面的正中间有很多种方法。

* 修改网格中每个顶点的坐标，将它们向(-2.5, -2.5, 0)方向移动。
* 改变摄像机的位置，将其向(2.5, 2.5, 0)方向移动。

第一种方法比较麻烦，我们要重新计算每个顶点的坐标。

第二种方法看似比较可行，但我是拒绝的：作为一名真正的强迫症，必须不能接受这种小花招。虽然看似居中了，但是实际上这个六边形的中心并不在原点，我骗不了我自己！

所以我倾向于第三种方法，移动整个Geometry。

    geom.move(-2.5f, -2.5f, 0);

或者用另一个看起来更加高大上的方法让它自己居中。

    geom.center();

![hex_white_center](/content/images/2017/03/hex_white_center.png)

OK，现在你们爽了吧？

## 程序生成网格

### 基本形状

很多3D引擎都会用代码来生成一些常用的基本形状，jME3也不例外。`Mesh`有很多子类，分别位于`com.jme3.scene.shape`和`com.jme3.scene.debug`这2个包中。

* com.jme3.scene.shape
 * Line 线段
 * Curve 曲线
 * Quad 平面
 * Surface 曲面
 * Box 方块
 * Cylinder 圆柱体
 * Sphere 球体
 * Dome 半球
 * Toru 圆环
* com.jme3.scene.debug
 * Arrow 箭头
 * Grid 网格
 * WireBox 线框盒
 * WireFrustum 线框锥
 * WireSphere 线框球

关于这些形状，jme3-example中有一些有趣的例子，位于`jme3test.model.shape`包中，大家感兴趣的话可以运行一下看看效果。

### 实例：球体

下面是一个简单的例子，创建一个有10根纬线、16根经线、半径为2的球体。

    package net.jmecn;
    
    import com.jme3.app.SimpleApplication;
    import com.jme3.material.Material;
    import com.jme3.scene.Geometry;
    import com.jme3.scene.shape.Sphere;
    
    public class HelloShape extends SimpleApplication {
    
        @Override
        public void simpleInitApp() {
            flyCam.setMoveSpeed(10);
            
            // 创建球体
            Geometry geom = new Geometry("球体", new Sphere(10, 16, 2));
            
            // 创建材质，并显示网格线
            Material mat = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");
            mat.getAdditionalRenderState().setWireframe(true);
            geom.setMaterial(mat);
            
            // 将物体添加到场景图中
            rootNode.attachChild(geom);
    
        }
    
        public static void main(String[] args) {
            HelloShape app = new HelloShape();
            app.start();
        }
    
    }

效果如下：

![sphere](/content/images/2017/03/sphere.png)

### 实例：坐标轴

在前面的六边形例子中加入3个箭头，做成参考坐标系。

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
    import com.jme3.util.BufferUtils;
    
    /**
     * 网格
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
        }
    
        /**
         * 创建一个六边形
         */
        private void createHex() {
            // 六个顶点
            float[] vertex = {
                    2.5f, 4f, 0f, // 零
                    1f, 3.26f, 0f, // 壹
                    1f, 1.74f, 0f, // 贰
                    2.5f, 1f, 0f, // 叁
                    4f, 1.74f, 0f, // 肆
                    4f, 3.26f, 0f// 伍
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
            mesh.setBuffer(Type.Index, 1, BufferUtils.createIntBuffer(indices));
    
            mesh.updateBound();
            mesh.setStatic();
    
            // 创建材质，使我们可以看见这个六边形
            Material mat = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");
            // mat.getAdditionalRenderState().setWireframe(true);
    
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

效果如下图：

![hex_white_arrows](/content/images/2017/03/hex_white_arrows.png)

## 扩展阅读：渲染管线

渲染管线的工作如今已经被游戏引擎接管，我们基本上不需要直接接触它，如果你对偏理论的部分不太感兴趣，可以跳过这一节。

* [3D图形技术概念和渲染管线的处理](http://chaimzane.iteye.com/blog/1880367)
* [如何理解OpenGL中着色器，渲染管线，光栅化等概念？](https://www.zhihu.com/question/29163054)
* [可编程渲染管线比固定管线的优势在哪？有什么应用？](https://www.zhihu.com/question/28024422)
