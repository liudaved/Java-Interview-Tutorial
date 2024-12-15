# 01-视频基础知识

图像或视频，能感知到色彩差异、清晰度、明暗对比，这些画面是咋形成展示的？内部原理是啥？从视频/图像的原始数据格式、视频逐行/隔行扫描、帧率、图像分辨率、色域等方面，对视频基础知识做整体了解。

## 1 视频、图像像素点数据格式

看视频时会看到很多图像，这些图像的展现形式由一个个像素点组成的线，又由一条条线组成面，这个面铺在屏幕展现就是我们看到的图像。

这些图像有黑白，也有彩色，因为图像输出设备支持规格不同，色彩空间不同，不同色彩空间能展现的色彩明暗程度，颜色范围等不同。

### 1.1 色彩格式

- GRAY 色彩空间
- YUV 色彩空间
- RGB 色彩空间
- HSL 和 HSV 色彩空间

#### 1.1.1 GRAY 灰度模式表示

黑白电视的图像以GRAY方式展现的图像，即Gray灰度模式，8位展示的灰度，取值0至255，表示明暗程度，0为最黑暗的模式，255为最亮的模式，色彩表示范围如图所示：

![](https://upload-images.jianshu.io/upload_images/16782311-4534351e55d4100e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

由于每个像素点用8位深展示的，一个像素点等于占用一个字节，一张图像占用的存储空间大小：
$$
图像占用空间 = 图像宽度 (W) * 图像高度(H) * 1
$$
举个例子，如果图像为352x288的分辨率，那么一张图像占用的存储空间应该是352x288，也就是101376个字节大小。

#### 1.1.2 YUV 色彩表示

视频领域通常以YUV格式存储和显示图像:

- Y表示视频的灰阶值，也可理解为亮度值
- UV表示色彩度，若忽略UV值，看到的图像与前面提到的GRAY相同，为黑白灰阶形式的图像

YUV最大优点：每个像素点的色彩表示值占用的带宽或存储空间很少。

原图与YUV的Y通道、U通道和V通道的图像示例：

![](https://upload-images.jianshu.io/upload_images/16782311-a581805fb01979d8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

为节省带宽，大多YUV格式平均使用的每像素位数都少于24位。主要的色彩采样格式有：

- YCbCr 4：2：0
- YCbCr 4：2：2
- YCbCr 4：1：1
- YCbCr 4：4：4

YUV的表示法也称为A：B：C表示法。

352x288的图像大小为例看各采样格式的区别。

##### YUV 4：4：4 格式

yuv444表示4比4比4的yuv取样，水平每1个像素（即1x1的1个像素）中y取样1个，u取样1个，v取样1个，所以每1x1个像素y占有1个字节，u占有1个字节，v占有1个字节，平均yuv444每个像素所占位数为：

![](https://upload-images.jianshu.io/upload_images/16782311-20d3be76148b4dae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

352x288分辨率的一帧图像占用的存储空间：

![](https://upload-images.jianshu.io/upload_images/16782311-2a71ff46961b10ea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### YUV 4：2：2 格式

yuv422表示4比2比2的yuv取样，水平每2个像素（即2x1的2个像素）中y取样2个，u取样1个，v取样1个，所以每2x1个像素y占有2个字节，u占有1个字节，v占有1个字节，平均yuv422每个像素所占位数为：

![image-20230517154219772](https://p.ipic.vip/klmv96.png)

那么352x288分辨率的一帧图像占用的存储空间为：

![image-20230517154242836](https://p.ipic.vip/fyelc0.png)

##### yuv4：1：1 格式

yuv411表示4比1比1的yuv取样，水平每4个像素（即4x1的4个像素）中y取样4个，u取样1个，v取样1个，所以每4x1个像素y占有4个字节，u占有1个字节，v占有1个字节，平均yuv411每个像素所占位数为：

![image-20230517154323464](https://p.ipic.vip/5p2aub.png)

那么352x288分辨率的一帧图像占用的存储空间为：

![image-20230517154558633](https://p.ipic.vip/xned2b.png)

##### yuv4：2：0 格式

yuv420表示4比2比0的yuv取样，水平每2个像素与垂直每2个像素（即2x2的2个像素）中y取样4个，u取样1个，v取样1个，所以每2x2个像素y占有4个字节，u占有1个字节，v占有1个字节，平均yuv420每个像素所占位数为：

![image-20230517154809317](https://p.ipic.vip/fkzuna.png)

那么352x288分辨率的一帧图像占用的存储空间为：

![image-20230517154711115](https://p.ipic.vip/yh6ijl.png)

为方便理解YUV在内存中的存储方式，以宽度=6、高度=4的yuv420格式为例，一帧图像读取和存储在内存中的方式如图：

![image-20230517154647438](https://p.ipic.vip/y22ied.png)

#### 1.1.3 RGB 色彩表示

三原色光模式（RGB color model），又称RGB颜色模型或红绿蓝颜色模型，一种加色模型，将红（Red）、绿（Green）、蓝（Blue）三原色的色光按照不同的比例相加，来合成各种色彩光。

每象素24位编码的RGB值：使用三个8位无符号整数（0到255）表示红色、绿色和蓝色的强度。主流的标准表示方法，用于交换真彩色和JPEG或者TIFF等图像文件格式里的通用颜色。可产生一千六百万种颜色组合。

使用每原色8位的全值域，RGB能有256个级别的白-灰-黑深浅变化，255个级别的红色、绿色和蓝色及它们等量混合的深浅变化，但其他色相的深浅变化相对少。

典型使用上，数字视频的RGB不是全值域的。视频RGB有比例和偏移量的约定，即 （16, 16, 16）是黑色，（235, 235, 235）是白色。例如，这种比例和偏移量就用在CCIR 601的数字RGB定义。

RGB常见的展现方式：

- 16位模式

  16位模式（RGB565、BGR565、ARGB1555、ABGR1555）分配给每种原色各为5位，其中绿色为6位，因为人眼对绿色分辨的色调更敏感。但某些情况下每种原色各占5位，余下的1位不使用或者表示Alpha通道透明度

- 32位模式

  32位模式中主要用其中24位表示RGB

32位模式（ARGB8888），实际就是24位模式，余下的8位不分配到象素中，这种模式是为了提高数据处理的速度。同样在一些特殊情况下，在有些设备中或者图像色彩处理内存中，余下的8位用来表示象素的透明度（Alpha通道透明度）。

即RGB图像色彩表示，对照RGB色彩分布直方图来理解：

![image-20230517161055268](https://p.ipic.vip/m70gjy.png)

#### 1.1.4 HSL 与 HSV 色彩表示

HSL和HSV是将RGB色彩模型中的点放在圆柱坐标系中的表示法，视觉上比RGB模型更直观。

HSL就是：

- 色相（Hue）
- 饱和度（ Saturation）
- 亮度（ Lightness）

HSV是：

- 色相（Hue）
- 饱和度（ Saturation）
- 明度（Value）

色相（H）是色彩的基本属性，即颜色名称，如红色、黄色等；饱和度（S）指色彩纯度，越高色彩越纯，低则逐渐变灰，取0～100%；明度（V）和亮度（L），同样取0～100%。

HSL和HSV都把颜色描述在圆柱坐标系里的点内，这个圆柱的中心轴取值为自底部的黑色到顶部的白色，而在它们中间的是灰色，绕这个轴的角度对应于“色相”，到这个轴的距离对应于“饱和度”，而沿着这个轴的高度对应于“亮度”、“色调”或“明度”。如图：

![](https://p.ipic.vip/f3pso6.png)

HSV色彩空间还可以表示为类似于上述圆柱体的圆锥体，色相沿着圆柱体的外圆周变化，饱和度沿着从横截面的圆心的距离变化，明度沿着横截面到底面和顶面的距离而变化。这种用圆锥体来表示HSV色彩空间的方式可能更加精确，有些图像在RGB或者YUV的色彩模型中处理起来并不精准，我们可以将图像转换为HSV色彩空间，再进行处理，效果会更好。例如图像的抠像处理，用圆锥体表示在多数情况下更实用、更精准。如图：

![image-20230517161446699](/Users/javaedge/Library/Application%20Support/typora-user-images/image-20230517161446699.png)

## 2 图像的色彩空间

了解了视频和图像的集中色彩表示方式，是不是用相同的数据格式就能输出颜色完全一样的图像呢？不一定，观察电视中的视频图像、电脑屏幕中的视频图像、打印机打印出来的视频图像，同一张图像会有不同颜色差异，甚至不同电脑屏幕看到的视频图像、不同的电视看到的视频图像，有时也存在色差，如：

![image-20230517163957286](https://p.ipic.vip/6jh1j8.png)

如果仔细观察的话，会发现右图的颜色比左图的颜色更深一些。之所以会出现这样的差异，主要是因为图像受到了色彩空间参数的影响。我们这里说的色彩空间也叫色域，指某种表色模式用所能表达的颜色构成的范围区域。而这个范围，不同的标准支持的范围则不同，下面，我们来看三种范围，分别为基于CIE模型表示的BT.601、BT.709和BT.2020范围。

![image-20230517161901189](https://p.ipic.vip/sxzv6h.png)

色彩空间除了BT.601、BT.709和BT.2020以外，还有很多标准格式，用到时，可使用参考标准（可参考标准：H.273）进行对比。当有人反馈偏色的问题时可以优先考虑是色彩空间的差异导致的，需要调整视频格式（Video Format）、色彩原色（Colour primaries）、转换特性（Transfer characteristics）和矩阵系数（Matrix coefficients）等参数。



色彩格式是图像显示的基础，但是视频技术不仅仅需要知道色彩格式，想要理解视频图像的话，还需要弄清楚一些现象，如：

- 有的视频图像运动的时候会有条纹，有的视频图像在运动的时候没有条纹
- 用一些工具导出电影视频的时候，一般会按照23.97fps的帧率导出，而很多公众号或者媒体在宣传支持60帧帧率

## 3 视频逐行、隔行扫描与帧率

老电视剧、老电影或者一些DV机拍摄的视频时，会发现视频中物体在移动时会出现条纹，主要因为视频采用隔行扫描的刷新方式。

### 隔行扫描与逐行扫描

隔行扫描（Interlaced）是一种将图像隔行显示在扫描式显示设备上的方法，例如早期的CRT电脑显示器。非隔行扫描的扫描方法，即逐行扫描（Progressive），通常从上到下地扫描每帧图像，这个过程消耗的时间比较长，占用的频宽比较大，所以在频宽不够时，很容易因为阴极射线的荧光衰减在视觉上产生闪烁的效应。而相比逐行扫描，隔行扫描占用带宽比较小。扫描设备会交换扫描偶数行和奇数行，同一张图像要刷两次，就产生条纹。

![](https://p.ipic.vip/sgu00l.png)

早期显示器设备刷新率低，不太适合使用逐行扫描，一般都使用隔行扫描。隔行扫描常见分辨率描述是720i、1080i。

“i”就是Interlaced。看视频播放器相关广告和说明时，720p、1080p的“p”又是啥？

由于现代的逐行扫描显示的刷新率提高，使用者不会感觉到屏幕闪烁。因此，隔行扫描技术逐渐被取代，逐行扫描更常见，即720p、1080p。

当我们拿到隔行扫描/逐行扫描的数据后，会看到：25fps、30fps、60fps等等。fps是啥？

### 帧率

帧率（FrameRate），1s刷新的视频图像帧数（Frames Per Second），视频一秒钟可以刷新多少帧，取决于显示设备的刷新能力。不同时代的设备，不同场景的视频显示设备，刷新的能力也不同，所以针对不同的场景也出现了很多种标准，例如：

1. NTSC标准的帧率是 30000/1001，大约为 29.97 fps；
2. PAL标准的帧率是 25/1，为25 fps；
3. QNTSC 标准的帧率是 30000/1001，大约为 29.97 fps；
4. QPAL标准的帧率是 25/1，为25 fps；
5. SNTSC标准的帧率是 30000/1001，大约为 29.97 fps；
6. SPAL标准的帧率是 25/1，为25 fps；
7. FILM标准的帧率是 24/1，为24 fps；
8. NTSC-FILM标准的帧率是 24000/1001，大约为 23.976 fps。

如果用心观察的话，你会发现NTSC标准的分辨率都不是整除的帧率，分母都是1001，为什么会这样呢？

NTSC 制式的标准为了解决因为色度和亮度频率不同引起失真色差的问题，将频率降低千分之一，于是就看到了有零有整的帧率。我们在电影院看的电影的帧率，实际上标准的是 23.97 fps，所以我们可以看到给院线做视频后期制作的剪辑师们最终渲染视频的时候，大多数会选择23.97 fps的帧率导出。关于视频刷新帧率背后更详细的知识，如果感兴趣的话，你可以继续阅读一下《The Black Art of Video Game Console Design》。

说到这里，我们再来解答一下前面的问题，为什么有些公众号宣传自己的编码和设备支持60帧帧率呢？这是因为科技在进步，有些显示设备的刷新率更高了，为了让我们的眼球看着屏幕上的物体运动更流畅，所以定制了60帧，这也是为了宣传自己设备的功能更加先进、强大。但是在院线标准中，60fps刷新率的设备并没有大范围升级完毕，当前我们看的依然还是以film、ntsc-film标准居多。

## 4 图像分辨率与比例

最后我们来看一下另一个与图像相关的重要概念——分辨率。当人们在谈论流畅、标清、高清、超高清等清晰度的时候，其实主要想表达的是分辨率。它是衡量图像细节表现力的重要的技术参数。

除了分辨率之外，我们还需要结合视频的类型、场景等设置适合的码率（单位时间内传递的数据量）。随着视频平台竞争越来越激烈，网络与存储的开销越来越高，有了各种定制的参数设置与算法，在分辨率相同的情况下做了更深层的优化，比如极速高清、极致高清、窄带高清等。但是目前人们对流畅、标清、高清、超高清等清晰度的理解，其实普遍还是指分辨率。

一般，分辨率越高代表图像质量越好，越能看到图像的更多细节，文件也就会越大。分辨率通常由宽、高与像素点占用的位数组成，计算方式为图像的宽乘以高。在提到显示分辨率的时候，人们还常常会提到宽高比，即DAR。DAR是显示宽高比率（display aspect ratio），表示不同分辨率的图像的差别。

![image-20230517163650217](https://p.ipic.vip/0ox7qz.png)

而分辨率在我们日常的应用中各家的档位定义均有不同，但是在国际的标准中还是有一个参考定义的，并且分辨率都有定义名称。为了方便理解，我们来看一下分辨率的示意图，如图：

![](https://p.ipic.vip/ex2uqo.png)

我们经常听到人们提到1080p、4K，其实它们还有更标准的称呼或者叫法，例如1080p我们又叫Full-HD，通常接近4K的分辨率我们叫4K也没太大问题，像有更标准的叫法，比如3840x2160 的分辨率应该是UHD-1。但是如果直接按标准叫法来叫的话，国内很多人可能不太习惯，为了便于区分，通常就直接说分辨率的宽乘以高的数值。因为4k的表述比较简洁，所以就可以模糊地说是4K了。

## 5 总结

从视频图像像素点数据格式、视频逐行/隔行扫描、帧率、分辨率与比例、色域几个方面带你做了一个概览，这几个方面是组成视频基础最重要的几块基石。

![](https://upload-images.jianshu.io/upload_images/16782311-427326a62599b5dd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

音视频技术与计算机图形学在图像处理方面略有相似，在做视频技术的时候，会频繁地用到图像色彩相关的知识。所以这节课我详细地介绍了GARY、YUV、RGB、HSL/HSV四种色彩表示模式。但如果想要做好视频技术，仅仅知道一些图像色彩知识是万万不行的。因为视频是连续的图像序列，所以关于视频逐行/隔行扫描、帧的刷新频率等相关知识也必不可少。

图像序列裸数据占用的存储和带宽极高，为了降低存储和传输带宽，我们就需要做图像的数据压缩，图像压缩以有损压缩为主，加上图像本身色彩格式多样，所以难免会有偏色等问题，学完今天的课程你应该能想到这主要是色彩空间的差异导致的，这时候我们需要调整各项参数来解决问题。用户观看视频的时候还需要解码视频数据包，为图像色彩的像素点表示数据，所以我们就又需要用到图像与色彩技术了。

## FAQ

YUV与MP4、H.264、RTMP之间什么关系？

YUV、MP4、H.264和RTMP是与视频相关的不同概念和技术，它们在视频处理和传输中扮演着不同的角色。

1. YUV：YUV是一种颜色编码格式，用于表示彩色图像中的亮度（Y）和色度（U、V）分量。YUV格式常用于数字视频处理中，它将亮度和色度分离存储，能够有效地压缩彩色图像数据并保持可接受的图像质量。YUV格式经常在视频编码、解码和处理过程中使用。
2. MP4：MP4是一种常见的视频文件格式，它是一种容器格式，可以用于存储音频、视频和其他相关媒体数据。MP4文件通常使用H.264（或其他视频编码器）进行视频压缩，并使用AAC（或其他音频编码器）进行音频压缩。MP4文件可以在各种设备和平台上播放，并广泛用于存储和传输视频内容。
3. H.264：H.264，也被称为AVC（Advanced Video Coding），是一种视频压缩标准，用于将视频数据进行压缩和编码。H.264采用先进的压缩算法，能够在保持较高视频质量的同时实现更低的比特率，从而减小存储空间和传输带宽的需求。H.264是当前最常用的视频编码标准之一，广泛应用于视频压缩、存储和传输。
4. RTMP：RTMP（Real-Time Messaging Protocol）是一种实时消息传输协议，用于实时的音视频流传输和互动。RTMP可以用于将实时音视频数据从源（如摄像头、编码器）传输到服务器，然后通过RTMP协议将音视频流传输到客户端进行实时播放或其他处理。RTMP常用于直播、视频会议和流媒体传输等场景。

综上，YUV是一种颜色编码格式，用于表示图像的亮度和色度分量。MP4是一种视频文件格式，可用于存储视频和音频数据。H.264是一种视频压缩标准，用于将视频进行压缩和编码。RTMP是一种实时消息传输协议，用于实时的音视频流传输和互动。它们在视频处理和传输中扮演不同的角色，并相互关联用于实现视频的压缩、存储、传输和播放等功能。