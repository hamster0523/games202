## GAMES 202

## GAMES202_Lecture_01

### What is GAMES202 about? 

**Real-Time** High Quality Rendering(高质量实时渲染)

- 高质量事实渲染是什么？

  每秒钟生成30副图可以认为是实时，别的领域的实时有不用的定义

- `Interactivity`：画面很卡很卡，每秒小于30帧可以认为

Real-Time **High Quality** Rendering(高质量实时渲染)

- 指的是真实感，通常在为了保证"实时"，牺牲一定的质量，但任然要保证高质量
- 保证在物理上是真实的，近似正确的

### Rendering

![1](D:\BaiduNetdiskWorkspace\games202\1\1.png)

### Topic

![2](D:\BaiduNetdiskWorkspace\games202\1\2.png)

- 阴影实现
- 全局光照
- 基于物理的着色
- 实时光线追踪
- 散射介质，参与介质的渲染，图像空间的操作

![3](D:\BaiduNetdiskWorkspace\games202\1\3.png)

- 非真实感渲染，非真实感渲染存在非真实性，是反物理的

![4](D:\BaiduNetdiskWorkspace\games202\1\4.png)

- AA抗锯齿技术，DLSS

![5](D:\BaiduNetdiskWorkspace\games202\1\5.png)

202不会包含如下内容：

- 建模，引擎（不教工具使用方法，只会教这些工具背后的工程或科学原理，太偏应用不会提及）
- 离线渲染（卫星？难度极大）
- 神经渲染（Neural Rendering，做不到“实时”和“高质量”，但会提一些成功的应用，如DLSS）
- 怎么用OpenGL之类的着色语言，以及有关着色器的优化，或高性能计算（CUDA）

### How to study GAMES202

- 对渲染或图形学感兴趣（很重要）
- 图形学基础和一些高数知识储备
- 基本的GLSL基础，课程用WebGL，跨平台没有任何问题
- 不需要买显卡，不需要买书（rtr4可以作为参考），参考资料在BBS上都有
- 没必要用IDE，作业独立完成

## GAMES202_Lecture_02

### 渲染管线

![1](D:\BaiduNetdiskWorkspace\games202\2\1.png)

- `Vertex Processing`:obj文件存储的是空间中的三角形顶点，需要将空间中的点转换到屏幕坐标之下，他们的相对关系不会被改变
- 光栅化：（Rasterization)将原本连续表示的三角形离散化表示为屏幕上的一串像素。之后会使用一系列的AA技术，比如MSAA，TAA啊，抗锯齿
- 打散的过程中也涉及三角形远近的处理，那些靠近相机等，只有离相机最近的三角形当然才会被相机看见，在这里做的处理是维持一个靠近相机的最近的Z-buffer（逐像素的记录最近的深度）
- 确定了所有可见的像素之后，就可以对所有可见的像素进行着色，计算每个像素应该的颜色（也就是对每个像素进行shading)(简单的着色模型：Bling-phong着色模型)（简单的Bling-Phong模型是经验性的着色模型，包括了常数的环境光项，漫反射项，高光项）
- 最后完成光栅化渲染，好处是整套管线比较简单
- 整个一套渲染管线可以对直接光照处理的较好，但是对于全局的现象：比如说阴影，光线的多次弹射都处理的不好

![2](D:\BaiduNetdiskWorkspace\games202\2\2.png)

- 纹理贴图：在shading这一步当然也可以使用像素的坐标唯一的查询一个可能的纹理，将纹理贴图到当前像素上。（纹理也就是一张图）（应用三角形三个顶点所对应的纹理坐标u,v去唯一的插值三角形内任意一点的纹理坐标u,v使用的是三角形的重心坐标插值）（但也就存在了一个问题：纹理太小但是像素过大怎么办：使用纹理放大；像素过少但是纹理又太大了怎么办：使用MipMap)

### OpenGL

![3](D:\BaiduNetdiskWorkspace\games202\2\3.png)

使用OpenGL：类比于画画的过程

