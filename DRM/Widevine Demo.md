# source
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
MediaDrm.getProvisionRequest()&MediaDrm.provideProvisionResponse()&MediaDrm.getKeyRequest()&MediaDrm.provideKeyResponse()的request和response**使用Base64解码后需要使用protocol buffer解析**。Base64 refer to [DRM/Base64.md](https://github.com/BugMaker93/Media/blob/main/DRM/Base64.md)

idevine通过license_protocol.proto,device_files.proto提供了protocol格式。

license_protocol.proto（Android/vendor/widevine/libwvdrmengine/cdm/core/src/device_files.proto）定义了所有Widevine的消息，例如MediaDrm.getProvisionRequest()&MediaDrm.provideProvisionResponse()&MediaDrm.getKeyRequest()&MediaDrm.provideKeyResponse()。

device_files.proto（Android/vendor/widevine/libwvdrmengine/cdm/core/src/device_files.proto）定义offline相关消息。

| 接口 | 对应协议消息 |
|---|---|
| MediaDrm.getProvisionRequest() | license_protocol.proto->SignedProvisioningMessage（optional bytes message 用的是 ProvisioningRequest） |
| MediaDrm.provideProvisionResponse() | license_protocol.proto->SignedProvisioningMessage（optional bytes message 用的是 ProvisioningResponse） |
| MediaDrm.getKeyRequest() | license_protocol.proto->SignedMessage(type=LICENSE_REQUEST) |
| &MediaDrm.provideKeyResponse() | license_protocol.proto->SignedMessage(type=LICENSE) |

google提供了[tools](https://integration.widevine.com/diagnostics)去解析keyRequest&keyResponse即**Base64解码后再使用protocol buffer解析**，没有提供provisionRequest&provisionResponse,但原理相同。

# MediaDrm.getProvisionRequest()
这是Base64编码值，不可读
```
ProvisionRequest: Q3VrSEVnUTNhRlkyR2dRSUFCSUFLcmdIQ2d4M2FXUmxkbWx1WlM1amIyMFNFRkZEVC1La1RIWTd6
Q3lDYWkxdS1hY2FnQVZUMkV2dEcxaC00RVZ5bDRoNTdCdFFFVU1sOEUycXRwOWNJZW9COVlDNzhY
YXQzSTBOQTI2OF9nQkt4RnhyY25wTi1CNGxZR1haRkhqaG9SMzZFeFRWX2ozQnBnT0RBV25pelhR
NnBSRjNsNkVJUTl3dDZGNEtZUXNGSkZab0JjS1ROenRCeTd3UG5vb1lYZVg0MnNuVzBtVTI2ajhl
UDVMQVFRY29OaHBFNWlpT1FZTnhXN3Z0enc1ODdjVkc3Ukh2aG9BclhlRnFSY3N5cUdBWks1OEVL
Z2RFZmhaWjhDWi1OTWxkQ3ZQWjdaX3ctd1JNaGlva3ZmQkpETjZlZXZwTmpkcENTN0V1SndUcGkt
al9KaTk0ZmkwSDhsdHJHUUpBTktSaDU2dVQ4blk0ODFiMG9BLVh5VHpTaFlENDRxRU94cVRCQmNG
c1RSTmhmdkR3WjlmNWlrVmlXZDBmRzBUanZ4TW5YMXNWdk5sX3dhaHFzZ3VyUjZiSWdJdXFEU3BF
NmZUUkpJUmhlVENnaENraGhqNVBQZ3pSRTFnMFVWRUlJQmtRejRYaF9odFhZR0pheTk4aEhoUTRT
ZVo5QjJSUGZRc1RuekhwQnJoNnN2N1lPaDBnS3hIQkZUVHZpV0duOWo3TUt6T0Y4S3RDN1IyZzlP
QWtGcnJaS284VEhBSDI0dHMzaFlKbWp3OEZaSkl0d3h1enV1WHB3eVgtQnRTY0VpTzRUNVA1ZXQx
ZjdEcG5icG0wQkV0Q1BYemZXUEdVdHFjbVlRZGh3UE0xaUVyWi1QVmlILUpDNHl6OUd1eWdYR1hC
blBDcjZKb3R5RGdvclhKTkJxRGh5SWxVQlFkZ1BWWnhxOUVaNDljZ0tNbTJtUV90cjlqNEYtUTlU
Y2RsWXNzTzJ3Rkt4NzA4UU5xSGl5TEc5d0t1RXNKdURrYkNOUG56TmMzeGxwaHhpN0FNcWVmUnBW
QUlPQmZaaVZ3RGQwazdBSlVEbktIVE45Tmlvc0RHSWNPY1Mzc0V3Qmk1NWdidnVCMExod3Nvdjlh
RGdvbm45N0VTNkI0Y3lQVVlMZmpGRmJkdE9zQVYtYU5rLXFTQ3NvTmMwV3V3WlQ4NnlGT3h0dHdi
TGJzUzZodWdJLWN1SWhBSkM4TThrbzdHWmxfQzdvT3pydzNuS29BQ1M3T09iSWNRMlhlTVYwMG9i
ZzdlSXEtQzJOd0hLLUtSVWlOYnRXNVhQaXNzQnpYRzNQUUZvRnZqclBGYXV2OEpneDBiSHFrVEtx
VGtJUkxKeFl3dXFJYmJOOVE5SV9ITU5mcm9Gb1ZtNWozLTI2RHd0YlVOU2VEWElvSlExalU0TzlO
Snk5ZFdfZGlGVE1YVXI3SWhFZjRYR08yUG5TYzEySkRxRlhocEFwSVk3VTdhdngwVzRSMmpCa29i
VzZrZTM4MEFqTTBoWEdKaS1TejctV3RyVmZ1cTB2YXpwME9ocW1ON1RDWUJxcklHaG9HQnR1Uk53
elJLTV9lNUdxQkZheWloTnFTT09sT0R0Mk9YV2EyeWJQRW04aUFYM2RTUk1BVkZCa196ZE5odUtV
NllPOXlMVTM5bUpodE8yWVNPRWw1T3FQYklhXzUzS1RyV3lvaFBKem9nWm9VeExkakkyNkxDSHRO
M2M0MXM4TVZ6WVAyemlyenFfbmpEaFV5VXVCOFNJTF94dHBFeG4zRXBqSGtuOVlkVF9QdU5adU0t
NGd5M0RxMk1ucjFUVFk3VEdBSXlYZ0FBQUFVQUFBQmVBQUVBRWpaV2FEY0FBQUFEQUFBQVFBQUFB
QUFBQUFBQUFBQUFCd0FBQUFZQUFNWm1BQklBQVFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFB
QUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFCQUFn
```
这是Base64解码后并使用protocol buffer解析后值，可读
```
{
    "message": {
        "encrypted_client_id": {
            "provider_id": "widevine.com",
            "service_certificate_serial_number": "UUNP4qRMdjvMLIJqLW75pw==",
            "encrypted_client_id": "U9hL7RtYfuBFcpeIeewbUBFDJfBNqrafXCHqAfWAu/F2rdyNDQNuvP4ASsRca3J6TfgeJWBl2RR44aEd+hMU1f49waYDgwFp4s10OqURd5ehCEPcLeheCmELBSRWaAXCkzc7Qcu8D56KGF3l+NrJ1tJlNuo/Hj+SwEEHKDYaROYojkGDcVu77c8OfO3FRu0R74aAK13hakXLMqhgGSufBCoHRH4WWfAmfjTJXQrz2e2f8PsETIYqJL3wSQzennr6TY3aQkuxLicE6Yvo/yYveH4tB/JbaxkCQDSkYeerk/J2OPNW9KAPl8k80oWA+OKhDsakwQXBbE0TYX7w8GfX+YpFYlndHxtE478TJ19bFbzZf8GoarILq0emyICLqg0qROn00SSEYXkwoIQpIYY+Tz4M0RNYNFFRCCAZEM+F4f4bV2BiWsvfIR4UOEnmfQdkT30LE58x6Qa4erL+2DodICsRwRU074lhp/Y+zCszhfCrQu0doPTgJBa62SqPExwB9uLbN4WCZo8PBWSSLcMbs7rl6cMl/gbUnBIjuE+T+XrdX+w6Z26ZtARLQj1831jxlLanJmEHYcDzNYhK2fj1Yh/iQuMs/RrsoFxlwZzwq+iaLcg4KK1yTQag4ciJVAUHYD1WcavRGePXICjJtpkP7a/Y+BfkPU3HZWLLDtsBSse9PEDah4sixvcCrhLCbg5GwjT58zXN8ZaYcYuwDKnn0aVQCDgX2YlcA3dJOwCVA5yh0zfTYqLAxiHDnEt7BMAYueYG77gdC4cLKL/Wg4KJ5/exEugeHMj1GC34xRW3bTrAFfmjZPqkgrKDXNFrsGU/OshTsbbcGy27EuoboCPnLg==",
            "encrypted_client_id_iv": "CQvDPJKOxmZfwu6Ds68N5w==",
            "encrypted_privacy_key": "S7OObIcQ2XeMV00obg7eIq+C2NwHK+KRUiNbtW5XPissBzXG3PQFoFvjrPFauv8Jgx0bHqkTKqTkIRLJxYwuqIbbN9Q9I/HMNfroFoVm5j3+26DwtbUNSeDXIoJQ1jU4O9NJy9dW/diFTMXUr7IhEf4XGO2PnSc12JDqFXhpApIY7U7avx0W4R2jBkobW6ke380AjM0hXGJi+Sz7+WtrVfuq0vazp0OhqmN7TCYBqrIGhoGBtuRNwzRKM/e5GqBFayihNqSOOlODt2OXWa2ybPEm8iAX3dSRMAVFBk/zdNhuKU6YO9yLU39mJhtO2YSOEl5OqPbIa/53KTrWyohPJw=="
        },
        "nonce": "N2hWNg==",
        "options": {
            "certificate_type": "WIDEVINE_DRM",
            "certificate_authority": ""
        },
        "spoid": "ZoUxLdjI26LCHtN3c41s8MVzYP2zirzq/njDhUyUuB8="
    },
    "signature": "v/G2kTGfcSmMeSf1h1P8+41m4z7iDLcOrYyevVNNjtM=",
    "provisioning_type": "PROVISIONING_20",
    "oemcrypto_core_message": "AAAABQAAAF4AAQASNlZoNwAAAAMAAABAAAAAAAAAAAAAAAAHAAAABgAAxmYAEgABAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA==",
    "protocol_version": "VERSION_1_1"
}
```

# MediaDrm.provideProvisionResponse()
这是Base64编码值，不可读
```
ProvisionResponse: ewogICJoZWFkZXIiOiB7fSwKICAic2lnbmVkUmVzcG9uc2UiOiAiQ3B3VkN0QUpvdWNDQ0Z1cGRE
amJkSDNjbzFGMnlRUThhcnBsX2NBUnFZeEQyd1JkUjM4SFZoWWhBbXpJdXlabDBYS0RLR0RyRXVM
M0NPZEdZend3QmRZUkFCR3BMNE83YnkxYTFBaWIwcVNfbW1EX0pvQU5ydmc5NGpiMFU5bkZucjZF
RlpmMDFpanJucS1wUVpERjBOUlFRdnJ5T2ZxZGxDbk9ORXFTc2tPSmgtNENpcE80LWhzdFlRSXY4
c0gtWjh0MEp1LWVnMEctcjdCbkV4ZWxJczNTaGVxM21kY042b0VIRkN2emlzVGxhcEZaOXgyVldf
ekdlMGhaQUFlVlFkcjlId2ZaOTdVNUNVTkNVa2dLbGRiS1J0LWxWQWVvTkxjd19xVUdteFdhT1k5
YjFMNXprNjY4eFdOZVc0OTlVUVdSbkxjczFuc3FzbFZNQjUxSTdqcFIzV285NDZKb2MwbVMybzNy
V1RfQ2Nxd0tvQ3NXcUFtNVZONk9PUmdGN09XcDF5V0hvSTVQbWRNY0k3d2NOdlNodVdDTjkzSFpI
N01nMWFjNXZYX1k1cXRscnV3SnVtQUpSYkg1alpRdUtYdXdGd0R3MUFYelJMS09ZSFVxTWl3RjEx
LUUzVGpmaDVSVWZSS2NHNUpGeWViY09lelZRdHNPN1Y4SFFxSnV4dnAzTEJMRDlXV0RYTDlYS3R2
b3dLeW83NExpRVg4Q1psZlRDNVNPVXVNclBPRmJOVk1pdXNNbkxmbkVjUEVVLURQeHN1Z0Npal92
OUtkcGc4Y2RqbDlWWmI5U3pmR1dyTEZDR1JFTXgxMUtSRm5QZTg3c0RfNFpMbW13bmFYOTZaLWF6
SFdZX1l2SHJyeG5pRENtUTBYVDB6YkpnelVWUDRHWWFwSEJZN2dUVGRzalVPNV8zTkcwLXpDbW1R
RzVzQ0Z5ZXdMUmt4bzhiM3hHM0JLYnBUeWdSWjZsQ3dPZ0ZwTS1UTFFUZ0hoS01aZGFGNXFIRnJB
S25raFVqSTB5Snk1N193bEFlUlI1WENLMC1TblFvRTNYTVNXTWRnTmFtRGFDS1BqSTMyVFFqZVcy
UV84THQwTHBpWlEzLWtNam9hbUdNVF8xaU15M0RGQm9wSVp0NF9CeVdvVGJfbUhSemxmM0FJcFhi
dEVhNnM1dmYwdllXTzhGX0JQWGJ5Vkdqa1NIRTduNm0wNEFRZ3Y0VHlQdVBFVndQdXBxZTRRN21N
aUItWVFWMFFQOWh2TDFqSl9qMUhZdWx6Y3d4NDRhU2h2NzJVVHNqNEFNck1qUEh5RkVwR0ZXMzdm
bm5vdHQtdnVma3F2cnJ0WXBVN2xXaUJXVHhSTmNpZGxKSFdlQ2o5ZHhoeFZZa200WEw2Vk03VFpR
dVNOOXVORmllNFRjaWM4TjJSZ0ZFSzZDbU5wcHR0YU92WkUwdW04RERjdTlaSHZ2WGU2c3ptbDJu
MVljTnpsbDVaVUNVZVdoZWtlTFZkbWJUUFlKcDNvamlmeWh1dXpjNWJBNGhYUDMxRERlTWp2QVNO
Uk0wTk1EbWU3QWZsMC1pVndseFRlWXpORTJFWXljSDdocDF5eDRlZm8ycEplSVl4S3ZJYVFlZ1NF
WDZQTUVwc0JXSURmVzVhQnNQSi1tR2RHWlo5c1BDYjU1aHVZcFQ4ODVwSDluQVVUWjcyYVg3VGFt
ei1YZ1BjSDd6T3JkY09jWXc4YXg0M1ctMjFYT1B6Z0FSN2tydkF3Y0w5QlRFWTcxMDF3SW9rXzFL
aXEwZjlreXFEalJ3RUtubUpzX0ZYYlBpN29lWDJJc1Fadmk2dXFUSDRoeHFLTHdZeW4wT2FnU0U2
MHNHQXFjdHJWVFBHbGZHLVlmaHJmNHhaV0U4QVJ3ZTR3YU4yRkRscnBTN2pIOXVGanktT2RETUJB
MEJXQklzNWZtM3BEQXpVMGFOeXU5ZGlpTVRPckFiQnNPekNBcmlIVVM5aWtnTF8za3FsMzFtMy1L
RHNERjFVaEpFM2xSNHBuWFpYVUJYeUZsWFdDa0tGLWIxWEZEUnFxZ096d2JwUlRFTjh6aFdnQXNi
cm5BajNmU2QxdEVwUzE0bVdJVFNERFdvcjNZSU1leFpwMzZkV1phQnMtUDMxcWJMb3lQNDUwY3dw
Z1l6YkExSXlGUEN0VllrWUhsMlFDc2hOd1RUcWpIYjRDMUgtTVNFSUhqVHNBOHlTWE1KZ3Y1N1Yt
MEFJd2FyZ3NLN2dNSUFoSWdab1V4TGRqSTI2TENIdE4zYzQxczhNVnpZUDJ6aXJ6cV9uakRoVXlV
dUI4WXliMkV1QVlpamdJd2dnRUtBb0lCQVFDMXpuNXNHZVFaWGUtNEdqRGlCaDF6ajBtSG5LRXQz
T1FLRV9uOXE3a3JZcFJfRTdkNGduNUlueUFXREZoOFhWbEtoZW1RYnAyemNnS21hXzFlVVFZTVpt
NEEycFpBSzRIYzhjZTJ4RUpzX2xWQ1BjTG9rWThwTnZjcG9FeExEMHNrZ3dRS3pwYWNnNjZUSGNt
WHZoUUdiR2l5ZzA1Z3l4cTlUUDJwbEJmdWVCY3RtU3hDVjQ1cExCWWg0S2FWUEg5YVE1YVhQeU1Q
ZmFNeVA3R0otTVJ5U3hDSDlUeWhxUzlsMkQxS0xGUnpJcDUxU0ZhM2VCTmF3TzlUdXRWcUhxUEhw
UmF4YTlaemdrdVF4NWIzMGNndDc2UWlFVDZLSEp5bTZiMHd4SlBUUVdGMXprTmFHOUx2Sm5aN1V6
T2Q4WDRsY2ZZNGNuNDJxNkxnSXUtNzRXZWZBZ01CQUFFbzk5WUJTQUZTcWdFSUFSQUFHb0VCQks0
SkdYMi1YWGZONmFIUy14bHlObl9oZ0dmeTRCY0Z5UGdnNkZMcEdZTWpoLV8zaWFKLTQ0RDBKSzRt
OExrQW9Od0N0VmZ5T3RiYzFMcXlCeFgxcTVOdUlTWENKOVUyLXBQX3FLSnlRUEFvLW1wbXM2X3N3
aDlvVVMwRGg5STNIZ2tTVjFoZWRKMmN3d0dKeVI4bEk1R2pCV2lvUGFQNkRmVk9GYUVuRHd3eklp
Qm1hb2NEN29MaVBfUnhUa1lTVkRSdFFpbFl5MkFNUE9rckxTbXVLLUpQYWhLQUFpY0lSaU91RWVM
WjU3WXR0ZUkxT1JaM0RiVk5rT2lzWVQxeEJkWDFfUy1WS0RIX1hYRmM4bzJXVml6NmFGNzdrd0ZP
OE05RC1aUk4weEQ3eGM4RV9VX2s5QXFXekRaWGNSRjBPR0dTdEJZVVpZWE00SmlaaEQ5VTJwMUtE
VDlRekJNV2dMbHR5dGRFMFJzTFVlTWl1WFZ0bkV0YmY4TVBfMGc5MWxaVVhJNURVRVRUaFNqZnFX
cl9zcUlVNnhyakdJN3ZsWGo1Y2h3Zzg0d0Y4VndmYmE3eDFMVm90TnFMTmJHTlBOVnF2UGVpUVRS
bkRVZm1uMW5pRTZYbWhxNVBSd0ZSd0JpdkJTZjlDWTIxbzV5aWxVRnlOZzE4MHlyNHl1LWgyd3U0
SGtJNWl4Q0U0M0R1MVFTMlpUYTlWRTZwSDdEYUh5MzVCbGlwRm5IZ2tnYVJiTE1hdHdVS3NRSUlB
UklRSVhjNVplb1lYTFBLM3NNLVVHVzYwUmllOU1TWUJpS09BakNDQVFvQ2dnRUJBTDNqNGkwaDdS
YlBsRlRpT0lJeUhGb1ZwRmRaMnVfZmNsa2k1OV9xSGVvNHN4RjNLLVV4ZDc3ZXdsSXJpbTBsVWRP
Q0RoOFV1dHVNQlMzRHpTRkkwQUdVbHl0N3dWaVRjU2xLTXhRR0lIbmxDRU1YSVI0aE5wU2hYWkhp
OVN0UmROeHVnZ3RQTG1VRks3Y2Z1T3N4NUpleTc5dC1mdW84UG1BTXZOOUhpVGRneGprejdWb2lf
Q3QzVGdkVVc2cHg3Qm9fdy1nWV9YeWVHckx6Q3hFNWhGZm9saUVYRWJscUFyYVYtRHl5eUR5b1pK
dVpaWjdIUXNSRXdWUzZNYmhwN1IxRXhmbDFDLUJKOUljMjRmOWkzd3IzcXMwUWRvZ0ZvS2NVOHlM
b3M1ai1kQi1mak0tMkpuNkhIcWRWVVZiVFgyMDlPQ0NEWVlqSHRSYWJfNWRfNTQ4Q0F3RUFBU2oz
MWdGSUFSS0FBeDlsaWZZdl82YTlYT3ktQjJ6MXc5dzdIRGJyU1A3TFg0YV9rZzl3MWRSamZFVU1Z
MEpoNVJzRk5FdEl2bjdhVmw4bmx1d2lEaGQ2QjRkMFc5Znl1RXYyelNkcGIyTHN6eE44UXpBZW5Q
Y0hEVlVZQ2IyWktQcVlIMUtBZDZtVUJnVDV1amJrUjdoNEZWRnRST25fbDRTd1gzc3ZGREVnaHdN
eEJtSzBpbVJFX3kxVUkyck1EMTJzQlp4WUlOeHo0bGFYdDk5QXRtU0pVR1JTejNLZGN0OHRhT2pM
V1RXWVctMnVjcFhVTU81T3lvNkJxemRra0VlUmloYkJFREJOZkkxQTNTeURzOFJyRFZWODhPV3JU
TTNpWDdRazRrVzBuZmwxaE9mMk1Pa2JWMURSbldOOEVyOTdoZHpDRnlfSFF0bHV0WHp4UUczYWs5
Ul9ZRnpEM25ETjRIbFlZVGxoaXhQM19EbnRFRkVaZXFiaDZnTlZlZDNRMUtqbFJweW1RakI5MEtt
eU5jb1lpLTF2bmgxNDJVNjNUMVM5b1JhZVJlY29RcUxBSW9VMmpQSjVnYXFMQmdMd01jaDd3Z3dt
b29jT0dvNmJVX19Ea1RuMHlSMENvZ2F6MkZyNXFiOU5vU3JMU0UxMDdRSkR2Tm5lWHJnWmpTOFNB
OURTUWRxcUJDSUVOMmhXTmhJZ3otXzc5QlRFWjVpVkVoaTF3aDcyRTVwQmFnZEJiZ0xzNF9PWU9h
QVJncjB5TUFBQUFBWUFBQUF3QUFFQUVqWldhRGNBQUFBREFBQUFBQUFBQUFNQUFBVFFBQUFFMVFB
QUFCQUFBQUFBQUFBQUFBPT0iLAogICJraW5kIjogImNlcnRpZmljYXRlcHJvdmlzaW9uaW5nI2Nl
cnRpZmljYXRlUHJvdmlzaW9uaW5nUmVzcG9uc2UiCn0K
```
这是Base64解码后并使用protocol buffer解析后值，可读
```
{
    "message": {
        "device_rsa_key": "oucCCFupdDjbdH3co1F2yQQ8arpl/cARqYxD2wRdR38HVhYhAmzIuyZl0XKDKGDrEuL3COdGYzwwBdYRABGpL4O7by1a1Aib0qS/mmD/JoANrvg94jb0U9nFnr6EFZf01ijrnq+pQZDF0NRQQvryOfqdlCnONEqSskOJh+4CipO4+hstYQIv8sH+Z8t0Ju+eg0G+r7BnExelIs3Sheq3mdcN6oEHFCvzisTlapFZ9x2VW/zGe0hZAAeVQdr9HwfZ97U5CUNCUkgKldbKRt+lVAeoNLcw/qUGmxWaOY9b1L5zk668xWNeW499UQWRnLcs1nsqslVMB51I7jpR3Wo946Joc0mS2o3rWT/CcqwKoCsWqAm5VN6OORgF7OWp1yWHoI5PmdMcI7wcNvShuWCN93HZH7Mg1ac5vX/Y5qtlruwJumAJRbH5jZQuKXuwFwDw1AXzRLKOYHUqMiwF11+E3Tjfh5RUfRKcG5JFyebcOezVQtsO7V8HQqJuxvp3LBLD9WWDXL9XKtvowKyo74LiEX8CZlfTC5SOUuMrPOFbNVMiusMnLfnEcPEU+DPxsugCij/v9Kdpg8cdjl9VZb9SzfGWrLFCGREMx11KRFnPe87sD/4ZLmmwnaX96Z+azHWY/YvHrrxniDCmQ0XT0zbJgzUVP4GYapHBY7gTTdsjUO5/3NG0+zCmmQG5sCFyewLRkxo8b3xG3BKbpTygRZ6lCwOgFpM+TLQTgHhKMZdaF5qHFrAKnkhUjI0yJy57/wlAeRR5XCK0+SnQoE3XMSWMdgNamDaCKPjI32TQjeW2Q/8Lt0LpiZQ3+kMjoamGMT/1iMy3DFBopIZt4/ByWoTb/mHRzlf3AIpXbtEa6s5vf0vYWO8F/BPXbyVGjkSHE7n6m04AQgv4TyPuPEVwPupqe4Q7mMiB+YQV0QP9hvL1jJ/j1HYulzcwx44aShv72UTsj4AMrMjPHyFEpGFW37fnnott+vufkqvrrtYpU7lWiBWTxRNcidlJHWeCj9dxhxVYkm4XL6VM7TZQuSN9uNFie4Tcic8N2RgFEK6CmNppttaOvZE0um8DDcu9ZHvvXe6szml2n1YcNzll5ZUCUeWhekeLVdmbTPYJp3ojifyhuuzc5bA4hXP31DDeMjvASNRM0NMDme7Afl0+iVwlxTeYzNE2EYycH7hp1yx4efo2pJeIYxKvIaQegSEX6PMEpsBWIDfW5aBsPJ+mGdGZZ9sPCb55huYpT885pH9nAUTZ72aX7Tamz+XgPcH7zOrdcOcYw8ax43W+21XOPzgAR7krvAwcL9BTEY7101wIok/1Kiq0f9kyqDjRwEKnmJs/FXbPi7oeX2IsQZvi6uqTH4hxqKLwYyn0OagSE60sGAqctrVTPGlfG+Yfhrf4xZWE8ARwe4waN2FDlrpS7jH9uFjy+OdDMBA0BWBIs5fm3pDAzU0aNyu9diiMTOrAbBsOzCAriHUS9ikgL/3kql31m3+KDsDF1UhJE3lR4pnXZXUBXyFlXWCkKF+b1XFDRqqgOzwbpRTEN8zhWgAsbrnAj3fSd1tEpS14mWITSDDWor3YIMexZp36dWZaBs+P31qbLoyP450cwpgYzbA1IyFPCtVYkYHl2QCshNwTTqjHb4C1H+M=",
        "device_rsa_key_iv": "geNOwDzJJcwmC/ntX7QAjA==",
        "device_certificate": {
            "drm_certificate": {
                "type": "DEVICE",
                "serial_number": "ZoUxLdjI26LCHtN3c41s8MVzYP2zirzq/njDhUyUuB8=",
                "creation_time_seconds": 1728126665,
                "public_key": "MIIBCgKCAQEAtc5+bBnkGV3vuBow4gYdc49Jh5yhLdzkChP5/au5K2KUfxO3eIJ+SJ8gFgxYfF1ZSoXpkG6ds3ICpmv9XlEGDGZuANqWQCuB3PHHtsRCbP5VQj3C6JGPKTb3KaBMSw9LJIMECs6WnIOukx3Jl74UBmxosoNOYMsavUz9qZQX7ngXLZksQleOaSwWIeCmlTx/WkOWlz8jD32jMj+xifjEcksQh/U8oakvZdg9SixUcyKedUhWt3gTWsDvU7rVah6jx6UWsWvWc4JLkMeW99HILe+kIhE+ihycpum9MMST00Fhdc5DWhvS7yZ2e1MznfF+JXH2OHJ+Nqui4CLvu+FnnwIDAQAB",
                "system_id": 27511,
                "algorithm": "RSA",
                "rot_id": {
                    "version": "ROOT_OF_TRUST_ID_VERSION_1",
                    "key_id": 0,
                    "encrypted_unique_id": "BK4JGX2+XXfN6aHS+xlyNn/hgGfy4BcFyPgg6FLpGYMjh+/3iaJ+44D0JK4m8LkAoNwCtVfyOtbc1LqyBxX1q5NuISXCJ9U2+pP/qKJyQPAo+mpms6/swh9oUS0Dh9I3HgkSV1hedJ2cwwGJyR8lI5GjBWioPaP6DfVOFaEnDwwz",
                    "unique_id_hash": "ZmqHA+6C4j/0cU5GElQ0bUIpWMtgDDzpKy0priviT2o="
                }
            },
            "signature": "JwhGI64R4tnnti214jU5FncNtU2Q6KxhPXEF1fX9L5UoMf9dcVzyjZZWLPpoXvuTAU7wz0P5lE3TEPvFzwT9T+T0CpbMNldxEXQ4YZK0FhRlhczgmJmEP1TanUoNP1DMExaAuW3K10TRGwtR4yK5dW2cS1t/ww//SD3WVlRcjkNQRNOFKN+pav+yohTrGuMYju+VePlyHCDzjAXxXB9trvHUtWi02os1sY081Wq896JBNGcNR+afWeITpeaGrk9HAVHAGK8FJ/0JjbWjnKKVQXI2DXzTKvjK76HbC7geQjmLEITjcO7VBLZlNr1UTqkfsNofLfkGWKkWceCSBpFssw==",
            "signer": {
                "drm_certificate": {
                    "type": "DEVICE_MODEL",
                    "serial_number": "IXc5ZeoYXLPK3sM+UGW60Q==",
                    "creation_time_seconds": 1662073374,
                    "public_key": "MIIBCgKCAQEAvePiLSHtFs+UVOI4gjIcWhWkV1na799yWSLn3+od6jizEXcr5TF3vt7CUiuKbSVR04IOHxS624wFLcPNIUjQAZSXK3vBWJNxKUozFAYgeeUIQxchHiE2lKFdkeL1K1F03G6CC08uZQUrtx+46zHkl7Lv235+6jw+YAy830eJN2DGOTPtWiL8K3dOB1RbqnHsGj/D6Bj9fJ4asvMLETmEV+iWIRcRuWoCtpX4PLLIPKhkm5llnsdCxETBVLoxuGntHUTF+XUL4En0hzbh/2LfCveqzRB2iAWgpxTzIuizmP50H5+Mz7Ymfocep1VRVtNfbT04IINhiMe1Fpv/l3/njwIDAQAB",
                    "system_id": 27511,
                    "algorithm": "RSA"
                },
                "signature": "H2WJ9i//pr1c7L4HbPXD3DscNutI/stfhr+SD3DV1GN8RQxjQmHlGwU0S0i+ftpWXyeW7CIOF3oHh3Rb1/K4S/bNJ2lvYuzPE3xDMB6c9wcNVRgJvZko+pgfUoB3qZQGBPm6NuRHuHgVUW1E6f+XhLBfey8UMSCHAzEGYrSKZET/LVQjaswPXawFnFgg3HPiVpe330C2ZIlQZFLPcp1y3y1o6MtZNZhb7a5yldQw7k7KjoGrN2SQR5GKFsEQME18jUDdLIOzxGsNVXzw5atMzeJftCTiRbSd+XWE5/Yw6RtXUNGdY3wSv3uF3MIXL8dC2W61fPFAbdqT1H9gXMPecM3geVhhOWGLE/f8Oe0QURl6puHqA1V53dDUqOVGnKZCMH3QqbI1yhiL7W+eHXjZTrdPVL2hFp5F5yhCosAihTaM8nmBqosGAvAxyHvCDCaihw4ajptT/8OROfTJHQKiBrPYWvmpv02hKstITXTtAkO82d5euBmNLxID0NJB2qoE"
            }
        },
        "nonce": "N2hWNg=="
    },
    "signature": "z+/79BTEZ5iVEhi1wh72E5pBagdBbgLs4/OYOaARgr0=",
    "oemcrypto_core_message": "AAAABgAAADAAAQASNlZoNwAAAAMAAAAAAAAAAwAABNAAAATVAAAAEAAAAAAAAAAA"
}
```

# MediaDrm.getKeyRequest()
这是Base64编码值，不可读
```
KeyRequest: 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
```
这是Base64解码后并使用protocol buffer解析后值，可读
```
{
    "type": "LICENSE_REQUEST",
    "msg": {
        "client_id": {
            "client_info": [
                {
                    "name": "application_name",
                    "value": "com.google.android.exoplayer2.demo"
                },
                {
                    "name": "origin",
                    "value": ""
                },
                {
                    "name": "package_certificate_hash_bytes",
                    "value": "I4poqEeZDbWVqdqzov/M/E24WB9SduMQpHChvY5D6Wc="
                },
                {
                    "name": "company_name",
                    "value": "Harman International Industries Inc."
                },
                {
                    "name": "model_name",
                    "value": "AOSP on Harman Platform"
                },
                {
                    "name": "architecture_name",
                    "value": "arm64-v8a"
                },
                {
                    "name": "device_name",
                    "value": "pf_vfs_space4_ea9"
                },
                {
                    "name": "product_name",
                    "value": "pf_vfs_space4_ea9"
                },
                {
                    "name": "build_info",
                    "value": "HARMAN/pf_vfs_space4_ea9/pf_vfs_space4_ea9:14/UQ1A.240205.004.B1/1347:userdebug/test-keys"
                },
                {
                    "name": "widevine_cdm_version",
                    "value": "18.0.0@666"
                },
                {
                    "name": "oem_crypto_security_patch_level",
                    "value": "3"
                },
                {
                    "name": "oem_crypto_build_information",
                    "value": "{\"soc_vendor\":\"SEC\",\"soc_model\":\"Exynosautov9\",\"ta_ver\":\"1.0\",\"uses_opk\":false,\"tee_os\":\"\",\"tee_os_ver\":\"1.0\",\"is_debug\":false}"
                }
            ],
            "type": "DRM_DEVICE_CERTIFICATE",
            "token": {
                "drm_certificate": {
                    "type": "DEVICE",
                    "serial_number": "ZoUxLdjI26LCHtN3c41s8MVzYP2zirzq/njDhUyUuB8=",
                    "creation_time_seconds": 1728126665,
                    "public_key": "MIIBCgKCAQEAtc5+bBnkGV3vuBow4gYdc49Jh5yhLdzkChP5/au5K2KUfxO3eIJ+SJ8gFgxYfF1ZSoXpkG6ds3ICpmv9XlEGDGZuANqWQCuB3PHHtsRCbP5VQj3C6JGPKTb3KaBMSw9LJIMECs6WnIOukx3Jl74UBmxosoNOYMsavUz9qZQX7ngXLZksQleOaSwWIeCmlTx/WkOWlz8jD32jMj+xifjEcksQh/U8oakvZdg9SixUcyKedUhWt3gTWsDvU7rVah6jx6UWsWvWc4JLkMeW99HILe+kIhE+ihycpum9MMST00Fhdc5DWhvS7yZ2e1MznfF+JXH2OHJ+Nqui4CLvu+FnnwIDAQAB",
                    "system_id": 27511,
                    "algorithm": "RSA",
                    "rot_id": {
                        "version": "ROOT_OF_TRUST_ID_VERSION_1",
                        "key_id": 0,
                        "encrypted_unique_id": "BK4JGX2+XXfN6aHS+xlyNn/hgGfy4BcFyPgg6FLpGYMjh+/3iaJ+44D0JK4m8LkAoNwCtVfyOtbc1LqyBxX1q5NuISXCJ9U2+pP/qKJyQPAo+mpms6/swh9oUS0Dh9I3HgkSV1hedJ2cwwGJyR8lI5GjBWioPaP6DfVOFaEnDwwz",
                        "unique_id_hash": "ZmqHA+6C4j/0cU5GElQ0bUIpWMtgDDzpKy0priviT2o="
                    }
                },
                "signature": "JwhGI64R4tnnti214jU5FncNtU2Q6KxhPXEF1fX9L5UoMf9dcVzyjZZWLPpoXvuTAU7wz0P5lE3TEPvFzwT9T+T0CpbMNldxEXQ4YZK0FhRlhczgmJmEP1TanUoNP1DMExaAuW3K10TRGwtR4yK5dW2cS1t/ww//SD3WVlRcjkNQRNOFKN+pav+yohTrGuMYju+VePlyHCDzjAXxXB9trvHUtWi02os1sY081Wq896JBNGcNR+afWeITpeaGrk9HAVHAGK8FJ/0JjbWjnKKVQXI2DXzTKvjK76HbC7geQjmLEITjcO7VBLZlNr1UTqkfsNofLfkGWKkWceCSBpFssw==",
                "signer": {
                    "drm_certificate": {
                        "type": "DEVICE_MODEL",
                        "serial_number": "IXc5ZeoYXLPK3sM+UGW60Q==",
                        "creation_time_seconds": 1662073374,
                        "public_key": "MIIBCgKCAQEAvePiLSHtFs+UVOI4gjIcWhWkV1na799yWSLn3+od6jizEXcr5TF3vt7CUiuKbSVR04IOHxS624wFLcPNIUjQAZSXK3vBWJNxKUozFAYgeeUIQxchHiE2lKFdkeL1K1F03G6CC08uZQUrtx+46zHkl7Lv235+6jw+YAy830eJN2DGOTPtWiL8K3dOB1RbqnHsGj/D6Bj9fJ4asvMLETmEV+iWIRcRuWoCtpX4PLLIPKhkm5llnsdCxETBVLoxuGntHUTF+XUL4En0hzbh/2LfCveqzRB2iAWgpxTzIuizmP50H5+Mz7Ymfocep1VRVtNfbT04IINhiMe1Fpv/l3/njwIDAQAB",
                        "system_id": 27511,
                        "algorithm": "RSA"
                    },
                    "signature": "H2WJ9i//pr1c7L4HbPXD3DscNutI/stfhr+SD3DV1GN8RQxjQmHlGwU0S0i+ftpWXyeW7CIOF3oHh3Rb1/K4S/bNJ2lvYuzPE3xDMB6c9wcNVRgJvZko+pgfUoB3qZQGBPm6NuRHuHgVUW1E6f+XhLBfey8UMSCHAzEGYrSKZET/LVQjaswPXawFnFgg3HPiVpe330C2ZIlQZFLPcp1y3y1o6MtZNZhb7a5yldQw7k7KjoGrN2SQR5GKFsEQME18jUDdLIOzxGsNVXzw5atMzeJftCTiRbSd+XWE5/Yw6RtXUNGdY3wSv3uF3MIXL8dC2W61fPFAbdqT1H9gXMPecM3geVhhOWGLE/f8Oe0QURl6puHqA1V53dDUqOVGnKZCMH3QqbI1yhiL7W+eHXjZTrdPVL2hFp5F5yhCosAihTaM8nmBqosGAvAxyHvCDCaihw4ajptT/8OROfTJHQKiBrPYWvmpv02hKstITXTtAkO82d5euBmNLxID0NJB2qoE"
                }
            },
            "client_capabilities": {
                "client_token": true,
                "session_token": true,
                "max_hdcp_version": "HDCP_V2_2",
                "oem_crypto_api_version": 18,
                "anti_rollback_usage_table": true,
                "srm_version": 0,
                "can_update_srm": false,
                "supported_certificate_key_type": [
                    "RSA_2048"
                ],
                "analog_output_capabilities": "ANALOG_OUTPUT_NONE",
                "can_disable_analog_output": false,
                "resource_rating_tier": 3,
                "watermarking_support": "WATERMARKING_NOT_SUPPORTED"
            }
        },
        "content_id": {
            "widevine_pssh_data": {
                "pssh_data": [
                    "CAESATUaDXdpZGV2aW5lX3Rlc3QiCjIwMTVfdGVhcnMqAlNE"
                ],
                "license_type": "STREAMING",
                "request_id": "MkY5RENBNTBEMUVEOTY5MTBFMDAwMDAwMDAwMDAwMDA="
            }
        },
        "type": "NEW",
        "request_time": 1728203280,
        "protocol_version": "VERSION_2_1",
        "key_control_nonce": 1496987003
    },
    "signature": "r1V5BjC936g8HSGvjWPxD8nlKjFiQk//lFZTj4fgOuK0apAiWsE5vQGpl2iT/jj/LNf/ZyqAKYg0Bkk3N515BmjnKlJayWQC91sKCjJ+6JImxrrVfGAGW/NjiRzYdKEbOhtCpnmXeThFGYR0/SrFSptmj+IwJIILOtSreD/pWp+yDZdovYhUkUZ8JNhv4m++vfS8Xx7bNycv+TSV8iNrd38MFGljx+xq/R5wH/xwYOjw/VOpREdF9AlY5KIZQrRBusgRSvzbdbYJUxp6MAUCeZ75f3NF7XOKj4szJ+XJqzOM6uQmTK6UVxgXKg30xiKdBcAuTWDHPiRYstOrGGRz9Q==",
    "oemcrypto_core_message": "AAAAAQAAAFoAAQASWTo1ewAAAAIAAAAAAAAAAAAAAAoAAAARAAuCZQASAAEAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"      
}
```

# MediaDrm.getKeyResponse()
这是Base64编码值，不可读
```
KeyResponse: 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
```
这是Base64解码后并使用protocol buffer解析后值，可读
```
{
    "type": "LICENSE",
    "msg": {
        "id": {
            "request_id": "MkY5RENBNTBEMUVEOTY5MTBFMDAwMDAwMDAwMDAwMDA=",
            "session_id": "MkY5RENBNTBEMUVEOTY5MTBFMDAwMDAwMDAwMDAwMDA=",
            "purchase_id": "",
            "type": "STREAMING",
            "version": 0,
            "original_rental_duration_seconds": 30,
            "original_playback_duration_seconds": 30,
            "original_start_time_seconds": 1728203280
        },
        "policy": {
            "can_play": true,
            "can_persist": false,
            "can_renew": false,
            "rental_duration_seconds": 30,
            "playback_duration_seconds": 30,
            "license_duration_seconds": 30,
            "soft_enforce_playback_duration": false,
            "soft_enforce_rental_duration": false
        },
        "key": [
            {
                "iv": "z6INq4c4rSEBn56NOyDAOg==",
                "key": "K1Wik0+3LCzL7PwTh5unudJH3zhGFOk8rMtFGOtUhYdlESKcYZgaJ2bhGV21QAnywaFjAJzq8k5kEGeiot22APnGAuzN8eX7jZllIAOOIK8=",
                "type": "SIGNING"
            },
            {
                "id": "MDAwMDAwMDAwMDAwMDAwMw==",
                "iv": "FVC9MZG10jA9JBCfyO5HDw==",
                "key": "rZflLQ/H4nPd4P9XAjVBMxkvRzpEhjVrcLZ6/WYsIbc=",
                "type": "CONTENT",
                "level": "SW_SECURE_CRYPTO",
                "required_protection": {},
                "requested_protection": {},
                "key_control": {
                    "key_control_block": "a2MxOAAAAB5ZOjV7gAAACA=="
                },
                "track_label": "HD"
            },
            {
                "id": "MDAwMDAwMDAwMDAwMDAwMg==",
                "iv": "fvfggz7Uj74qNrxYor5lbQ==",
                "key": "c1VB0xhdItiGXLBdxX16VXdBzINp+99Q3N0KDiJJBpU=",
                "type": "CONTENT",
                "level": "SW_SECURE_CRYPTO",
                "required_protection": {},
                "requested_protection": {},
                "key_control": {
                    "key_control_block": "a2MxOAAAAB5ZOjV7gAAACA=="
                },
                "track_label": "HD"
            },
            {
                "id": "MDAwMDAwMDAwMDAwMDAwMA==",
                "iv": "hioH/kl9iJFxgGN/5q8AyQ==",
                "key": "lWmK82mdycTCRvQjWLP5U7/R5jUia/CBWxiTW0YMfNo=",
                "type": "CONTENT",
                "level": "SW_SECURE_CRYPTO",
                "required_protection": {},
                "requested_protection": {},
                "key_control": {
                    "key_control_block": "a2MxOAAAAB5ZOjV7gAAACA=="
                },
                "track_label": "AUDIO"
            },
            {
                "id": "MDAwMDAwMDAwMDAwMDAwNw==",
                "iv": "h9TKpOvqH/ADWfiuMWp7Mg==",
                "key": "IBWnvyuabdUdaVLaFnGuIKIogmmUrrVHFl0eCp2s6wk=",
                "type": "CONTENT",
                "level": "SW_SECURE_CRYPTO",
                "required_protection": {},
                "requested_protection": {},
                "key_control": {
                    "key_control_block": "a2MxOAAAAB5ZOjV7gAAACA=="
                },
                "track_label": "HD"
            },
            {
                "id": "MDAwMDAwMDAwMDAwMDAwMQ==",
                "iv": "gFbC72y24OB464VL6ByC0Q==",
                "key": "BGs7Dzip/CrCfgnn2uIySpRP3+EYv+yliLA+X+21ObY=",
                "type": "CONTENT",
                "level": "SW_SECURE_CRYPTO",
                "required_protection": {},
                "requested_protection": {},
                "key_control": {
                    "key_control_block": "a2MxOAAAAB5ZOjV7gAAACA=="
                },
                "track_label": "SD"
            },
            {
                "id": "MDAwMDAwMDAwMDAwMDAwNA==",
                "iv": "iRFjuNIhLxsPUcBlkQTXsg==",
                "key": "D473dmcUA/F6DXTWHLZDYpMafNiVovicg89FAewMMZ8=",
                "type": "CONTENT",
                "level": "SW_SECURE_CRYPTO",
                "required_protection": {},
                "requested_protection": {},
                "key_control": {
                    "key_control_block": "a2MxOAAAAB5ZOjV7gAAACA=="
                },
                "track_label": "SD"
            },
            {
                "id": "MDAwMDAwMDAwMDAwMDAwNQ==",
                "iv": "8WSLPAmwCoPyobeWy/jV1g==",
                "key": "6LQj8O5jwl2EbpiRUEYvBteRMJhWmWnGpU1eUR9pwdI=",
                "type": "CONTENT",
                "level": "SW_SECURE_CRYPTO",
                "required_protection": {},
                "requested_protection": {},
                "key_control": {
                    "key_control_block": "a2MxOAAAAB5ZOjV7gAAACA=="
                },
                "track_label": "SD"
            },
            {
                "id": "MDAwMDAwMDAwMDAwMDAwNg==",
                "iv": "pDLiLITcpSoZKJW4AQkERQ==",
                "key": "UGYYqyv/7No6v/CFgpWnxU/khsRzwLQ3YJ1nUyJuqu0=",
                "type": "CONTENT",
                "level": "SW_SECURE_CRYPTO",
                "required_protection": {},
                "requested_protection": {},
                "key_control": {
                    "key_control_block": "a2MxOAAAAB5ZOjV7gAAACA=="
                },
                "track_label": "SD"
            }
        ],
        "license_start_time": 1728203280,
        "protection_scheme": 0
    },
    "signature": "XPkaKg+o7YAhbe1w4lzMm+xcSBHxLU6iFsnwFNt3M+M=",
    "session_key": "AXC45gjPG8EGWNTO17bw77T+HzomGRExKAQvjAm14dO7fyIyyBxQ9M+DwvUOgbuarAvcAkMH2K3HbEPBK54jgEugreqqS/O762PVELN+O1oUy+juavsIaA0pELbXPEFhYB0ejAKgzcERZ9Y1O274BuAKNR4W8A3I8pTwjd0yUdbRs1BL6Uz9pWLwJdwywPib9PfEXMG9yhqd2/MeO7BG26L5l4npeUYLL8NHVS2zSC6rBmj+pT6upsldZw8FvlRyZ+3FpKzdWOpZtHLfBK2oPSq1bDisKOX9iva/Vkn2wHiDA4Q+VpXLOamma585qun3cMvg+jQFcqk1lTDGCcJXyw==",
    "service_version_info": {
        "license_sdk_version": "19.0.7 Built on Aug 28 2024 10:55:19 (1724867719)"
    },
    "session_key_type": "WRAPPED_AES_KEY",
    "oemcrypto_core_message": "AAAAAgAAAbEAAQASWTo1ewAAAAIAAABsAAAAEAAAAH4AAABQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAeAAAAAAAAAB4AAAAAAAAAAAAAAAAAAAAAAAAAAAgAAADUAAAAEAAAAOYAAAAQAAAA+AAAABAAAAAAAAAAAAAAASQAAAAQAAABPAAAABAAAAFOAAAAEAAAAWAAAAAQAAAAAAAAAAAAAAEkAAAAEAAAAaQAAAAQAAABtgAAABAAAAHIAAAAEAAAAAAAAAAAAAABJAAAABAAAAIPAAAAEAAAAiEAAAAQAAACMwAAABAAAAAAAAAAAAAAASQAAAAQAAACdwAAABAAAAKJAAAAEAAAApsAAAAQAAAAAAAAAAAAAAEkAAAAEAAAAt8AAAAQAAAC8QAAABAAAAMDAAAAEAAAAAAAAAAAAAABJAAAABAAAANHAAAAEAAAA1kAAAAQAAADawAAABAAAAAAAAAAAAAAASQAAAAQAAADrwAAABAAAAPBAAAAEAAAA9MAAAAQAAAAAAAAAAAAAAEkAAAAEA==",
    "using_secondary_key": false
}
```

# CDM Decode&OEMCrypto Decode

CDM和OEMCrypto都要解码。

两者解码的字段不一样。

两者环境不一样，CDM在software，OEMCrypto在hardware。

License Server**解码**license request信息，验证后再加入其他信息**编码**成license response发送给Application。

license response中包括**加密**过的Content keys（这个是用来解码播放内容的关键keys，这个keyLicense Server本来就已知）。

license response会传到CDM和OEMCrypto，CDM和OEMCrypto都会**解码**license response获得自己想要的信息。
