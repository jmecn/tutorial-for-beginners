# 第十四章：特效

在jMonkeyEngine中，我们有3种实现特殊视觉效果的工具。

* 后期滤镜 FilterPostProcessor & Filter
* 场景处理器 SceneProcessor
* 粒子发射器 PraticleEmitter

## 后期滤镜

在游戏场景渲染完成后，我们可以使用滤镜（Filter）对画面进一步加工。在3D游戏中，下列效果通常都是用滤镜来实现的：

* 雾化（Fog）
* 马赛克（Mosaic）
* 玻璃（Glass）
* 雨雪（Rain / Snow）
* 景深（Depth of Field）
* 水面反射（Water Reflection）
* 泛光（Bloom）
* 高光（Glow Effect）
* 柔光（Blur）
* 光线散射（Light Scattering）
* 环形光遮蔽（Ambient Occlusion）
* 卡通边缘线/轮廓线（Cartoon Edge / Outline）
* 等等..

下面两幅图就是通过滤镜实现的效果。

水下

![水下](/content/images/2017/06/under_water.png)

雾化

![雾化](/content/images/2017/06/fog.png)

### 滤镜的用法

在jME3中，所有滤镜都是 `com.jme3.post.Filter` 的子类，由 `com.jme3.post.FilterPostProcessor` 统一管理。

#### 依赖

在使用滤镜之前，请确保你的项目用包含了对 `jme3-effects.jar` 的依赖。

Gradle

    compile 'org.jmonkeyengine:jme3-effects:3.1.0-stable'

Maven

    <dependency>
        <groupId>org.jmonkeyengine</groupId>
        <artifactId>jme3-effects</artifactId>
        <version>3.1.0-stable</version>
    </dependency>

jME3 SDK

![](/content/images/2017/06/lib.png)

#### 使用方法

滤镜的用法非常简单：

1. 把滤镜处理器（FilterPostProcessor）添加到视口（ViewPort）中；
2. 把滤镜（Filter）的实例添加到滤镜处理器中。

例如使用雾化滤镜（FogFilter）：

    @Override
    public void simpleInitApp() {
        // 初始化滤镜处理器
        FilterPostProcessor fpp = new FilterPostProcessor(assetManager);
        viewPort.addProcessor(fpp);

        // 添加雾化滤镜
        FogFilter fogFilter = new FogFilter(ColorRGBA.White, 1.5f, 100f);
        fpp.addFilter(fogFilter );
    }

视口（ViewPort）代表了玩家通过虚拟摄像机（Camera）所看到的场景画面。你可以创建多个摄像机和视口，但每个视口只应该和一个滤镜处理器关联。

滤镜处理器（FilterPostProcessor）需要通过视口拿到渲染后的画面，随后才能用滤镜来加工这些图像。滤镜工作的时候，需要通过AssetManager来加载材质（或着色器），因此创建滤镜处理器时应该通过构造方法把 AssetManager对象传递给它。

    fpp = new FilterPostProcessor(assetManager);

每个滤镜处理器中可以添加多个滤镜（Filter），**滤镜工作的顺序与添加顺序有关**。如果顺序不对，滤镜可能就达不到你期望的效果。

不要重复添加相同的滤镜。

#### 禁用滤镜

使用 `T getFilter(Class<T> class)` 方法来查询特定类型的滤镜对象。

    FogFilter fog = fpp.getFilter(FogFilter.class);
    if (fog == null) {
        // 说明没有添加过雾化滤镜
    }

使用 `removeFilter(Filter filter)` 方法来移除处理队列中的某个滤镜。

    fpp.removeFilter(fogFilter);

使用 `Filter#setEnabled(boolean active)` 方法来激活/禁用一个滤镜。

    fogFilter.setEnabled(false);// 禁用雾化滤镜

使用 removeFilter() 方法把滤镜从处理队列中移除，它就不会工作了。但如果你还要再次把它加回到滤镜处理器中，画面可能就跟以前不一样了，这是因为滤镜的顺序发生了变化。

有时我们希望暂时禁用一个滤镜，就应该使用 setEnabled() 方法，而不是直接移除它。这样可以保持滤镜在处理队列中的位置不变。

#### 抗拒齿

将 FilterPostProcessor 添加到视口中后，你会发现画面抗拒齿功能失效了。此时你需要调用 FilterPostProcessor 的 `setNumSamples(int sample)` 方法来设置抗拒齿的倍率。

    fpp.setNumSamples(4);// 开启4倍抗拒齿

你也可以根据用户启动程序时设置的参数来设置抗拒齿的倍率。

    @Override
    public void simpleInitApp() {
        ...
        // 检查用户是否设置过抗拒齿
        int numSamples = getContext().getSettings().getSamples();
        if (numSamples > 0) {
            fpp.setNumSamples(numSamples);
        }
    }

有些开发者把jME3和 Swing、JavaFX等框架集成，此时通过 setNumSamples() 设置的多重采样抗拒齿会失效，那么你就得用**快速近似抗拒齿**过滤器（com.jme3.post.filter.FXAAFilter）了。为保证正常工作，这个滤镜应该最后加入滤镜处理器中。

       fpp.addFilter(...);
       fpp.addFilter(...);
       fpp.addFilter(new FXAAFilter());

### 实例：雾化

#### 准备测试场景

上一章我们已经学习了 AppState，现在我想使用AppState专门制作一个测试场景，这样我就不必在每个SimpleApplication都重新写一遍类似的代码了。

