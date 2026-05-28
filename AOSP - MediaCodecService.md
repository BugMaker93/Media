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

HIDL设计决定C2ComponentStore作为ComponentStore构造方法的参数传入。可以理解C2ComponentStore是ComponentStore的utils，namely不以实现方式的实现。

<img width="1611" height="815" alt="image" src="https://github.com/user-attachments/assets/1fee2a04-9e3a-4406-a745-9067de264e73" />

HIDL设计决定C2Component也是作为IComponent的构造方法的参数传入。

C2Component对应所有软解的实现。

## allocate component

MediaCodec:android/frameworks/av/media/libstagefright/MediaCodec.cpp

以MediaCodec为分界，右边都是c2codec

<img width="2479" height="1813" alt="image" src="https://github.com/user-attachments/assets/c42c1632-d085-4a95-b16a-5cdb0c884de7" />

承接

<img width="1509" height="500" alt="image" src="https://github.com/user-attachments/assets/8c74dbfd-5191-49f0-b84b-dd58c8c7f2b6" />

Codec2Clinet/Codec2Clinet：：Component属于codec2，codec2和hidl打交道。

Codec2Clinet对应ComponentStore。

Codec2Clinet：：Component对应Component。

CCodecBufferChannel：管理输入/输出 buffer。需要特别关注`onInputBufferAvailable()/onOutputBufferAvailable()`的来源。

<img width="1241" height="917" alt="image" src="https://github.com/user-attachments/assets/21c2a699-3f6d-478c-980c-f350ada470af" />

MediaCodec.CreateByType()中会去MediaCodecList中找支持的codec。

MediaCodec/CCodec/so是一一对应，这里通过componentName找到对应codec，并new MediaCodec。

## configure component

<img width="2464" height="1387" alt="image" src="https://github.com/user-attachments/assets/bb045d43-cd7c-4d6c-a7f7-d128d7b37793" />

承接

<img width="1416" height="447" alt="image" src="https://github.com/user-attachments/assets/d9139a38-cf89-4df0-aac7-9317a5ae33ba" />

暂无重点。

## start

<img width="1818" height="1063" alt="image" src="https://github.com/user-attachments/assets/209dd0ae-a8e6-41b1-b851-139806d867c1" />

承接

<img width="1454" height="488" alt="image" src="https://github.com/user-attachments/assets/be5163a2-8b1e-411b-afee-979cd96661a4" />

CCodecBufferChannel：管理输入/输出 buffer。这里是输入onInputBufferAvailable()。MediaCodec在收到onInputBufferAvailable()后会queueInputbuffer()。

## CCodecBufferChannel start

<img width="1463" height="2037" alt="image" src="https://github.com/user-attachments/assets/2dbb994f-4cef-4667-8ad9-e6df67a1fa4d" />

针对CCodecBufferChannel更详细的start。

## buffer transfer

<img width="1746" height="1128" alt="image" src="https://github.com/user-attachments/assets/9cb818ee-25f3-4008-86db-bc89cac49652" />

MediaCodec在收到onInputBufferAvailable()后会调用queueInputbuffer()。

MediaCodec调用queueInputbuffer()后，会在Codec decode后，MediaCodec收到onOutputBufferAvailable()。

NuPlayerDecoder处理完output buffer后，调用MediaCodec：：renderOutputBufferRelease()，最后MediaCodec又会收到onInputBufferAvailable()。形成buffer的闭环。

## load all C2 Codec Services

<img width="792" height="903" alt="image" src="https://github.com/user-attachments/assets/bd8c40d9-270c-4495-9758-5b11c1519d2b" />

# Buffer
>主要CCodecBufferChannel中Buffer，MediaCodec中Buffer也来自CCodecBufferChannel中Buffer，MediaCodec只是自己维护了下，可关注MediaCodec.mPortBuffers/mAvailPortBuffers

## InputBuffers
```
//Android\frameworks\av\media\codec2\sfplugin\CCodecBuffers.h
class CCodecBuffers{
    virtual bool isArrayMode() const { return false; }
}

class InputBuffers ：public CCodecBuffers{
	
}

//有许多具有FlexBuffersImpl的Buffers继承InputBuffers，且isArrayMode都为false。后续都以GraphicInputBuffers为例。
class GraphicInputBuffers : public InputBuffers {
    FlexBuffersImpl mImpl;
}

class FlexBuffersImpl {
    struct Entry {
        sp<Codec2Buffer> clientBuffer;
        std::weak_ptr<C2Buffer> compBuffer;
    };
    std::vector<Entry> mBuffers;
}

//只有一个具有BuffersArrayImpl的Buffers继承InputBuffers且isArrayMode都为true。
class InputBuffersArray : public InputBuffers {

    BuffersArrayImpl mImpl;

    bool isArrayMode() const final { return true; }
}

class BuffersArrayImpl {
    struct Entry {
        const sp<Codec2Buffer> clientBuffer;
        std::weak_ptr<C2Buffer> compBuffer;
		//比FlexBuffersImpl多的属性
        bool ownedByClient;
    };
    std::vector<Entry> mBuffers;
}
```

