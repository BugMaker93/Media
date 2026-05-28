>以MediaCodecService中IComponentStore.hal为例

**IComponentStore.hal**：Android/hardware/interfaces/media/c2/1.0/

**IComponentStore相关的generated file**（IComponentStore.h/BpHwComponentStore.h/BnHwComponentStore.h/ComponentStoreAll.cpp）：Android\out\soong\.intermediates\hardware\interfaces\media\c2\1.0\

# 提供Bn的impl
Bp和Bn会自动生成用来进行IPC，**IComponentStore用来提供Bn的具体实现**。

```
struct BnHwComponentStore : public ::android::hidl::base::V1_0::BnHwBase {
	//使用IComponentStore注入BnHwComponentStore的实现，调用BnHwComponentStore=调用IComponentStore
    explicit BnHwComponentStore(const ::android::sp<IComponentStore> &_hidl_impl);

    static ::android::status_t _hidl_copyBuffer(
            ::android::hidl::base::V1_0::BnHwBase* _hidl_this,
            const ::android::hardware::Parcel &_hidl_data,
            ::android::hardware::Parcel *_hidl_reply,
            TransactCallback _hidl_cb);    
}
```

```
struct BpHwComponentStore : public ::android::hardware::BpInterface<IComponentStore>, public ::android::hardware::details::HidlInstrumentor {
    //_hidl_impl是BpBinder
	explicit BpHwComponentStore(const ::android::sp<::android::hardware::IBinder> &_hidl_impl);
	
    static ::android::hardware::Return<::android::hardware::media::c2::V1_0::Status>  _hidl_copyBuffer
	(::android::hardware::IInterface* _hidl_this,
	::android::hardware::details::HidlInstrumentor *_hidl_this_instrumentor, 
	const ::android::hardware::media::c2::V1_0::Buffer& src, 
	const::android::hardware::media::c2::V1_0::Buffer& dst);
}
```

```
//Android\frameworks\av\media\codec2\hal\hidl\1.0\utils\include\codec2\hidl\1.0\ComponentStore.h
struct ComponentStore : public IComponentStore {
    //C2ComponentStore是IComponentStore的helper，调用IComponentStore=调用C2ComponentStore
    ComponentStore(const std::shared_ptr<C2ComponentStore>& store);
}
```

```
//Android\frameworks\av\media\module\codecserviceregistrant\CodecServiceRegistrant.cpp
extern "C" void RegisterCodecServices() {
    std::shared_ptr<C2ComponentStore> store = android::GetCodec2PlatformComponentStore();

    android::sp<V1_2::IComponentStore> storeV1_2 = new V1_2::utils::ComponentStore(store);
	
	if (storeV1_2->registerAsService("software") != android::OK) {
		LOG(ERROR) << "Cannot register software Codec2 v1.2 service.";
		return;
    }
	
}
```

```
//Android\frameworks\av\media\codec2\vndk\C2Store.cpp

class C2PlatformComponentStore : public C2ComponentStore {
}

std::shared_ptr<C2ComponentStore> GetCodec2PlatformComponentStore() {
    static std::mutex mutex;
    static std::weak_ptr<C2ComponentStore> platformStore;
    std::lock_guard<std::mutex> lock(mutex);
    std::shared_ptr<C2ComponentStore> store = platformStore.lock();
    if (store == nullptr) {
        store = std::make_shared<C2PlatformComponentStore>();
        platformStore = store;
    }
    return store;
}
```

# HIDL的Parcel
>以_hidl_copyBuffer中::android::hardware::media::c2::V1_0::Buffer为例,Buffer是struct是不能进行IPC通信的，需要转换成Parcel

```
//Android\out\soong\.intermediates\hardware\interfaces\media\c2\1.0\android.hardware.media.c2@1.0_genc++_headers\gen\android\hardware\media\c2\1.0\types.h
struct Buffer final {
    /**
     * Metadata associated with the buffer.
     */
    ::android::hardware::hidl_vec<uint8_t> info __attribute__ ((aligned(8)));
    /**
     * Blocks contained in the buffer.
     */
    ::android::hardware::hidl_vec<::android::hardware::media::c2::V1_0::Block> blocks __attribute__ ((aligned(8)));
};

static_assert(offsetof(::android::hardware::media::c2::V1_0::Buffer, info) == 0, "wrong offset");
static_assert(offsetof(::android::hardware::media::c2::V1_0::Buffer, blocks) == 16, "wrong offset");
static_assert(sizeof(::android::hardware::media::c2::V1_0::Buffer) == 32, "wrong size");
static_assert(__alignof(::android::hardware::media::c2::V1_0::Buffer) == 8, "wrong alignment");
```