下面是CubeAppState.java的代码。

    package net.jmecn.state;
    
    import com.jme3.app.Application;
    import com.jme3.app.SimpleApplication;
    import com.jme3.app.state.BaseAppState;
    import com.jme3.asset.AssetManager;
    import com.jme3.light.AmbientLight;
    import com.jme3.light.DirectionalLight;
    import com.jme3.light.PointLight;
    import com.jme3.material.Material;
    import com.jme3.math.ColorRGBA;
    import com.jme3.math.FastMath;
    import com.jme3.math.Vector2f;
    import com.jme3.math.Vector3f;
    import com.jme3.scene.Geometry;
    import com.jme3.scene.Node;
    import com.jme3.scene.Spatial;
    import com.jme3.scene.shape.Box;
    import com.jme3.scene.shape.Quad;
    import com.jme3.util.SkyFactory;
    import com.jme3.util.SkyFactory.EnvMapType;
    
    /**
     * 这是一个测试用场景
     * 
     * @author yanmaoyuan
     *
     */
    public class CubeAppState extends BaseAppState {
    
        private Node rootNode = new Node("Scene root");
    
        private AmbientLight ambient;
        private PointLight point;
        private DirectionalLight sun;
        private Vector3f sunDirection = new Vector3f(-0.65093255f, -0.11788898f, 0.7499261f);
        
        private AssetManager assetManager;
    
        @Override
        protected void initialize(Application app) {
    
            this.assetManager = app.getAssetManager();
    
            // 创造地板
            Material mat = assetManager.loadMaterial("Textures/Terrain/Pond/Pond.j3m");
    
            Quad quad = new Quad(200, 200);
            quad.scaleTextureCoordinates(new Vector2f(20, 20));
            Geometry geom = new Geometry("Floor", quad);
            geom.setMaterial(mat);
            geom.rotate(-FastMath.HALF_PI, 0, 0);
            rootNode.attachChild(geom);
    
            float scalar = 20;
            float side = 3f;
            for (int y = 0; y < 9; y++) {
                for (int x = 0; x < 9; x++) {
                    geom = new Geometry("Cube", new Box(side, side*2, side));
                    geom.setMaterial(getMaterial(new ColorRGBA(1 - x / 8f, y / 8f, 1f, 1f)));
                    geom.move((x + 1) * scalar, side*2, -(y + 1) * scalar);
                    rootNode.attachChild(geom);
                }
            }
    
            // 天空
            Spatial sky = SkyFactory.createSky(assetManager, "Scenes/Beach/FullskiesSunset0068.dds", EnvMapType.CubeMap);
            sky.setLocalScale(350);
            rootNode.attachChild(sky);
    
            // 创造光源
            sun = new DirectionalLight();
            sun.setDirection(sunDirection);
            sun.setColor(new ColorRGBA(0.6f, 0.6f, 0.6f, 1f));
    
            ambient = new AmbientLight();
            ambient.setColor(new ColorRGBA(0.4f, 0.4f, 0.4f, 1f));
            
            point = new PointLight();
            point.setPosition(new Vector3f(100, 200, 100));
            point.setRadius(1000);
        }
    
        private Material getMaterial(ColorRGBA color) {
            Material mat = new Material(assetManager, "Common/MatDefs/Light/Lighting.j3md");
            mat.setColor("Diffuse", color);
            mat.setColor("Ambient", color);
            mat.setColor("Specular", ColorRGBA.White);
            mat.setFloat("Shininess", 20f);
            mat.setBoolean("UseMaterialColors", true);
    
            return mat;
        }
        
        public Vector3f getSunDirection() {
            return sunDirection;
        }
    
        @Override
        protected void cleanup(Application app) {
        }
    
        @Override
        protected void onEnable() {
            SimpleApplication app = (SimpleApplication) getApplication();
    
            app.getRootNode().attachChild(rootNode);
            app.getRootNode().addLight(ambient);
            app.getRootNode().addLight(point);
            app.getRootNode().addLight(sun);
        }
    
        @Override
        protected void onDisable() {
            SimpleApplication app = (SimpleApplication) getApplication();
    
            app.getRootNode().detachChild(rootNode);
            app.getRootNode().removeLight(ambient);
            app.getRootNode().removeLight(point);
            app.getRootNode().removeLight(sun);
        }
    
    }

#### 主类

创建一个HelloFog类，在这个类中使用CubeAppState，并初始化滤镜处理器和雾化滤镜。

下面是HelloFog.java的代码：

    package net.jmecn.effect;
    
    import com.jme3.app.SimpleApplication;
    import com.jme3.math.ColorRGBA;
    import com.jme3.post.FilterPostProcessor;
    import com.jme3.post.filters.FogFilter;
    
    import net.jmecn.state.CubeAppState;
    
    /**
     * 演示雾化滤镜(FogFilter)的作用
     * 
     * @author yanmaoyuan
     *
     */
    public class HelloFog extends SimpleApplication {
    
        @Override
        public void simpleInitApp() {
            stateManager.attach(new CubeAppState());
            
            FilterPostProcessor fpp = new FilterPostProcessor(assetManager);
            fpp.setNumSamples(4);// 4倍抗拒齿
            viewPort.addProcessor(fpp);
    
            // 雾化滤镜
            FogFilter fogFilter = new FogFilter(ColorRGBA.White, 1.5f, 100f);
            fpp.addFilter(fogFilter);
    
            flyCam.setMoveSpeed(10f);
        }
    
        public static void main(String[] args) {
            HelloFog app = new HelloFog();
            app.start();
        }
    
    }

效果对比如下：

![](/content/images/2017/06/no_fog.png)

![](/content/images/2017/06/with_fog.png)

### 实例：多滤镜

下面的例子，将演示多个滤镜结合起来使用的效果。

#### FilterAppState

