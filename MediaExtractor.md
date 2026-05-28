# launch MediaExtractorService
>android/frameworks/av/services/mediaextractor/main_extractorservice.cpp

<img width="1413" height="588" alt="image" src="https://github.com/user-attachments/assets/fec2b3f2-d726-4c25-91f4-de37f842d6ca" />

## IMediaExtractorService

<img width="1179" height="669" alt="image" src="https://github.com/user-attachments/assets/50f90436-f9c0-4613-b549-35bf61f0b56b" />

# prepare()

<img width="8482" height="4863" alt="image" src="https://github.com/user-attachments/assets/4103310a-498b-45b0-b4b7-2d808a039fab" />

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
