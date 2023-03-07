# Audio-Video-Study
 音视频开发
https://github.com/zhanxiaokai


# 一、音频渲染
AudioQueue（AVAudioSession），AudioUnit（AUGraph、AVAudioEngine）

音频的渲染源是 PCM 数据
## 1.iOS 平台的音频格式
ASBD（AudioStreamBasicDescription）

mFormatID 这个参数是用来指定音频的编码格式，此处音频编码格式指定为 PCM 格式；

mSampleRate 声音的采样率；

mChannelsPerFrame 声道数；

mFramesPerPacket每个 Packet 有几个 Frame；

mFormatFlags 是用来描述声音表示格式的参数，代码中的第一个参数指定每个 sample 的表示格式是 Float 格式。这个类似于我们之前讲解的每个 sample 使用两个字节（SInt16）来表示；然后是后面的参数 NonInterleaved，表面理解这个单词的意思是非交错的，其实对音频来说，就是左右声道是非交错存放的，实际的音频数据会存储在一个 AudioBufferList 结构中的变量 mBuffers 中。如果 mFormatFlags 指定的是 NonInterleaved，那么左声道就会在 mBuffers[0]里面，右声道就会在 mBuffers[1]里面，而如果 mFormatFlags 指定的是 Interleaved 的话，那么左右声道就会交错排列在 mBuffers[0]里面，理解这一点对后续的开发是十分重要的；

mBitsPerChannel 表示的是一个声道的音频数据用多少位来表示，前面我们已经知道每个采样使用 Float 来表示，所以这里就使用 8 乘以每个采样的字节数来赋值；

mBytesPerFrame 和 mBytesPerPacket 的赋值，这里需要根据 mFormatFlags 的值来分配。如果在 NonInterleaved 的情况下，就赋值为 bytesPerSample（因为左右声道是分开存放的）；但如果是 Interleaved 的话，那么就应该是 bytesPerSample * channels（因为左右声道是交错存放的），这样才能表示一个 Frame 里面到底有多少个 byte。
## 2.AudioUnit 的分类
(1)Effect Unit

(2)Mixer Units

(3)I/O Units

(4)Format Converter Units

(5)Generator Units
# 二、视频渲染
视频的渲染源是 YUV 或者 RGBA 格式的数据，这种数据是描述画面最基础的格式，其中 YUV 常用在视频的原始格式中，RGBA 常用在一些图像的原始格式上。

iOS 平台的 SDK 层也提供了 Quartz 2D 把位图（Bitmap）绘制到屏幕上，在一些自定义 View 以及获取 snapshot 的场景下使用。

iOS 平台提供了 Metal 来作为 OpenGL 的替代者，在 iOS12 系统之后弃用 OpenGL ES，系统的一些框架全面改成默认 Metal 支持。由于 Metal 的设计和 OpenGL 比较类似，开发者上手难度并不大，绘制性能也有比较明显的提升，主要缺点是跨平台性比较差。
## OpenGL ES（OpenGL for Embedded Systems）
OpenGL 的全称是 Open Graphics Library，它用于二维或三维图像的处理与渲染，是一个功能强大、调用方便的底层图形库。它定义了一套跨编程语言、跨平台编程的专业图形程序接口。而 OpenGL ES（OpenGL for Embedded Systems）是它在嵌入式设备上的版本，这个版本是针对智能手机等嵌入式设备设计的，可以理解为是 Open GL 的一个子集。

每个平台提供渲染的基础实现，我们称之为 OpenGL 的上下文环境。另外在 OpenGL 的设计中，OpenGL 是不负责管理窗口的，窗口的管理也交由每个平台自己来完成。与上下文环境一样，窗口管理在每个平台也都有自己的实现。具体来讲，作为 OpenGL ES 的上下文环境，iOS 平台为开发者提供了 EAGL，而 Android 平台（Linux 平台）为开发者提供了 EGL。

一个开源项目——GPUImage。它的实现非常优雅、完备，尤其是在 iOS 平台，提供了视频录制、视频编辑、离线保存这些场景。

