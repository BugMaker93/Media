# Sequence
<img width="1051" height="1537" alt="image" src="https://github.com/user-attachments/assets/9e73f36d-7ec2-4cb8-a020-e752164c6aa9" />

**JNI_MediaPlayer**：Android/frameworks/base/media/jni/android_media_MediaPlayer.cpp。

**MediaPlayer**：IMediaPlayerClient。

**JNIMediaPlayerListener**：MediaPlayerListener。MediaPlayer通过JNIMediaPlayerListener回调JNI_MediaPlayer。

**MediaPlayerService**:IMediaPlayerService。

**Client:IMediaPlayer**:IMediaPlayerService的BnMediaPlayer。

<img width="1177" height="1361" alt="image" src="https://github.com/user-attachments/assets/a377af00-565f-4f01-8141-11973b254b92" />

**NuPlayerDriver:MediaPlayerInterface:MediaPlayerBase**:NuPlayerDriver创建时会创建NuPlayer。

**Listener:MediaPlayerBase::Listener**:NuPlayerDriver创建时需要此Listener，此Listener是Client创建，则NuPlayerDriver通过Listener回调Client。

<img width="2469" height="3840" alt="image" src="https://github.com/user-attachments/assets/9467d024-f534-4d51-8ed0-96ce71d2cd32" />

<img width="2377" height="2333" alt="image" src="https://github.com/user-attachments/assets/351a0b62-0d04-4df3-ad72-32a31acb59bf" />

**setDataSource**()的flow会到创建GenericSource。

**prepare**()会创建Extractor，并通过GenericSource和Extractor通信。

**start**()会new NuPlayer：：Renderer()/NuPlayer：：Decoder()/NuPlayer：：CCDecoder()。

## NuPlayer::GenericSource:NuPlayer::Source




# Package Flow
>如果一个h有两个cpp，则是Bp和Bn。以include\media\IMediaPlayerClient.h为例，Bp的实现是IMediaPlayerClient.cpp，Bn的实现是mediaplayer.cpp

## setDataSource

<table border="1">
  <thead>
    <tr>
      <th colspan="6">libmedia(Android\frameworks\av\media\libmedia)</th>
      <th colspan="3">libmediaservice(Android\frameworks\av\media\libmediaservice)</th>
    </tr>
    <tr>
      <th colspan="2">include\media\IMediaPlayerClient.h</th>
      <th colspan="2">include\media\IMediaPlayerService.h</th>
      <th colspan="2">include\media\IMediaPlayer.h</th>
      <th>nuplayer\NuplayerDriver.h</th>
      <th>nuplayer\Nuplayer.h</th>
      <th>nuplayer\GenericSource.h</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>IMediaPlayerClient.cpp</td>
      <td>mediaplayer.cpp</td>
      <td>IMediaPlayerService.cpp</td>
      <td>Android\frameworks\av\media\libmediaservice\MediaPlayerService.cpp</td>
      <td>IMediaPlayer.cpp</td>
      <td>Android\frameworks\av\media\libmediaservice\MediaPlayerService.cpp</td>
      <td>nuplayer\NuplayerDriver.cpp</td>
      <td>nuplayer\Nuplayer.cpp</td>
      <td>nuplayer\GenericSource.cpp</td>
    </tr>
  </tbody>
</table>

## prepare
>从GenericSource开始

<table border="1">
  <thead>
    <tr>
      <th colspan="3">libmediaservice(Android\frameworks\av\media\libmediaservice)</th>
      <th colspan="2">mediaextractor (Android\frameworks\av\service\mediaextractor)</th>
      <th colspan="1">libstagefright(Android\frameworks\av\media\libstagefright)</th>
      <th colspan="2">libmedia(Android\frameworks\av\media\libmedia)</th>
    </tr>
    <tr>
      <th>nuplayer\NuplayerDriver.h</th>
      <th>nuplayer\Nuplayer.h</th>
      <th>nuplayer\GenericSource.h</th>
      <th colspan="2">aidl(Android/frameworks/av/media/libmedia/aidl/android/IMediaExtractorService.aidl)</th>
      <th>include\media\stagefright\MediaExtractorFactory.h</th>
      <th colspan="2">include\android\IMediaExtractor.h</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>nuplayer\NuplayerDriver.cpp</td>
      <td>nuplayer\Nuplayer.cpp</td>
      <td>nuplayer\GenericSource.cpp</td>
      <td>bp aidl auto</td>
      <td>MediaExtractorService.h/MediaExtractorService.cpp</td>
      <td>MediaExtractorFactory.cpp</td>
      <td>IMediaExtractor.cpp</td>
      <td>Android\frameworks\av\media\libstagefright\RemoteMediaExtractor.cpp</td>
    </tr>
  </tbody>
</table>

## start
>从NuPlayer开始，准确是从NuPlayerDecoder开始。

<table border="1">
  <thead>
    <tr>
      <th colspan="1">libstagefright(Android\frameworks\av\media\libstagefright)</th>
      <th colspan="3">codec2(Android\frameworks\av\media\codec2)</th>
    </tr>
    <tr>
      <th>include\media\stagefright\MediaCodec.h</th>
      <th>sfplugin\include\media\stagefright\CCodec.h</th>
      <th>sfplugin\CCodecBufferChannel.h</th>
      <th>hidl\client\include\codec2\hidl\client.h</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>MediaCodec.cpp</td>
      <td>sfplugin/CCodec.cpp</td>
      <td>sfplugin\CCodecBufferChannel.cpp</td>
      <td>hidl\client\client.cpp</td>
    </tr>
  </tbody>
</table>