我又写了一个 AppState，用它来管理 FilterPostProcessor。

    package net.jmecn.state;
    
    import java.util.ArrayList;
    import java.util.List;
    
    import com.jme3.app.Application;
    import com.jme3.app.state.BaseAppState;
    import com.jme3.asset.AssetManager;
    import com.jme3.post.Filter;
    import com.jme3.post.FilterPostProcessor;
    import com.jme3.post.SceneProcessor;
    import com.jme3.renderer.ViewPort;
    
    /**
     * 滤镜功能测试
     * @author yanmaoyuan
     *
     */
    public class FilterAppState extends BaseAppState {
    
        private AssetManager assetManager;
        private FilterPostProcessor fpp;
        private ViewPort viewPort;
        
        private List<Filter> filtersToAdd = new ArrayList<Filter>();
        private List<Filter> filtersToRemove = new ArrayList<Filter>();
        
        @Override
        protected void initialize(Application app) {
            this.assetManager = app.getAssetManager();
            this.viewPort = app.getViewPort();
            
            /**
             * 检查用户是否已经设置过FilterPostProcessor
             */
            for( SceneProcessor processor : viewPort.getProcessors()) {
                if (processor instanceof FilterPostProcessor) {
                    fpp = (FilterPostProcessor) processor;
                    break;
                }
            }
            
            /**
             * 初始化 FilterPostProcessor
             */
            if (fpp == null) {
                // 创建滤镜处理器
                fpp = new FilterPostProcessor(assetManager);
                
                // 检查用户是否设置过抗拒齿
                int numSamples = app.getContext().getSettings().getSamples();
                if (numSamples > 0) {
                    fpp.setNumSamples(numSamples);
                }
            }
        }
        
        @Override
        public void update(float tpf) {
            if (filtersToAdd.size() > 0) {
                // 添加滤镜
                addFilters();
                filtersToAdd.clear();
            }
            
            if (filtersToRemove.size() > 0) {
                // 移除滤镜
                removeFilters();
                filtersToRemove.clear();
            }
        }
        
        /**
         * 添加滤镜
         * @param filter
         */
        public void add(Filter filter) {
            if (filter == null)
                return;
            
            filtersToAdd.add(filter);
        }
        
        /**
         * 移除滤镜
         * @param filter
         */
        public void remove(Filter filter) {
            if (filter == null)
                return;
            
            filtersToRemove.add(filter);
        }
        
        /**
         * 添加所有滤镜
         */
        private void addFilters() {
            int length = filtersToAdd.size();
             // 按顺序添加到 FilterPostProcessor 中
            for(int i=0; i<length; i++) {
                Filter filter = filtersToAdd.get(i);
                // 相同滤镜只添加一次。
                if (null == fpp.getFilter(filter.getClass())) {
                    fpp.addFilter(filter);
                }
            }
        }
        
        /**
         * 移除所有滤镜
         */
        private void removeFilters() {
            int length = filtersToRemove.size();
            
            for(int i=0; i<length; i++) {
                Filter filter = filtersToRemove.get(i);
                fpp.removeFilter(filter);
            }
        }
        
        @Override
        protected void cleanup(Application app) {
            filtersToAdd.clear();
            filtersToRemove.clear();
        }
    
        @Override
        protected void onEnable() {
            viewPort.addProcessor(fpp);
        }
    
        @Override
        protected void onDisable() {
            viewPort.removeProcessor(fpp);
        }
    
    }


#### HelloFilters

我不打算详细介绍每一个Filter的作用，所以用这一个例子来尽量演示。我这个例子中制作了窗口，使用鼠标选择对应的过滤器，将会把它添加到FilterAppState中。

    package net.jmecn.effect;
    
    import java.util.ArrayList;
    import java.util.List;
    
    import com.jme3.app.DebugKeysAppState;
    import com.jme3.app.FlyCamAppState;
    import com.jme3.app.SimpleApplication;
    import com.jme3.app.StatsAppState;
    import com.jme3.app.state.ScreenshotAppState;
    import com.jme3.math.ColorRGBA;
    import com.jme3.math.Vector3f;
    import com.jme3.post.Filter;
    import com.jme3.post.filters.BloomFilter;
    import com.jme3.post.filters.CartoonEdgeFilter;
    import com.jme3.post.filters.ColorOverlayFilter;
    import com.jme3.post.filters.CrossHatchFilter;
    import com.jme3.post.filters.DepthOfFieldFilter;
    import com.jme3.post.filters.FogFilter;
    import com.jme3.post.filters.LightScatteringFilter;
    import com.jme3.post.ssao.SSAOFilter;
    import com.jme3.water.WaterFilter;
    import com.jme3.water.WaterFilter.AreaShape;
    import com.simsilica.lemur.Button;
    import com.simsilica.lemur.Checkbox;
    import com.simsilica.lemur.Command;
    import com.simsilica.lemur.Container;
    import com.simsilica.lemur.GuiGlobals;
    import com.simsilica.lemur.Label;
    import com.simsilica.lemur.style.BaseStyles;
    import com.simsilica.lemur.style.ElementId;
    
    import net.jmecn.state.CubeAppState;
    import net.jmecn.state.FilterAppState;
    
    /**
     * 演示滤镜的作用
     * 
     * @author yanmaoyuan
     *
     */
    public class HelloFilters extends SimpleApplication {
    
        /**
         * 全部滤镜
         */
        private List<Filter> filters = new ArrayList<Filter>();
        
        // 全部滤镜
        private BloomFilter bloom;
        private CartoonEdgeFilter cartoonEdge;
        private ColorOverlayFilter colorOverlay;
        private CrossHatchFilter crossHatch;
        private DepthOfFieldFilter depthOfField;
        private FogFilter fog;
        private LightScatteringFilter lightScattering;
        private SSAOFilter ssao;
        private WaterFilter water;
        // 我们自定义的Filter
        private GrayScaleFilter grayScale;
    
        public HelloFilters() {
            super(new FlyCamAppState(), new StatsAppState(), new DebugKeysAppState(), new CubeAppState(),
                    new FilterAppState(), new ScreenshotAppState("screenshots/", System.currentTimeMillis() + "_"));
        }
    
        @Override
        public void simpleInitApp() {
            flyCam.setMoveSpeed(10f);
            flyCam.setDragToRotate(true);
    
            GuiGlobals.initialize(this);
            BaseStyles.loadGlassStyle();
            GuiGlobals.getInstance().getStyles().setDefaultStyle("glass");
    
            initFilters();
    
            initGui();
        }
    
        /**
         * 初始化所有滤镜
         */
        private void initFilters() {
            // 发光特效
            bloom = new BloomFilter(BloomFilter.GlowMode.SceneAndObjects);
    
            // 卡通边缘
            cartoonEdge = new CartoonEdgeFilter();
            cartoonEdge.setEdgeColor(ColorRGBA.Black);
    
            // 纯色叠加
            colorOverlay = new ColorOverlayFilter(new ColorRGBA(1f, 0.8f, 0.8f, 1f));
    
            // 交叉阴影
            crossHatch = new CrossHatchFilter();
    
            // 景深
            depthOfField = new DepthOfFieldFilter();
            depthOfField.setFocusDistance(0);
            depthOfField.setFocusRange(20);
            depthOfField.setBlurScale(1.4f);
    
            // 雾化
            fog = new FogFilter(ColorRGBA.White, 1.5f, 200f);
    
            // 灰度化
            grayScale = new GrayScaleFilter();
    
            // 光线散射
            Vector3f sunDir = stateManager.getState(CubeAppState.class).getSunDirection();
            lightScattering = new LightScatteringFilter(sunDir.mult(-3000));
    
            // 屏幕空间环境光遮蔽
            ssao = new SSAOFilter(7f, 14f, 0.4f, 0.6f);
    
            // 水
            water = new WaterFilter();
            // 设置水面的范围。若不设置，则为无限范围。
            //water.setCenter(new Vector3f(100, 0, -100));
            //water.setRadius(100);// 水面半径
            //water.setShapeType(AreaShape.Square);// 水面形状，可以是圆形，也可以是方形。
            
            water.setDeepWaterColor(new ColorRGBA(0.8f, 1f, 0.8f, 1f));// 水下颜色
            water.setLightDirection(sunDir);// 阳光方向
            water.setWaterHeight(6f);// 水面高度
            water.setUnderWaterFogDistance(80);// 水下视距
            
            filters.add(bloom);
            filters.add(cartoonEdge);
            filters.add(colorOverlay);
            filters.add(crossHatch);
            filters.add(depthOfField);
            filters.add(fog);
            filters.add(grayScale);
            filters.add(lightScattering);
            filters.add(ssao);
            filters.add(water);
        }
    
        private void initGui() {
            Container window = new Container();
            guiNode.attachChild(window);
            
            window.addChild(new Label("Filters", new ElementId("title")));
            
            for(int i=0; i<filters.size(); i++) {
                Filter filter = filters.get(i);
                window.addChild(createCheckbox(filter));
            }
            
            window.setLocalTranslation(10, cam.getHeight() - 10, 0);
        }
        
        /**
         * 实例化一个Checkbox，作为滤镜的开关。
         * @param filter
         * @return
         */
        @SuppressWarnings("unchecked")
        private Checkbox createCheckbox(final Filter filter) {
            
            String name = filter.getClass().getSimpleName();
            final Checkbox cb = new Checkbox(name);
            
            cb.addClickCommands(new Command<Button>() {
                @Override
                public void execute(Button source) {
                    FilterAppState state = stateManager.getState(FilterAppState.class);
                    if (cb.isChecked()) {
                        state.add(filter);
                    } else {
                        state.remove(filter);
                    }
                }
            });
            
            return cb;
        }
    
        public static void main(String[] args) {
            HelloFilters app = new HelloFilters();
            app.start();
        }
    
    }