- 放置物体模型
- 放置画架子放在哪，相当于相机的位置
- 画架上放一块画布
- 在画布上花花
- E：如果还想画更多，贴另一块画布，继续画即可
- F：使用之前画过的画继续使用

将上述过程类比到OpenGL中即

![4](D:\BaiduNetdiskWorkspace\games202\2\4.png)

- A：放置物体
  - 放的是什么物体（Model specification)
  - 模型怎么摆放
  - VBO:GPU中的一块区域，由于存储模型，十分类似于obj文件，都是存储一堆顶点信息，连接方式。以一定的格式，组织了模型放在GPU的什么位置。
  - 使用OpenGL的API获得矩阵，比如平移矩阵，旋转矩阵，透视矩阵等
  
  ![5](D:\BaiduNetdiskWorkspace\games202\2\5.png)

- B：放置画布：类比于放置相机，就对应着视口变换。第二个更重要的是在其中建议一个OpenGL中的画架，就是建立一个framebuffer(意思就是用哪一个画架)

  - 只需要规定相机的一些属性即可，只需要调用这个函数，就可以得到视图变换的矩阵。
  - 视图变换的目的是将camera的位置变化到原点位置，右手边位置为x轴，头顶为y轴，相机面向的位置为-z轴，经典的右手坐标系。为了让看到的场景不产生形变，应该对所有的物体都进行相同的试图变换

  ![6](D:\BaiduNetdiskWorkspace\games202\2\6.png)

- C：使用画架，为了使用画架需要固定一个画布。**E：MultipleRenderTarget. 可以使用同一个画架可以画不同的图的：也就是只要指定了一个framebuffer,可以使用这个framebuffer去渲染不同的图的。指定一个framebuffer,可以让其输出不同的纹理**
  - 垂直同步，双重缓存（即将渲染的结果存在一个缓冲区中，在渲染结束确定没问题才将其在屏幕上显示），三重缓冲

![7](D:\BaiduNetdiskWorkspace\games202\2\7.png)

- 在画布上画画：类比于OpenGL中做shading。用到两种shading,一种是顶点着色器，一个是像素着色器（片段着色器）
  - 具体怎么做呢？每一个顶点都要做一个操作，这些操作都是什么，首先需要做MVP的映射（ModelView,Projection操作），做完之后可能需要一些插值，之后将插值信息送到片段着色器，片段着色器的输入就是顶点着色器的输出，即定点上的信息就会被插值到每一个像素上属性。两个重要的事情：顶点的变换运算以及顶点插值的信息需要通知片段着色器

![8](D:\BaiduNetdiskWorkspace\games202\2\8.png)

- 对于每个像素的处理：自己写像素着色器，设计对于每个像素上的计算。可以在片段着色器上进行深度测试
- 写shader整个最重要的过程就是怎么写顶点着色器，怎么写片段着色器

---

对于OpenGL的总结：

![9](D:\BaiduNetdiskWorkspace\games202\2\9.png)

在每一次的PASS（每一趟渲染）中告诉GPU我该做什么：

- 定义物体，相机的位置，以及各种变换矩阵是什么

- 我要选什么framebuffer，以及framebuffer的输出都是几个纹理呢得申明

- 告诉GPU怎么渲染，也就是写好顶点着色器以及片段着色器

- 一切都准备好就可以开始渲染了

