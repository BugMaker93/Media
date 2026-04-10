# overview

HAL IPC通常使用HIDL，但现在已经转向AIDL。

AIDL &rarr; aidl文件

HIDL &rarr; hal文件

# HIDL
>以drm为例

**android所有的hal文件位置**：Android\hardware\interfaces

**android的hal生成的文件位置**:Android\out\soong\.intermediates\hardware\interfaces

**android的hal生成文件的脚本位置**:Android\hardware\interfaces
<img width="803" height="78" alt="image" src="https://github.com/user-attachments/assets/d710d7d1-c1d2-4971-878a-7f88631f309c" />

**drm hal文件位置**：Android\hardware\interfaces\drm
<img width="862" height="280" alt="image" src="https://github.com/user-attachments/assets/9fea4a04-0fb0-4637-8327-f16216b7effc" />

**drm hal生成文件位置**:Android\out\soong\.intermediates\hardware\interfaces\drm\1.0
<img width="661" height="281" alt="image" src="https://github.com/user-attachments/assets/39d82565-bdd1-4825-b07e-6ff191c8eed4" />

**android.hardware.drm@1.0 就是模块对应的库文件**：
<img width="832" height="53" alt="image" src="https://github.com/user-attachments/assets/5953bd44-147c-4bb5-b5c7-1ee67671b9fe" />

**android.hardware.drm@1.0_genc++ 为生成对应的 C++文件**：
<img width="937" height="299" alt="image" src="https://github.com/user-attachments/assets/ff6b432f-8d51-4a5d-abc6-5b6f773d8ad2" />

**android.hardware.drm@1.0_genc++_headers 为生成的 C++ 所需的头文件**：
<img width="925" height="1070" alt="image" src="https://github.com/user-attachments/assets/c2a6cb3d-1b6c-4913-838f-93835a6f75fd" />

对于Client端集成so后则可以和HAL层的Service进行通信。refer to [AOSP -HIDL IPC.md](https://github.com/BugMaker93/Media/blob/main/AOSP%20-HIDL%20IPC.md)

HAL层的Service需要HAL自己实现。但Android\hardware\interfaces\drm\1.0\default会生成一个默认的service提供给HAL实现。
