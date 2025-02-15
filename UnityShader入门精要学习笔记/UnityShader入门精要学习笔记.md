# GAMES101学习笔记



# 前言

这里的笔记原先是Excel形态，被复制到MarkDown中。

创建时间是2022年3月20日，最后一次修订是2022年8月30日。

观看体验不太好是真的，因为早期没有什么经验。。。

建议作为查找用的资料使用，想要知道什么，试试在这里面查找一下，说不定会有收获。

我之后会不定期继续修订这个笔记。

感谢闫令琪老师，真心的。🙏🏻🙏🏻



# 内容

| 问题                                                         | 回答                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 什么是Shader                                                 | 着色器。是渲染流程中的一环，输入是摄像机、光源、要渲染的图元等，输出是屏幕上的显示图像 |
| 粗略地看渲染流水线                                           | 分为应用阶段、几何阶段和光栅化阶段                           |
| 应用阶段                                                     | 由CPU主导，负责处理场景中的数据，最终会输出点线面等渲染图元  |
| 几何阶段                                                     | 由GPU主导，负责处理所有要绘制的几何相关的事情，进行逐顶点、逐多边形的操作。最终输出屏幕空间的二维顶点坐标、每个顶点的深度值、着色等信息。 |
| 光栅化阶段                                                   | 由GPU主导，负责渲染出最终呈现的图像。期间会处理上一阶段传入的顶点颜色、纹理等信息，进行逐像素的处理。在Unity中，还会在这里处理背面剔除。 |
| 应用阶段详细流程                                             | 1.CPU把数据（mesh、材质、贴图等等）从硬盘加载到内存，然后又从内存加载到显存，这么做是因为GPU对显存的访问很快，不直接访问内存是因为大多是显卡没有那个权限。      2.设置渲染状态。载入的数据没有关联，设置渲染状态就是指定Mesh和这些材质与贴图之间的关系，用来告诉GPU这个Mesh要如何渲染，用哪个Shader（材质）。     3.Draw Call。CPU处理完前两步的信息后，会用DrawCall叫GPU开始渲染，函数只有一个参数，就是一个需要被渲染的图元列表。 |
| 几何和光栅阶段详细流程                                       | 几何阶段：（收入DrawCall传入的图元列表）->  顶点着色器 -> 曲面细分着色器 -> 几何着色器 ->   裁剪 -> 屏幕映射 ->     光栅化阶段： （收入屏幕映射后拿到的每个图元的屏幕坐标系的信息） -> 三角形设置 -> 三角形遍历 -> 片元着色器 ->  逐片元操作 -> （呈现屏幕图像） |
| 顶点着色器                                                   | 完全可编程，用于顶点的空间变换、改变顶点的颜色等。顶点着色器的处理单位是顶点，也就是说每个顶点都要调用一次顶点着色器，并且，顶点之间相互独立，无法得知这个顶点以外的信息。顶点着色器最基本的工作是把顶点坐标从模型空间转换到齐次裁剪空间，对应的语句是：O.Pos  = mul(UNITY_MVP, v.position);最终在裁剪之后经过硬件处理，会得到归一化的设备坐标（NDC）。 |
| 曲面细分着色器                                               | 可选的，可有可无的。用于细分图元。详细的见百人计划笔记。     |
| 几何着色器                                                   | 可有可无的。执行逐图元操作，或者生成更多的图元。详细见百人计划笔记。 |
| 裁剪                                                         | 可配置不可编程的。目的是将不在摄像机范围内的顶点删除，并剔除某些三角面。对于部分在屏幕内的图元而言，会剔除在屏幕外的顶点，而使用新的顶点代替，这些是自动完成的。 |
| 屏幕映射                                                     | 不可配置，不可编程。作用是把每个图元的坐标（此时应是NDC），转换到屏幕坐标系中（仅x、y的值），而z值不会改变。这样的x、y、z组成了新的坐标系，叫做窗口坐标系。 |
| 三角形设置                                                   | 固定函数，作用是计算输入的顶点的信息组成的三角面，计算三角形边界的信息，为下一个阶段做准备。 |
| 三角形遍历（扫描变换）                                       | 固定函数，作用是计算上一阶段传来的三角形边界的信息。会遍历屏幕上的每一个像素，看它是否被某一个三角面覆盖了，若是，则会生成一个片元，用来保存后续计算需要的信息。期间也会进行插值，计算片元的深度信息。最终会输出一个片元序列。 |
| 片元着色器                                                   | 片元着色器是可编程的。片元着色器接收到三角形遍历插值出来的片元序列，然后对每一个片元执行一次。期间会根据片元中的纹理坐标等信息，得到一个或多个颜色值，并把计算后的最终颜色值保存到片元中。和顶点着色器一样，片元是相互独立的，无法访问和改变到其他的片元。 |
| 逐片元操作                                                   | 逐片元操作是高度可配置的。     将片元的信息和缓冲区的信息进行比较或合并，得到最终的颜色值，并保存到缓冲区中。      首先会确定该片元的可见性，会通过模板测试和深度测试来确定其可见性，通过测试后，若半透明则会通过混合来确定像素最后的颜色值，否则直接确定计算出的颜色为这个屏幕像素的颜色。 |
| 模板测试                                                     | 如果开启了模板测试，则会经过这一流程。这是一个高度自定义的测试，会对比该片元的参考值和模板的参考值，你来决定如何算通过了该测试。若没通过，则会舍弃该片元，若通过会进入深度测试。 |
| 深度测试                                                     | 顾名思义，东西被挡住，就会看不见。测试会比较该片元的深度和深度缓冲区中的深度值，若通过，则会保留片元进入混合阶段，否则说明被挡住，会被舍弃。深度测试是可以配置开启或者关闭的，你还可以关闭或开启深度写入。 |
| 混合                                                         | 与PS的混合模式很像，你可以决定片元之间如何混合。若这个被渲染的物体是完全不透明的，则可以关闭混合。 |
| 双重缓冲                                                     | 这样的渲染像画画一样，可能先画出后面的远山，再画前景。如果你眼神好，可能会看到还没画好的样子。为了解决这个问题，就有了双重缓冲，这样的渲染发生在后置缓冲，当后置渲染好后，GPU会交换后置缓冲和前置缓冲的内容，你看到的画面就是渲好的了。     我猜测，之所以Unity渲染的帧调试器中，第一步都是Clear，就是因为交换前后置缓冲，刚拿到的缓冲保存了上一帧的东西，所以需要Clear。 |
| 减少DrawCall                                                 | DrawCall多了会造成CPU的负担，导致渲染速度下降。可以使用批处理，将静态的、同一渲染状态的物体合为一次DrawCall。详细的方法见该笔记的性能优化部分。 |
| ShaderLab                                                    | UnityShader专用的语言                                        |
| UnityShader的基本结构                                        | 1.属性：Properties     2.SubShader     3.Fallback            |
| Properties                                                   | 和游戏脚本里的属性类似，定义方法是：     变量名（“在面板中显示的名字”， 变量类型） = 默认值 |
| SubShader                                                    | 每个UnityShader可以有多个SubShader，这是用来看显卡的，比如你的显卡太拉了，用不了第一个SubShader，就会自动调用下一个。都用不了就会用FallBack。一个SubShader就是一套解决方案，但是并不是严格割裂的关系，比如我自定的一套SubShader中没有阴影投射器的方法，但是我开启了阴影，无法在这个SubShader中透射就会去下一个SubShader找，都找不到就会用FallBack的阴影投射器，而不是发现这个SubShader没有就直接用下一个SubShader或者FallBack。这就是不要乱写FallBack的原因。     SubShader中包含Tags（可选）、RenderSetUp（可选）、Pass{} |
| RenderSetUp                                                  | 对应渲染流程中的应用阶段中的设置渲染状态，比如告诉显卡是否开启深度测试，是否开启深度写入，是否开启混合等等。      当你在SubShader中设置渲染状态时，它会被应用到所有的Pass，每次Pass对应一次完整的渲染。如果你想分Pass设置渲染状态，就在Pass中设置RenderSetUp。 |
| Tags（SubShader）                                            | 是可选的，功能上与RenderSetUp类似，你给这个SubShader打上标签，就可以载入一些预设，同时拿到一些变量，这样可以更方便的满足需求，类似using。     定义方法：Tags{ “标签名” = “值”  “标签名2” = “值2”} |
| Pass                                                         | Pass中包含Name，Tags和RenderSetUp。功能和SubShader中的基本一致。Name使得Pass可以在不同的Shader中相互调用，调用时只需要UsePass  “Shader名/Pass名”，但是要注意Pass名会被换成全大写的 |
| FallBack                                                     | 当显卡所有的SubShader都跑不了，就会跑这个，与Pass没有什么特别大的区别。FallBack是可以关闭的，但最好还是不要关闭，因为内置的FallBack中有阴影相关的内容。 |
| 表面着色器                                                   | 是对顶点和片元着色器的抽象，是更易用但是相较局限的着色器，需要考虑性能问题，当光源多时使用更好。     其代码不写在Pass中，而是与Pass平行的一个存在，写在SubShader的CGPROGRAM和ENDCG中。 |
| CGPROGRAM和ENDCG                                             | CGPROGRAM和ENDCG表示你要在ShaderLab中插入Cg/HLSL语句，与一个花括号的功能基本一致 |
| 顶点、片元着色器的写法                                       | 写在Pass的CGPROGRAM和ENDCG中，灵活性最高。                   |
| 判断左右手坐标系                                             | 靠内的3根手指分别代表xyz，符合左手的就是左手坐标系。Unity中，模型空间和世界空间用的是左手坐标系，但观察空间是反的，是右手坐标系。 |
| 左右手法则判断旋转正方向                                     | 比一个点赞的手势，大拇指朝该轴的正方向，其余四指的方向就是旋转的正方向。左手坐标系用左手，右手坐标系用右手。 |
| 摄像机的观察空间和模型空间                                   | 模型空间下，默认摄像机的正前方是Z的正方向，而观察空间下前方是Z的负方向。 |
| 向量的点积（内积）                                           | 计算方法是每一个维相乘再相加，或者二者的模长相乘再乘Cos二者的夹角。      几何意义是显示二者的投影长度，可以用来计算两向量的夹角关系。比如投影大于0，说明两者夹角小于90这样。向量与自己点积，得到自己的模长的平方，通常用来比较，开方十分的消耗性能。还可以用来求两个单位向量的夹角，夹角  = arcCos（a点乘b） ab都是单位向量 |
| 向量的叉积（外积）                                           | 计算的结果是一个向量，计算方法是：向量的每一维都等于其他维交错相乘再相减。比如a（x，y，z）、b（p，q，l），则a  X b = （yl - zq， xl - zp， xq - yp）。     计算符合负的交换律，axb = - bxa。     a、b叉乘的模长为a模长乘b模长再乘二者夹角的Sin。     新向量的模长等于原二向量围成的四边形的面积。     新向量的方向由左右手决定，左手坐标系用左手。手心对着a的方向，四指向b的方向弯曲，拇指方向就是新向量的方向。     物理意义是算出一个垂直于原二向量的新向量。 |
| 矩阵相乘                                                     | 矩阵相乘得到一个新的矩阵。比如a（4X3）  X   b（3X6），会得到一个新的c（4X6）。如果a的列数不等于b的行数，则不可以相乘。新矩阵的每一项，比如c（1，1）等于a的第一行的每一项乘以b的第一列的每一项再相加，而c（1，2）等于a的第一行的每一项乘以b的第二列的每一项再相加。     矩阵乘法不满足交换律，但是满足结合律。 |
| 矩阵转置                                                     | 一种运算，矩阵的第一行是新矩阵的第一列，第二行是新矩阵第二列。     性质1：（MT）T = M     性质2：（AB）T = ATBT |
| 逆矩阵                                                       | 只有方阵有可能可逆（非奇异），当矩阵的行列式不为零时，矩阵可逆。     如果说矩阵代变了变换，那么逆矩阵就代变逆变换。有些变换矩阵是可逆的，比如说旋转矩阵。     对于方阵M，如果M-1可以满足 MM-1 == M-1M == I（单位矩阵），那么可以说M-1是M的逆矩阵     性质1：（M-1）-1 = M     性质2：I-1 = I（单位矩阵）     性质3：（MT）-1 = （M-1）T     性质4：（AB）-1 = B-1A-1 |
| 正交矩阵                                                     | 如果MMT  = MTM = I，则可以说M是正交矩阵。     正交矩阵的MT = M-1.     判断方法：一般不用上述定义判断，性能开销大。可以看矩阵的每一行和每一列，如果每一行都是单位向量，并且每一行的向量都互相垂直，则可以说是正交矩阵。 |
| unity矩阵变换规范                                            | 一般把目标变换向量（列矩阵）放在最右边。矩阵的相乘是右乘。     比如目标向量v和变换矩阵ABC，规范是：     CBAv；代表先对v使用A变换，然后是B变换，最后是C。CBAv = （C（B（Av）））。 |
| 矩阵即是变换                                                 | 在Unity中，矩阵即是变换，如平移、旋转、缩放等。一个列向量乘以对应的变换矩阵，得到的就是变换后的位置。 |
| 线性变换                                                     | 如果满足f（x）  + f（y） = f（x + y）且kf（x） =  f（kx），那么这个变换就是线性变换。缩放、旋转都是线性变换。如果是三维坐标，那么线性变换只需要3X3的矩阵就能表示所有变换的可能。 |
| 仿射变换                                                     | 因为非常常用的变换：平移并不是线性变换，为了解决平移的问题，提出了仿射变换，即把空间拉到4X4的矩阵中，这个4X4的坐标称为齐次坐标。这个4X4下的空间称为齐次坐标空间。仿射变换就是合并线性变换和平移变换的变换。 |
| 如何构建齐次坐标                                             | 齐次坐标就是为了矩阵计算方便把向量拔高一维的结果，那多出来的一维的值是多少呢？      对于表示位置的向量而言，是1，这样可以使得三种基本变换都能作用到点上。但是对于表示方向的向量而言，是0，这样会导致平移无法作用于方向，而方向本就不应该受到平移的影响。 |
| 用矩阵表示平移                                               | 对于一个三维向量，其变换矩阵是4X4的，这个矩阵以一个4X4的单位矩阵为基础，第四列的前三行的值，就表示向量的三维的变化值。 |
| 用矩阵表示缩放                                               | 类似上一条，以4X4的单位矩阵为基础，对角线的前三项分别表示点沿xyz方向的缩放程度。三轴的缩放系数若相同，则是统一缩放，否则是非统一缩放。 |
| 用矩阵表示旋转                                               | 与其他两个不太一样。还是以一个4X4的单位矩阵为基础，但是只需要修改左上角的3X3.     若是绕x轴旋转，则是3X3中，右下角c，-s；s，c；的2X2的子矩阵。     若是绕y轴旋转，则是3X3中，四个角的c， s；-s，c；     若是绕z轴旋转，则是3X3中，左上角c，-s；s，c； |
| 复合变换                                                     | 很容易，一个矩阵代表一个变换，那么，给向量多乘几个变换矩阵就是复合变换了。注意，为了符合规范，一般最先缩放、再旋转、再平移。     表现在算式上就是M平移M旋转M缩放v     注意，这个是右乘的，也就是说从右往左读。 |
| 关于旋转的顺序                                               | 有时我要你三个轴都要旋转，但是顺序怎么定呢？Unity中是zxy，写作：MZMXMY，但是从右乘来说应该是yxz。 |
| 一般来说的坐标空间转换流程                                   | 模型空间  --模型变换--》 世界空间 --观察变换--》 观察空间 --投影变换--》 裁剪空间 --底层逻辑自动计算--》NDC/归一化的设备坐标系  --屏幕映射--》 屏幕空间      前三个变换通常在顶点着色器中被串联（即相乘）成一个矩阵，称为MVP矩阵。     这些坐标系中，只有观察空间是右手坐标系（unity）。 |
| 一个真正的UnityShader                                        | 首先Pass中要有一个编译命令，然后主要是Pass中的顶点和片元着色器。     编译命令：#pragma vertex vert     #pragma fragment frag     第一个词是不变的，表示编译。     第二个词表示你要编译哪一种着色器。     第三个词是你自己定义的这个着色器函数的名字，一般就取这两个名字就好了。 |
| 一个顶点片元着色器函数的格式                                 | 返回类型  函数名（参数类型 参数名 ： Cg/HLSL语义） ： 返回的Cg/HLSL语义     比如：float4 vert (float4 v : POSITION) : SV_POSITION{       return mul(UNITY_MATRIX_MVP,  v);     }     这是非常自由的，很多的类型都是极其自由的，也可以自定义结构体。 |
| 想要获取更多顶点的信息                                       | 我要写顶点着色器的话，光知道位置怎么够呢。其实在应用阶段顶点中存了很多信息，位置啦、法线啦等等，怎么获得呢？     可以自己定义一个结构体a2v（application to  vert），里面可以定义自己想要的顶点的信息，在创建结构体实例的时候，就会根据语义拿到顶点的信息了。     Struct a2v     {       float4 vertex : POSITION;       float3 Normal : NORMAL;       …...     }     这个怎么用到顶点着色器的函数里面呢？很简单，你把那个函数的参数改成这个a2v  类型的实例就可以了，这样的话，在遍历顶点，给每一个顶点执行Shader的时候，就会把语义中的数据存到a2v的对象中，再传到vert的函数中。 |
| 如何在顶点和片元着色器间交流数据？                           | 使用结构体就可以了。     你可以再建一个结构体叫v2f，表示顶点到片元，里面可以包含一些后面可能会用到的信息，比如法线啊、顶点颜色之类的。     struct v2f     {       float4 pos : SV_POSITION;       float3 normal : NORMAL;     }     这些信息在vert函数中指定。然后返回值为v2f，就能把这些信息一并发到下一个环节。      其实顶点和片元远远不是一一对应的，那如何匹配v2f的对象呢？比如我只有3个顶点，那不是只有3个v2f吗？其他的片元取哪一个v2f呢？是这样的，在三角形遍历的时候，也就是光栅化的时候，会给三角面片所覆盖的区域的片元根据三个顶点的值进行插值，这样就能保证每一个片元都有一个v2f了。     使用v2f会不会影响到Cg/HLSL的内部呢？我认为是不会的，在结构体中不是给自己定的变量和语义进行了匹配吗，这应该是一种桥梁的关系，这样的结构体应该只是为了人看着方便，并且可以一次返回多个值设计的。 |
| 使用Properties                                               | 你可以暴露Shader的参数，比如可以很轻松地修改颜色。      Properties是在Shader的层面中定义的，若要在Pass中使用这个Properties，则需要再在Pass中定义一个类型和名称都和这个Properties一模一样的变量才行，Unity会自动完成这个链接工作。      需要注意一个相对的类型的问题。比如Color，它对于Unity来说类型就是Color，你在Shader中定义Color的类型就是Color，然而在Pass中，Color被指定为fixed4  类型的变量，也就是4维的一个向量，表示RGBA。 |
| 使用内置文件                                                 | 像其他编程语言一样，你可以引用一些已有的文件，这些文件通常包含有用的结构体、函数等。     如：     #include "UnityCG.cginc" |
| 使用对的数据类型                                             | float、half、fixed都是浮点类型，但是精度逐个递减，性能消耗也逐个递减。对于移动平台来说，要尽量选择低精度的浮点来保证性能。其他平台最好也这样做。一般来说，颜色和单位向量用最低精度的就可以了。 |
| 标准光照模型                                                 | 看到一个物体必然是捕获到了光，这光可以分为四个部分：     1.自发光：作为光源才有。     2.高光反射：字面意思。     3.漫反射：物体受到光源照射后，反射出来的不同方向的光。     4. 环境光：表示其他所有物体的间接光照。其他物体受到光照后也会反射出光线，这些光线对观察物体的影响就是环境光。 |
| 游戏引擎中的环境光和自发光                                   | 通常是一个定值                                               |
| 漫反射的计算                                                 | 根据兰伯特定律计算出来的。：反射光线的强度与表面法线和光源方向之间的夹角的余弦值成正比。（夹角的余弦值不能小于0，也就是说照到背面是无效的）     分析一下，就是说正着照的地方最亮，随着光线和法线的夹角增大，漫反射会越来越弱最后到0。     公式是：Cdiffuse = （Clight * Mdiffuse）max（0，n*l）；     Mdiffuse指的是材质的漫反射信息，n指的是法线，l指的是入射光，默认n和l是单位向量，所以点乘的值就是Cos夹角值。 |
| 高光的计算/Phong高光模型                                     | 根据生活经验，高光是会随着视角变化的，所以计算高光也要使用视角方向。     高光是根据Phong模型算出来的，公式是：     Cspecular = （Clight *Mspecular）max（0，v*r）^Mgloss     Mspecular代表材质的高光反射颜色，v是观察方向，r是出射方向，Mgloss是材质的光泽度。     分析一下，和漫反射一样，背面是不计算高光的。V*r是视角和出射光线的夹角的Cos值，是小于1的，所以给其乘方的Mgloss越大，高光的范围就越小。 |
| 高光计算的另一种模型（Blinn）                                | Blinn模型是一种稍微简单一点的模型，公式是：     Cspecular = （Clight *Mspecular）max（0，n*h）^Mgloss     n是反射表面的法线，h是观察方向和光源的中线，也叫半程向量。这样就不用计算反射光线，在光源与模型较远时计算比Phong快。 |
| Phong着色/Phong Shading                                      | 是逐片元着色，会对每一个片元使用上面的公式来计算光照，计算量大但是效果好。这里指的是着色频率，请和经验模型区分开来。 |
| 高洛德着色（Gouraud）                                        | 是逐顶点着色，再通过三角面的每一个顶点对其所覆盖的片元插值而来。计算量小但计算高光会出问题，而且可能出现明显棱角。 |
| 标准光照模型的别称                                           | 叫Phone光照模型，后续因为融入了Blinn计算高光，则又被叫做Blinn-Phong光照模型。 |
| 半兰伯特光照模型                                             | 通过上面对漫反射的计算，得知背面是全黑的，如果没有环境光，那就是完全的黑暗。但这是符合物理的，虽然不好看。     为了解决这个问题，有了半兰伯特模型，公式是：     Cdiffuse = （Clight * Mdiffuse）（a（n*l） + b）；     改变不大，a、b通常是0.5，这样就把后半部分从-1~1映射到了0~1，可以理解为给本不受漫反射的地方给了一个最低的保证补贴，而亮的地方也不那么亮了。 |
| BRDF（Bidirectional Reflectance Disitribution Function）（双向反射分布函数） | 既然是一个函数，那么知道它的输入输出就可以了。对于模型表面的一个点，BRDF的输入是入射光线的辐照度和方向，输出是出射光线某个方向上的光照能量分布。就是说BRDF定义了光如何与表面作用，所以BRDF即是材质的本质。 |
| 在Shader中使用纹理                                           | 1.要使用纹理，自然要先在Shader的Properties中先定义一张纹理：     _MainTex（“MainTex”，2D） = “white”{}      2.然后的Pass中要定义一个名字和类型都一样的变量，才可以使用它。但是对于纹理，还有一些区别。你不止需要声明贴图(采样器)，还需要声明纹理的缩放和偏移。     sampler2D _MainTex;     float4 _MainTex_ST;(ST代表Scale和Transform的变化，即缩放和偏移)（这一条是必须的，没有会报错）     3.要使用纹理，自然从a2v阶段就要开始拿到UV的信息     a2v中定义：     float4 texcoord : TEXCOORD0(语义：主纹理坐标、UV0)     4.片元着色器需要uv坐标来拿到颜色值，所以需要从v2f中传入：     v2f中：     float2 uv : TEXTCOORD2;     5.因为你设置了偏移，所有在顶点着色器中就需要把它的偏移计算好，以便三角形遍历的时候插值,vert中：     o.uv = v.texcoord.xy * _MainText_ST.xy + _MainText.zw;(乘上缩放再加上偏移)     或者：o.uv = TRANSFORM_TEX(v.texcoord,_MainTex);     6.然后就可以给片元拿到对应的纹理上的值了，frag中     fixed3 albedo = tex2D (_MainTex, i.uv).rgb * _Color.rgb;     这是把物体本来的颜色和纹理上取到的颜色混合了。这个albedo会用在接下来计算环境光和漫反射的时候用。 |
| 过滤模式/滤波                                                | 按照很直球的理解来讲，uv坐标对应纹理的哪一个像素，这个片元的颜色就是多少。但是如果纹理不够清晰，涉及纹理的拉伸，这么做可能导致渲染出来的图像很硬，锯齿严重。于是有了过滤模式这种东西。上述这种就是点过滤模式，往下还有双线性和三线性，他们不仅会对uv坐标所指的那一个像素采样，还会根据信息分析周围像素对它的影响，达到一个软化的效果，虽然会消耗性能，但效果是好的。（原理移步101） |
| mipmapping技术（多级渐远技术）                               | 像一个金字塔一样，提前将缩放好的纹理保存下来，这样在渲染较远的东西的时候，就可以直接拿到缩放后的纹理了，而不用再临时去计算过滤，是一种空间换时间的做法，大约会多占33%的空间。（原理移步101） |
| 高度纹理                                                     | 高度纹理比较容易，它的灰度值就代表了向面法线突出的高度是多少。一般越白的地方越突出。高度贴图对性能的开销有点大，因为片元的法线需要重新计算。对于高度图的合理应用，移步百人计划笔记。 |
| 法线纹理                                                     | rgb通道分别记录了xyz在切线空间下的偏移量。在顶点着色器中，我们先计算各种需要计算的元素转到切线空间下，然后再在片元着色器中，通过对法线贴图采样来更新片元的法线，再把新的法线应用到光照中去即可。存在切线空间的原因，移步百人计划笔记。 |
| 渐变纹理                                                     | 有点像三渲二的颜色分段条。里面通常有好几段颜色，当通过对漫反射的计算在某范围内时，则均取到同一块的颜色。是的，这样的话，uv就没用了，因为不会用uv来对颜色采样，而是用漫反射过程中计算出的值来采样。 |
| 遮罩纹理                                                     | 原理很简单，遮罩纹理的像素的rgba的任意一维都可以保存信息，你可以保存高光的系数、漫反射的系数等等，只要在计算相应的光照时，附加上遮罩纹理的系数影响就可以了。 |
| 透明度测试                                                   | 一般不用，就是利用透明度测试，当不通过透明度测试时，就把这个片元删掉。 |
| 透明度混合                                                   | 更接近真实的半透明效果，需要在Pass中开启混合并关闭深度写入，还需要你设置一下渲染队列，一般来说半透明物体渲染在普通物体之后的TransParent（这个单词的意思就是半透明）队列中 |
| 开启深度写入的透明度混合                                     | 关闭深度写入的话，如果渲染物体有复杂的缠绕等结构，会导致渲染出错误的结果，所以有时需要开启深度写入。具体的做法是使用两个Pass，第一个只写入被渲染物体的深度，但是不输出颜色，这样就能提前分析好这个物体的内部的遮挡关系，下一个Pass正常渲染半透明混合即可。缺点是用了两个Pass，效率不高。 |
| ClolorMask                                                   | Pass中的渲染命令，可以表示该Pass输出的颜色。比如我不想输出颜色，则命令ColorMask0，我只想输出R，则ColorMask  R。 |
| 混合类型                                                     | 如PS中一样，颜色的混合有很多种类型，正片叠底啊、变亮啊之类的。这些在Pass中开启混合的时候都可以设置，具体参数查表。 |
| 双面渲染                                                     | 有时候物体的内部或者说另一面可以可见的，但是在光栅化阶段会剔除背面，所以为了准确的渲染物体，则需要关闭背面剔除。操作很简单，在Pass中写Cull  off就可以了。还可以更自由的选择剔除背面、或者剔除正面等等。 |
| 双面渲染半透明物体                                           | 半透明物体需要关闭深度写入，这样就会造成双面渲染的混乱。所以需要两个Pass，先渲染出背面，再渲染出正面，即可以得到正确的渲染结果。 |
| 渲染路径                                                     | 目前有：前向渲染路径，延迟~和顶点照明~三种，顶点照明~目前已经几乎不用了。一个项目中一般只用一种~。使用了正确的~才能从Unity拿到想要的信息。 |
| 前向渲染                                                     | 最常用的一种方式。大概来说就是，每个物体都会对其有影响的光源进行一次Pass。这样的Pass有三种方式，一种是逐顶点处理，一种是逐像素处理，一种是球谐函数处理，unity中可以设置光源的类型，如果光源是重要的，则会使用逐像素处理，否则使用逐顶点或者球谐函数。      对于多个光照而言，一个Shader中需要使用多个Pass，一个叫BasePass，其他的是AdditionalPass，BassPass用来计算光照纹理、环境光、自发光、阴影等，会计算一个逐像素光源和其他所有逐顶点和球谐函数的光源。AdditionalPass会对其他每一个影响该物体的逐像素光源进行计算。要标记这些Pass，就在Pass中用#pragma  ……来标记，还可以进行很多额外的配置。 |
| 逐顶点渲染路径                                               | 即逐顶点地计算光照，效果差但是效率高。现在已经几乎不用。     |
| 延迟渲染路径                                                 | 依赖一种叫做G缓冲（G-Buffer、几何体缓冲）的渲染方式，可以解决光源过多时，会产生很多Pass的问题。原理是：Shader中内含两个Pass，第一个用来渲染世界空间法线、观察方向、世界空间位置等等信息，保存到好几个Buffer中，这些Buffer统称G-buffer。第二个Pass才开始利用这些渲染好的信息计算光照。     延迟渲染是有缺点的：     1.不支持真正的抗锯齿功能     2.不能处理半透明物体     3.对显卡有要求 |
| 前向渲染中，BasePass对主光源的选取                           | 是自动的，会选取最亮的平行光。如果没有平行光，就是黑的。     |
| 关于AddtionPass                                              | 之前说了，会在这里计算其他的光源的影响。大概流程是这样：     1.判断光源类型，根据类型的不同计算影响。     2.根据需要计算光源在特定坐标系下的位置、方向、和衰减信息     3.按照正常的流程计算这些其他光源的漫反射和高光，再乘上衰减量。（Pass中要开启混合） |
| Unity获取光照衰减的方式                                      | 默认情况下，unity并不直接去计算光照的衰减，而是通过一张衰减的纹理，通过采样来获取衰减的值。步骤是：     1.计算点在光源空间下的坐标，右乘一个_LightMartex0。     2.用光源坐标系下的点的坐标的模长的平方对光源的衰减纹理采样，即可获得衰减值。 |
| 不透明物体的阴影的产生                                       | 原理是先把相机移到光源的位置，然后用一个标签为“ShadowCaster”的Pass，渲染出一张灯光坐标系下的深度图，判断一个点是否在阴影中时，只需要对比它在灯光坐标系下的深度和渲染出的深度图对应位置的深度即可。若比深度图的位置深，那说明这个点是被遮挡的，也就是在阴影之下的。否则说明其就是这个光源最近的点之一，可以被直接照射到。一般来说，生成投影这种事情是交给FallBack来做的，前面说FallBack不要乱写的原因就是这个，里面往往包含了阴影计算的内容。 |
| 不透明物体阴影的获得（被其他物体投影）                       | 这里的代码很多是被宏定义的。简单来说，想要一个物体可以被其他物体投影，则需要：     1.包含文件：#include "AutoLight.cginc"      2.在v2f结构体中加上：SHADOW_COORDS(2);大概内容就是声明了一组用于对阴影纹理采样的坐标。（注意，这个2不是随便来的，是根据目前的已经使用的纹理数量决定的。比如说我已经定义了TEXCOORD0和1，那么这个括号里才填2）     3.在顶点着色器中：TRANSFER_SHADOW(o);大概是计算纹理坐标。     4.最后，在frag中计算和应用阴影值：fixed shadow =  SHADOW_ATTENUATION(i);最后在计算光照模型的时候，乘上阴影这个系数就可以了。     *：需要注意，使用宏的时候，变量的名称是卡的很死的，在阴影的计算中，顶点着色器只能叫vert，顶点的位置只能叫pos。 |
| 使用内置函数同时计算光照衰减和阴影                           | 前面都和上一条一样，只是第4就不用了，改成：     UNITY_LIGHT_ATTENUATION(atten,I,i.worldPos);     函数会把光照和阴影的结果整合到第一个参数里，只需要在计算光照模型的时候乘上这个参数就可以了 |
| 透明物体的投影问题                                           | 如果使用上面的通过FallBack来生成阴影的话，对于不透明物体来说，会出现问题。比如说，若通过透明度测试来镂空物体，则投影并不会表现为镂空，而是一整个正方体，我们只需要修改FallBack至：“Transparent/Cutout/VertexLit”，这样做可以在生成光源坐标系下的深度图的时候，也会进行透明度测试，则可以正确地生成镂空的阴影。     若是一个半透明物体，目前都没有好的解决办法，只能强制生成阴影或者不生成阴影。     生成：VertexLit     不生成：Transparent/VertexLit |
| 立方体纹理                                                   | 类似全景图片一样，根据六个面的内容拼接起来的纹理，可以看到360度的东西，常常用来作为天空盒，或者用来做反射纹理，可以模拟出光滑的金属反射出环境的效果。 |
| 制作立方体纹理                                               | 如果想要制作一个立方体纹理，其实很容易，Unity有提供函数Camera.RenderToCubmap() |
| 制作反射效果                                                 | 原理是：通过计算世界坐标下的视线方向（摄像机到片元）以片元法线为法线的反射光线，作为一个采样向量，用来采样立方体纹理就可以了。一般在顶点着色器中计算反射光线的采样纹理坐标，因为这样结果不会相差很多，而且比较节省性能 |
| 制作折射效果                                                 | 与反射非常相似，只不过要用到一个计算折射的公式：斯涅尔定律：     n1*Sin a1 = n2*Sin a2；     用来表现折射率和夹角的关系。     与反射计算类似的，在顶点着色器中，计用refract函数计算出射光线的方向，以其为依据对立方体纹理采样即可。 |
| 制作菲涅尔反射                                               | 更加写实的反射。现实中的反射往往不是镜面反射，上面那种计算的只能是镜面反射。那么现实生活中的反射是什么样的呢？你可以参考一下水面，当你向脚下看，也就是视线方向与面法线夹角小的时候，反射不强烈。而你看远方的水面的时候，也就是视角和水面的面法线夹角大的时候，反射是强的。是只有水面这种比较透明的材质才会有菲涅尔反射吗？不是的，其实生活中的大多数反射都是菲涅尔反射。比如说我装满了咖啡豆的罐子，它就能当成一个不透明的玻璃圆柱体，仔细观察一下，就能发现，其实正面的反射很弱，能够清晰地看到里面的豆子。而比较侧面的地方反射就比较强，我几乎看不清豆子。     那么如何在Shader中写出来呢？其实只要用视角和面法线的夹角来影响反射的强度就好了。     我们可以在片元着色器中：     fixed fresnel = _FresnelScale + (1 - _FresnelScale) * pow(1 -  dot(worldViewDir,worldNormal), 5);     这是一个计算菲涅尔效应的近似的等式。     这之后，只要用它来拘束反射的强度就可以了，在光照模型的计算中：     fixed3 color = ambient + lerp(diffuse, reflection, saturate(fresnel)) *  atten;     ambient是环境光，lerp是插值函数，会返回diffuse  和reflection之间根据fresnel来插值的结果，saturate可以把里面的值拘束在0~1之间。atten是计算好的光线衰减和阴影的系数。这样就可以根据片元的菲涅尔系数来制作菲涅尔反射了，使用非常的广泛。 |
| 渲染纹理                                                     | 之前说GPU渲染完画面会放到双重缓冲的back中，但其实现代GPU还支持渲染结果放到一张渲染纹理中，可以通过给摄像机赋予一张用来渲染的渲染纹理，即可把它的渲染画面实时的显示在这张纹理上。 |
| 用渲染纹理实现镜子效果                                       | 只要在镜子的位置安置一个摄像机，再把摄像机的画面输出到这个镜子链接的渲染纹理上，就可以了。 |
| 透过玻璃折射看到后面的物体的效果                             | 书中使用了GrabPass，使用GrabPass{"_RefractionTex"}，这样的形式，可以把在此之前渲染好的物体放到这个名字的纹理里面。例子中玻璃属于半透明物体，本就是在不透明物体之后渲染的。      再说透过玻璃折射的效果，如果只是简单的混合，将得不到正确的折射效果，比如渲染一个球，透过凹凸不平的玻璃，你看到的还是球吗？肯定不是吧，肯定被扭曲了或者错位了吧。     为了达到这种效果，需要先渲好不透明物体，并保存到纹理中，然后再渲染半透明的玻璃，注意，为了达到这种效果，是不能开启混合的，半透明是假象，只是我们提前对后面的东西保存了纹理而已。然后我们只需要根据这个玻璃的片元再根据法线贴图偏移之后的法线方向来计算折射的出射方向，再用它来对之前渲染好的纹理来采样就可以了。 |
| 程序化生成纹理                                               | 很多比较规则的纹理都是可以用程序来生成的，方法也很简单，具体参考书例吧。 |
| 纹理动画                                                     | 把很多帧包含到一张大纹理的做法，然后根据时间来对uv进行偏移。 |
| 纹理滚轴动画                                                 | 和上面非常类似，只涉及一个轴的uv随时间的偏移                 |
| 顶点偏移动画                                                 | 和上面的思路基本一样，在顶点着色器中利用时间对顶点的位置进行偏移就行了。 |
| 使用了顶点动画的物体不能使用批处理                           | 所以需要tag"DisableBatching”  = "ture";                      |
| 广告牌效果                                                   | 即面片永远指向视角方向的效果。本质是构建模型空间下的用于旋转的基向量。具体的数学计算太深奥了，我没看懂。 |
| 顶点动画的阴影计算                                           | 顶点动画使用默认的Pass进行阴影计算的话，是得不到正确的效果的。因为默认的阴影计算的Pass中不包含顶点的偏移。若是要正确计算，必须自己重写一个tag为“ShadowCaster”的Pass，我们在那里面再进行一次一样的顶点运算就可以了。 |
| 屏幕后处理                                                   | 其实很简单，可以通过脚本控制Unity把当前的渲染结果保存在渲染纹理中，再去修改这个渲染纹理，最后把它输出在屏幕上就可以了。常见的处理如增亮、饱和度修改等，根据公式来改变纹理取得的颜色值就可以了。     这个过程需要用到Unity 的回调：OnRenderImage（RenderTexture a，RenderTexture  b）；输入是a，输出是b。在里面可以用Blit（）函数使用不同的材质或者Shader来对输入进行修改，然后保存到输出中。 |
| 基于屏幕后处理的边缘检测                                     | 利用的是卷积。图形处理里的卷积并不会像数学里那么复杂，其实就是一个根据周围像素的情况，判断自己的的梯度值的一个方法。这个所谓的周围，就叫卷积核，不止是一个范围而已，要计算梯度，当然需要有不同的权值。一些常用的卷积核，就叫算子。书中用的算子叫索贝尔算子，是一种3X3的算子，分为对x方向的算子和对y方向的，把二者的结果综合一下，一般是简单相加，就可以大概判断出这个片元是边缘的可能性。     对应到UnityShader中，需要先在顶点着色器中记录下以自己为中心的3X3片元区域的纹理采样的uv值。你可能会问，顶点着色器中，都还没有片元呢，要怎么拿到周围片元的uv呢？这需要用到用到Unity的一个宏:  _MainTex_TexelSize，这会帮我们计算相邻片元的uv坐标。使用例：     o.uv[1] = uv + _MainTex_TexelSize.xy * half2(0,-1);      片元中的信息会根据各顶点中插值而来，当然也包含它们的邻片元的uv坐标。因此我们可以直接在片元着色器中计算这个片元的梯度值。要注意，并没有内置宏帮我们定义算子，这些算子是要我们自己去敲的。首先，一般会把RGBA的颜色值转换为一维的亮度值来方便运算。然后我们对这个片元保存的像素进行xy分别的加权累加，就得到了一个轴上的梯度值。要综合xy，只需要简单的把它们加起来就可以了。书中使用了1  - x - y的值用来表示梯度值，这样表示的话，这个梯度值越小，说明是边缘的可能性越大。     已经拿到了梯度值，只需要利用它来对描边和原图进行插值，就可以得到后处理后的图像了。 |
| 高斯模糊                                                     | 基本的思路和是和边缘检测是一样的。     就是把自己这个像素的颜色值和周围的像素用权重拉一下平均。这里用的卷积核叫高斯核，通常是一个正方形，是一种滤波核。      书例使用了5X5的高斯核，要知道，这个卷积核是像一个锥形一样的，自己的影响最大，越远的影响越小，从顶上看和一个圆一样，所以5X5的高斯核并不需要记住25个格子的权重，第一步就可以简化成xy两个轴，然而，xy每个轴也是对称的，也就是说x  = 1 和 x = -1的值和y = 1的和y = -1的值，这四个是一样的，于是对于5X5的高斯核，我们只需要记住和使用3个权重就可以了。     在Unity的实现中：      1.我们需要考虑高斯模糊的处理次数和处理的半径，关于次数，很多时候只处理一次是不够模糊的，所以让用户自己来决定处理多少次，当然，次数越多越耗费性能。关于半径，即你可以决定高斯核隔几个单位采样一次。你以为卷积核必须是黏在一起的5X5格子？其实也可以设置采样的间隔，这样就可以用更远的颜色来和自己混合，这样的模糊效果往往更好，但是容易出现虚影（这种做法叫做降采样）。     2.设置用于替换的纹理。在脚本阶段，光进行一次Bilt是不够的，光高斯一次就要分xy两个Pass，所以无论如何都需要一个中间缓存来保存目前的处理结果，就像C语言里你要交换两个int的值一样，需要一个中间值t来保存。     3.进入Shader阶段，思路和描边大同小异，先在顶点着色器中对uv采样，填充xy两轴的各5个uv坐标的采样值。      4.在片元着色器中，填写上面提到的高斯核仅需要的3个权重，然后就可以遍历从顶点着色器传来的uv坐标了。5X5的话，其实只需要遍历两次，假如说这次是x轴的高斯，第一次计算x  = 1 和x = -1 处的像素值的影响，下一次遍历x = 2 和-2 的。把这些根据权重一拉平均，就能得到自己最终的颜色值了。 |
| CGINCLUDE/ENDCG                                              | 跟Pass平行的，你可以把代码定义在这两个关键字的内部，和头文件是一样的，里面可以定义变量、结构体和函数，这些东西都能在这个SubShader中公用。需要注意，如果是共用的顶点着色器之类的，需要在Pass的#Pragma命令中指定对应的函数名。 |
| Bloom泛光效果                                                | 可以做出一种亮的地方向外泛光的感觉，很好看，也比较简单。      原理是：先通过一个Pass渲染出一张渲染纹理，用来保存屏幕中较亮的地方，再对这个渲染纹理做高斯模糊，最后把这个渲染纹理和原图的颜色值直接相加即可，并没有开启混合。 |
| 实现运动模糊                                                 | Unity的一种做法比较简单，就是保存上一帧的渲染结果，再调整一下本帧的阿尔法值，再把它和上一帧的颜色混合就可以了。     在脚本阶段，我们用一张和屏幕一样大的纹理。使用其RestoreExpected（）函数，就可以把上一帧的画面存到这个纹理里面。 |
| 深度纹理和法线纹理                                           | 你知道，片元其实保存了法线和深度等信息。有时候可以把这些信息渲染成纹理，这样你就可以通过纹理来访问深度和法线信息了。     Unity中的做法是：     1.在脚本阶段，给Camera的depthTextureMode属性设置为渲染深度、或者渲染深度和法线等。     2.在Shader中，直接使用_CameraDepthNormalsTexture等名字访问即可。 |
| 把非线性的深度纹理转换成线性的                               | 什么叫非线性的？就是函数图像不是直线的。深度值被限制在NDC中被限制在【-1~1】之间，那么如何用如此有限的区间表示无限远的深度呢？很简单，用类似y  =  1/x这样的映射就可以了，这样即使X也就是深度再大，也可以用y表示出来了。但是这样的结果是非线性的，也就是说，如果三个深度值，1，2，3，你并不能说它们的间隔是一样大的，因为它们不是线性的，如果是线性的，那么就可以说它们的间隔是一样的。     在UnityShader中，可以使用Linear01Depth（）函数来计算0~1内的线性的深度值，也可以用LinearEyeDepth（）函数来计算深度纹理采样结果在视角空间下的深度值。     为什么要这么做，我猜测是和Gama矫正一样的原因，希望用更多的空间保存近处的信息，远处可以稍微粗略一些。 |
| 解读深度和法线纹理                                           | 设置摄像机的输出的深度纹理的属性，也可以得到深度+法线纹理。采样是容易的，可以直接采用tex2D函数，但是解读是困难的，需要Unity函数的支持，可以通过Shader中的DecodeDepthNormal（）函数来解读深度+  法线纹理，可以直接得到深度和法线的方向。 |
| 另一种运动模糊                                               | 原理是记录每个像素的速度，根据速度的方向和大小来决定如何模糊。书例的用来生成速度映射图的方法是：      在片元着色器中，利用变换矩阵，重构像素在世界坐标下的位置。随后，再使用前一帧的变换矩阵对其变换，这样就能得到前一帧的像素的世界坐标了，两个位置的差距就能当成是速度的参考值了。这种做法需要利用摄像机生成深度图，这样才能重构像素的世界坐标。 |
| 全局雾效                                                     | 如果让我做一个雾效，我首先想到的是给这个物体加上Shader，在有雾的地方混上雾的颜色。但是这是很蠢的，因为每一个物体都要再使用这种方法过一个Pass。所以有了基于屏幕后处理的全局雾效。      乍一想好像很容易，拿到片元的位置，再定一个限度，然后再混合颜色，不是和原来的差不多嘛。但是你搞错了，屏幕后处理的处理对象是一张渲染纹理，渲染纹理没有保存片元的位置等信息。那怎么办？其实可以利用之前提到的深度图，来重构像素的世界坐标。原理很复杂，我无法理解。但是使用到UnityShader里是这样的：     1.从脚本阶段传入需要的摄像机的信息，包括裁剪平面的距离、FOV、摄像机在世界空间下的各个方向等。      2.在片元着色器中，先利用宏对深度纹理解析，再获取这个片元的线性的深度值，再把它乘上各个摄像机的参数，最后加上世界坐标系下的摄像机位置，即可重构片元的世界坐标。     3.得到世界坐标后就好办了，按照需求规定哪些地方雾浓、衰减的系数等等，最后用lerp对雾和原图根据算出来的雾的浓度系数插值就可以了。 |
| 利用法线深度纹理描边                                         | 上面的描边效果不好，描了很多不需要的边，而且很容易受光影影响。为了解决这个问题，可以尝试用法线深度纹理来描边，这是不受光影影响的。      思路是对法线深度纹理卷积，书例中使用的是罗伯特算子，它表面上2X2，实际上还是3X3的算子，只不过只讨论对角线的值，所以每个轴的算子只考虑四个采样加上自己一共五个值来加权拉平均。这个算子也是可以加上采样间隔的，也就是降采样。     对法线深度纹理采样后可以得到一个四维的值，这个值的xy可以用来计算法线，zw可以用来计算深度。这样我们就可以在顶点着色器中，先保存好算子的uv坐标，再到片元着色器中对算子中的项采样，再分别根据人为指定阈值分析这个片元是不是一条边，若法线和深度都显示它是一条边，那么就可以告诉片元着色器，把它渲成描边的颜色。 |
| 卡通渲染                                                     | 书例的卡通渲染主要在讲描边、漫反射和高光的特殊处理。      1.先说说新的描边方法：通过单独的一个Pass来实现，首先剔除模型的正面，然后把剩下的顶点的位置沿着视角空间下的法线方向扩展一小段距离，想象一下，一个模型的顶点都沿着法线方向移动一点，会发生什么？没错，就是会变大一点嘛。可是这样单纯的变大可能会挡住后面要渲染的正面，所以可以可以把背面的顶点法线在扩张前先z值减小一定值，也就是放大前，先把它的z轴压缩一下。视角空间下的z就是表示远近，所以你可以理解为变得像一个面片了。     2.漫反射怎么办？很简单，用之前学的那个渐变纹理就可以了，用计算出来的漫反射的值，去采样一张渐变的纹理，再用这个得到的颜色来表示漫反射颜色就可以了。      3.高光比较复杂，卡通渲染的高光是一块一块、棱角相对分明的，真实视角的高光往往是渐变的。其实也可以用类似渐变纹理的思想，先按找传统方法计算出高光的值，再对这个值进行判断，如果大于阈值，鉴定为高光，如果小于，鉴定为非高光就可以了。但是这样往往高光不平滑，也就是锯齿严重，可以使用smoothstep函数对高光边缘的值进行一下插值。 |
| 素描风格渲染                                                 | 原理就是判断点的漫反射系数，然后根据它所处的梯度来决定使用哪些纹理来混合，事先需要准备不同漫反射系数的纹理。然后到片元着色器中，对所有梯度的纹理采样，并乘上在顶点着色器中根据梯度计算的每一张的权重即可。 |
| 使用噪声贴图制作消融效果                                     | 比如我需要制作一个箱子燃烧逐渐消失的效果，可以用一个Amount来控制燃烧的进度，0表示完全没开始，1表示已经烧完了。这个Amount其实就是一个阈值，我们先看消融的消失怎么做，其实很简单，在片元着色器中，通过噪声的采样值和燃烧进度的差值就可以判断这个片元处于哪个状态，如果已经被烧完了，就把它剔除，否则进入下面的渲染。      下面的渲染其实就是正常的渲染了，如果这个片元还没烧完，但其实已经快被烧完了，表现为差值已经比较小了，就把它的颜色和燃烧的颜色插值，这样就能在边缘营造一种火光一样的效果      还要注意阴影的处理，你在这个片元着色器剔除了片元，但是在产生阴影的着色器中没有剔除，就会导致已经烧穿的地方光线投不过去。解决方法和之前一样，只需要再自己写一个产生阴影的、标签为“ShadowCaster”的Pass，再在里面Clip掉已经被烧穿的片元就可以了 |
| 使用噪声法线贴图模拟水波                                     | 很像之前一些知识点的混合运用。     1.使用菲涅尔效应，对水面的折射和反射效果插值     2.使用正方体贴图，对反射效果采样     3.使用GrabPass，摘取当前渲染结果，并对其采样模拟折射效果（再次提醒，不是透明混合，是假的透明）     4.使用时间变量在采样时对uv进行偏移，如果是偏移的是法线贴图，还能模拟顶点动画的效果。这里是模拟水波的上下摇荡。 |
| 使用噪声贴图优化基于屏幕后处理的雾效                         | 之前的雾效是静止的、均匀的，不够好。可以考虑使用噪声，再加上uv偏移动画，模拟出飘动的雾效。      主要的做法和之前一样，先取得渲染纹理，再使用矩阵重构世界坐标系。原来的雾的浓度仅由用户指定的雾浓度插值决定，但现在的雾浓度还会受噪声的影响。用片元的uv坐标对噪声图采样，再通过运算影响到雾浓度，即可得到不均匀的雾。如何让雾飘动起来？很容易，雾的浓度不是还会受噪声贴图的采样值的影响吗，那只要根据时间不断偏移噪声贴图的采样坐标就可以了。其余的部分和原来的雾效基本一致 |
| 造成性能瓶颈的因素                                           | CPU：     DrawCall多     复杂的脚本逻辑和复杂的物理模拟     GPU：     过多的顶点     过多的逐顶点计算     过多的片元     过多的逐片元计算     使用了大分辨率且未压缩的纹理     使用了分辨率过高的帧缓存 |
| 如何优化？                                                   | CPU：     用批处理尽量减少DrawCall     GPU：     优化模型，减少点面     使用LOD技术，在远的时候，使用精度更低的模型和更简单的Shader     使用遮挡剔除技术，早一点进行Ztest，就不用渲染看不见的地方了。     控制绘制顺序，做到尽量先渲前面的东西     警惕半透明物体，半透明物体的overDeaw问题严重，能少用就少用吧     减少实时光照（阴影）尽量用烘焙好的光照吧     使用Shader中的LOD技术     减小纹理大小，使用各种压缩技术 |
| 批处理                                                       | 来仔细说下批处理吧     分为静态批处理和动态批处理     静态批处理：      是用户手动指定的，操作方法是在Inspector面板把这个物体勾成static，就会被自动和其他同材质的东西打为一批了。这更自由，但是可能会消耗很多的内存，而且静态批处理后的物体无法移动。     动态批处理：      打开项目设置中的动态批处理后，就不需要做任何操作，Unity会自动把同一材质的物体打成一个批，而且这样批处理的物体是可以移动的。缺点是要求苛刻，顶点过多、或者光照环境稍微复杂，就无法再使用了。 |
| 共享材质                                                     | 我会觉得批处理的先决条件就很苛刻，那就是要求必须同一材质。要知道，材质之所以可以表现出很多效果，是因为有很多参数，而这些参数和类的Static变量一样，一个材质只有一套，一个变了，全都变了。我想不出来如何用一个材质表现两种仅通过改变参数造成的不同的效果。所以有了共享材质这个东西。它可以用同一个材质表现多种效果，这样这些物体就可以打成一批了。如果是想用不同的纹理，比如说书例里的那个盒子，我想用一种材质做出两种图案不一样的盒子，怎么办？可以用一张更大的贴图，内含了两种图案的完整贴图。在采样的时候，用特殊的方法分别采样就可以了。如果还想区分别的参数，可以判断顶点颜色再区别计算。 |
| 为什么说安排好渲染顺序可以节省性能？                         | 因为存在overdraw的问题，也就是一个像素会被多次绘制的问题。你知道，Ztest是在片元着色器结束之后才进行的，如果你先画离屏幕远的，再画离屏幕近的，那远的的被遮挡的细节就被浪费了。  你大可以先画离屏幕近的，再画离屏幕远的，这样被遮挡的部分就不用画了。（这是EarlyZ技术，在片元着色器之前先进行一次Ztest，详情见百人笔记） |
| Shader的LOD技术                                              | 就是给SubShader加一个LOD值，当LOD低于设定的阈值时，这个Shader才会被调用。 |
| 表面着色器                                                   | 你不觉得每写一个Shader，无关紧要的东西太多了么，为什么我要再计算一下光照，为什么我要再自己算一下裁剪空间下的顶点坐标，这些不是每一次都要做的么，为什么要人为来做，太弱智了吧。确实，于是有了表面着色器，像我写的基类一样，表面着色器只需要你写最核心的东西就可以了 |
| 表面着色器的编译指令                                         | 还记得编译指令是什么么，就是那些个#pragma  …  在原先的UnityShader中，用它们来指定vert和frag函数。在表面着色器中也需要编译指令，最重要的任务是指明表面函数和光照函数。还可以指定顶点修改函数和最后颜色修改函数，还可以加上一些指令，来改变阴影之类的设置，非常方便。 |
| 表面着色器的两种结构体                                       | 顶点和片元着色器之间交流信息，靠的v2f这个结构体。表面着色器的思路是类似的，有一个输入结构体Input  和储存了表面信息的输出结构体SurfaceOutput（如果是基于物理的光照模型，输出的结构体会不一样）。Input是我们自己定义的，想要什么变量，就去找这个变量的名字，然后照抄下来就可以了，都不用语义映射。OutPut的结构体是无法修改的，我们一般会拿到这个输出结构体的对象实例，我们只要去改里面的变量就好了。 |
| 对表面着色器和UnityShader的对应关系的理解                    | Unity中，表面着色器最终是转换成顶点和片元着色器来运行的，它具有刚好的可读性，也更容易编写。      表面着色器的顶点修改函数是一个逐顶点的操作，有点像顶点着色器，但它不用计算复杂的矩阵运算，你只需要算你想要的就好，比如说根据法线方向移动顶点位置之类的。顶点修改函数的参数是一个inout  appdate_full 类型的参数，里面和a2v里的东西差不多。     随后是表面函数，有点像片元着色器，但是输出的不是颜色值了，而是表面的信息，在这个函数里，主要是进行一些表面的反射率啊、透明度啊、法线啊等计算，并把它们的值保存到SurfaceOutput的对象中。     之后是光照函数，名字是：Lighting+“……” ……为你指定的光照模型的名字。返回值是half4，也就是颜色值。参数是Input  和SurfaceOutput，和表面函数类似，把结果存到SurfaceOutput的对象中。光照函数是可以没有的，没有的话他就用你在#pragma中指定的，你写了的话他就用你的。和frag中对光照的计算是很像的。     最后是最后颜色修改函数，无返回值，参数是Input、SurfaceOutput和一个inout  fixed4，第三个参数代表颜色值，指前面全部渲完后的颜色，是一个逐像素的操作。只要修改第三个参数的值，就能修改最后输出的颜色了。 |
| 表面着色器的缺点                                             | 方便意味着更少的自由，无论在哪里都是这样。     表面着色器的性能不是很好。 |
| 超采样                                                       | 超级采样抗锯齿，SSAA。原理是，把分辨率成倍放大。然后再缩放到屏幕分辨率，对资源消耗非常大。 |