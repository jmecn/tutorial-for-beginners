# 第七章：光与影

## 感受光影

计算机中没有光，只有数据结构和算法，光照和阴影在3D引擎中是两种不同的事物。

光照能够让物体面向光源的一面看起来更加明亮，而背光面则更加灰暗。`Lighting.j3md`材质的作用就是如此。

但是光照并不能让物体在地板、墙壁上投射阴影。在3D引擎中，绘制影子需要额外的计算，这将影响程序的性能（降低FPS）。

观察下面2个场景的截图，第一场景中只有光照，第二个场景中加入了阴影。加入阴影之后FPS降低了一半不止。

![光照](/content/images/2017/04/Light.png)

![阴影](/content/images/2017/04/Shadow.png)

上图源代码：[HelloLight](https://github.com/jmecn/jME3Tutorials/blob/master/src/main/java/net/jmecn/HelloLight.java)

影子的效果越清晰、越柔和，对性能的消耗越大。引擎的性能和显卡的性能直接挂钩，开发游戏时需要考虑用户的不同的硬件性能，适当调整画面效果。

![GTX1080 vs GTX660](/content/images/2017/04/GraphicsCard.jpg)

## 光源

所有的光源都是`com.jme3.light.Light`的子类，通过`rootNode.addLight(myLight);`可以把一个光源添加到场景图中。

目前jME3提供了4种光源：

* 环境光 `com.jme3.light.AmbientLight`。
* 定向光 `com.jme3.light.DirectionalLight`。
* 点光源 `com.jme3.light.PointLight`。
* 聚光灯 `com.jme3.light.SpotLight`。例：车前灯，手电筒。

通过`light.setColor(ColorRGBA color)`方法可以改变光源的颜色和亮度。例如`setColor(new ColorRGBA(1f, 1f, 1f, 1f))`或`setColor(ColorRGBA.White)`可以把光源设置成白色。

`Spatial#getWorldLightLight()`用于查询场景中影响某个物体的所有光源；`Spatial#getLocalLightLight()`用于查询和某个物体绑定的光源。例如通过下列代码，可以查询整个场景中所有的光源。

    LightList list = rootNode.getLocalLightList();

### 点光源

![PointLight](/content/images/2017/04/PointLight.png)

点光源(PointLight)在场景中有一个具体的**位置**，向四面八方辐射光线。点光源的亮度根据光照半径(Lighting Radius)而定，离光源位置越远，亮度越低。

典型例子：灯泡、火把、蜡烛。


    PointLight lamp_light = new PointLight();
    lamp_light.setColor(ColorRGBA.Yellow);
    lamp_light.setRadius(4f);
    lamp_light.setPosition(new Vector3f(1, 1, 1));
    rootNode.addLight(lamp_light);

点光源可以使用`PointLightShadowRenderer` 来产生阴影。

### 定向光

![DirectionalLight](/content/images/2017/04/DirectionalLight.png)

定向光(DirectionalLight)没有具体的位置，只有方向。当添定向光以后，整个场景中都会弥漫着与其方向平行的光线。

定向光用于模拟无穷远处的光源产生的光线，生活中最典型的例子就是阳光、月光。

    DirectionalLight sun = new DirectionalLight();
    sun.setColor(ColorRGBA.White);
    sun.setDirection(new Vector3f(-.5f,-.5f,-.5f).normalizeLocal());
    rootNode.addLight(sun);

定向光可以使用`DirectionalLightShadowRenderer`来产生阴影。

### 聚光灯

![SpotLight](/content/images/2017/04/SpotLight.png)

聚光灯(SpotLight)可以发射出明亮的光线或光锥，它在场景中有具体的**位置**和**方向**。除此之外，聚光灯还有射程(Range)和2个角度：内角(inner angle)定义了靠近光锥中心的明亮区域，外角(outer angle)定义了光锥的边缘区域。光锥范围外的物体不会受到聚光灯的影响。

![手电筒](/content/images/2017/04/SpotLight3.jpg)

典型例子：手电筒、车前灯。

    SpotLight spot = new SpotLight();
    spot.setSpotRange(100f);                           // 射程
    spot.setSpotInnerAngle(5f * FastMath.DEG_TO_RAD);  // 光锥内角
    spot.setSpotOuterAngle(15f * FastMath.DEG_TO_RAD); // 光锥外角
    spot.setColor(ColorRGBA.White.mult(1.3f));         // 光源颜色
    spot.setPosition(new Vector3f(9.5f, 13.5f, 9f));   // 光源位置
    spot.setDirection(new Vector3f(-0.06764714f, -0.647349f, -0.7591859f));// 光源方向
    rootNode.addLight(spot);

聚光灯可以使用`SpotLightShadowRenderer`来产生阴影。

如果想让聚光灯跟着摄像机，可以在simpleUpdate方法中实时改变聚光灯的位置和方向。

    @Override
    public void simpleUpdate(float tpf) {
        // 使聚光灯始终跟随摄像机
        spotLight.setPosition(cam.getLocation());  // 光源位置：摄像机的位置
        spotLight.setDirection(cam.getDirection());// 光源方向：摄像机的方向
    }

### 环境光

![AmbientLight](/content/images/2017/04/AmbientLight.png)

环境光(AmbientLight)用于模拟间接光，如遍及室外场景的大气光线，它是照亮整个场景的常规光线。这种光具有均匀的强度，并且属于均质漫反射。它不具有可辨别的光源和方向，也不会照射物体产生阴影。它的用途是调整整个场景的亮度和颜色。

在现实生活中，白天一般会有明亮的环境光，即使不开灯人眼也能看清房间里的东西；夜晚的环境光亮度比较低，很难看清物体的形状和颜色。场景中的阴影不会比环境光颜色暗，因为环境光同样也会影响到背光的区域。

一般情况下，我们不会单独在场景中使用环境光，它总是配合其他光源一起使用，否则会让3D场景看起来像一个平面一样。

下面两幅图中，第一个只有定向光，第二个场景有定向光+环境光。

![No ambient light](/content/images/2017/03/3.png)

![Ambient light](/content/images/2017/03/4.png)

例：在场景中添加环境光：

    AmbientLight al = new AmbientLight();
    al.setColor(ColorRGBA.White.mult(0.2f));
    rootNode.addLight(al);

你可以将颜色乘以一个系数，以此来调整光源的亮度，
例如：乘以一个大于1的数值，可以让颜色看起来更加明亮：

    mylight.setColor(ColorRGBA.White.mult(1.3f));

## 阴影

对于除环境光之外的每种光源，jME3都提供了2种产生阴影的方式。

* 阴影渲染器(ShadowRenderer)：应用于ViewPort。
* 阴影滤镜(ShadowFilter)：通过FilterPostProcessor应用于ViewPort。

<table>
 <tr>
  <th>光源</th>
  <th>阴影渲染器(ShadowRenderer)</th>
  <th>阴影滤镜(ShadowFilter)</th>
 </tr>
 <tr>
  <td>DirectionalLight</td>
  <td>DirectionalLightShadowRenderer</td>
  <td>DirectionalLightShadowFilter</td>
 </tr>
 <tr>
  <td>PointLight</td>
  <td>PointLightShadowRenderer</td>
  <td>PointLightShadowFilter</td>
 </tr>
 <tr>
  <td>SpotLight</td>
  <td>SpotLightShadowRenderer</td>
  <td>SpotLightShadowFilter</td>
 </tr>
 <tr>
  <td>AmbientLight</td>
  <td>无</td>
  <td>无</td>
 </tr>
</table>

例如下面为一个点光源产生阴影。

    private void addPointLight() {
        // 点光源
        PointLight pointLight = new PointLight();
        pointLight.setPosition(new Vector3f(8, 5, 8));
        pointLight.setRadius(1000);
        pointLight.setColor(new ColorRGBA(0.8f, 0.8f, 0f, 1f));
        rootNode.addLight(pointLight);
        // 点光源影子
        PointLightShadowRenderer plsr = new PointLightShadowRenderer(assetManager, 1024);
        plsr.setLight(pointLight);// 设置点光源
        plsr.setEdgeFilteringMode(mode);
        viewPort.addProcessor(plsr);
    }

对于每个光源，只需要1种方式就可以产生阴影。如果已经使用了阴影渲染器(ShadowRenderer)，就不需要再使用阴影滤镜(ShadowFilter)了。所有绘制阴影的类都有相似的接口，学会使用其中一个就可以很容易掌握其他的。

阴影计算十分消耗性能，使用的时候要有所节制。阴影渲染器(ShadowRenderer)可以控制场景中每个物体的阴影模式(产生或接受)，例如：

    spatial.setShadowMode(ShadowMode.Inherit);     // 默认状态，继承父节点的阴影模式。
    rootNode.setShadowMode(ShadowMode.Off);        // 禁止整个场景产生阴影，除非某个物体单独设置为Cast。
    wall.setShadowMode(ShadowMode.CastAndReceive); // 既产生阴影，也接受阴影。
    floor.setShadowMode(ShadowMode.Receive);       // 只接受阴影，不产生阴影。
    airplane.setShadowMode(ShadowMode.Cast);       // 不接受阴影，只产生阴影。
    ghost.setShadowMode(ShadowMode.Off);           // 既不产生阴影，也不接受阴影。

阴影滤镜(ShadowFilter)在绘制阴影的时候只考虑Cast模式，不考虑物体的Receive模式。这意味着就算你把地板设置为ShadowMode.Cast，其它物体的影子也会投射在它上面。

下面是使用DirectionalLightShadowFilter来为定向光产生影子的关键代码：

        DirectionalLight sun = new DirectionalLight();
        sun.setColor(ColorRGBA.White);
        sun.setDirection(cam.getDirection());
        rootNode.addLight(sun);

        /* 产生阴影 */
        final int SHADOWMAP_SIZE=1024;

        DirectionalLightShadowFilter dlsf = new DirectionalLightShadowFilter(assetManager, SHADOWMAP_SIZE, 3);
        dlsf.setLight(sun);
        dlsf.setEnabled(true);
        FilterPostProcessor fpp = new FilterPostProcessor(assetManager);
        fpp.addFilter(dlsf);
        viewPort.addProcessor(fpp);

构造方法阐述说明：

 * AssetManager对象
 * 阴影图像的分辨率(单位：像素)，取值为512、1024、2048等。
 * 渲染的次数，次数越多，阴影质量越高FPS越低，取值范围（1~4）。

**注意!!!!!**

jME3中很多的特效滤镜都是通过`FilterPostProcessor`来工作的。`FilterPostProcessor`在工作的时候会禁用全局抗锯齿设置，需要通过代码来告诉它开启几倍抗锯齿。

例：

    fpp.setNumSamples(4);// 4倍抗锯齿


## 光与材质

本节源代码：[HelloMaterial](https://github.com/jmecn/jME3Tutorials/blob/master/src/main/java/net/jmecn/HelloMaterial.java)

### 无光 VS 有光
Unshaded材质不需要光照，而Lighting材质必须得有光源才能看见。例如我们注释掉`simpleInitApp`中的`addLight()`方法，再次运行程序就会发现，使用Lighting材质的物体变成了深渊的颜色。另外，由于场景中不需要计算光照，FPS将会有显著提升。

![没有光源时，左边的小球和方块都变黑了](/content/images/2017/04/Materials_no_light.png)

如果上图中并没有把viewPort改成淡蓝色，你会看到什么？什么都看不到！

![连背景色都没有了](/content/images/2017/04/Materials_no_light_no_sky.png)

### 不同的光

让我们先直观感受一下不同的光在Lighting.j3md材质上的效果。

首先只留下定向光。

    // #3 将模型和光源添加到场景图中
    rootNode.addLight(sun);
    //rootNode.addLight(ambient);

被光照射到的地方比较亮，没有被照射到的部分则是黑色的。这种光能够凸显出3D的效果。

![没有环境光](/content/images/2017/04/Materials_no_ambient.png)

然后换一边，注释掉定向光，只保留环境光。

    // #3 将模型和光源添加到场景图中
    //rootNode.addLight(sun);
    rootNode.addLight(ambient);

在环境光的影响下，物体表现出均匀的颜色，看起来就是它们固有的颜色一样。但是这种光没有光暗对比，人眼区分不出看到的是一个平面还是立体的物体。

![没有定向光](/content/images/2017/04/Materials_no_sun.png)

当这两种光都存在于同一个场景图中时，光照计算将会叠加在一起，结果就是我们最开始看到的效果。

![光照效果叠加在一起](/content/images/2017/04/Materials_two_lights.png)

### 颜色和亮度

在这个例子中，我们使用了一个ColorRGBA对象来设置光照的颜色和亮度。

    // 调整光照亮度
    ColorRGBA lightColor = new ColorRGBA();
    sun.setColor(lightColor.mult(0.8f));
    ambient.setColor(lightColor.mult(0.2f));

由于这两个光源在同一个场景中，其颜色最终会叠加在一起，因此我将2个光源的亮度按比例调整成了0.8倍和0.2倍，这样叠加之后的光照比较自然。

如果两种光照的亮度都比较大，最终会导致场景中的物体过于明亮刺眼。下面我们做个试验，将lightColor后面的亮度系数都改为1，看看效果如何。

    // 调整光照亮度
    ColorRGBA lightColor = new ColorRGBA();
    sun.setColor(lightColor.mult(1f));
    ambient.setColor(lightColor.mult(1f));

![1倍亮度](/content/images/2017/04/Materials_too_bright.png)

继续调整为2倍亮度，现在更加刺眼了。

    // 调整光照亮度
    ColorRGBA lightColor = new ColorRGBA();
    sun.setColor(lightColor.mult(2f));
    ambient.setColor(lightColor.mult(2f));

![2倍亮度](/content/images/2017/04/Materials_too_bright_2.png)

结论1：慎重对待场景中的光源，不要太暗也不要太亮。

结论2：Unshaded.j3md完全不受光照影响。


