# Code
source:Android/frameworks/av/drm

aidl/hidl:Android/hardware/interfaces/drm

plugin:
- clearkey:Android\frameworks\av\drm\mediadrm\plugins\clearkey

- widevine:Android\vendor\widevine

# ClearKey

refer to [DRM/ClearKey.md](https://github.com/BugMaker93/Media/blob/main/DRM/ClearKey.md)

# WideVine

refer to

[DRM/Widevine Overview.md](https://github.com/BugMaker93/Media/blob/main/DRM/Widevine%20Overview.md)

[DRM/Widevine Demo.md](https://github.com/BugMaker93/Media/blob/main/DRM/Widevine%20Demo.md)

wvts就是验证drm的cts。

## NoProvisionException
没有做过 provision，路 "/data/vender/mediadrm/L1" 中不存在 DRM Certificate ，检测到文件不存在，widevine 会通知 provision required event，通知到 client，会触发 NoProvision 异常。

<img width="1317" height="877" alt="image" src="https://github.com/user-attachments/assets/e043c6ac-46a2-4a0b-a5dc-a927cc254920" />

## liboemcrypto.so
```
//Android/vendor/widevine/libwvdrmengine/cdm/core/src/oemcrypto_adapter_dynamic.cpp
OEMCryptoResult Initialize() {
    LoadLevel3();
	//LoadLevel1前会dlopen，找liboemcrypto.so，有才支持L1。L3不需要此so。
    if (LoadLevel1()) {
	}else{
     	FallBackToLevel3();
	}
}
```
