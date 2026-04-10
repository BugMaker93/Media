# Service
>Android\frameworks\av\services\mediaextractor

注册service：android/frameworks/av/services/mediaextractor/**main_extractorservice.cpp**

实现service：android/frameworks/av/services/mediaextractor/**MediaExtractorService.cpp**

# MediaExtractorFactory
```
//在MediaExtractorService创建时会对所有支持的extractor进行注册。Android/frameworks/av/media/module/extractors/下是所有支持的extractor。
void MediaExtractorFactory::LoadExtractors()


//CreateFromService是MediaExtractorService创建对应MediaExtractor时调用。CreateFromService调用sniff()。sniff就是遍历所有ExtractorPlugin，作用是找到最合适的Extractor。
sp<IMediaExtractor> MediaExtractorFactory::CreateFromService(
        const sp<DataSource> &source, const char *mime)
```

# ExtractorPlugin
refer to [AOSP - 加载so.md](https://github.com/BugMaker93/Media/blob/main/AOSP%20-%20%E5%8A%A0%E8%BD%BDso.md)

ExtractorPlugin在MediaExtractorFactory中定义。

**MediaExtractorFactory和so通信涉及到很多数据结构的转换**。涉及的数据结构在Android/frameworks/av/include/media/MediaExtractorPluginApi.h和Android/frameworks/av/include/media/MediaExtractorPluginHelper.h中。

MediaExtractorPluginApi.h针对MediaExtractorFactory，定义了CMediaTrack,CMediaExtractor...

MediaExtractorPluginHelper.h针对so的数据转换。

**so的实现**：Android/frameworks/av/media/module/extractors/

# Version
<img width="1057" height="529" alt="image" src="https://github.com/user-attachments/assets/a0ad5188-5658-44d6-8547-1d24b832811e" />

同一个extractor，使用version大的。
