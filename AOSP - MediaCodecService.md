# Service
>Android\frameworks\av\services\mediacodec

**OMX的注册service**：Android/frameworks/av/services/mediacodec/main_codecservice.cpp

**CCodec的注册service**:Android/frameworks/av/services/mediacodec/main_swcodecservice.cpp

通常**实现service**会以xxxxservice命名，例如MediaExtractorService.cpp被注册到main_extractorservice.cpp

但也有例外，例如MediaCodec。所以注意registerService/addService(),只要传入Bn就行。

# Sequence

## service start
<img width="1109" height="479" alt="image" src="https://github.com/user-attachments/assets/77cf07a5-48ee-4d35-b108-09ef4cb0cfe6" />

**C2Store.cpp**：Android\frameworks\av\media\codec2\vndk\C2Store.cpp，其中定义了**C2PlatformComponentStore**。

<img width="1252" height="745" alt="image" src="https://github.com/user-attachments/assets/50f2d067-da28-4e8b-9ce6-296987559cbc" />

HIDL是什么，refer to [AOSP -HIDL.md](https://github.com/BugMaker93/Media/blob/main/AOSP%20-HIDL.md)

HIDL设计决定C2ComponentStore作为ComponentStore构造方法的参数传入。可以理解C2ComponentStore是ComponentStore的utils，namely不以实现方式的实现。

<img width="1611" height="815" alt="image" src="https://github.com/user-attachments/assets/1fee2a04-9e3a-4406-a745-9067de264e73" />

HIDL设计决定C2Component也是作为IComponent的构造方法的参数传入。

C2Component对应所有软解的实现。

## allocate component

MediaCodec:android/frameworks/av/media/libstagefright/MediaCodec.cpp

以MediaCodec为分界，右边都是c2codec

<img width="2479" height="1813" alt="image" src="https://github.com/user-attachments/assets/c42c1632-d085-4a95-b16a-5cdb0c884de7" />

Codec2Clinet/Codec2Clinet：：Component属于codec2，codec2和hidl打交道。

Codec2Clinet对应ComponentStore。

Codec2Clinet：：Component对应Component。

CCodecBufferChannel：管理输入/输出 buffer。需要特别关注`onInputBufferAvailable()/onOutputBufferAvailable()`的来源。