- 并且可以多次渲染。之前渲染的结果也是一种纹理，可以用来多次渲染。比如之前讲到的shadowmap的做法就是两趟渲染：先看light可以看到什么，再看camera看到的物体是否可以被light看到

  ### Shadow Mapping（阴影映射）

  Shadow Mapping 是一种用于实时渲染阴影的技术。在3D图形中，正确地渲染阴影对于增强场景的真实感非常重要。Shadow Mapping 技术通过两个主要步骤来实现阴影的渲染：

  1. **从光源的视角渲染场景**：首先，场景从光源的视角进行渲染。在这一步，不是渲染出完整的场景图像，而是记录场景中各个表面距离光源的距离。这个过程生成了一个深度图或称为“Shadow Map”。每个像素在Shadow Map中存储的是从光源到达该像素对应位置最近表面的深度值。

  2. **从相机视角渲染场景**：然后，场景从相机的视角进行最终渲染。在这一步，对于场景中的每个点，通过Shadow Map来判断它是否处在阴影中。具体来说，**如果一个点从相机视角看是可见的，那么会检查这个点从光源视角看是否也是可见的**。**如果从光源视角看该点被其他物体遮挡（即该点的深度大于Shadow Map中存储的深度），那么这个点就在阴影中。**

  ### 多次渲染和纹理的使用

  在Shadow Mapping技术中，第一次渲染（从光源视角）产生的Shadow Map实际上就是一种特殊的纹理。在第二次渲染（从相机视角）时，这个纹理被用来判断场景中的各个点是否应该在阴影中。

  这种方法的优点是它相对简单且广泛适用于各种光源类型，但也有缺点，比如阴影边缘可能会显得不够平滑（锯齿状）

  它通过从光源和相机的视角进行两次渲染，并使用深度图作为纹理来判断物体是否应处于阴影中。

-----

### OpenGL Shading Lauguage(GLSL)

用于描述着色器不同的操作，而着色器的操作是用来描述顶点或者像素进行的各种可能的变化。着色语言是不面向对象的

![10](D:\BaiduNetdiskWorkspace\games202\2\10.png)

- 发明的各种着色语言最终是会被汇编到GPU的汇编指令，最终GPU也是执行各种汇编指令

所以怎么用呢

![11](D:\BaiduNetdiskWorkspace\games202\2\11.png)

- program可以认为是集合了所有写的shader
- Link 就是将各种着色器链接起来
- shader的源文件就是你写的一串字符串罢了

---

![12](D:\BaiduNetdiskWorkspace\games202\2\12.png)

- 一个Vertex Shader：顶点的变化需要做

  - `aVertexPosition`：表示的是顶点的属性，attribute表示的是顶点的属性
  - 相似的顶点的法线
  - 顶点的纹理坐标

- 顶点着色器还需要告诉片段着色器里要被插值的量。使用Varying关键字指明。

  - `vTextureCoord`代表不止需要顶点的纹理坐标，还需要三角形内部的纹理坐标
  - 法线也需要插值
  - varing关键字指定的关键字在像素着色器中一定也有一份完全相同的副本

- uniform就是全局变量，顶点着色器片段着色器都共享这些变量

  - 这里的全局变量就是
  - MVP变换的所需要的矩阵
  - mat4代表的就是一个四阶矩阵
- 这里的`gl_position`相当于顶点的位置经过变换后是什么，也就是要写入到这个变量中去
  

  ![13](D:\BaiduNetdiskWorkspace\games202\2\13.png)

- 可以看到Varying申明的关键字在片段着色器任然适用
- 用到了更多的全局变量
  - kd:就是漫反射系数kd
  - ks：就是高光项的不吸收系数
  - 光源位置
  - 相机位置
  - 光强
  - sample2D值得就是一个定义的纹理，即定义的二维纹理坐标
  - 比如在后续的`texture2D(usampler, vTextureCoord).rgb`就是查这个坐标下对于usampler的纹理,只要其中的三通道部分，后面的vec3(2.2)不太懂是啥意思
- 

![14](D:\BaiduNetdiskWorkspace\games202\2\14.png)

- `gl_FragColor`:最后向告诉OpenGL这个片段的颜色是什么，就需要往这个变量中写入颜色

---

### Debugging Shader

![15](D:\BaiduNetdiskWorkspace\games202\2\15.png)

为什么难调试：因为程序是编译到GPU上，也就比较难进行调试

### 渲染方程

![16](D:\BaiduNetdiskWorkspace\games202\2\16.png)

- 自发光
- 收到的所有光往观测方向的部分有多少
- cos衰减项，转换为于光线垂直的面
- BRDF表示材质的作用

在实时渲染中的微微区别

![17](D:\BaiduNetdiskWorkspace\games202\2\17.png)

映入一个Visibility项，会显式的考虑Visibility

![18](D:\BaiduNetdiskWorkspace\games202\2\18.png)