GPUImage 框架是使用 OpenGL ES 中的 GLSL 书写出了上述的各种 Shader。GLSL（OpenGL Shading Language）是 OpenGL 的着色器语言，也是 2.0 版本中最出色的功能。开发人员可以使用这种语言编写程序运行在 GPU 上来进行图像处理或渲染。使用 GLSL 写的代码最终可以编译、链接成为一个 GLProgram。一个 GLProgram 分为两部分，一是顶点着色器（Vertex Shader），二是片元着色器（Fragment Shader），这两部分分别完成各自在 OpenGL 渲染管线中的功能。

我们平时说的点、直线、三角形都是几何图元，是在顶点着色器中指定的，用这些几何图元创建的物体叫做模型，而根据这些模型创建并显示图像的过程就是我们所说的渲染。在渲染过程结束之后，图像就会以像素点的形式绘制到屏幕上了。

OpenGL 的渲染管线具体是指什么呢？其实就是 OpenGL 引擎把图像（内存中的 RGBA 数据）一步步渲染到屏幕上去的步骤。

函数原型中的参数 shaderType 有两种类型：  
一是 GL_VERTEX_SHADER，创建顶点着色器时开发者应传入的类型；  
二是 GL_FRAGMENT_SHADER，创建片元着色器时开发者应传入的类型。

