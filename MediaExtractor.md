# launch MediaExtractorService
>android/frameworks/av/services/mediaextractor/main_extractorservice.cpp

<img width="1413" height="588" alt="image" src="https://github.com/user-attachments/assets/fec2b3f2-d726-4c25-91f4-de37f842d6ca" />

## IMediaExtractorService

<img width="1179" height="669" alt="image" src="https://github.com/user-attachments/assets/50f90436-f9c0-4613-b549-35bf61f0b56b" />

# prepare()
>libstagefright中MediaExtractorFactory是MediaPlayer和MediaExtractor的分界点。

<img width="8481" height="4871" alt="image" src="https://github.com/user-attachments/assets/5280c409-7333-4fd1-b7ab-ad7c2ccf5618" />

## ExtractorPlugin/ExtractorPluginHelper
在MediaExtractorFactory::sniff()中产生关系。

所有Extractor（e.g. AAExtractor）继承ExtractorPluginHelper。

## DataSouce

### 数据转换
CreateDataSourceFromIDataSource/CreateIDataSourceFromDataSource:android/frameworks/av/media/libstagefright/InterfaceUtils.cpp

### 通信结构
```
DataSourceHelper（CDataSource（TinyCacheSource2（BpDataSource2（RemoteDataSource2（TinyCacheSource1（BpDataSource1（RemoteDataSource1（FileSource））））））））
```

# AnotherPacketSource
```
/GenericSource.h
struct Track {
	size_t mIndex;
	sp<IMediaSource> mSource;
	sp<AnotherPacketSource> mPackets;
};


//GenericSource.cpp
void NuPlayer::GenericSource::readBuffer(）{
	Track *track;
	sp<IMediaSource> source = track->mSource;
	MediaBufferBase *mbuf = NULL;
    source->read(&mbuf);
    sp<ABuffer> buffer = mediaBufferToABuffer(mbuf);
    track->mPackets->queueAccessUnit(buffer);
}
```

**queueAccessUnit**()负责把**解复用后的数据**入队，并更新队列时序状态。

**dequeueAccessUnit**()负责从队列取出**解复用后的数据**给**C2Codec解码**链路，并同步消费侧状态。具体如何dequeueAccessUnit() refer to 
