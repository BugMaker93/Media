# initiateAllocateComponent

<img width="3569" height="1792" alt="image" src="https://github.com/user-attachments/assets/7b1e177c-5417-4989-afe6-e40b39510fef" />

Codec2Client代理ComponentStore:IComponentStore代理C2PlatformComponentStore:C2ComponentStore。

Codec2Client::Component代理Component：IComponent代理C2SoftRawDec：SimpleC2Component：C2Component。

# initiateConfigureComponent

<img width="1165" height="756" alt="image" src="https://github.com/user-attachments/assets/d1cbe73a-a954-4b48-bd2a-d957e5dcfded" />

Codec2Client::Component后续的调用没有画，会一路调到SimpleC2Component中。refer to initiateAllocateComponent中的代理。

# initiateStart

LinearInputBuffer中存储的是decode前数据，即解复用的数据。

LinearOutputBuffers中存储时decode后的数据。

C2Store.cpp涉及到buffer的转换，图会忽略。buffer具体如何转换refer to [CCodec - Buffer.md](https://github.com/BugMaker93/Media/blob/main/CCodec%20-%20Buffer.md)

LinearInputBuffer/LinearOutputBuffer会使用C2Store.cpp中的buffer转换。

LinearInputBuffer的操作(releaseBuffer()/requestNewBuffer())对应onInputBufferAvailable()。

LinearOutputBuffers的操作(pushToStash()/popFromStashAndRegister())对应onOutputBufferAvailable()。

CCodecBufferChannel：：queueInputBufferInternal()其实会同时操作LinearInputBuffer和LinearOutputBuffers，只是分开画了。

## onInputBufferAvailable(initiateStart)

<img width="2766" height="1401" alt="image" src="https://github.com/user-attachments/assets/59f67687-18ab-4127-a64b-65e5bb3901d8" />

## onOutputBufferAvailable(queueInputBuffer)

<img width="1607" height="823" alt="image" src="https://github.com/user-attachments/assets/00a69268-e620-422d-b7df-d4a1b47b3936" />

Codec2Client::Component后续的调用没有画，会一路调到SimpleC2Component中。refer to initiateAllocateComponent中的代理。

这里queue的目的就是调到SimpleC2Component，通过onWorkDone获得SimpleC2Component decode后数据。