```
//Android\out\soong\.intermediates\hardware\interfaces\media\c2\1.0\android.hardware.media.c2@1.0_genc++\gen\android\hardware\media\c2\1.0\types.cpp
::android::status_t writeEmbeddedToParcel(
        const Buffer &obj,
        ::android::hardware::Parcel *parcel,
        size_t parentHandle,
        size_t parentOffset) {
    ::android::status_t _hidl_err = ::android::OK;

    size_t _hidl_info_child;

    _hidl_err = ::android::hardware::writeEmbeddedToParcel(
            obj.info,
            parcel,
            parentHandle,
            parentOffset + offsetof(::android::hardware::media::c2::V1_0::Buffer, info), &_hidl_info_child);

    if (_hidl_err != ::android::OK) { return _hidl_err; }

    size_t _hidl_blocks_child;

    _hidl_err = ::android::hardware::writeEmbeddedToParcel(
            obj.blocks,
            parcel,
            parentHandle,
            parentOffset + offsetof(::android::hardware::media::c2::V1_0::Buffer, blocks), &_hidl_blocks_child);

    if (_hidl_err != ::android::OK) { return _hidl_err; }

    for (size_t _hidl_index_0 = 0; _hidl_index_0 < obj.blocks.size(); ++_hidl_index_0) {
        _hidl_err = writeEmbeddedToParcel(
                obj.blocks[_hidl_index_0],
                parcel,
                _hidl_blocks_child,
                _hidl_index_0 * sizeof(::android::hardware::media::c2::V1_0::Block));

        if (_hidl_err != ::android::OK) { return _hidl_err; }

    }

    return _hidl_err;
}

::android::status_t readEmbeddedFromParcel(
        const Buffer &obj,
        const ::android::hardware::Parcel &parcel,
        size_t parentHandle,
        size_t parentOffset) {
    ::android::status_t _hidl_err = ::android::OK;

    size_t _hidl_info_child;

    _hidl_err = ::android::hardware::readEmbeddedFromParcel(
            const_cast<::android::hardware::hidl_vec<uint8_t> &>(obj.info),
            parcel,
            parentHandle,
            parentOffset + offsetof(::android::hardware::media::c2::V1_0::Buffer, info), &_hidl_info_child);

    if (_hidl_err != ::android::OK) { return _hidl_err; }

    size_t _hidl_blocks_child;

    _hidl_err = ::android::hardware::readEmbeddedFromParcel(
            const_cast<::android::hardware::hidl_vec<::android::hardware::media::c2::V1_0::Block> &>(obj.blocks),
            parcel,
            parentHandle,
            parentOffset + offsetof(::android::hardware::media::c2::V1_0::Buffer, blocks), &_hidl_blocks_child);

    if (_hidl_err != ::android::OK) { return _hidl_err; }

    for (size_t _hidl_index_0 = 0; _hidl_index_0 < obj.blocks.size(); ++_hidl_index_0) {
        _hidl_err = readEmbeddedFromParcel(
                const_cast<::android::hardware::media::c2::V1_0::Block &>(obj.blocks[_hidl_index_0]),
                parcel,
                _hidl_blocks_child,
                _hidl_index_0 * sizeof(::android::hardware::media::c2::V1_0::Block));

        if (_hidl_err != ::android::OK) { return _hidl_err; }

    }

    return _hidl_err;
}
```

所有在IComponent中需要转换成parcel的数据结构

会在typc.h定义其格式

会在type.cpp实现其readEmbeddedFromParcel/writeEmbeddedToParcel

这些都是自动生成

**HIDL中参数转HIDL格式可参考Android/frameworks/av/media/codec2/hal/hidl/1.0/utils/types.cpp，关注createParamsBlob()/copyParamsFromBlob()**
