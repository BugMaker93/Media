# MediaCodec.Buffer
>libstagefright

主要关注CCodecBufferChannel.Buffer，MediaCodec.Buffer来自CCodecBufferChannel.Buffer。MediaCodec.Buffer关注mPortBuffers/mAvailPortBuffers。


# CCodecBufferChannel.Buffer
>codec2

<img width="1586" height="498" alt="image" src="https://github.com/user-attachments/assets/4b389dfe-82fc-4354-861e-99b85f9a982a" />

Codec2Buffer同时持有ABuffer和C2Buffer。ABuffer和C2Buffer会内存映射，**操作ABuffer想到与操作C2Buffer**。

## MediaCodecBuffer
>android/frameworks/av/media/libmedia/include/media/MediaCodecBuffer.h

```
class MediaCodecBuffer{

public:
    MediaCodecBuffer(const sp<AMessage> &format, const sp<ABuffer> &buffer);

    virtual std::shared_ptr<C2Buffer> asC2Buffer() { return nullptr; }

    ...
}

```


## Codec2Buffer
>android/frameworks/av/media/codec2/sfplugin/Codec2Buffer.h

```
class Codec2Buffer : public MediaCodecBuffer {
protected:
    bool copyLinear(const std::shared_ptr<C2Buffer> &buffer);

...
}
```

#


C2ComponentStore：我能提供哪些 codec 组件，怎么创建它们
C2AllocatorStore：我能提供哪些 allocator，按 id 取哪个
C2Allocator：底层内存到底怎么分
C2BlockPool：把 allocator 包装成可复用的 block 来源，供组件拿 buffer
