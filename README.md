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
