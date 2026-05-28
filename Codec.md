# ACodec/CCodec
OMX的注册service：main_codecservice.cpp

CCodec的注册service：main_swcodecservice.cpp

<img width="890" height="398" alt="image" src="https://github.com/user-attachments/assets/c8598ea7-91d8-43de-895e-487ad38b7fa6" />

现在使用的是CCodec。

# launch CCodec

<img width="1960" height="847" alt="image" src="https://github.com/user-attachments/assets/24abad31-02a5-4e4b-91e9-c644430d650c" />

## IComponentStore
>用于IPC

<img width="1252" height="745" alt="image" src="https://github.com/user-attachments/assets/b7df1347-2d8d-40d0-8055-fa0882a5c1db" />

## C2Component

<img width="1611" height="815" alt="image" src="https://github.com/user-attachments/assets/b35e5b8a-0e16-450a-9a99-cae10a2a9a4a" />

## ComponentModule/ComponentLoader
ComponentLoader中有所以ccodec.so,以ComponentLoader形式对外部提供。

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
