
##  一、先看你原来能成功的情况

> “我之前用一个本地反代的软件，手动设置 Git 代理地址后，能稳定提交”

说明：

- 你的 Git 客户端是**明确知道代理地址和协议的**（例如 `http://127.0.0.1:10809`）。
    
- 那个“反代软件”可能是基于 **HTTP/SOCKS5** 的通用代理。
    
- 它能正确把 Git 的 HTTPS 流量（`git push`, `git pull`）转发到外网。
    

 所以连接非常稳定。

---

##  二、后来换了新的代理软件后为什么就不行了？

> “翻墙没问题（能上外网），但 git 提交却连接失败”

这是一个非常常见的现象，根本原因是：

> **新的代理软件工作方式变了！**

---

###  代理方式对比

|模式|对浏览器有效|对命令行程序有效|特点|
|---|---|---|---|
|**系统代理 / PAC 模式**|✅（浏览器会自动使用）|❌（Git、Curl 默认不会）|常见于 GUI 翻墙软件，如 Clash Verge、Netch 等|
|**TUN 模式（虚拟网卡）**|✅✅（所有流量走）|✅（理论上 Git 也能）|但部分软件对 localhost 连接有例外|
|**HTTP/SOCKS5 显式代理**|✅（浏览器可手动设）|✅（Git 可配置 `git config http.proxy`）|最兼容，但要你手动设置|
|**透明代理 / 分流**|✅（自动识别外网）|❌（Git 不一定被接管）|一般由 Clash、sing-box 在路由层处理|

---

###  换句话说：

你的浏览器“能翻墙”，  
不代表你的 Git 命令行也“知道”该走代理。

因为命令行工具不会自动读取 GUI 软件的代理规则。

---

##  三、Git 实际是怎么联网的

Git 在执行 `git push` 或 `git pull` 时：

- 通过 HTTPS（443 端口）连接到远端（如 `github.com`）；
    
- 它不会自动读取系统代理；
    
- 需要你在配置里明确告诉它代理：
    

```bash
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890
```

（如果你的代理软件是 socks5，则改成：）

```bash
git config --global http.proxy socks5://127.0.0.1:1080
git config --global https.proxy socks5://127.0.0.1:1080
```

---

##  四、为什么以前的代理能行，现在的不行？

可能是这几种情况之一：

|情况|原因|
|---|---|
|新代理只提供“系统代理”，而非真正监听端口|Git 没有走代理|
|新代理用了 “TUN 模式”，Git 走 localhost 时被绕过|Clash/sing-box 的分流规则里通常跳过 127.0.0.1|
|新代理的监听端口或协议不同|旧的是 `http://127.0.0.1:10809`，新的是 `socks5://127.0.0.1:7890`|
|代理软件未启用允许“局域网连接”|Git 连接失败（连接被拒绝）|

---

##  五、你可以这样排查（非常有效）

1. **查看你的代理软件设置**
    
    - 找到“本地代理端口”（例如：HTTP 7890、SOCKS5 1080）。
        
    - 记下协议类型。
        
2. **命令行测试**
    
    ```bash
    curl -x http://127.0.0.1:7890 https://github.com
    ```
    
    如果成功返回 GitHub 页面 → 说明代理通了。  
    否则 → 代理地址不对。
    
3. **在 Git 中重新设置代理**
    
    ```bash
    git config --global --unset http.proxy
    git config --global --unset https.proxy
    git config --global http.proxy http://127.0.0.1:7890
    git config --global https.proxy http://127.0.0.1:7890
    ```
    
4. **再测试推送**
    
    ```bash
    git push
    ```
    
5. 若仍不行 → 试试用 **socks5** 协议设置：
    
    ```bash
    git config --global http.proxy socks5://127.0.0.1:1080
    git config --global https.proxy socks5://127.0.0.1:1080
    ```
    

---

##  六、一个额外陷阱：TUN 模式绕过 localhost

有些代理软件（例如 **Clash Verge / Meta / sing-box TUN 模式**）为了防止死循环，会**自动跳过本机 127.0.0.1** 流量。

Git 就在本机 → 它访问 `127.0.0.1` 的代理时被忽略。  
解决方式：

- 打开软件的 “允许局域网连接（Allow LAN）” 选项；
    
- 或在配置文件里取消对 localhost 的绕过；
    
- 或改走 HTTP 代理端口，而不是 SOCKS。
    

---

##  总结

|现象|根本原因|解决方式|
|---|---|---|
|浏览器能上 GitHub，但 Git push 不行|Git 没走代理|配置 `git config --global http.proxy ...`|
|代理软件换了后出问题|新软件不再暴露 HTTP/SOCKS5 端口|查端口号、协议类型|
|TUN 模式下 Git 不通|TUN 绕过 localhost|开启“允许局域网连接”或切换成显式代理|