## OutputBuffers
```
//Android\frameworks\av\media\codec2\sfplugin\CCodecBuffers.h
class OutputBuffers : public CCodecBuffers {
}

//有许多具有FlexBuffersImpl的Buffers继承InputBuffers，且isArrayMode都为false。后续都以FlexOutputBuffers为例。
class FlexOutputBuffers : public OutputBuffers {
	FlexBuffersImpl mImpl;
}

//只有一个具有BuffersArrayImpl的Buffers继承OutputBuffers且isArrayMode都为true。
class OutputBuffersArray : public OutputBuffers {
	//区别于FlexBuffersImpl 
    BuffersArrayImpl mImpl
}
```

## 创建GraphicInputBuffers.Entry.Codec2Buffer
内部预创建的Entry.Codec2Buffer，为了MediaCodec.mPortBuffers/mAvailPortBuffers，即为了外部dequeueInputBuffers()时能获取index。

```
CCodec::initiateStart()
CCodecBufferChannel::prepareInitialInputBuffers()
GraphicInputBuffers::requestNewBuffer()
GraphicInputBuffers::createNewBuffer()
AllocateInputGraphicBuffer()
FlexBuffersImpl::assignSlot()
CCodecBufferChannel::requestInitialInputBuffers()
//操作MediaCodec.mPortBuffers/mAvailPortBuffers，并向外通知，例如通知MediaPlayer。
MediaCodec::onInputBufferAvailable()
```

## 创建InputBuffersArray.Entry
内部创建真正的Entry.Codec2Buffer，外部getInputBuffers()返回的是这个。

```
CCodecBufferChannel::getInputBufferArray()
//GraphicInputBuffers.Entry->InputBuffersArray.Entry,转换来的ownedByClient设置为true，新添加的为false
GraphicInputBuffers::toArrayMode() 
InputBuffersArray::initialize()
BuffersArrayImpl::initialize()
BuffersArrayImpl::getArray()
```

## 填充InputBuffersArray.Entry
外部填充了Entry.Codec2Buffer后，内部转成Entry.C2Buffer，并送到Service进行处理。

```
CCodecBufferChannel::queueInputBufferInternal()
//把对应的Codec2Buffer转换成C2Buffer，ownedByClient设置为false
InputBuffersArray::releaseBuffer()
BuffersArrayImpl::returnBuffer()
GraphicBlockBuffer::asC2Buffer()
C2Buffer::CreateGraphicBuffer()
//后面会专门讲PipelineWatcher作用
PipelineWatcher::onWorkQueued()
//C2Buffer会放入C2Work，把C2Work传给Component，不专门讲C2Work，只需要知道C2work.C2FrameData.C2Buffer中是处理前数据，C2work.C2Worklet.C2FrameData.C2Buffer是处理后数据
Component::queue()
```

## 填充OutputBuffersArray.Entry

```
//Component::queue()回调到此方法,关注Mutexed<std::list<std::unique_ptr<C2Work>>> mWorkDoneQueue
CCodec::onWorkDone()
CCodecBufferChannel::onWorkDone()
CCodecBufferChannel::handleWork()
//对应PipelineWatcher::onWorkQueued()
PipelineWatcher：：onWorkDone()
OutputBuffers::pushToStash()
CCodecBufferChannel::sendOutputBuffers()
OutputBuffers::popFromStashAndRegister()
/*
 *可能调用FlexOutputBuffers::registerBuffer()也可能调用OutputBuffersArray::registerBuffer()。
 *BuffersArrayImpl::grabBuffer()把C2Buffer转成Codec2Buffer，并把OutputBuffersArray中(!Entry[i].ownedByClient && Entry[i].compBuffer.expired())的Entry找到，把转成的Codec2Buffer放入Entry.Codec2Buffer，并把ownedByClient设置成true。
*/
FlexOutputBuffers::registerBuffer()->FlexBuffersImpl::assignSlot()/OutputBuffersArray::registerBuffer()->BuffersArrayImpl::grabBuffer()
//操作MediaCodec.mPortBuffers/mAvailPortBuffers，并向外通知，例如通知MediaPlayer。
MediaCodec::onOutputBufferAvailable()
```

## 消费后处理InputBuffersArray.Entry
InputBuffersArray.Entry.C2Buffer交给Component处理后，回调到CCodec::onWorkDone()，相当于此InputBuffersArray.Entry已经被消费。

