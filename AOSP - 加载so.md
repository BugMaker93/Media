# 找到对应so
opendir 和 readdir 是用于目录操作的标准库函数。它们主要用于打开目录、读取目录中的条目以及关闭目录。以下是它们的具体使用方法：

## opendir 函数
opendir 用于打开一个目录，并返回一个指向目录流的指针。其函数原型如下：
```
#include <sys/types.h>
#include <dirent.h>

DIR *opendir(const char *name);

```
-   **参数**：`name` 是要打开的目录路径，可以是绝对路径或相对路径。
-   **返回值**：成功时返回一个指向 `DIR` 结构体的指针，失败时返回 `NULL`。

## readdir 函数
readdir 用于读取目录流中的下一个目录项。其函数原型如下：
```
#include <dirent.h>

struct dirent *readdir(DIR *dirp);

```
-   **参数**：`dirp` 是由 `opendir` 返回的目录流指针。
-   **返回值**：成功时返回一个指向 `struct dirent` 结构体的指针，表示目录中的下一个条目；到达目录末尾或出错时返回 `NULL`。

## closedir 函数
closedir 用于关闭目录流并释放资源。其函数原型如下：
```
#include <sys/types.h>
#include <dirent.h>

int closedir(DIR *dirp);

```
-   **参数**：`dirp` 是由 `opendir` 返回的目录流指针。
-   **返回值**：成功时返回 0，失败时返回 -1。

## 示例代码
以下是一个简单的示例，演示如何使用 opendir 和 readdir 打开目录并读取其中的文件名：

```
#include <stdio.h>
#include <stdlib.h>
#include <dirent.h>

int main(void) {
    DIR *dirp;
    struct dirent *entry;

    // 打开目录
    dirp = opendir("/path/to/directory");
    if (dirp == NULL) {
        perror("opendir");
        return EXIT_FAILURE;
    }

    // 读取目录中的条目
    while ((entry = readdir(dirp)) != NULL) {
        printf("Found file: %s\n", entry->d_name);
    }

    // 关闭目录
    closedir(dirp);
    return EXIT_SUCCESS;
}


```
在这个示例中，程序首先使用 opendir 打开指定目录，然后使用 readdir 逐个读取目录中的条目，并打印每个文件的名称。最后，使用 closedir 关闭目录流

# 加载so

android_dlopen_ext 和 dlsym 是用于动态加载和解析共享库符号的函数。它们在运行时加载共享库并获取库中函数或变量的地址。以下是它们的具体使用方法：

## android_dlopen_ext 函数
android_dlopen_ext 是 Android 特有的扩展函数，用于加载共享库。它允许传递额外的参数来控制加载行为。其函数原型如下：
```
#include <dlext.h>

void* android_dlopen_ext(const char* __filename, int __flags, const android_dlextinfo* __info);

```

-   **参数**：

    -   `__filename`：要加载的共享库的路径。
    -   `__flags`：加载标志，与 `dlopen` 的标志类似，如 `RTLD_NOW`、`RTLD_LAZY` 等。
    -   `__info`：指向 `android_dlextinfo` 结构体的指针，用于传递 Android 特有的加载选项。
-   **返回值**：成功时返回共享库的句柄，失败时返回 `NULL`。
-   
## dlsym 函数
dlsym 用于在已加载的共享库中查找符号（函数或变量）的地址。其函数原型如下：
```
#include <dlfcn.h>

void* dlsym(void* __handle, const char* __symbol);

```

-   **参数**：

    -   `__handle`：由 `dlopen` 或 `android_dlopen_ext` 返回的共享库句柄。
    -   `__symbol`：要查找的符号名称。
-   **返回值**：成功时返回符号的地址，失败时返回 `NULL`。

## 示例代码
以下是一个简单的示例，演示如何使用 android_dlopen_ext 和 dlsym 加载共享库并调用其中的函数：

```
#include <dlfcn.h>
#include <dlext.h>
#include <stdio.h>

int main() {
    const char* lib_path = "/path/to/library.so";
    android_dlextinfo extinfo = {0};
    // 设置额外的加载选项（如果需要）
    extinfo.flags = ANDROID_DLEXT_USE_LIBRARY_FD;

    // 加载共享库
    void* handle = android_dlopen_ext(lib_path, RTLD_NOW, &extinfo);
    if (!handle) {
        fprintf(stderr, "Failed to load library: %s\n", dlerror());
        return -1;
    }

    // 查找符号
    void (*func)() = (void (*)())dlsym(handle, "function_name");
    if (!func) {
        fprintf(stderr, "Failed to find symbol: %s\n", dlerror());
        dlclose(handle);
        return -1;
    }

    // 调用函数
    func();

    // 关闭共享库
    dlclose(handle);
    return 0;
}

```

## extractor加载so
Android/frameworks/av/media/libstagefright/**MediaExtractorFactory.cpp**

```
void MediaExtractorFactory::RegisterExtractors(
        const char *libDirPath, const android_dlextinfo* dlextinfo,
        std::list<sp<ExtractorPlugin>> &pluginList) {
    ALOGV("search for plugins at %s", libDirPath);

    DIR *libDir = opendir(libDirPath);
    if (libDir) {
        struct dirent* libEntry;
        while ((libEntry = readdir(libDir))) {
            if (libEntry->d_name[0] == '.') {
                continue;
            }
            String8 libPath = String8(libDirPath) + "/" + libEntry->d_name;
            if (!libPath.contains("extractor.so")) {
                continue;
            }
            void *libHandle = android_dlopen_ext(
                    libPath.string(),
                    RTLD_NOW | RTLD_LOCAL, dlextinfo);
            if (libHandle == nullptr) {
                ALOGI("dlopen(%s) reported error %s", libPath.string(), strerror(errno));
                continue;
            }

            GetExtractorDef getDef =
                (GetExtractorDef) dlsym(libHandle, "GETEXTRACTORDEF");
            if (getDef == nullptr) {
                ALOGI("no sniffer found in %s", libPath.string());
                dlclose(libHandle);
                continue;
            }

            ALOGV("registering sniffer for %s", libPath.string());
            RegisterExtractor(
                    new ExtractorPlugin(getDef(), libHandle, libPath), pluginList);
        }
        closedir(libDir);
    } else {
        ALOGI("plugin directory not present (%s)", libDirPath);
    }
}
```
