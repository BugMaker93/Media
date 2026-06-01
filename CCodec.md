
CCodecCallbackImpl：CCodecCallback的作用是通过CCodec调用到MediaCodec::CodecCallback：CodecBase::CodecCallback。

Codec2Client代理ComponentStore:IComponentStore代理C2PlatformComponentStore:C2ComponentStore。
Codec2Client::Component代理Component：IComponent代理C2SoftRawDec：SimpleC2Component：C2Component。


<img width="1165" height="756" alt="image" src="https://github.com/user-attachments/assets/cf604ccf-bd78-4d28-9c63-827cbb7c1597" />

Codec2Client::Component后续的调用没有画，会一路调到SimpleC2Component中。