环境光项的考虑，又之前的Cubic的表示，也有右边的球体的表示。之后的八面体的表示

----

全局光照对于直接光照加间接光照

---

### 关于OpenGL的使用

接口

1. `gl.createBuffer()`
   - 这是一个WebGL方法，用于创建一个新的WebGLBuffer对象。这个对象可以存储多种类型的数据，比如顶点数据或颜色数据。在这个例子中，它用于存储顶点位置数据。
2. `gl.bindBuffer(gl.ARRAY_BUFFER, positionBuffer)`
   - `bindBuffer`方法将定义的缓冲区绑定到一个特定的缓冲区类型上。在这里，`gl.ARRAY_BUFFER`是绑定点，用于包含顶点属性，如顶点坐标、纹理坐标数据等。`positionBuffer`是之前创建的缓冲区对象。
3. `new Float32Array(positions)`
   - 这是创建一个32位浮点数组的JavaScript语法。`Float32Array`用于存储顶点数据或其他类型的数据，以便可以将其传输到WebGL。
4. `gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(positions), gl.STATIC_DRAW)`
   - `bufferData`方法用于向绑定到`gl.ARRAY_BUFFER`的缓冲区对象中传输数据。这里传输的是顶点数据（顶点坐标）。`gl.STATIC_DRAW`是一种提示，表明数据不会经常改变，主要用于绘制。
5. 

## GAMES202_Lecture_03

主题：实时的软阴影实现

`PCSS: Percentage-Closer Soft Shadows`

### Shadow Mapping简单回顾

渲染场景渲染两遍

①(第一个PASS）第一遍从light所在的地方看向场景，记录一个light可以看到的最浅的深度。记录为一个shadowmap

②（第二个PASS）从相机位置出发渲染场景，就可以检测场景中任何一点是否处在阴影中（即从这个点出发记录一个从该点到光源的距离，如果距离大于第一遍光源记录的最近位置，说明这个点处在阴影中）

![1](D:\BaiduNetdiskWorkspace\games202\3\1.png)

好处：一旦shadowmap生成了，就可以作为场景中的一个几何表示

坏处：

- 自遮挡现象
- 走样现象

![2](D:\BaiduNetdiskWorkspace\games202\3\2.png)

获得从光源出发的深度图

![3](D:\BaiduNetdiskWorkspace\games202\3\3.png)

第一个点可以被光源看到说明可见，第二个点明显处在阴影中

![4](D:\BaiduNetdiskWorkspace\games202\3\4.png)

其中从light视角看去，看过去渲染应该是这样的结果

![5](D:\BaiduNetdiskWorkspace\games202\3\5.png)

但其实是完全没有必要的，因为我们根本用不到从光源看过去的渲染结果，只需记录一个深度图即可

![6](D:\BaiduNetdiskWorkspace\games202\3\6.png)

shadowMapping怎么做：由于透视矫正的存在，两次的深度比较，要么都比较两个透视投影之后的深度，要么比较没有经过透视投影之前，在空间中点的距离即可。

> 什么是透视矫正？
>
> 在计算机图形学中，透视矫正通常涉及调整3D场景的投影，以使其在2D显示上更加自然或符合特定的视觉要求。透视矫正的目的是为了纠正由于透视投影造成的形状变形，特别是在模拟相机拍摄建筑或物体时。当场景中的直线在2D图像上显示为会聚于一点的线条时，透视矫正可以帮助恢复这些线条的平行性，使得图像看起来更接近观察者的实际视角或期望的视觉效果。这在模拟现实世界的相机操作或制作精确的技术图纸时尤为重要。

![7](D:\BaiduNetdiskWorkspace\games202\3\7.png)

自遮挡现象：传统shadowmap的缺点：地板上一圈一圈的纹路，由于数值精度造成的。首先生成shadowmap，shadowmap记录一个深度（也就是在一个像素内的深度都是一个常数）在shadowmap看来看到的都是离散的深度图片而不是连续的场景(比如上述图片中的红色小片就是因为被像素所离散化了），比如这里再第一次深度图中记录的深度是黑色线条所代表的深度。

在第二次渲染时，从camera出发（即蓝色线条），穿过的像素是上述黑色线所穿过的像素，记录的深度也就是黑色线的长度，将黑色线移动到蓝色虚线的长度，可以明显发现蓝色线明显长于黑色线，这个时候也就发生了记录深度小于计算深度了（虽然只差了一点点，差一点点也是一点点），因此就发生了其实根本没有发生遮挡，但是数值上他就是发生了遮挡的现象。 

----

**解决自遮挡**：添加一个偏差，即认为只有记录的深度与真实深度的差值的绝对值大于一个阈值（threshhold)时才认为这时候这个像素位于阴影中。

![8](D:\BaiduNetdiskWorkspace\games202\3\8.png)

并且这个阈值是可以动态变化的。也就是说：在光源垂直地板直射的时候，这个时候可以把这个阈值设小一些。在光源倾斜射向地板时，可以把这个阈值稍微设大一些。（只要计算地板微平面发现和光源法线之间的夹角大小即可）

----

引入BIAS后出现了第二个问题：脚部分的阴影断了：bias设置不好会造成某些阴影的丢失。没有真正解决这个问题的办法。。。（detach_shadow和self_occlusion之间是难以平衡的）

----

其他的学术界的方法：Second-depth shadow mapping:思想是每个像素不仅存最小的深度，也要存次小的深度

![9](D:\BaiduNetdiskWorkspace\games202\3\9.png)

这里灯是从上往下照的，先记录第一次碰到物体表面的深度（最小深度front faces),在记录第二次碰到物体的深度（second-depth),用两次的中间的深度进行后续的阴影比较。