结果，就是我们在本章开头看到的画面。

![](/content/images/2017/06/depth_of_field.png)

### 自定义滤镜

滤镜的背后是着色器（Shader）。

3D场景渲染完毕后，GPU将会把画面数据返回给CPU，并保存为FrameBuffer对象。滤镜处理器在工作的时，会把FrameBuffer中的数据渲染到一个内存中的纹理（Texture）上，然后把这个纹理交给Filter去进行后期处理。

基本上每个滤镜都有对应的Shader程序，在程序中加载为一个Material对象。处理器顺序调用每一个滤镜的Material，对内存中的纹理进行渲染，最后再把画面数据写回到FrameBuffer中。

下面我来实现一个非常简单的灰度化滤镜（GrayScaleFilter），它的作用是对画面中每个像素的R、G、B三元色求均值，把画面变成灰色。

    float gray = (color.r + color.g + color.b) / 3.0;

#### 片元着色器

GLSL100版本：Materials/GrayScale/GrayScale.frag

    uniform sampler2D m_Texture;
     
    varying vec2 texCoord;
     
    void main() {
         
        // Convert to grayscale
        vec3 color = texture2D(m_Texture, texCoord).rgb;
        float gray = (color.r + color.g + color.b) / 3.0;
        vec3 grayscale = vec3(gray);
         
        gl_FragColor = vec4(grayscale, 1.0);
    }

GLSL150版本：Materials/GrayScale/GrayScale15.frag

    #import "Common/ShaderLib/MultiSample.glsllib"
     
    uniform COLORTEXTURE m_Texture;
      
    in vec2 texCoord;
    out vec4 fragColor;
      
    void main() {
          
        // Convert to grayscale
        vec3 color = getColor(m_Texture, texCoord).rgb;
        float gray = (color.r + color.g + color.b) / 3.0;
        vec3 grayscale = vec3(gray);
          
        fragColor = vec4(grayscale, 1.0);
    }

#### 材质定义脚本

由于这个着色器只需要片元着色，因此顶点着色器直接用jME3自带的Post.vert即可。

Materials/GrayScale/GrayScale.j3md

    MaterialDef GrayScale {
      
        MaterialParameters {
            Int NumSamples
            Texture2D Texture
        }
      
        Technique {
            VertexShader GLSL150:   Common/MatDefs/Post/Post15.vert
            FragmentShader GLSL150: Materials/GrayScale/GrayScale15.frag
      
            WorldParameters {
                WorldViewProjectionMatrix
            }
     
            Defines {
                RESOLVE_MS : NumSamples          
            }
        }
      
        Technique {
            VertexShader GLSL100:   Common/MatDefs/Post/Post.vert
            FragmentShader GLSL100: Materials/GrayScale/GrayScale.frag
      
            WorldParameters {
                WorldViewProjectionMatrix
            }
        }
    }

#### GrayScaleFilter

继承Filter，实现一个GrayScaleFilter子类。由于这个过滤器特别简单，因此只需要加载材质，然后重写getMaterial方法返回我们的材质即可。

    package net.jmecn.effect;
    import com.jme3.asset.AssetManager;
    import com.jme3.material.Material;
    import com.jme3.post.Filter;
    import com.jme3.renderer.RenderManager;
    import com.jme3.renderer.ViewPort;
    
    public class GrayScaleFilter extends Filter {

    	public GrayScaleFilter() {
    		super("GrayScaleFilter");
    	}

    	@Override
    	protected Material getMaterial() {
    		return this.material;
    	}
    
    	@Override
    	protected void initFilter(final AssetManager manager, final RenderManager renderManager, final ViewPort vp,
    			final int w, final int h) {
    		this.material = new Material(manager, "Materials/GrayScale/GrayScale.j3md");
    	}
    
    }