通过CCodecBufferChannel::feedInputBufferIfAvailableInternal()让InputBuffersArray.Entry能再次被外部使用。内部调用feedInputBufferIfAvailableInternal()的时机很多，这只是其中之一。

```
CCodecBufferChannel::feedInputBufferIfAvailableInternal()
InputBuffersArray::requestNewBuffer()
BuffersArrayImpl::grabBuffer()
MediaCodec::onInputBufferAvailable()
```

## 消费后处理OutputBuffersArray.Entry
外部使用完OutputBuffersArray.Entry,即消费完OutputBuffersArray.Entry后。

外部需要调用releaseOutputBuffer()让OutputBuffersArray.Entry能再次被内部使用。

```
//外部调用releaseOutputBuffer()回调到这里
MediaCodec::onReleaseOutputBuffer()
//没有Surface相当于外部消费完了直接discard，有Surface，则等Surface消费完再discard，这里只写discardBuffer流程
CCodecBufferChannel::discardBuffer()/CCodecBufferChannel::renderOutputBuffer()
OutputBuffersArray::releaseBuffer()
BuffersArrayImpl::returnBuffer()
```

# ps

## PipelineWatcher作用
>作用在InputBuffer，和OutputBuffer无关

BuffersArrayImpl和FlexBuffersImpl中C2Buffer是**弱引用**，C2Buffer是否expired取决于外部强引用。PipelineWatcher则以强引用持有这个弱引用。

需关注

- 上面个流程中PipelineWatcher::onWorkQueued()/PipelineWatcher：：onWorkDone()时机

- CCodecBufferChannel::feedInputBufferIfAvailableInternal()中通过PipelineWatcher控制速度

## ownedByClientt

| type | ownedByClient=true | ownedByClient=false |
|---|---|---|
| InputBuffersArray | 可供外部使用。外部需要填充。 | 外部已经使用完毕。外部填充完毕，内部需要消费。 |
| OutputBuffersArray | 可供外部使用。内部填充完毕，外部需要消费。 | 外部已经使用完毕。外部消费完毕，内部需要再次填充。 |

## C2PlatformComponentStore::ComponentModule::init
dlopen加载so点地方

<img width="1481" height="1605" alt="image" src="https://github.com/user-attachments/assets/4303a240-3b61-4cc7-9e55-c760ec0296fa" />

Android\frameworks\av\media\codec2\vndk\C2Store.cpp

## OutputBuffers中mReorderStash和mPending的作用
```
// 数据流向图
Codec --|-> mReorderStash --> mPending --|-> slots --|-> client
        |                                |           |
  pushToStash()                    popFromStashAndRegister()
```

```
// 缓冲区条目结构
struct StashEntry {
    std::shared_ptr<C2Buffer> buffer;    // C2Buffer数据
    bool notify;                         // 是否通知客户端
    int64_t timestamp;                   // 时间戳
    int32_t flags;                       // 标志位
    sp<AMessage> format;                 // 格式信息
    C2WorkOrdinalStruct ordinal;         // 排序键值
};

// 两个关键成员
std::list<StashEntry> mPending;          // FIFO队列，无大小限制
std::list<StashEntry> mReorderStash;     // 有序列表，有大小限制
uint32_t mDepth{0};                      // mReorderStash的大小限制
C2Config::ordinal_key_t mKey{C2Config::ORDINAL}; // 排序键类型
```

```
void OutputBuffers::pushToStash(
        const std::shared_ptr<C2Buffer>& buffer,
        bool notify,
        int64_t timestamp,
        int32_t flags,
        const sp<AMessage>& format,
        const C2WorkOrdinalStruct& ordinal) {
    
    // 查找插入位置 - 维持有序
    auto it = mReorderStash.begin();
    for (; it != mReorderStash.end(); ++it) {
        if (less(ordinal, it->ordinal)) {  // 按ordinal排序
            break;
        }
    }
    // 有序插入
    mReorderStash.emplace(it, buffer, notify, timestamp, flags, format, ordinal);
    
    // 溢出处理 - 超过深度限制时移到mPending
    while (!mReorderStash.empty() && mReorderStash.size() > mDepth) {
        mPending.push_back(mReorderStash.front());
        mReorderStash.pop_front();
    }
}
```

```
// 编码顺序: I0 P3 B1 B2 P6 B4 B5
// 显示顺序: I0 B1 B2 P3 B4 B5 P6

pushToStash(I0, ordinal.frameIndex=0);
pushToStash(P3, ordinal.frameIndex=3);  // 进入mReorderStash，排序
pushToStash(B1, ordinal.frameIndex=1);  // 插入到正确位置
pushToStash(B2, ordinal.frameIndex=2);  // 有序插入
// mReorderStash: [I0, B1, B2, P3] (按frameIndex排序)
```
