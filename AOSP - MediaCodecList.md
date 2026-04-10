# OMX和C2加载对应Codec
```
//Android\frameworks\av\media\libstagefright\MediaCodecList.cpp
std::vector<MediaCodecListBuilderBase *> GetBuilders() {
    std::vector<MediaCodecListBuilderBase *> builders;
    // if plugin provides the input surface, we cannot use OMX video encoders.
    // In this case, rely on plugin to provide list of OMX codecs that are usable.
    sp<PersistentSurface> surfaceTest = CCodec::CreateInputSurface();
    if (surfaceTest == nullptr) {
        ALOGD("Allowing all OMX codecs");
        builders.push_back(&sOmxInfoBuilder);
    } else {
        ALOGD("Allowing only non-surface-encoder OMX codecs");
        builders.push_back(&sOmxNoSurfaceEncoderInfoBuilder);
    }
    builders.push_back(GetCodec2InfoBuilder());
    return builders;
}


MediaCodecListBuilderBase *GetCodec2InfoBuilder() {
    Mutex::Autolock _l(sCodec2InfoBuilderMutex);
    if (!sCodec2InfoBuilder) {
        sCodec2InfoBuilder.reset(new Codec2InfoBuilder);
    }
    return sCodec2InfoBuilder.get();
}
```

# C2.Codec具体位置

**so的实现**：Android/frameworks/av/media/codec2/components/

```
adb shell
//能找到对应配置codec的xml
find . -name media_codec* 2>/dev/null

./vendor/etc/media_codecs.xml
./vendor/etc/media_codecs_c2.xml
./vendor/etc/media_codecs_google_audio.xml
./vendor/etc/media_codecs_google_telephony.xml
./vendor/etc/media_codecs_google_video.xml
./vendor/etc/media_codecs_performance.xml
./vendor/etc/media_codecs_performance_c2.xml
```

```
//Android\frameworks\av\media\codec2\sfplugin\Codec2InfoBuilder.cpp
status_t Codec2InfoBuilder::buildMediaCodecList(MediaCodecListWriter* writer) {

 	// Obtain Codec2Client
    std::vector<Traits> traits = Codec2Client::ListComponents();

    // parse APEX XML first, followed by vendor XML.
    // Note: APEX XML names do not depend on ro.media.xml_variant.* properties.
    MediaCodecsXmlParser parser;
	//xml位置对应
    parser.parseXmlFilesInSearchDirs(
            { "media_codecs.xml", "media_codecs_performance.xml" },
            { "/apex/com.android.media.swcodec/etc" });

    // TODO: remove these c2-specific files once product moved to default file names
    parser.parseXmlFilesInSearchDirs(
            { "media_codecs_c2.xml", "media_codecs_performance_c2.xml" });

    // parse default XML files
    parser.parseXmlFilesInSearchDirs();
}
```
Traits中对应属性会从xml中得到。

# MediaCodecList和MediaCodec名字对应
Android\frameworks\av\media\codec2\vndk\C2Store.cpp

<img width="1329" height="835" alt="image" src="https://github.com/user-attachments/assets/c625c70e-722c-429a-8ca4-537d41bbd354" />
