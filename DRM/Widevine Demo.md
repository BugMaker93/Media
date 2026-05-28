# code
google提供mpd测试uri:

https://storage.googleapis.com/wvmedia/cenc/h264/tears/tears.mpd

下载得到

```
<?xml version="1.0" encoding="UTF-8"?>
<!--Generated with https://github.com/google/shaka-packager version 97fc982-release-->
<MPD xmlns="urn:mpeg:dash:schema:mpd:2011" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xlink="http://www.w3.org/1999/xlink" xsi:schemaLocation="urn:mpeg:dash:schema:mpd:2011 DASH-MPD.xsd" xmlns:cenc="urn:mpeg:cenc:2013" minBufferTime="PT2S" type="static" profiles="urn:mpeg:dash:profile:isoff-on-demand:2011" mediaPresentationDuration="PT734S">
  <Period id="0">
    <AdaptationSet id="0" contentType="audio" lang="en">
        <ContentProtection value="cenc" schemeIdUri="urn:mpeg:dash:mp4protection:2011" cenc:default_KID="30303030-3030-3030-3030-303030303030"/>
        <ContentProtection schemeIdUri="urn:uuid:edef8ba9-79d6-4ace-a3c8-27dcd51d21ed">
          <cenc:pssh>AAAARHBzc2gAAAAA7e+LqXnWSs6jyCfc1R0h7QAAACQIARIBNRoNd2lkZXZpbmVfdGVzdCIKMjAxNV90ZWFycyoCU0Q=</cenc:pssh>
        </ContentProtection>
      <Representation id="0" bandwidth="134359" codecs="mp4a.40.2" mimeType="audio/mp4" audioSamplingRate="44100">
        <AudioChannelConfiguration schemeIdUri="urn:mpeg:dash:23003:3:audio_channel_configuration:2011" value="2"/>
        <BaseURL>tears_audio_eng.mp4</BaseURL>
        <SegmentBase indexRange="974-1893" timescale="44100">
          <Initialization range="0-973"/>
        </SegmentBase>
      </Representation>
    </AdaptationSet>
    <AdaptationSet id="1" contentType="video" maxWidth="1920" maxHeight="856" frameRate="12288/512" par="38:17">
        <ContentProtection value="cenc" schemeIdUri="urn:mpeg:dash:mp4protection:2011" cenc:default_KID="30303030-3030-3030-3030-303030303035"/>
        <ContentProtection schemeIdUri="urn:uuid:edef8ba9-79d6-4ace-a3c8-27dcd51d21ed">
          <cenc:pssh>AAAARHBzc2gAAAAA7e+LqXnWSs6jyCfc1R0h7QAAACQIARIBNRoNd2lkZXZpbmVfdGVzdCIKMjAxNV90ZWFycyoCU0Q=</cenc:pssh>
        </ContentProtection>
      <Representation id="1" bandwidth="772315" codecs="avc1.42c01e" mimeType="video/mp4" sar="852:857" width="320" height="142">
        <BaseURL>tears_h264_baseline_240p_800.mp4</BaseURL>
        <SegmentBase indexRange="1127-1902" timescale="12288">
          <Initialization range="0-1126"/>
        </SegmentBase>
      </Representation>
      <Representation id="2" bandwidth="1777315" codecs="avc1.4d401f" mimeType="video/mp4" sar="2242:2249" width="854" height="380">
        <BaseURL>tears_h264_main_480p_2000.mp4</BaseURL>
        <SegmentBase indexRange="1131-1906" timescale="12288">
          <Initialization range="0-1130"/>
        </SegmentBase>
      </Representation>
      <Representation id="3" bandwidth="7206998" codecs="avc1.4d4028" mimeType="video/mp4" sar="855:857" width="1280" height="570">
        <BaseURL>tears_h264_main_720p_8000.mp4</BaseURL>
        <SegmentBase indexRange="1133-1908" timescale="12288">
          <Initialization range="0-1132"/>
        </SegmentBase>
      </Representation>
      <Representation id="4" bandwidth="18320008" codecs="avc1.64002a" mimeType="video/mp4" sar="856:857" width="1920" height="856">
        <BaseURL>tears_h264_high_1080p_20000.mp4</BaseURL>
        <SegmentBase indexRange="1137-1912" timescale="12288">
          <Initialization range="0-1136"/>
        </SegmentBase>
      </Representation>
    </AdaptationSet>
  </Period>
</MPD>

```

解析

```
下载BaseURL例如tears_audio_eng.mp4就获得加密的mp4
https://storage.googleapis.com/wvmedia/cenc/h264/tears/tears_audio_eng.mp4

//cenc指的是加密标准
<ContentProtection value="cenc" schemeIdUri="urn:mpeg:dash:mp4protection:2011" cenc:default_KID="30303030-3030-3030-3030-303030303030"/>

//schemeIdUri的值指的是Widevine的UUID，这UUID是固定的，来自Android\vendor\widevine\libwvdrmengine\src\WVUUID.cpp
<ContentProtection schemeIdUri="urn:uuid:edef8ba9-79d6-4ace-a3c8-27dcd51d21ed">

//cenc:pssh的值和mp4 pssh box相对应
<cenc:pssh>AAAARHBzc2gAAAAA7e+LqXnWSs6jyCfc1R0h7QAAACQIARIBNRoNd2lkZXZpbmVfdGVzdCIKMjAxNV90ZWFycyoCU0Q=</cenc:pssh>

//indexRange的值和mp4 sidx box相对应，从这个值能知道要下载哪个moof和mdat box
<SegmentBase indexRange="974-1893" timescale="44100">
```

# Request&Response
对应DRM Provisioning Server的provisioning request&provisioning response
```
MediaDrm.getProvisionRequest()
MediaDrm.provideProvisionResponse()
```

对应License Server的license request&license response
```
MediaDrm.getKeyRequest()
MediaDrm.provideKeyResponse()
```

# Widevine Protocol Buffer