实际中没有人会用：有一个要求，要求所有的物体都是water-tighte的（防水的）

> "Watertight" 在模型和3D打印领域中是一个重要的概念。一个模型被称为 "watertight" 或 "水密"，意味着它是完全封闭的，没有任何缺口或漏洞。这是指模型的所有边缘都完全连接，没有任何未连接的顶点或边缘。在三维模型中，这通常意味着每个面都完全由边缘封闭，且所有的面共同封闭一个体积。

----

---

Shadow mapping的另一个问题：走样问题

![10](D:\BaiduNetdiskWorkspace\games202\3\10.png)

动态分辨率的ShadowMap

CSM:cascade shadowmap

### The Math behind shadow mapping

![11](D:\BaiduNetdiskWorkspace\games202\3\11.png)

在RTR中不关心不等，将不等式看作近似相等

![12](D:\BaiduNetdiskWorkspace\games202\3\12.png)

将函数乘积的积分拆开算，什么时候可以这样认为，分母是一个归一化常数（为了使得不等式前面g(x)的积分结果的数量级和后面g(x)积分的数量级大致相等）。什么时候可以认为他准确：当积分域足够的小时候，或者，g(x)足够的光滑的时候，可以认为左右两遍的积分大致相等。

利用得到的约等式进行改写渲染方程

![13](D:\BaiduNetdiskWorkspace\games202\3\13.png)

右边没有红框框的部分就是Shading的部分（即原本的渲染方程的结果直接Shading的结果）左边红框里的就是Visibility项。就是最基本的思路：只需要先做shading，后面把Visibility乘上去就行。这样一个积分在积分范围足够小的时候式准确的（一个点光源，方向光源），当g(x)足够光滑的时候，g代表的是光照，即BRDF

![14](D:\BaiduNetdiskWorkspace\games202\3\14.png)

什么时候g(x)是smooth的？

- 光照是smooth的，即类似于面光源，即$L_i(p,w_i)$变化不大时候
- BRDF是diffuse的时候可以认为他是变化比较小的，是smooth的（也就是说对于Glossy的BRDF是不太适合用于做shadowmapping的）

-----

### PCSS-做实时的软阴影

![15](D:\BaiduNetdiskWorkspace\games202\3\15.png)

为什么会有软阴影存在：生活中大多数光源都是面光源，所以软阴影比较明显。

umbra本影, penumbra半影

----