#### 使用灰度化滤镜

把 GrayScaleFilter 对象加到 FilterPostProcessor 中即可。
效果如下：

![](/content/images/2017/06/gray_scale.png)

## 场景处理器

场景处理器（SceneProcessor）可以在场景渲染前后对画面进行处理。SceneProcessor 是一个接口，它的子类通常都命名为xxProcessor或xxRenderer。

例如 FilterPostProcessor 就是 SceneProcessor 的一个实现，只不过它更加专注于后期处理。

在[第七章：光与影](http://blog.jmecn.net/chapter-7-light-and-shadow/)中，我们已经接触过阴影渲染器（ShadowRenderer）和阴影过滤器（ShadowFilter），它们分别是使用 SceneProcessor 和 Filter 实现的。

* 基于 SceneProcessor 实现
 * DirectionalLightShadowRenderer
 * PointLightShadowRenderer
 * SpotLightShadowRednerer
* 基于 Filter 实现
 * DiectionalLightShadowFilter
 * PointLightShadowFilter
 * SpotLightShadowFilter

### SceneProcessor的使用

SceneProcessor的用法很简单。其实在介绍 FilterPostProcessor 时，我们已经知道SceneProcessor的用法了：把它添加到viewPort中。

    private DirectionalLight sunLight;

    @Override
    public void simpleInitApp() {

        ...

        // 定向光阴影渲染器
        DirectionalLightShadowRenderer dlsr = new DirectionalLightShadowRenderer(assetManager, 1024, 4);
        dlsr.setLight(sunLight);// 设置定向光源

        viewPort.addProcessor(dlsr);
    }

### 工作顺序问题

除了阴影渲染器以外，jME3还提供了**高动态光照渲染器**（HDRRenderer）、**水面反射渲染器**（SimpleWaterProcessor）等实用场景渲染器。

甚至，jME3 的截屏（ScreenshotsAppState）、录像（VideoRecordAppState）功能也是通过 SceneProcessor 实现的，因为 SceneProcessor 很容易拿到 ViewPort 中的画面。

请注意，每个ViewPort可以添加多个 SceneProceesor，这些**SceneProcessor 的工作顺序与添加到ViewPort中的顺序有关**。

如果你在截图的时候发现画面中并没有体现出滤镜的特效，可能就是因为 FilterPostProcessor 是在
 ScreenshotsAppState 之后被添加到ViewPort中的。

### 自定义SceneProcessor

SceneProcessor 的背后也是着色器（Shader），你可以根据自己的需要来实现特殊功能的 SceneProcessor，这与 Filter的实现没有什么本质区别。

不过，这些内容已经超出本章要介绍的范围，就不在本文中讨论了。

## 粒子系统

当你在游戏中看到下列内容之一时，可能是使用粒子系统实现的。

* 火焰，闪光
* 雨滴，雪花，瀑布，喷泉，落叶
* 爆炸，碎片，冲击波
* 云，雾，烟，尘
* 蜂群，星云，流星尾迹
* 魔法飞弹
* ..

这类场景元素很难通过建模来实现，想要制作模型动画几乎是不可能的。在3D游戏中，我们使用粒子系统来模拟其它传统的渲染技术难以实现的抽象视觉效果。

![](/content/images/2017/06/ParticleFire.png)

### 相关概念

#### 粒子系统的定义

粒子系统是由总体具有相同的表现规律，个体却随机表现出不同的特征的大量显示元素构成的集合。

这个定义有几个要素：

1. 群体性
粒子系统是由“大量显示元素”构成的。因此，用粒子系统来描述一群蜜蜂是正确的，但描述一只蜜蜂没有意义。
2. 统一性
粒子系统的每个元素具有相同的表现规律。比如组成火堆的每一个火苗，都是红色，发亮，向上跳动，并且会在上升途中逐渐变小以至消失。
3. 随机性
粒子系统的每个元素又随机表现出不同特征。比如蜂群中的每一只蜜蜂，它的飞行路线可能会弯弯曲曲，就象布郎运动一般无规则可寻，但整个蜂群，却是看起来直线向一个方向运动（这就是上一点所说的统一性）。

粒子系统既可以是动态的（火花、雨滴），也可以是静态的（草坪、毛发）。有些看似完全不同的东西，却可以使用相同的模式来实现。

* “爆炸”和“烟尘”的区别，仅仅是粒子运动速度不同。
* “火焰”和“瀑布”的区别，在于粒子的颜色和运动方向不同。

#### 粒子系统的组成

一般来说，粒子系统是由3大部分组成的：

1. 粒子发射器
 * 粒子数量
 * 生成速率
 * 生成区域
 * 持续时间
 * 循环模式
2. 粒子外观
 * 形状（点状、片状、3D网格）
 * 颜色
 * 材质
 * 大小
3. 粒子行为
 * 运动速度（线速度、角速度）
 * 速度的变化（散开、旋转、加速、减速）
 * 变形

**粒子发射器**定义了粒子系统的**群体性**，包括粒子的数量、生成区域、生命周期等。

**粒子外观**和**粒子行为**共同定义了粒子系统的**统一性**，系统中的每个元素都应该遵循这些规则。

粒子系统的**随机性**则是通过下列方式实现的：

* 虽然粒子系统的生成区域是一个固定形状，但每个粒子生成时可以在范围内**随机挑选一个出生点**。
* **设置多种可选的粒子外观**，每个粒子生成时可以随机选择一个，有些粒子系统还允许粒子在活动过程中**随机改变外观**。
* 粒子行为有一定的变量和范围，**每个粒子的行为参数都在一定区间内随机选择**。比如粒子运动的初速度可以设置最大值和最小值，运动方向也可以在一定范围内调整。

#### jME3中的粒子系统

在jME3中，粒子系统是由这么几个部分组成的：

1. 粒子 `Particle`
定义粒子的位置、速度、大小、外观、生存时间等信息。
2. 粒子发射器 `ParticleEmitter`
用于随机生成一定数量的粒子，驱动粒子动画。
3. 粒子生成区域 `EmitterShape`
规定粒子生成的区域，可以是从固定点生成，也可以在球体、方块、平面或者任意网格范围内生成。
4. 粒子网格 `ParticleMesh`
保存所有粒子的网格数据。
5. 粒子材质 `Common/MatDefs/Misc/Particle.j3md`
粒子也是由着色器绘制的，需要使用材质来定义如何渲染。
6. 粒子行为 `ParticleInfluencer`
用于控制粒子的运动方式，例如直行、旋转、布朗运动等。

### ParticleEmitter的用法

粒子发射器（ParticleEmitter）是Geometry的子类，把它的实例添加到场景图中，玩家就能看到它所产生的粒子。

jME3粒子为点状或片状，通过设置不同的材质和纹理，可以改变粒子的外观。jME3现在不支持复杂形状的粒子，如果你希望看到方块、小球形状的粒子，可以自己实现。

为粒子系统配置不同的参数，可以得到完全不同的效果。置于哪一种效果是你需要的，这就需要一点点来尝试了。

#### 创建发生器

有时，游戏中的粒子特效并不是单一的粒子系统，你需要为特效中的每个粒子系统创建一个发生器。

例如，“爆炸”的粒子特效是由闪光、火云、烟尘、碎片、冲击波等多种粒子组合而成的，通常还会伴随着瞬间的强光。

![](/content/images/2017/06/explosion.jpg)

创建一个发生器非常简单，构造参数为：发生器名称、粒子形状、粒子总数。

    ParticleEmitter explosion = new ParticleEmitter(
    "explosion", ParticleMesh.Type.Triangle, 30);

将发生器添加到场景图中，并把它安放到适合的位置：

    rootNode.attachChild(explosion);
    explosion.setLocalTranslation(bomb.getLocalTranslation());

现在发生器就会开始按照一定的速率随机产生粒子，默认值是每秒20个粒子。如果想要改变粒子产生速率，可以调用下面的方法：

    explosion.setParticlesPerSec(30);

你也可以调用下面的方法，把全部粒子一次性全部喷发出来！

    explosion.emitAllParticles();

粒子发射器会持续工作，当粒子的生命周期到达终点后，又会有新的粒子产生。如果你希望所有粒子只生成一次，可以将上面两个方法结合起来使用。先把粒子产生速率设为0，然后一次性全喷出来。

    explosion.setParticlesPerSec(0);
    explosion.emitAllParticles();

不需要粒子发射器继续工作，可以调用下面的方法来停止它：

    explosion.killAllParticles()

粒子的形状有两种类型，分别是点状和面状，你只能二选一。

    ParticleMesh.Type.Point
    ParticleMesh.Type.Triangle

点状很好理解，面状则是由2个三角形拼成的方形，大概是这个样子。

![](/content/images/2017/06/type_triangle.png)

#### 设置粒子的材质

粒子发射器有专用的材质 `Common/MatDefs/Misc/Particle.j3md`。使用不同的纹理贴图，可以让粒子呈现出不同的外形。你可以自己创建所需要的粒子贴图，从而获得碎石、烟雾、水滴、雪花等不同的粒子特效。

![flash](/content/images/2017/06/flash.png)

上图是我们接下来要是用的“闪光”特效贴图，这个图片可以在 [jme3-testdata](https://github.com/jMonkeyEngine/jmonkeyengine/tree/master/jme3-testdata/src/main/resources/Effects/Explosion) 中找到。

    Material flash_mat = new Material(
        assetManager, "Common/MatDefs/Misc/Particle.j3md");
    flash_mat.setTexture("Texture",
        assetManager.loadTexture("Effects/Explosion/flash.png"));
    flash.setMaterial(flash_mat);
    flash.setImagesX(2); // 列
    flash.setImagesY(2); // 行
    flash.setSelectRandomImage(true);

特效贴图可以只有一张图片，也可以是精灵动画 —— 一系列稍有不同的图片，以相同的行距和列距保存在一张图中。如果你使用了精灵动画，那么你需要：

* 使用 `setImageX(2)` 和 `setImageY(2)` 来设定特效贴图的列数和行数。
* 使用 `setSelectRandomImage(true)` 来设定精灵动画的播放顺序。true 为随机播放，false 为顺序播放。

下面我们来看几个粒子特效贴图的范例，很快你就知道该怎样自己做一个特效贴图了。

##### 默认的特效贴图

`Common/MatDefs/Misc/Particle.j3md` 材质要搭配灰度贴图一起使用。下图中的黑色部分其实是完全透明的，白色部分则是半透明的。下表中的特效贴图都可以在 jme3-testdata 中找到，你也可以按照这个规则来自己制作特图。

<table>
  <tr><th>资源路径</th><th>列数*行数</th><th>预览</th></tr>
  <tr>
    <td>Effects/Explosion/Debris.png</td>
    <td>3*3</td>
    <td><img src="content/images/2017/06/Debris.png" /></td>
  </tr>
  <tr>
    <td>Effects/Explosion/flame.png</td>
    <td>2*2</td>
    <td><img src="content/images/2017/06/flame.png" width="128"/></td>
  </tr>
  <tr>
    <td>Effects/Explosion/flash.png</td>
    <td>2*2</td>
    <td><img src="content/images/2017/06/flash.png" width="128"/></td>
  </tr>
  <tr>
    <td>Effects/Explosion/roundspark.png</td>
    <td>1*1</td>
    <td><img src="content/images/2017/06/roundspark.png" /></td>
  </tr>
  <tr>
    <td>Effects/Explosion/shockwave.png</td>
    <td>1*1</td>
    <td><img src="content/images/2017/06/shockwave.png" /></td>
  </tr>
  <tr>
    <td>Effects/Explosion/smoketrail.png</td>
    <td>1*3</td>
    <td><img src="content/images/2017/06/smoketrail.png" width="128" /></td>
  </tr>
  <tr>
    <td>Effects/Explosion/spark.png</td>
    <td>1*1</td>
    <td><img src="content/images/2017/06/spark.png" /></td>
  </tr>
  <tr>
    <td>Effects/Smoke/Smoke.png</td>
    <td>15*1</td>
    <td><img src="content/images/2017/06/Smoke.png" width="200" style="background-color:#000"></img></td>
  </tr>
</table>

> 使用粒子发射器的 `setStartColor()` 和 `setEndColor()` 方法可以设置特效贴图的半透明部分在渲染时的实际颜色。

#### 配置粒子参数

粒子发射器有很多参数可以配置，通常来说，每个粒子系统只用得上其中几个参数。如果你没有专门去修改这些参数的话，粒子发射器将会使用它们的默认值。

<table>
 <tr><th>参数</th><th>方法</th><th>默认值</th><th>说明</th></tr>
  <tr>
    <td>粒子数量</td>
    <td><code>setNumParticles()</code></td>
    <td></td>
    <td>同一时刻能见到的最大粒子数量，通过构造方法初始化。</td>
  </tr>
  <tr>
    <td>发射速率</td>
    <td><code>setParticlesPerSec()</code></td>
    <td>20</td>
    <td>特效的密度，定义每秒发射多少个粒子。<br>设为0可以控制发射器的开关。<br>设为大于0的数字将持续发射粒子。</td>
  </tr>
  <tr>
    <td>尺寸</td>
    <td><code>setStartSize()</code>, <code>setEndSize()</code></td>
    <td>0.2f, 2f</td>
    <td>特效贴图的放大系数。设为不同的值可以制造出逐渐缩小/放大的粒子子效果。</td>
  </tr>
  <tr>
    <td>颜色</td>
    <td><code>setStartColor()</code>, <code>setEndColor()</code></td>
    <td>灰色</td>
    <td>控制特图中半透明部分的颜色。设为相同的值颜色可以保持粒子颜色不变。设为不同的颜色，则会出现渐变效果（例：火的焰心和焰尖）。</td>
  </tr>
  <tr>
    <td>运动方向/速度</td>
    <td><code>getParticleInfluencer() .setInitialVelocity()</code></td>
    <td>Vector3f(0,0,0)</td>
    <td>使用向量定义粒子运动的初始方向和速度，向量越长，速度越快。</td>
  </tr>
  <tr>
    <td>散开</td>
    <td><code>getParticleInfluencer() .setVelocityVariation()</code></td>
    <td>0.2f</td>
    <td>设置粒子运动方向的偏移程度，取值范围为0~1。改变这个值，可以制造出不同形状的粒子云。<br> 1 = 最大偏移，粒子可在初速度方向360°范围内随机发射。(例：爆炸)<br> 0.5f = 粒子可在初速度方向180°范围内随机发射。<br> 0 = 无任何偏移，粒子沿初速度方向直线发射。 (例：激光枪)</td>
  </tr>
  <tr>
    <td>粒子正面方向<br> (二选一)</td>
    <td><code>setFacingVelocity()</code></td>
    <td>false</td>
    <td>true = 粒子正面朝向它正在运动的方向(例：导弹)。<br> false = 粒子正面始始终和初始方向保持一致(例：残片)。</td>
  </tr>
  <tr>
    <td>粒子正面方向<br> (二选一)</td>
    <td><code>setFaceNormal()</code></td>
    <td>Vector3f.NAN</td>
    <td>Vector3f = 粒子正面朝向给定的方向 (例：冲击波始终保持正面朝上 = Vector3f.UNIT_Y).<br>Vector3f.NAN = 粒子始终正对摄像机。</td>
  </tr>
  <tr>
    <td>粒子生存时间</td>
    <td><code>setLowLife()</code>, <code>setHighLife()</code></td>
    <td>3f, 7f</td>
    <td>设置粒子消失之前的存活时间。每个粒子的生存时间将在最小值和最大值之间随机取值；LowLife的值应当比HightLife的值更小。<br>LowLife &lt; 1f 会让粒子系统看起来更加活跃，调高LowLife的取值则会当粒子系统看起来更稳定。<br>HighLieft &lt; 1f 会让粒子系统瞬间爆炸，设为更大的值则会更加持久（例：烟、雾）。<br>将LowLife和HighLife设为相同的值，则会让粒子均匀发射（例：喷泉），设为不同的值可以制造出“扭曲”的感觉（例：具有超高火焰的火堆）。</td>
  </tr>
  <tr>
    <td>旋转速度</td>
    <td><code>setRotateSpeed()</code></td>
    <td>0f</td>
    <td>等于 0 表示粒子运动时不发生旋转 (例：烟雾、苍蝇)。<br>大于 0 表示粒子运动时旋转的快慢 (例：弹片、飞镖、失控的导弹)。</td>
  </tr>
  <tr>
    <td>旋转方向</td>
    <td><code>setRandomAngle()</code></td>
    <td>false</td>
    <td>设为 true 粒子发射时将向随机角度旋转(例：爆炸弹片)。<br>设为 false 粒子将保持它们在特效贴图上的姿态。</td>
  </tr>
  <tr>
    <td>出生区域</td>
    <td><code>setShape()</code></td>
    <td>EmitterPointShape()</td>
    <td>默认情况下，粒子将会在发射器所在的位置（原点）生成。你可以改变出生区域，让粒子在一定范围内随机生成，从而使粒子系统看起来更加不规则。<br> EmitterSphereShape() 球形区域。<br>EmitterBoxShape() 方块区域。<br>EmitterMeshVertexShape() 从给定网格的随机顶点发射。<br>EmitterMeshFaceShape() 从给定网格的随机表面发射。<br>EmitterMeshConvexHullShape() 从给定网格内部的随机位置发射。</td>
  </tr>
  <tr>
    <td>重力</td>
    <td><code>setGravity()</code></td>
    <td>Vector3f(0.0f, 0.1f, 0.0f)</td>
    <td>粒子在运动过程中，将朝重力向量的方向坠落。该向量越长，坠落速度越快（例：喷泉）。<br>设为 (0,0,0) 粒子将不受重力影响，永远保持初始运动方向（例：火焰）。</td>
  </tr>
</table>

### 实例：火焰

让我们通过一个实例来看看jME3的粒子系统。

    package net.jmecn.effect;
    
    import com.jme3.app.SimpleApplication;
    import com.jme3.effect.ParticleEmitter;
    import com.jme3.effect.ParticleMesh;
    import com.jme3.material.Material;
    import com.jme3.math.ColorRGBA;
    import com.jme3.math.Vector3f;
    
    /**
     * 使用粒子发射器实现火焰。
     * 
     * @author yanmaoyuan
     *
     */
    public class HelloParticle extends SimpleApplication {
    
        public static void main(String[] args) {
            HelloParticle app = new HelloParticle();
            app.start();
        }
    
        @Override
        public void simpleInitApp() {
            
            // 粒子发射器
            ParticleEmitter fire = new ParticleEmitter("Emitter", ParticleMesh.Type.Triangle, 30);
            
            // 粒子的生存周期
            fire.setLowLife(1f);// 最短1秒
            fire.setHighLife(3f);// 最长3秒
            
            /**
             * 粒子的外观
             */
            // 加载材质
            Material mat = new Material(assetManager, "Common/MatDefs/Misc/Particle.j3md");
            mat.setTexture("Texture", assetManager.loadTexture("Effects/Explosion/flame.png"));
            fire.setMaterial(mat);
            
            // 设置2x2的动画
            fire.setImagesX(2);
            fire.setImagesY(2);
            
            // 初始颜色，结束颜色
            fire.setEndColor(new ColorRGBA(1f, 0f, 0f, 1f)); // red
            fire.setStartColor(new ColorRGBA(1f, 1f, 0f, 0.5f)); // yellow
            
            // 初始大小，结束大小
            fire.setStartSize(1.5f);
            fire.setEndSize(0.1f);
            
            /**
             * 粒子的行为
             */
            // 初速度
            fire.getParticleInfluencer().setInitialVelocity(new Vector3f(0, 2, 0));
            // 速度的变化
            fire.getParticleInfluencer().setVelocityVariation(0.3f);
            // 不受重力影响
            fire.setGravity(0, 0, 0);
            
            rootNode.attachChild(fire);
        }
    }

运行结果如下：

![](/content/images/2017/06/ParticleFire.png)

### t0neg0d粒子发射器

[t0neg0d](https://hub.jmonkeyengine.org/u/t0neg0d/summary)是曾经活跃于jME社区的一名开发者，主要活动时间是2011年到2015年。

![](/content/images/2017/06/t0neg0d.png)

t0neg0d是一位相当富有创造力的图形工程师，在她活跃的这段时间，她持续推出了很多具有非常意义的jME3插件。不仅非常实用，而且效果炫酷。

1. t0neg0d-gui 图形用户组件
在 Lemur 诞生之前，t0neg0d 创造的这套GUI组件是jME3社区唯一能与 Nifty-gui 抗衡的 GUI。它比 Nifty 更易用，又吸收了 Nifty 使用XML进行样式定义的特点，吸引了大批用户。
t0neg0d 已经2年没露面了，现在这套 GUI 组件由它的用户自行维护，比如 [Stephen Gold](https://github.com/stephengold/tonegodgui) 和 [JavaSaBr](https://github.com/JavaSaBr/tonegodgui)。
2. t0neg0d-emitter 强大的粒子发射器
jME3的粒子系统其实主要是定义了粒子系统的架构，但是它的功能和其他3D游戏引擎比起来还有相当的差距。毕竟jME3本身就是一个开源设计，官方鼓励开发者自己去扩展jME3系统。
t0neg0d就是怎么一个人，她觉得jME3自带的粒子系统不够酷炫，于是自己制造了一个更加强大的粒子系统。这个粒子系统能够定义更加复杂的粒子行为，并且还能使用Geometry作为单个粒子的形状，甚至还实现了粒子碰撞检测。
我fork了一份 t0neg0d-emitter 的[源码](https://github.com/jmecn/tonegodemitter)，并在jME3.1环境下编译、测试通过。
3. SkyControl 炫丽的天穹
这个组件更是帅到没朋友。多层天球，多层云层，昼夜更替，光照变化，经纬度变化。。[SkyControl](https://github.com/jMonkeyEngine-Contributions/SkyControl) 基于着色器实现，在获得 t0neg0d 本人同意后，被Stephen Gold移植到他的[jme3-utilities](https://github.com/stephengold/jme3-utilities)项目中。

![t0neg0d_fire.jpg](/content/images/2017/06/t0neg0d_fire.jpg)

![t0neg0d_ice.jpg](/content/images/2017/06/t0neg0d_ice.jpg)

![t0neg0d_spark.png](/content/images/2017/06/t0neg0d_spark.png)

![](/content/images/2017/06/Sky.png)
![](/content/images/2017/06/Sky2.png)

## 性能问题

有些特效工作时消耗的资源很少，玩家几乎感觉不到画面的卡顿。但是有些特效的算法非常复杂，如果玩家的机器配置不够，使用起来非常影响游戏性。

我说的就是 SSAOFilter。

说到SSAO，先要从AO说起。AO是“环境光遮蔽”的缩写，SS是“屏幕空间”。SSAO合起来就是“屏幕空间环境光遮蔽”。

我先不解释这个复杂的术语是什么意思，请看下面两张图。这两张图分别是关闭、开启SSAO时的效果，请注意画面效果和刷新率（FPS）的区别。

![](/content/images/2017/06/No_AO.png)
![](/content/images/2017/06/SSAO.png)

在现实生活中，物体表面的光并不仅仅来自光源，还包括从很多其他物体表面多次反射的环境光。与直接光源相比，这些环境光称为间接光。当光线在房间中传播和反射时，有一些地方是不容易被照到的：角落、缝隙、褶皱等等。这导致了这些区域看起来比它们周围要暗一些。

这个现象被称为环境遮蔽(Ambient Occlusion, AO)，一般用于模拟这种区域变暗的方法是：对于每个面，测试它被其它面“阻挡”了多少。或者，跟踪光线的每一次传播、反射、折射，记录每个像素受到的光强度。

这些算法说起来简单，但是运算量相当大。3D游戏中所有的画面都是实时渲染出来的，每一帧的计算时间要足够短才行，否则玩家就会觉得卡。如果玩家的显卡性能不够，压根就开不起SSAO特效。即使显卡性能足够，开启SSAO也会显著降低画面的刷新率（FPS）。

不仅是SSAO，事实上开启任何特效都会在一定程度上降低FPS。比如：想在游戏中用100万个粒子模拟真实的星空。用天空盒不好吗？为什么一定要跟显卡作对？

**请慎用特效！请慎用特效！请慎用特效！重要的话要说三遍！！**

**使用任何特效，都应该给用户提供关闭特效的接口！**