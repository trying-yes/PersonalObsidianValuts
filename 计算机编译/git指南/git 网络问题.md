
**Git 不信任 Windows 系统添加的证书颁发机构 (CA)**

1. **Git 使用的加密库：**
    
    - Git for Windows **自带**了 OpenSSL 库来处理 HTTPS 连接（如 `git clone https://...`, `git push` 等）。
        
    - Windows 操作系统本身有自己内置的加密通信组件叫 **SChannel**。
        
2. **证书信任的差异：**
    
    - **SChannel：** 信任的是 **Windows 系统的证书存储**。当你通过“管理用户证书”或“管理计算机证书”等系统工具安装一个新的根证书或中间 CA 证书（比如公司内部 CA 的证书），SChannel 就知道并信任这些证书。
        
    - **OpenSSL (Git)：** 信任的是它 **自己捆绑的证书包**（通常是一个文件，如 `ca-bundle.crt`）。它 **不看** Windows 系统的证书存储。
        
3. **问题的根源 (WT 添加的 CA 不被承认)：**
    
    - 文中提到的 “WT” 可能是指 **Windows Trust** 机制或者泛指在 **Windows 系统里**添加受信任 CA 的操作。
        
    - 当你把一个新的 CA 证书（比如你公司的自签名 CA 或防火墙的 CA）添加到 **Windows 系统证书存储**（让浏览器、Edge 等信任）时：
        
        - **SChannel 知道了也信任了**，所以用 IE/Edge 访问公司内部 HTTPS 网站没问题。
            
        - **但是 Git 不知道！** Git 的 OpenSSL 只认它自己带的那个 `ca-bundle.crt` 文件里的 CA。你新加进去的 CA 不在这个文件里。
            
4. **后果 (SSL 握手异常)：**
    
    - 当你用 Git 操作一个 HTTPS 仓库（比如 `git clone https://your.company.git`），而这个仓库的服务器证书是由你刚刚添加到 _Windows 系统_ 里的那个 CA 签发的：
        
        - Git 的 OpenSSL 会检查服务器证书。
            
        - 它发现签发证书的 CA **不在**它自带的信任列表 (`ca-bundle.crt`) 里。
            
        - OpenSSL 认为这个连接**不安全**，因为它无法验证服务器身份。
            
        - 于是 Git 报错：通常是 **`SSL certificate problem: unable to get local issuer certificate`** 或类似的 **SSL 握手失败/证书验证失败**的错误。
            
5. **解决方案（两个选择）：**
    
    - **选项一：把证书弄进去 (推荐但稍麻烦)：**
        
        - 找到你添加到 Windows 系统里的那个 **CA 证书文件**（通常是 `.crt` 或 `.pem` 格式）。
            
        - 把这个 CA 证书 **添加** 到 Git 信任的那个 `ca-bundle.crt` 文件里。
            
        - 或者，配置 Git 使用一个**包含了你额外 CA 证书**的自定义证书包文件（通过 `git config --global http.sslCAInfo /path/to/your/custom-ca-bundle.crt`）。
            
        - **效果：** Git 的 OpenSSL 现在信任这个 CA 了，能成功验证服务器证书，HTTPS 操作正常。
            
    - **选项二：关掉 SSL 验证 (简单但非常不安全！)：**
        
        - 运行命令：`git config --global http.sslVerify false`
            
        - **效果：** Git 会**完全跳过**所有的 SSL/TLS 证书验证。
            
        - **严重警告：** 这就像蒙着眼睛过马路！虽然能暂时绕过错误让你操作成功，但**极其危险**。你无法确认连接的另一端是不是真正的 Git 服务器，中间人攻击可以轻易窃取你的密码或代码。**强烈不推荐！** 只在绝对安全可控的内部测试环境，且明确知道风险时才临时使用。
            

**简单总结：**

> 因为 Git for Windows 用自己的小本本 (`ca-bundle.crt`) 记信任的 CA，不看 Windows 的大账本（系统证书存储）。你在 Windows 账本里新加了一个 CA（比如公司的），Git 的小本本上没有，所以当它连接用这个 CA 签名的服务器时，就不认账，报 SSL 错误。要解决，要么把你公司的 CA 抄一份到 Git 的小本本里（安全），要么让 Git 别检查证书了（危险）。

**建议：** 优先尝试 **选项一（添加证书到 Git 的信任链）**，这是安全且根本的解决方法。只有在万不得已且完全了解风险时才考虑临时关闭验证（选项二）。