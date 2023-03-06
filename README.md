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
## 音视频输入模块
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
## 音视频同步模块
使用 PThread 来创建线程会是一个好的选择，原因是两个平台都支持这种线程模型。  
音视频同步模块向外界暴露获取音频数据、视频数据的接口，这两个接口提供数据的同时要保持同步。音视频同步模块在内部组装输入模块，负责解码线程的调度。然后我们把音视频同步模块、音频输出模块、视频输出模块封装到调度器模块中，调度器模块会分别向音频输出模块和视频输出模块注册回调函数，调度器模块的回调函数中就调用音视频同步模块来获取音频数据和视频数据。   
音视频同步的策略一般分为三种：音频向视频同步；视频向音频同步（业内常用）；音频视频统一向外部时钟同步。  
## 控制器模块
在开始播放的时候，需要把资源的地址（有可能是本地的文件，也有可能是网络的资源文件）传递给 AVSynchronizer。如果能够成功地打开文件，那么就去实例化 VideoOutput 和 AudioOutput。  
在实例化这两个类的同时，要传入回调函数，这两个回调函数又分别去调用 AVSynchronizer 里获取音频和视频帧的方法，这样就可以有序地组织多个模块，最终如果暂停、继续的指令调用下来，也相应地去调用各个模块对应的生命周期方法。  

解码模块内部使用 FFmpeg 实现，但是对外界会隐藏内部的实现，提供统一的封装接口。音频播放模块也会隐藏内部的实现，提供一个获取 PCM 数据的方法来拉取要播放的数据。画面播放模块直接获取出 YUV 数据使用 OpenGL ES 进行绘制，需要注意的是所有 OpenGL ES 的操作必须要在构建的 GL 线程中进行操作。