iOS 平台的上下文环境搭建：iOS 平台使用的是 EAGL，与 EGL 的双缓冲机制比较类似，iOS 平台也不允许我们直接渲染到屏幕上，而是渲染到一个叫 renderBuffer 的对象上。这个对象你可以理解为一个特殊的 FrameBuffer，最终再调用 EAGLContext 的 presentRenderBuffer 方法，就可以将渲染结果输出到屏幕上去了。RenderBuffer 需要绑定一个 CAEAGLLayer，而这个 Layer 实际上就可以一对一地关联到我们自定义的一个 UIView。
# 三、播放器
播放器要提供哪些功能给用户？最基本的功能自然是从零开始播放视频，能听到声音、看到画面，并且声音和画面是要对齐的，然后还需要支持暂停和继续播放功能；另外，需要支持 seek 功能，即可以随意拖动到任意位置，并立即从这个位置继续播放；高级一点的也会支持切换音轨（如果视频中有多个音轨的话）、添加字幕等功能。  
输入资源有可能是不同的协议，比如本地磁盘的文件（file）或者是 HTTP、RTMP、HLS 等协议，也有可能是不同的封装格式，比如 MP4、FLV、MOV。这些封装格式里通常会有两个 Stream（轨道 / 流），分别是音频流（轨道）和视频流（轨道）。每个轨道里面存储的都是压缩后的编码格式，音频一般为 AAC、视频一般为 H264。  
输入模块，负责将多媒体文件处理成音视频裸数据；音视频的队列负责存储音视频的裸数据；音视频同步模块，负责音视频的同步；音频输出与视频输出模块负责音视频的渲染。
## 音视频输入模块（解码模块）
从输入文件到最终得到裸数据，会经历解析协议、解封装、解码三个步骤。  
输入源和解码线程  
FFmpeg 中的 libavformat 模块可以处理各种不同的协议以及不同的封装格式，先用 libavformat 模块把文件解封装成每一路流，之后再进行解码。
FFmpeg 在解码场景下的核心流程：  
建立连接、准备资源阶段：使用 openInput 方法向外提供接口。  
读取数据进行拆封装、解码、处理数据阶段：使用 decodeFrames 方法向外提供接口。  
释放资源阶段：使用 releaseResource 方法向外提供接口。
## 音频输出模块
不论音频的输出还是视频的输出，都需要用一个独立的线程来管理，这两个线程会先去输入模块管理的队列中拿出音视频的裸数据，然后分别进行音视频的渲染，最终让用户听得到声音、看得到画面。  
iOS 平台，比较常见的就是 AudioQueue 和 AudioUnit，AudioQueue 是更高层次的音频 API，是建立在 AudioUnit 的基础之上的，提供的 API 更加简单，在这里选用 AudioQueue 其实也是可以的，但是我们最终选择了 AudioUnit，首先是因为音频渲染过程中有可能存在音频格式的转换，这时使用 AudioUnit 会更加方便；其次我们也要为后续的录音、音效处理等打下使用 AudioUnit 的基础。
## 视频输出模块
我们可以利用 OpenGL ES 处理图像的巨大优势，来对视频做一个后处理，比如增加去块滤波器、对比度等效果器，让用户感觉视频更加清晰。  
在 iOS 平台使用 EAGL 来为 OpenGL ES 提供上下文环境，自己定义一个继承自 UIView 的 View，使用 EAGLLayer 作为渲染对象，最终渲染到这个自定义的 View 上。  
解码模块内部使用 FFmpeg 实现，但是对外界会隐藏内部的实现，提供统一的封装接口。音频播放模块也会隐藏内部的实现，提供一个获取 PCM 数据的方法来拉取要播放的数据。画面播放模块直接获取出 YUV 数据使用 OpenGL ES 进行绘制，需要注意的是所有 OpenGL ES 的操作必须要在构建的 GL 线程中进行操作。
## 音视频同步模块
使用 PThread 来创建线程会是一个好的选择，原因是两个平台都支持这种线程模型。  
音视频同步模块向外界暴露获取音频数据、视频数据的接口，这两个接口提供数据的同时要保持同步。音视频同步模块在内部组装输入模块，负责解码线程的调度。然后我们把音视频同步模块、音频输出模块、视频输出模块封装到调度器模块中，调度器模块会分别向音频输出模块和视频输出模块注册回调函数，调度器模块的回调函数中就调用音视频同步模块来获取音频数据和视频数据。   
音视频同步的策略一般分为三种：音频向视频同步；视频向音频同步（业内常用）；音频视频统一向外部时钟同步。  
提供初始化接口，内部实现为：使用外界传递过来的 URI 去实例化解码器模块，实例化成功之后，创建音频队列与视频队列，并且创建解码线程，将音视频解码后的数据放入队列中；  
提供获取音频数据接口，内部实现为：如果音频队列中有音频就直接去返回，同时要记录下这个音频帧的时间戳，如果音频队列中没有音频就返回静音数据；  
提供获取视频帧接口，内部实现为：返回与当前播放的音频帧时间戳对齐的视频帧。  
提供销毁接口，内部实现为：先停掉解码线程，然后销毁解码器，最后再销毁音视频队列。    
## 控制器模块
在开始播放的时候，需要把资源的地址（有可能是本地的文件，也有可能是网络的资源文件）传递给 AVSynchronizer。如果能够成功地打开文件，那么就去实例化 VideoOutput 和 AudioOutput。  
在实例化这两个类的同时，要传入回调函数，这两个回调函数又分别去调用 AVSynchronizer 里获取音频和视频帧的方法，这样就可以有序地组织多个模块，最终如果暂停、继续的指令调用下来，也相应地去调用各个模块对应的生命周期方法。  

