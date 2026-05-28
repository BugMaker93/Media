# 没有整理！！！



<img width="2469" height="3840" alt="image" src="https://github.com/user-attachments/assets/1cb7076a-3068-4037-b501-3fcc7734065b" />

<img width="2377" height="2333" alt="image" src="https://github.com/user-attachments/assets/5a1ff9a5-00ad-4d73-849b-c348a3591865" />



# NuPlayerDecoder:NuPlayerDecoderBase
Decoder和Codec IPC通信。Decoder事件会回调给NuPlayer。也会和Render通信。

Decoder 解码出一帧 buffer，调用 Renderer::queueBuffer()，并传入一个 notifyConsumed 消息（AMessage），这个消息的 handler 是解码器自己。

Renderer 渲染完该帧后，调用 entry->mNotifyConsumed->post()，即把消息发回解码器。

Decoder 收到该消息后，执行 buffer 释放、解码下一个帧等后续操作。

NuPlayer::instantiateDecoder()：创建Decoder的位置。

# NuPlayerRender
Render事件会回调给NuPlayer。也会如上和Decoder通信。



Renderer 负责将解码后的音频数据送到音频输出设备（如 AudioSink），将视频帧送到显示表面（如 Surface），并确保音视频同步播放。

对于视频输出：NuPlayer::Renderer::onQueueBuffer()->NuPlayer::Renderer::postDrainVideoQueue()->NuPlayer::Renderer::onDrainVideoQueue()->MediaCodec::renderOutputBufferAndRelease()->Android-Graphic

对于音频输出：NuPlayer::Renderer::onQueueBuffer()->NuPlayer::Renderer::postDrainAudioQueue_l()->NuPlayer::Renderer::onDrainAudioQueue()->AudioSink::write()



Renderer 通常维护一个媒体时钟（MediaClock），用于协调音视频帧的输出时序，实现精准同步。

NuplayerRender记录写入多少Audio数据，从AudioSink获取当前Audio时间戳信息，向MediaClock打入锚点信息，视频以此时间计算Render时间，如果视频帧的render过时40ms，render标记置为false，该帧将被discard。对于Audio buffer，调用MediaCodec releaseOutputBuffer，直接discard掉。

# NuplayerCCDecoder
CCDecoder 只有Video有。

CCDecoder 负责解码和处理视频流中的隐藏字幕（Closed Caption，CC）数据，将其提取出来并在需要时显示在屏幕上。

CCDecoder 需要解析视频帧中的特定数据区（如 SEI、user data、VBI 等）来提取字幕信息。



# MediaCodec：：onInputBufferAvailable()/onOutputBufferAvailable()

onInputBufferAvailable()：MediaCodec可以给codec decode前数据。

onOutputBufferAvailable()：codec有decode后数据给MediaCodec。


后续refer to [AOSP - MediaCodecService.md](https://github.com/BugMaker93/Media/blob/main/AOSP%20-%20MediaCodecService.md)
