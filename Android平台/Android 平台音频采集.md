# Android 平台的音频采集技术选型
## SDK 层提供的采集方法
Android SDK 提供了两套音频采集的 API 接口，分别是 MediaRecorder 和 AudioRecord。前者是一个端到端的 API，它可以直接把手机麦克风录入的音频数据进行编码压缩（如 AMR、MP3 等）并存储成文件；而后者则更底层一些，可以让开发者更加灵活地控制麦克风采集到的 PCM 数据。
如果想简单地做一个录音机，并且录制出一个音频文件，首选肯定是 MediaRecorder，而如果需要对音频做进一步的算法处理，或者需要采用第三方的编码库进行编码压缩，那么就需要使用 AudioRecord 了。我们的视频录制器场景显然更适合选用第二种方式，使用 AudioRecord 来采集音频。
## NDK 层提供的采集方法
Android NDK 也提供了两套音频采集的 API 接口，就是 OpenSL ES 和 AAudio，其中 AAudio 是在 Android 8.0 系统以上才可以使用的。这两个 API 接口也都属于底层接口，可以获取实时采集到的 PCM 数据。相比于 AudioRecord，Native 层提供的采集方法具有更低的延迟与更高的性能，但是由于 Android 设备与系统厂商碎片化的问题，兼容性比不上 AudioRecord，比如某些品牌的手机上会有声音采集音量过小，采集的音频有杂音等问题。
如果在一些需要低延迟音频采集的场景比较适合使用这两个音频采集的方法，比如实时耳返场景、更低延迟的 VOIP 场景等。如果想为自己的工程构建音频采集框架，是需要同时支持 SDK 层 AudioRecord 采集和 Native 层采集的。而在 Native 层直接使用 Google 开源的 Oboe 框架是一个比较好的选择，因为它屏蔽掉了 OpenSL ES 与 AAudio 的内部实现细节，提供了统一的 API 接口。所以在这节课的最后一部分，我会以 Oboe 为例给你讲解 Native 层录制音频的方法。

Android 平台音频采集的技术选型，在 SDK 层和 NDK 层各有两套音频采集方法，考虑到我们的视频录制器的场景，我们选取了 SDK 层的 AudioRecord 和 Native 层的 Oboe 采集音频的方法。
AudioRecord 采集音频主要分为配置参数，初始化内部音频缓冲区、采集、提取数据，以及停止采集，释放资源四步。
使用 Oboe 录制音频也分为四步，分别是创建 Audio Stream、录制、启动线程读取音频、停止录音时关闭 Audio Stream。