首先实现了输入模块，也叫做解码模块，输出的音频帧是 AudioFrame，里面的主要数据就是 PCM 裸数据，输出的视频帧是 VideoFrame，里面的主要数据就是 YUV420P 的裸数据。  
然后实现了音频播放模块，输入就是我们解码出来的 AudioFrame，是 SInt16 表示一个 sample 格式的数据，输出就是输出到 Speaker 让用户直接听到声音。
接着实现了视频播放模块，输入就是解码出来的 VideoFrame，里面存放的是 YUV420P 格式的数据，在渲染过程中使用 OpenGL ES 的 Program 将 YUV 格式的数据转换为 RGBA 格式的数据，并最终显示到物理屏幕上。
之后就是音视频同步模块了，它的工作主要由两部分组成，第一是负责维护解码线程，即负责输入模块的管理；另外一个就是音视频同步，向外部暴露填充音频数据的接口和获取视频帧的接口，保证提供出去的数据是同步的。
最后书写一个中控系统，负责将 AVSync 模块、AudioOutput 模块、VideoOutput 模块组织起来，最重要的就是维护这几个模块的生命周期，由于这里面存在多线程的问题，所以比较重要的就是在初始化、运行、销毁的各个阶段保证这几个模块可以协同有序地运行，同时中控系统向外暴露用户可以操作的接口，比如开始播放、暂停、继续、停止等接口。
# 四、音频录制与编码
AVAudioRecorder，类似于 AVAudioPlayer，属于端到端的 API，存在于 AVFoundation 框架中。当我们想指定一个路径将麦克风的声音录制下来的时候，就可以使用这一个 API。优点是简单易用，缺点是无法操控中间的数据。  
AudioQueue，之前我们使用 AudioQueue 渲染过音频，其实 AudioQueue 也可以录制音频，也是对 AudioUnit 的封装，它允许开发者获取、操控中间的数据（按照配置的数据格式）。优点是灵活性较强，缺点是上手难度较高。  
AudioUnit，是音频最底层的 API 接口，之前我们使用 AudioUnit 渲染过音频，和 AudioQueue 一样，我们也可以使用它录制音频。当我们需要使用 VPIO（VoiceProcessIO）等处理音频的 AudioUnit、需要使用实时耳返或在低延迟场景下，必须使用这一层的 API。优点是灵活性最强，缺点是上手难度更高。  
渲染音频是需要业务端来填充数据，然后给 AudioQueue 进行播放；采集音频是需要业务端把采集到的数据消费掉，然后再返回给 AudioQueue 来填充音频。  
AAC 编码的方式有两种，一种是使用软件编码，另一种是使用 Android 与 iOS 平台的硬件编码。软件编码的实现方式我们会基于 FFmpeg 来讲解，后期无论你想用什么格式，都可以自己配置编码库来实现，编码部分的代码是可以复用的。  
## AAC的编码规格
AAC 编码器常用的编码规格有三种，分别是 LC-AAC、HE-AAC、HE-AACv2。  
其中 LC-AAC 的 Profile 是最基础的 AAC 的编码规格，它的复杂度最低，兼容性也是最好的，双声道音乐在 128Kbps 的码率下可以达到全频带（44.1kHz）的覆盖。  
在 LC-AAC 的基础上添加 SBR 技术，形成 HE-AAC 的编码规格，SBR 全称是 Spectral Band Replication，其实就是消除频域上的冗余信息，可以在降低码率的情况下保持音质。内部实现原理就是把频谱切割开，低频单独编码保存，来保留主要的频谱部分，高频单独放大编码保存以保留音质，这样就保证在降低码率的情况下，更大程度地保留了音质。  
在 HE-AAC 的基础上添加 PS 技术，就形成了 HE-AACv2 的编码规格，PS 全称是 Parametric Stereo，其实就是消除立体声中左右声道之间的冗余信息，所以使用这个编码规格编码的源文件必须是双声道的。内部实现原理就是存储了一个声道的全量信息，然后再花很少的字节用参数描述另一个声道和全量信息声道有差异的地方，这样就达到了在 HE-AAC 基础上进一步提高压缩比的效果。  
## AAC 的封装格式
我们日常生活中常见的 AAC 编码的封装格式有两种，一种是 ADTS 封装格式，可以简单理解为以 AAC 为后缀名的文件，另外一种是 ADIF 封装格式，可以简单地理解为以 M4A 为后缀名的文件。  
ADIF 全称是 Audio Data Interchange Format，是 AAC 定义在 MPEG4 里面的格式，字面意思是交换格式，是将整个流的 Meta 信息（包括 AAC 流的声道、采样率、规格、时长）写到头部，解码器只有解析了头部信息之后才可以解码具体的音频帧，像 M4A 封装格式、FLV 封装格式、MP4 封装格式都是这样的。  
ADTS 全称是 Audio Data Transport Stream，是 AAC 定义在 MPEG2 里面的格式，含义就是传输流格式，特点就是从流中的任意帧位置都可以直接进行解码。这种格式实现的原理是在每一帧 AAC 原始数据块的前面都会加上一个头信息（ADTS+ES），形成一个音频帧，然后不断地写入文件中形成一个完整可播放的 AAC 文件。
## 使用软件编码器编码 AAC
我们用 FFmpeg 的 API 来编写的主要原因是，如果我们以后想使用别的编码格式，只需要调整相应的编码器 ID 或者编码器 Name 就可以了。原理就是 FFmpeg 帮我们透明掉了内部的细节，做了和各家编码器 API 对接的工作，给开发者暴露出了统一的面向 FFmpeg API 的接口。这里使用的编码器是 libfdk_aac，既然要使用第三方库 libfdk_aac，那么就必须在做交叉编译的时候，将 libfdk_aac 这个库编译到 FFmpeg 中去。  
由于我们想书写一个同时运行在 Andorid 平台和 iOS 平台上编码器工具类，所以构造一个 C++ 的类，叫做 audio_encoder，向外暴露三个接口，分别是初始化、编码以及销毁方法。  
## iOS 平台的硬件编码器 AudioToolbox
PCM 到 PCM：可以转换位深度、采样率以及表示格式，也包括交错存储和平铺存储之间的转换，这与 FFmpeg 里的重采样器非常类似。  
PCM 到压缩格式的转换：相当于一个编解码器，当然可以应用在编解码场景。
AudioToolbox 里编码出来的 AAC 数据也是裸数据，在写入文件之前也需要手动添加上 ADTS 头信息，最终写出的文件才可以被系统播放器播放。  
# 五、视频录制与编码
基于摄像头采集驱动，中间可以支持视频特效处理，最终用 OpenGL ES 渲染到 UIView 上，且支持扩展插入编码节点。  
凡是需要输入纹理对象的，都是 Input 类型。  
凡是需要向后级节点输出纹理对象的，都是 Output 类型。
AVCaptureSession  
这里是一定要转换为 RGBA 的，因为在 OpenGL 中通用的渲染纹理格式都是 RGBA 的，包括纹理处理、渲染到屏幕上以及最终编码器，都是以 RGBA 格式为基础进行转换和处理的。
## ELImageInput
因为需要别的组件给它输入纹理对象，所以这个 Protocol 里面定义了两个方法，第一个方法是设置输入的纹理对象。  
第二个方法是执行渲染操作。
## ELImageOutput
这个类可以向自己的后级节点输出目标纹理对象，其中 Camera、Filter 节点是需要继承自这个类。根据这个特点我们建立两个属性，一个是渲染目标的纹理对象，一个是后级节点列表。
## ELImageProgram
把 GLProgram 的构建、查找属性、使用等这些操作以面向对象的形式封装起来，每一个节点都会组合这个类。
## ELImageTextureFrame
每一个节点的输入都是一个纹理对象（实际上是一个纹理 ID），使用 GLProgram 将这个纹理对象渲染到一个目标纹理对象的时候，还需要建立一个帧缓存对象（FBO），并且要将这个目标纹理对象 Attach 到这个帧缓存对象上。将纹理对象和帧缓存对象的创建、绑定、销毁等操作，以面向对象的方式封装起来，让每一个节点使用起来都更加方便。
## ELImageContext
封装 EAGLContext 和渲染线程  
一个 CMSampleBuffer 由以下三部分组成：
CMTime，代表这一帧图像的时间。
CMVideoFormatDescription，代表对这一帧图像格式的描述。
CVPixelBuffer，代表这一帧图像的具体数据。  
在渲染过程中需要注意两点，一是确定物体坐标和纹理坐标；二是在 FragmentShader 中，怎么把 YUV 转换成 RGBA 的表示格式。  
这里是一定要将 YUV 转换为 RGBA 的，因为在 OpenGL 中通用的渲染纹理格式都是 RGBA 的，包括纹理处理、渲染到屏幕上以及最终编码器，都是以 RGBA 格式为基础进行转换和处理的。
