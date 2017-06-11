# 第八章：场景图

## 概念

jMonkeyEngine3是一个基于场景图的3D游戏引擎，因此有必要对场景图的概念进行一些说明。jME3的场景图通过Spatial、Geometry、Node这3个类来实现，它们之间的关系如下图：

![SceneGraph](/content/images/2017/04/SceneGraph.svg)

场景图(Scene Graph)是一种数据结构，用于管理游戏场景中的物体，场景中的每个物体都被称为Spatial。

`Spatial`表示3D空间中的一个物体，它在空间中有三种线性变换：位移(Translation)、旋转(Rotation)、缩放(Scale)。Spatial是Geometry和Node的父类。

`Geometry`在前面的章节中已经介绍过了，它存储了物体的网格和材质，代表游戏中的可视物体。

`Node`是一个空间中的节点，每个节点(Node)中可以包含多个Spatial，每个Spatial只能有一个父节点(Node)。Node之间通过**父子关系**形成树形结构。

为了让你能够更容易理解这些概念，我从画了一幅画，用来做类比：一个气球就是一个Geometry，多个气球被一个小朋友牵在手里，小朋友的手心就是一个Node。

![小孩牵着气球](/content/images/2017/04/Spatials.png)

## 实例：HelloNode

下面我们创建2个球体，然后把它们添加到一个Node中。[源代码](https://github.com/jmecn/jME3Tutorials/blob/master/jME3Tutorials/src/main/java/net/jmecn/HelloNode.java)

	package net.jmecn;
	
	import com.jme3.app.SimpleApplication;
	import com.jme3.light.AmbientLight;
	import com.jme3.light.DirectionalLight;
	import com.jme3.material.Material;
	import com.jme3.math.ColorRGBA;
	import com.jme3.math.Vector3f;
	import com.jme3.scene.Geometry;
	import com.jme3.scene.Mesh;
	import com.jme3.scene.Node;
	import com.jme3.scene.shape.Sphere;
	
	/**
	 * 场景图、节点
	 * @author yanmaoyuan
	 *
	 */
	public class HelloNode extends SimpleApplication {
	
		@Override
		public void simpleInitApp() {
			// 球体网格
			Mesh mesh = new Sphere(16, 24, 1);
			
			// 创建2个球体
			Geometry geomA = new Geometry("红色气球", mesh);
			geomA.setMaterial(newLightingMaterial(ColorRGBA.Red));
			
			Geometry geomB = new Geometry("青色气球", mesh);
			geomB.setMaterial(newLightingMaterial(ColorRGBA.Cyan));
			
			// 将两个球体添加到一个Node节点中
			Node node = new Node("原点");
			node.attachChild(geomA);
			node.attachChild(geomB);
			
			// 设置两个球体的相对位置
			geomA.setLocalTranslation(-1, 3, 0);
			geomB.setLocalTranslation(1.5f, 2, 0);
			
			// 将这个节点添加到场景图中
			rootNode.attachChild(node);
			
			// 添加光源
			addLight();
		}
		
		/**
		 * 创建一个感光材质
		 * @param color
		 * @return
		 */
		private Material newLightingMaterial(ColorRGBA color) {
			// 创建材质
			Material mat = new Material(assetManager, "Common/MatDefs/Light/Lighting.j3md");
			
			mat.setColor("Diffuse", color);
			mat.setColor("Ambient", color);
			mat.setColor("Specular", ColorRGBA.White);
	        mat.setFloat("Shininess", 24);
	        mat.setBoolean("UseMaterialColors", true);
	        
			return mat;
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
	
		public static void main(String[] args) {
			HelloNode app = new HelloNode();
			app.start();
		}
	
	}

运行效果如下：

![HelloNode](/content/images/2017/04/HelloNode.png)

## Node

这份代码与前几章教程中的代码大同小异，最大区别在于第34~36行。Geometry不再是直接添加到rootNode中，而是添加到了另一个Node对象中。

        // 将两个球体添加到一个Node节点中
        Node node = new Node("原点");
        node.attachChild(geomA);
        node.attachChild(geomB);

为了让这两个球体能在屏幕上显示，这个node对象必须被添加到场景图的根节点rootNode中。

        // 将这个节点添加到场景图中
        rootNode.attachChild(node);

这里有几个需要注意的地方：

* SimpleApplication中的场景图`rootNode`其实就是一个Node对象。
* `rootNode`是场景中所有物体的父节点。
* 如果想要在摄像机中看到一个3D模型，就要把它添加到`rootNode`中，无论直接还是间接。

除此之外，还有一点更重要：**对父节点做任何操作，都会影响所有的子节点。**下面我们在HelleNode这个例子中增加一点特别的代码，用以证明这个结论。

(1)首先，定义一个`Spatial`类型的引用，用于指向场景中的某个物体。

(2)然后，重写主循环`simpleUpdate`方法，使用spatial对象的`rotate`方法让这个物体绕Y轴以每秒180°的速度旋转。

(3)最后，在`simpleInitApp`方法的末尾，把node对象的引用赋予这个spatial对象。

	private Spatial spatial;
	
	@Override
	public void simpleUpdate(float tpf) {
		if (spatial != null) {
			// 绕Y轴旋转
			spatial.rotate(0, 3.1415926f * tpf, 0);
		}
	}
	
	@Override
	public void simpleInitApp() {
		// ...
		
		this.spatial = node;
	}

运行程序，将会看到红蓝2个小球正在绕Y轴旋转。

在`simpleInitApp`中，还可以通过`scale`方法来同时缩放两个球体的大小。比如我们让它们变成原来的1/2大小。

	@Override
	public void simpleInitApp() {
		// 前略..
		node.scale(0.5f);
	}

![Scaled Node](/content/images/2017/04/HelloNode_scale.png)

## 遍历场景图

jME3提供了2中方式来遍历场景图：

* 广度优先遍历
 * `public void breadthFirstTraversal(SceneGraphVisitor visitor)`
* 深度优先遍历
 * `public abstract void depthFirstTraversal(SceneGraphVisitor visitor)`

`SceneGraphVisitor`是一个接口([查看源代码](https://github.com/jMonkeyEngine/jmonkeyengine/blob/master/jme3-core/src/main/java/com/jme3/scene/SceneGraphVisitor.java))，无论你是用任何一种方式来遍历场景图，都需要实现这个接口。

    package com.jme3.scene;
    public interface SceneGraphVisitor {
        public void visit(Spatial spatial);
    }

例如，编写一个方法，用于遍历场景图，将场景中所有Geometry的渲染状态都改变为线框模式。

    private void toggleWireframe(final boolean flag) {
        // 深度优先遍历
        rootNode.depthFirstTraversal(new SceneGraphVisitor() {
            @Override
            public void visit(Spatial spatial) {
                if (spatial instanceof Geometry) {
                    Geometry geom = (Geometry)spatial;
					
                    Material mat = geom.getMaterial();
                    if (mat != null) {
                        mat.getAdditionalRenderState().setWireframe(flag);
                    }
                }
            }
        });
    }
