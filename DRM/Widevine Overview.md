# Structure

<img width="1594" height="1035" alt="image" src="https://github.com/user-attachments/assets/f3ddcd41-542e-4de4-92cd-c04cbeb422f8" />

ODK和OEMCrypto属于TEE，是OEM flash进去的。

ODK是OEMCrypto的工具类。

# CDM&OEMCrypto
Widevine CDM（Content Decryption Module）和 OEMCrypto 是 Widevine DRM 体系中的两个关键组件，它们在设备认证和内容保护中扮演不同但互补的角色。以下是它们的主要关系和功能：

Widevine CDM（Content Decryption Module）
	定义：CDM 是一个软件模块，负责解密受保护的内容。它通常嵌入在浏览器或设备中，确保内容在播放时的安全性。
	功能：
		内容解密：处理来自许可证服务器的解密密钥，并使用这些密钥解密受保护的内容。
		与播放器集成：与浏览器或媒体播放器集成，确保解密后的内容可以顺利播放.

OEMCrypto
	定义：OEMCrypto 是一个硬件抽象层，用于在设备上实现安全的内容解密和密钥管理。
	功能：
		消息验证：验证与许可证服务器之间的消息真实性。
		密钥管理：管理设备密钥和内容密钥，确保它们在可信执行环境（TEE）中安全存储和使用。
		内容解密：使用内容密钥解密媒体数据，并将解密后的数据传递给播放器.

关系
	协同工作：Widevine CDM 和 OEMCrypto 协同工作，确保内容在传输和播放过程中的安全性。CDM 负责处理高层次的解密逻辑和与播放器的集成，而 OEMCrypto 负责底层的密钥管理和硬件级别的解密操作.
	安全级别：OEMCrypto 提供了硬件级别的安全性，确保内容密钥和解密操作在 TEE 中进行，从而提高了整体安全性。CDM 则利用这些硬件功能来实现安全的内容播放.

ps
	CDM和OEMCrypto都要解码。
	两者解码的字段不一样。
	两者环境不一样，CDM在software，OEMCrypto在hardware。

# Widevine Level
<img width="959" height="239" alt="image" src="https://github.com/user-attachments/assets/ca3ba667-9f7a-46d3-962e-3f922264e824" />

从Structure也能看出level3不通过TEE。

# Flow
<img width="1059" height="898" alt="image" src="https://github.com/user-attachments/assets/d95bb8d1-e9b4-4f39-9804-16ae8e61a1ab" />

OEMCrypto最后用Content keys进行解密

# OEM Cert 和 DRM Cert区别

在 Widevine DRM 体系中，OEM 证书（OEM cert）和 DRM 证书（DRM cert）是两个不同的概念，它们在设备认证和内容保护中起着不同的作用。以下是它们的主要区别：

OEM 证书（OEM cert）
	定义：OEM 证书是由设备制造商（OEM）生成并安装在设备上的证书，用于设备的初始认证和信任建立。
	用途：
		设备认证：确保设备是由可信的制造商生产的，并且符合 Widevine 的安全要求。
		密钥管理：用于保护设备密钥和其他敏感信息，确保设备能够安全地处理受保护的内容.
		安装：通常在设备制造过程中安装，确保设备在出厂时已经具备必要的安全认证.
DRM 证书（DRM cert）
	定义：DRM 证书是由 DRM 系统（如 Widevine）颁发的证书，用于设备在播放受保护内容时的认证和授权。
	用途：
		内容解密：用于验证设备的身份，并确保设备有权访问和解密受保护的内容。
		许可证请求：在设备请求播放受保护内容时，DRM 证书用于生成和发送许可证请求，获取解密密钥.
		管理：由 DRM 系统管理和分发，确保只有经过认证的设备才能获取解密密钥和播放受保护的内容.

关系和区别
OEM 证书：主要用于设备的初始认证和信任建立，由设备制造商生成和安装。
DRM 证书：主要用于内容解密和播放过程中的认证和授权，由 DRM 系统颁发和管理。

总结来说，OEM 证书确保设备在制造和出厂时的安全性，而 DRM 证书确保设备在播放受保护内容时的安全性和授权。
