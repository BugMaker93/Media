# ACodec/CCodec
OMX的注册service：main_codecservice.cpp

CCodec的注册service：main_swcodecservice.cpp

<img width="890" height="398" alt="image" src="https://github.com/user-attachments/assets/c8598ea7-91d8-43de-895e-487ad38b7fa6" />

现在使用的是CCodec。**so的实现**：Android/frameworks/av/media/codec2/components/...

# launch CCodec

<img width="1960" height="847" alt="image" src="https://github.com/user-attachments/assets/24abad31-02a5-4e4b-91e9-c644430d650c" />

## IComponentStore
>用于IPC

<img width="1252" height="745" alt="image" src="https://github.com/user-attachments/assets/b7df1347-2d8d-40d0-8055-fa0882a5c1db" />

## C2Component

<img width="1611" height="815" alt="image" src="https://github.com/user-attachments/assets/b35e5b8a-0e16-450a-9a99-cae10a2a9a4a" />

硬解一定会继承C2Component，Qcom不会提供硬解的源码，samsung源码在：Android/hardware/samsung_slsi/codec2/...

## ComponentModule/ComponentLoader
ComponentLoader中有所以ccodec.so,以ComponentLoader形式对外部提供。

# start
>libstagefright中MediaCodec是MediaPlayer和CCodec的分界点。

<img width="3757" height="2427" alt="image" src="https://github.com/user-attachments/assets/21bca2f0-a31c-4ae9-a3cd-dd4ea28c90bb" />

prepare时MediaExtractor解复用的数据通过AnotherPacketSource.queueAccessUnit()存储在GenericSource。

## onInputBufferAvailable
- 通过onInputBufferAvailable(),libstagefright得知CCodec需要获得解复用的数据。

- GenericSource通过AnotherPacketSource.dequeueAccessUnit()，把解复用数据给到CCodec。

<img width="1611" height="1165" alt="image" src="https://github.com/user-attachments/assets/4637f47d-2148-4e50-9bdc-293fd1539fad" />

CCodecCallbackImpl：CCodecCallback的作用是通过CCodec调用到MediaCodec::CodecCallback：CodecBase::CodecCallback。

onInputBufferAvailable()怎么来的？onInputBufferAvailable()中的buffer转换？queueInputBuffer()是怎么触发onOutputBufferAvailable()的？refer to [Codec.md](https://github.com/BugMaker93/Media/blob/main/Codec.md)

## onOutputBufferAvailable
- 通过onOutputBufferAvailable()，libstagefright得知CCodec已经把解复用数据decode完成，并把decode完成的的数据传回来。

<img width="2042" height="2569" alt="image" src="https://github.com/user-attachments/assets/e43954cb-9cbc-4fa9-8927-94ee709879a0" />

onOutputBufferAvailable()怎么来的？onInputBufferAvailable()/onOutputBufferAvailable()完美闭环中buffer的转换？refer to [Codec.md](https://github.com/BugMaker93/Media/blob/main/Codec.md)

Audio会call到AudioTrack。Video会call到IGraphicBufferProducer。IGraphicBufferProducer后续流程refer to [Graphic.md](https://github.com/BugMaker93/Media/blob/main/Graphic.md)

# class

## NuPlayerDecoder:NuPlayerDecoderBase
NuPlayerDecoder和CCodec IPC通信。NuPlayerDecoder事件会回调给NuPlayerRender。也会和NuPlayerRender通信。

NuPlayerDecoder解码出一帧buffer，调用NuPlayerRender::queueBuffer()，并传入一个notifyConsumed 消息（AMessage），这个消息的handler是NuPlayerDecoder。

NuPlayerRender渲染完该帧后，调用 entry->mNotifyConsumed->post()，即把消息发回NuPlayerDecoder。

NuPlayerDecoder收到该消息后，执行buffer释放、解码下一个帧等后续操作。

NuPlayer::instantiateDecoder()：创建Decoder的位置。

## NuPlayerRender
NuPlayerRender事件会回调给NuPlayer。也会和NuPlayerDecoder通信。

NuPlayerRender负责将解码后的音频数据送到音频输出设备（如 AudioSink），将视频帧送到显示表面（如 Surface），并确保音视频同步播放。

对于视频输出：NuPlayer::Renderer::onQueueBuffer()->NuPlayer::Renderer::postDrainVideoQueue()->NuPlayer::Renderer::onDrainVideoQueue()->MediaCodec::renderOutputBufferAndRelease()->Android-Graphic

对于音频输出：NuPlayer::Renderer::onQueueBuffer()->NuPlayer::Renderer::postDrainAudioQueue_l()->NuPlayer::Renderer::onDrainAudioQueue()->AudioSink::write()

NuPlayerRender通常维护一个媒体时钟（MediaClock），用于协调音视频帧的输出时序，实现精准同步。

NuplayerRender记录写入多少Audio数据，从AudioSink获取当前Audio时间戳信息，向MediaClock打入锚点信息，视频以此时间计算Render时间，如果视频帧的render过时40ms，render标记置为false，该帧将被discard。

## NuplayerCCDecoder
NuplayerCCDecoder只有Video有。

NuplayerCCDecoder负责解码和处理视频流中的隐藏字幕（Closed Caption，CC）数据，将其提取出来并在需要时显示在屏幕上。

NuplayerCCDecoder需要解析视频帧中的特定数据区（如 SEI、user data、VBI 等）来提取字幕信息。

## MediaCodec：：onInputBufferAvailable()/onOutputBufferAvailable()

onInputBufferAvailable()：MediaCodec给CCodec **decode前**数据。

onOutputBufferAvailable()：CCode把**decode后**数据给MediaCodec。
