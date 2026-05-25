# Overview

refer to [AOSP/Binder.md](https://github.com/BugMaker93/Media/blob/main/AOSP/Binder.md)

Client：使用BpInterface，BpBinder不需要自己创建。

Server：使用BnInterface，BBinder需要自己创建。

# Case
>以IMediaPlayerClient为例
>
Android/frameworks/av/media/libmedia/include/media/IMediaPlayerClient.h:

```
class IMediaPlayerClient: public IInterface
{
public:
	//IInterface.h中定义的宏，其为声明，实现是IMPLEMENT_META_INTERFACE
    DECLARE_META_INTERFACE(MediaPlayerClient);
	//独有
    virtual void notify(int msg, int ext1, int ext2, const Parcel *obj) = 0;
};

// ----------------------------------------------------------------------------

class BnMediaPlayerClient: public BnInterface<IMediaPlayerClient>
{
public:
	//独有
    virtual status_t    onTransact( uint32_t code,
                                    const Parcel& data,
                                    Parcel* reply,
                                    uint32_t flags = 0);
};


```

Android/frameworks/av/media/libmedia/IMediaPlayerClient.cpp:

```
enum {
    NOTIFY = IBinder::FIRST_CALL_TRANSACTION,
};

class BpMediaPlayerClient: public BpInterface<IMediaPlayerClient>
{
public:
	//impl是BpBinder，ProcessState::getStrongProxyForHandle可知道BpBinder的创建
    explicit BpMediaPlayerClient(const sp<IBinder>& impl)
        : BpInterface<IMediaPlayerClient>(impl)
    {
    }

    virtual void notify(int msg, int ext1, int ext2, const Parcel *obj)
    {
        Parcel data, reply;
        data.writeInterfaceToken(IMediaPlayerClient::getInterfaceDescriptor());
        data.writeInt32(msg);
        data.writeInt32(ext1);
        data.writeInt32(ext2);
        if (obj && obj->dataSize() > 0) {
            data.appendFrom(const_cast<Parcel *>(obj), 0, obj->dataSize());
        }
		//此remote()返回的是BpBinder，来自BpRefBase::mRemote
        remote()->transact(NOTIFY, data, &reply, IBinder::FLAG_ONEWAY);
    }
};

//IInterface.h中定义的宏，其为实现，声明是DECLARE_META_INTERFACE
IMPLEMENT_META_INTERFACE(MediaPlayerClient, "android.media.IMediaPlayerClient");

// ----------------------------------------------------------------------

status_t BnMediaPlayerClient::onTransact(
    uint32_t code, const Parcel& data, Parcel* reply, uint32_t flags)
{
    switch (code) {
        case NOTIFY: {
            CHECK_INTERFACE(IMediaPlayerClient, data, reply);
            int msg = data.readInt32();
            int ext1 = data.readInt32();
            int ext2 = data.readInt32();
            Parcel obj;
            if (data.dataAvail() > 0) {
                obj.appendFrom(const_cast<Parcel *>(&data), data.dataPosition(), data.dataAvail());
            }

            notify(msg, ext1, ext2, &obj);
            return NO_ERROR;
        } break;
        default:
            return BBinder::onTransact(code, data, reply, flags);
    }
}

```

Android/frameworks/av/media/libmedia/include/media/mediaplayer.h:

```
//BnMediaPlayerClient需要被继承后使用
class MediaPlayer : public BnMediaPlayerClient,
                    public virtual IMediaDeathNotifier
{

	......
}
```

# IInterface Structure
>创建BpBinder的位置

Android/frameworks/native/libs/binder/include/binder/IInterface.h：

```
class IInterface: public virtual RefBase
{
public:
            IInterface();
			//作用是IInterface->IBinder转换
            static sp<IBinder>  asBinder(const IInterface*);
            static sp<IBinder>  asBinder(const sp<IInterface>&);

protected:
    virtual                     ~IInterface();
    virtual IBinder*            onAsBinder() = 0;
};

// ----------------------------------------------------------------------


template<typename INTERFACE>
inline sp<INTERFACE> interface_cast(const sp<IBinder>& obj)
{
	//特别关注
    return INTERFACE::asInterface(obj);
}

// ----------------------------------------------------------------------

/*
 * 继承INTERFACE&BBinder
 * 会实现IBinder的queryLocalInterface(const String16& _descriptor)&getInterfaceDescriptor()
 */
template<typename INTERFACE>
class BnInterface : public INTERFACE, public BBinder
{
public:
    virtual sp<IInterface>      queryLocalInterface(const String16& _descriptor);
    virtual const String16&     getInterfaceDescriptor() const;
    typedef INTERFACE BaseInterface;

protected:
    virtual IBinder*            onAsBinder();
};

// ----------------------------------------------------------------------

/*
 * 继承INTERFACE和BpRefBase，但没有继承BpBinder！！BpBinder是BpRefBase::mRemote
 * 更不会不会实现IBinder的queryLocalInterface(const String16& _descriptor)&getInterfaceDescriptor()
 */ 
template<typename INTERFACE>
class BpInterface : public INTERFACE, public BpRefBase
{
public:
    explicit                    BpInterface(const sp<IBinder>& remote);
    typedef INTERFACE BaseInterface;

protected:
    virtual IBinder*            onAsBinder();
};

// ----------------------------------------------------------------------

//继承IInterface的类会用到，其为声明，实现是IMPLEMENT_META_INTERFACE
#define DECLARE_META_INTERFACE(INTERFACE)                                                         \
public:                                                                                           \
    static const ::android::String16 descriptor;                                                  \
    static ::android::sp<I##INTERFACE> asInterface(const ::android::sp<::android::IBinder>& obj); \
    virtual const ::android::String16& getInterfaceDescriptor() const;                            \
    I##INTERFACE();                                                                               \
    virtual ~I##INTERFACE();                                                                      \
    static bool setDefaultImpl(::android::sp<I##INTERFACE> impl);                                 \
    static const ::android::sp<I##INTERFACE>& getDefaultImpl();                                   \
                                                                                                  \
private:                                                                                          \
    static ::android::sp<I##INTERFACE> default_impl;                                              \
                                                                                                  \
public:

#define __IINTF_CONCAT(x, y) (x ## y)

#ifndef DO_NOT_CHECK_MANUAL_BINDER_INTERFACES

//继承BpInterface的类会用到，其为实现，声明是DECLARE_META_INTERFACE
#define IMPLEMENT_META_INTERFACE(INTERFACE, NAME)                       \
    static_assert(internal::allowedManualInterface(NAME),               \
                  "b/64223827: Manually written binder interfaces are " \
                  "considered error prone and frequently have bugs. "   \
                  "The preferred way to add interfaces is to define "   \
                  "an .aidl file to auto-generate the interface. If "   \
                  "an interface must be manually written, add its "     \
                  "name to the whitelist.");                            \
    DO_NOT_DIRECTLY_USE_ME_IMPLEMENT_META_INTERFACE(INTERFACE, NAME)    \

#else

#define IMPLEMENT_META_INTERFACE(INTERFACE, NAME)                       \
    DO_NOT_DIRECTLY_USE_ME_IMPLEMENT_META_INTERFACE(INTERFACE, NAME)    \

#endif

/*
 * 此例中DO_NOT_DIRECTLY_USE_ME_IMPLEMENT_META_INTERFACE0(ITYPE, INAME, BPTYPE)参数ITYPE=IMediaPlayerClient，INAME=IMediaPlayerClient，BPTYPE=BpMediaPlayerClient
 * 此例中asInterface(obj)如果obj为BpBinder则创建BpMediaPlayerClient。因为BpBinder并没有实现queryLocalInterface，所以intr一定为空。如果obj为BBinder则返回BnMediaPlayerClient
 */
#define DO_NOT_DIRECTLY_USE_ME_IMPLEMENT_META_INTERFACE0(ITYPE, INAME, BPTYPE)                     \
    const ::android::String16& ITYPE::getInterfaceDescriptor() const { return ITYPE::descriptor; } \
    ::android::sp<ITYPE> ITYPE::asInterface(const ::android::sp<::android::IBinder>& obj) {        \
        ::android::sp<ITYPE> intr;                                                                 \
        if (obj != nullptr) {                                                                      \
            intr = ::android::sp<ITYPE>::cast(obj->queryLocalInterface(ITYPE::descriptor));        \
            if (intr == nullptr) {                                                                 \
                intr = ::android::sp<BPTYPE>::make(obj);                                           \
            }                                                                                      \
        }                                                                                          \
        return intr;                                                                               \
    }                                                                                              \
    ::android::sp<ITYPE> ITYPE::default_impl;                                                      \
    bool ITYPE::setDefaultImpl(::android::sp<ITYPE> impl) {                                        \
        /* Only one user of this interface can use this function     */                            \
        /* at a time. This is a heuristic to detect if two different */                            \
        /* users in the same process use this function.              */                            \
        assert(!ITYPE::default_impl);                                                              \
        if (impl) {                                                                                \
            ITYPE::default_impl = std::move(impl);                                                 \
            return true;                                                                           \
        }                                                                                          \
        return false;                                                                              \
    }                                                                                              \
    const ::android::sp<ITYPE>& ITYPE::getDefaultImpl() { return ITYPE::default_impl; }            \
    ITYPE::INAME() {}                                                                              \
    ITYPE::~INAME() {}

// Macro for an interface type.
#define DO_NOT_DIRECTLY_USE_ME_IMPLEMENT_META_INTERFACE(INTERFACE, NAME)                        \
    const ::android::StaticString16 I##INTERFACE##_descriptor_static_str16(                     \
            __IINTF_CONCAT(u, NAME));                                                           \
    const ::android::String16 I##INTERFACE::descriptor(I##INTERFACE##_descriptor_static_str16); \
    DO_NOT_DIRECTLY_USE_ME_IMPLEMENT_META_INTERFACE0(I##INTERFACE, I##INTERFACE, Bp##INTERFACE)

// Macro for "nested" interface type.
// For example,
//   class Parent .. { class INested .. { }; };
// DO_NOT_DIRECTLY_USE_ME_IMPLEMENT_META_NESTED_INTERFACE(Parent, Nested, "Parent.INested")
#define DO_NOT_DIRECTLY_USE_ME_IMPLEMENT_META_NESTED_INTERFACE(PARENT, INTERFACE, NAME)  \
    const ::android::String16 PARENT::I##INTERFACE::descriptor(NAME);                    \
    DO_NOT_DIRECTLY_USE_ME_IMPLEMENT_META_INTERFACE0(PARENT::I##INTERFACE, I##INTERFACE, \
                                                     PARENT::Bp##INTERFACE)

#define CHECK_INTERFACE(interface, data, reply)                         \
    do {                                                                \
      if (!(data).checkInterface(this)) { return PERMISSION_DENIED; }   \
    } while (false)                                                     \


// ----------------------------------------------------------------------

template<typename INTERFACE>
inline sp<IInterface> BnInterface<INTERFACE>::queryLocalInterface(
        const String16& _descriptor)
{
    if (_descriptor == INTERFACE::descriptor) return sp<IInterface>::fromExisting(this);
    return nullptr;
}

template<typename INTERFACE>
inline const String16& BnInterface<INTERFACE>::getInterfaceDescriptor() const
{
    return INTERFACE::getInterfaceDescriptor();
}

template<typename INTERFACE>
IBinder* BnInterface<INTERFACE>::onAsBinder()
{
    return this;
}

template<typename INTERFACE>
inline BpInterface<INTERFACE>::BpInterface(const sp<IBinder>& remote)
    : BpRefBase(remote)
{
}

template<typename INTERFACE>
inline IBinder* BpInterface<INTERFACE>::onAsBinder()
{
    return remote();
}

......


```

# Binder Structure

Android/frameworks/native/libs/binder/include/binder/IBinder.h:

```
class IBinder: public virtual RefBase {
    /*
     * 对于BpBinder，并没有实现，返回null
     * 对于BBinder，在BnInterface中实现 ，作用是IBinder->IInterface转换
     */
    virtual sp<IInterface>  queryLocalInterface(const String16& descriptor);

    /*
     * 对于BpBinder，作用是使用rpc和BBinder进程间通信
     * 对于BBinder，调用自己的onTransact，作用是处理BpBinder通过rpc来的消息
     */ 
    virtual status_t        transact(   uint32_t code,
                                        const Parcel& data,
                                        Parcel* reply,
                                        uint32_t flags = 0) = 0;

     /*
     * 对于BpBinder，并没有实现，返回null
     * 对于BBinder，在BnInterface中实现，作用是唯一标识符
     */ 
	virtual const String16& getInterfaceDescriptor() const = 0;

	//BBinder实现，而BpBinder不实现
    virtual BBinder*        localBinder();
    //BpBinder实现，而BBinder不实现
    virtual BpBinder*       remoteBinder();

    ......
}
```

Android/frameworks/native/libs/binder/include/binder/BpBinder.h:

```
class BpBinder : public IBinder{
	//此handle能够让BpBinder找到对应的BBinder
     static sp<BpBinder> create(int32_t handle);

    //作用是使用rpc和BBinder进程间通信
	 virtual status_t    transact(   uint32_t code,
                                    const Parcel& data,
                                    Parcel* reply,
                                    uint32_t flags = 0) final;

	//返回this
	virtual BpBinder*   remoteBinder();
    ......
}
```

Android/frameworks/native/libs/binder/include/binder/Binder.h:

```
class BBinder : public IBinder {
	//调用自己的onTransact，作用是处理BpBinder通过rpc来的消息
    virtual status_t    transact(   uint32_t code,
                                    const Parcel& data,
                                    Parcel* reply,
                                    uint32_t flags = 0) final;

	//独有，处理BpBinder通过rpc来的消息的具体实现
    virtual status_t    onTransact( uint32_t code,
                                    const Parcel& data,
                                    Parcel* reply,
                                    uint32_t flags = 0);

	//返回this
	virtual BBinder*    localBinder();
    ......
}

// ----------------------------------------------------------------------

class BpRefBase : public virtual RefBase
{
protected:
    explicit                BpRefBase(const sp<IBinder>& o);
    virtual                 ~BpRefBase();
    inline IBinder* remote() const { return mRemote; }

private:
                            BpRefBase(const BpRefBase& o);
    BpRefBase&              operator=(const BpRefBase& o);
	//此为BpBinder
    IBinder* const          mRemote;

	......

};




```