PCF：Percentage Closer Filtering

![16](D:\BaiduNetdiskWorkspace\games202\3\16.png)

最初是用来做抗锯齿的

Fileter就等于模糊

需要注意的是：PCF的filtering并不是指最后对生成的阴影做filtering,而是在做阴影判断的时候做filtering(类似于怎么做反走样，不能是先得到走样的结果，在走样的结果上进行处理（模糊)；同时也不是filter那个shadowmap(因为filtershadowmap后，对于后续的深度判断也是非零即一的（要么在阴影中要么不在阴影中)

做法：

![17](D:\BaiduNetdiskWorkspace\games202\3\17.png)

做法：之前在做任何一个点在不在阴影里的判断（原本的做法是将这个点连向light,在与ShadowMap上这一点深度进行比较，只做了一次比较），PCF的区别是什么：投影到light之后不是找对应的唯一一个像素（p点），而是找周围一圈的像素，把周围的每一个深度都是实际的深度进行比较（卷积思想，也有类似于不做点光源而是对于区域光源的判断）。做完比较之后将所有得到的非零即一的结果平均起来得到可视度visibility（visibility是一个处于零到一之间的数）

![18](D:\BaiduNetdiskWorkspace\games202\3\18.png)

有什么缺点：之前只比较一次的时候，一个空间中的点shanding point只需要查一次ShadowMap对应的深度值。而现在如果是7*7的网格，则需要查49次深度。会很慢！

![19](D:\BaiduNetdiskWorkspace\games202\3\19.png)

根据这个窗口查询的大小，窗口太小可能会出现锐利的阴影类似于硬阴影。而窗口太大会出现很模糊的阴影。

！！！先生成硬阴影，能不能动态的决定n的大小（不同的位置有不同的filter,把他变成软阴影，应该给他多![20](D:\BaiduNetdiskWorkspace\games202\3\20.png)

大的filter呢？

笔尖是硬阴影（近/远：阴影所在的接收物到阴影的投射物之间的距离大小：即笔尖（阴影投射物）离阴影的接受物（纸面）之间距离很近所以阴影hard,反之）

所以想实现软阴影的效果，就需要在硬阴影不同位置上做不同大小的filtering。filter的大小应该与一个距离有关，距离叫blocker distance。即阴影投射物（笔）和阴影接受物（纸）之间的距离。![21](D:\BaiduNetdiskWorkspace\games202\3\21.png)

$w_{penumbra}$：阴影软的程度半影范围，w越大越软

- 和light大小有关
- 和blokcer与接受物之间相对距离有关
- 根据相似三角形定理

这里有一个问题：现实中的Blocker不一定是一个点，完全有可能是一个球之类物体，那么$d_{blocker}$怎么算？用一个物体（球）的average blocker depth作为$d_{blocker}$的代替。

----

对于面光源第一步怎么生成深度图shadowmap？把面光源看作一个点光源，即把camera放在面光源的中间（中点）去看向场景。

---

PCSS：

第一步：和shadowmap的第一步相同：从light所在的地方看向场景（即上面说的），记录一个light可以看到的最浅的深度。记录为一个shadowmap

第二步：计算$d_{blocker}$:从相机位置（camera_pos)出发，就可以检测场景中任何一点是否处在阴影中（即从这个点出发记录一个从该点到光源的距离，如果距离大于第一遍光源记录的最近位置，说明这个点处在阴影中）,利用这个距离减去第一次记录的深度不就是每一个像素的$d_{blocker}$,之后在将一个物体计算他平均的blocker的深度（这里有个问题：即在多大的范围内去做blocker的平均？）

第三步：统计计算的d_blocker计算w_penumbra

第四步：运用计算的w_pemumbra去卷积每个像素周围的阴影值

![22](D:\BaiduNetdiskWorkspace\games202\3\22.png)

怎么觉得blocker计算的深度的卷积核大小：超参数。或者是

![23](D:\BaiduNetdiskWorkspace\games202\3\23.png)

light离shading point距离比较近，说明这个blocker卷积核应该要比较大。反之

---

缺点：很慢。。

