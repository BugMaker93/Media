<img width="924" height="603" alt="image" src="https://github.com/user-attachments/assets/81c8a004-8908-42f7-800f-a13e10e197c3" />

上图是 clearkey DRM plugin 的类图，DrmFactory 实现 IDrmFactory，向外提供创建DrmPlugin的功能， CryptoFactory实现ICryptoFactory接口，向外提供常见CryptoPlugin的功能，SessionLibray 作为一个 sessionManager 的角色，提供创建销毁查找session的功能，管理所有创建的 session，IDrmPlugin 和 ICryptoPlugin 发过来的请求，会转发到session内部，处理getKeyRequest，provideKeyResponse，decrypt等重要的工作，InitDataParser 用于解析getKeyRequest传入的InitData，解析出KeyId(经过base64编码)，生成json格式的keyrequest。JsonWebKey 解析 provideKeyResponse 传入的 keyReaponse，内部包含解密的 key，AesCtrDecryptor 用于解密。

<img width="1030" height="524" alt="image" src="https://github.com/user-attachments/assets/07d119d5-9a5e-41dd-8d20-9b9555b918d0" />

上图描述Clearkey 的 PSSH data 的格式，详细可参考链接：

https://w3c.github.io/encrypted-media/format-registry/initdata/cenc.html

<img width="919" height="254" alt="image" src="https://github.com/user-attachments/assets/13f8ca56-3440-42b8-af0e-fa88427c0d3b" />

上图描述ClearKey的 keyRequest 请求和 keyResponse 的格式，是Json格式；kids 为 key id，type取值 temporary 和 persistent-license，前者为 STREAMING，后者为OFFLINE；k 描述的是 key 的值，用于解密，kty为 key type。关于协议可以参考如下连接：

https://dvcs.w3.org/hg/html-media/raw-file/default/encrypted-media/encrypted-media.html#clear-key

<img width="1888" height="2189" alt="image" src="https://github.com/user-attachments/assets/8af2a04c-05b0-42fe-95c2-d0f599b925a2" />

创建 ClearKey 的 DrmPlugin 和 Crypto 之前，先判断传入的 UUID 是否是 ClearKey 的 UUID；创建 DrmPlugin 是会初始化一些 properties；可以调用 getPropertyXXX 和 setPropertyXXX 修改这些属性；openSession 会从 SessionLibary 中创建一个 session；将从文件中获得的 initData (比如 mp4 中的 pssh )，以参数调用 getKeyRequest，会在 initDataParser 中解析 pssh 中的 keyid，将其转化为 Base64 格式，然后生成如上的一个 Json 格式的 reques；。request 发给服务器后，服务器回返回一个 keyResponse，在 Session 中解析出 其中的所有 key (包含 keyid，keyval )，记录在 KeyMap 中；如果是OFFLINE license，会生成一个 ID 与该 response 关联，再将该 keyResponse 保持在 device file 中；以 ID 为参数调用 restoreKey 可以重新从 device file 中加载 License，以 license 为参数调用 provideKeyRequest 来加载 Offline key；

创建 CryptoPlugin 时，有个参数为session id，可从sessionLibary中获取 session 实例；调用 setShareBufferBase 设置输入输出 buffer，decrypt 会使用上面解析的 keymap，然后解密数据，放入 buffer 中。
