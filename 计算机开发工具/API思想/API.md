### **API（Application Programming Interface）**

- **本质**：API 是一种 **规范** 或 **接口定义**，它描述了软件组件之间如何交互。它就像是一份 **说明书** 或 **协议**，告诉开发者如何使用某个功能或服务。
    
- 存在形式：**API 本身并不是一个实际的代码文件或实体，而是一组规则、方法和数据结构的定义** 。它通常以文档的形式存在，比如 Twitter API 文档 或 Google Maps API 文档
    
- **作用**：API 允许开发者通过特定的接口与某个系统或服务进行交互，而不需要了解其内部实现细节。例如，Web API 通过 HTTP 请求和响应的方式，让开发者能够访问远程服务器上的数据和服务。

| 特点       | API                         | 库                       |
| -------- | --------------------------- | ----------------------- |
| **本质**   | 规范、接口定义                     | 实际存在的代码集合               |
| **存在形式** | 文档、规范、接口定义                  | 文件、模块、代码集合              |
| **作用**   | 定义如何与系统或服务交互                | 提供具体实现，供开发者调用           |
| **示例**   | Twitter API、Google Maps API | `requests`、`numpy`、`os` |

### 举个例子：
假设你正在开发一个应用程序，需要获取用户的地理位置信息。你可以选择以下两种方式：

1. **使用 Web API**：
    
    - 你找到一个提供地理位置信息的 Web API，比如 [IPinfo API](https://ipinfo.io/developers)。
        
    - API 文档告诉你如何通过 HTTP 请求获取地理位置信息。例如，你可以发送一个请求到 `https://ipinfo.io/json`，API 会返回用户的地理位置信息。
        
    - 在这里，API 是一个规范，告诉你如何与远程服务交互。
        
2. **使用库**：
    
    - 你安装了一个 Python 库，比如 `requests`，来发送 HTTP 请求。
        
    - `requests` 库是一个实际存在的代码集合，它封装了 HTTP 请求的细节，让你可以通过简单的代码调用 API：
        
        Python复制
        
        ```python
        import requests
        response = requests.get("https://ipinfo.io/json")
        location = response.json()
        print(location)
        ```
        
    - 在这里，`requests` 库是一个实际的代码文件，它帮助你实现与 API 的交互。


函数长什么样（API 签名、枚举、结构体）。 调用顺序和出错行为（valid usage）。 GPU 和驱动必须支持哪些能力（feature、limit）。着色器用 SPIR-V 格式，中间语言如何组织。

**谁来实现这份规范？**  
GPU 厂商（NVIDIA、AMD、Intel、Arm、Qualcomm…）各自在自己的驱动里写一套“Vulkan 驱动”，只要它符合规范并通过 Khronos 的官方测试，就能宣称“支持 Vulkan 1.x”。因此：

- 你电脑上真正跑的是 `vulkan-1.dll` / `libvulkan.so` / `libvulkan.dylib`，它们由显卡驱动提供。
    
- 你写的程序只是按规范调用这些函数，不必关心驱动内部怎么实现。
    

一句话总结：  
**Vulkan = 一份公开的技术说明书**；  
**驱动 = 各厂商按说明书做的“产品”**；  
**你的代码 = 按说明书去调用产品的用户程序**。


**不同语言在API时的语法必然会出现差异**，但差异的“层”要分清：

1. **API 规范本身没变**  
    Khronos 发布的只是 C 头文件（`.h`），里面全是 C 函数指针、C 结构体、C 枚举。规范层面没有任何语法，只描述“应该有哪些符号、参数、返回值”。
    
2. **语言绑定/封装层把 C 语法“翻译”成目标语言的语法**  
    • C 里：`VkInstance instance; VkInstanceCreateInfo ci = {…}; vkCreateInstance(&ci, NULL, &instance);`  
    • Rust（ash）：`let instance = entry.create_instance(&create_info, None)?;`  
    • C#（Silk.NET）：`var instance = vk.CreateInstance(createInfo, null);`  
    • Python（vulkan）：`instance = vkCreateInstance(createInfo, None)`
    
    这些函数名、参数顺序、对象生命周期、错误处理方式都跟 C 的写法不同，看上去就是“换了语法”。
    
3. **差异来源**
    
    - 语言本身没有 C 的裸指针、预处理宏、位域语法，于是要用类、接口、属性、装饰器来替代。
        
    - 命名规范：C 的 `VK_STRUCTURE_TYPE_INSTANCE_CREATE_INFO` 在 Rust 里通常变成 `vk::StructureType::INSTANCE_CREATE_INFO`。
        
    - 内存管理：C 要手动 `vkDestroyInstance`，Rust/ C# 用 RAII 或 `Drop`/`Dispose`，Python 用 `__del__` 或上下文管理器。
        
    - 错误码：C 用 `VkResult` 宏判断，Rust 用 `Result<T, vk::Result>`，C# 可能抛异常。