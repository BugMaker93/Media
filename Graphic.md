# BufferQueue

<img width="481" height="149" alt="image" src="https://github.com/user-attachments/assets/fbf4c1eb-221b-4730-8f47-3c8cf379434d" />

上图是**BufferQueue**使用示意图，生产者调用 dequeueBuffer (dequeue) 从 queue 中获取一个 GraphicBuffer，对应 BufferSlot 变成 DEQUEUEED 状态；填充数据后调用 queueBuffer(queue) 放入 BufferQueue，变成 QUEUED 状态；消费者通过 acquireBuffer 从 BufferQueue 获取一个 GraphicBuffer，变成 ACQUIRED 状态，接着传给 SurfaceFlinger 用于渲染显示；SurfaceFlinger 使用完通过，通知消费者，消费者调用 releaseBuffer 放入BufferQueue(mFreeBuffer)，状态为 FREE；这样生产者就可以再次 dequeue 该 buffer；

<img width="663" height="222" alt="image" src="https://github.com/user-attachments/assets/7227e698-f016-4ead-8ccc-053f37adbc92" />

图展示的是Buffer的集中状态

mUnusedSlots     ->  mFreeSlots     ->  mFreeBuffers   ->  mActiveBuffers
unused&&no buf  ->  free&&no buf  ->  free&&has buf  ->  no free&&has buf

BufferQueueCore 中有几个数据成员来管理 BufferSlot ，这几个成员存储的都是 BufferSlot 的值(int)；mUnusedSlots 表示没有使用的slot，mFreeSlots 表示状态时FREE，但没有关联GraphicBuffer；mFreeBuffers 表示状态是FREE且关联了GraphicBuffer；mActiveBuffers 表示关联了 GraphicBuffer 且状态是 non-FREE；示意图在此链接

https://blog.csdn.net/hexiaolong2009/article/details/99225637

# IGraphicBufferProducer
>对应BufferQueue的producer

# queueBuffer

<img width="1886" height="1943" alt="image" src="https://github.com/user-attachments/assets/4da9514c-f99d-445e-9ebf-fda341b75cdb" />

OutputBufferQueue属于Codec2Client::Component。路径和Codec2Client同级，在Android\frameworks\av\media\codec2\hidl\client\output.cpp。

BufferQueue所有相关代码在Android/frameworks/native/libs/gui中。

# dequeueBuffer

<img width="2340" height="1353" alt="image" src="https://github.com/user-attachments/assets/a1909805-8ee9-448d-b4a6-e12f5b8b644d" />
