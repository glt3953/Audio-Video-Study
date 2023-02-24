# 在 iOS 平台采集音频数据，比较常用的就是 AVAudioRecoder，AudioQueue 以及 AudioUnit 三套接口。
AVAudioRecorder 使用起来比较简单，如果是简单的录音，使用 AVAudioRecorder 就可以。但因为无法操控中间的数据，它提供不了更高级的能力支持。
使用 AudioQueue 录制音频，灵活性比较强。如果只是获取内存中的录音数据，然后编码、输出，使用 AudioQueue 来采集音频会更适合。
使用 AudioUnit 录制音频灵活性最强，如果要使用更多的音效处理以及实时的监听功能，那么使用 AudioUnit 会更方便一些。
