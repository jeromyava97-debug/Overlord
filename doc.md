

---

# Overlord 项目架构文档（中文说明）

## 一、项目概述（Project Overview）

**Overlord** 是一个 **客户端-服务器架构的远程管理系统（Remote Administration System）**，支持多平台：

* Windows
* Linux
* macOS

它通过一个中心服务器，统一管理和控制大量客户端。

---

## 二、技术栈（Technology Stack）

### 服务端

* **语言 / 运行时**：TypeScript + Bun
* **通信**：WebSocket（WSS）
* **数据库**：SQLite
* **架构模式**：集中式 **C2（Command & Control，命令与控制）模型**

### 客户端

* **语言**：Go

### 通信协议

* **传输层**：WebSocket over TLS（WSS）
* **序列化**：

  * 主：MessagePack（二进制）
  * 备用：JSON

---

## 三、项目结构（Project Structure）

### 服务端（Overlord-Server）

* WebSocket 服务
* Web 管理控制台
* 客户端连接管理
* 权限控制（RBAC）
* 数据库存储
* 客户端构建系统（自动编译客户端）

### 客户端（Overlord-Client）

* 后台 Agent（常驻进程）
* 命令执行模块
* 远程桌面模块
* 文件管理
* 进程管理
* 插件系统（WASM）
* 开机自启动 / 持久化机制

---

## 四、客户端 ↔ 服务端 通信机制

### 通信方式

* **WSS（加密 WebSocket）**
* 地址示例：

  ```
  wss://server:5173/api/clients/{id}/stream/ws
  ```

### 认证

* Web 控制台用户：JWT
* 客户端 Agent：可选 Token
* TLS 证书校验：可选（可关闭）

---

## 五、消息类型说明

### 客户端 → 服务端

客户端会向服务器上报或反馈：

* `hello`：上线握手（系统信息）
* `ping / pong`：心跳
* `frame`：屏幕画面数据
* `command_result`：命令执行结果
* `console_output`：终端输出
* `file_list_result`：文件列表
* `process_list_result`：进程列表
* `notification`：活动窗口监控通知
* `plugin_event`：插件事件

---

### 服务端 → 客户端

服务器可以下发：

* `command`：让客户端执行操作
* `desktop_start / stop`：远程桌面控制
* `notification_config`：关键字监控规则
* `plugin_event`：插件通信

---

## 六、命令能力（服务器能让客户端做什么）

### 📁 文件系统

* 浏览、上传、下载、删除
* 搜索文件
* 修改权限
* 执行文件
* 压缩 / 解压

### 🖥 远程桌面

* 实时屏幕画面
* 鼠标 / 键盘控制
* 多显示器支持
* 帧率、画质控制

### ⚙️ 进程管理

* 查看进程
* 杀死进程

### 🖧 终端 / 控制台

* 交互式 Shell
* 支持多会话
* 支持窗口大小调整

### 📜 脚本执行

* PowerShell
* Bash / sh
* CMD
* Python
* 可静默执行（无窗口）

---

## 七、远程桌面数据协议

客户端持续发送二进制帧：

```
FRM + 版本 + 显示器编号 + FPS + 格式 + JPEG数据
```

也就是：
👉 **实时屏幕截图流**

---

## 八、核心功能模块说明

### 1️⃣ 远程桌面

* Windows：BitBlt
* Linux：X11
* macOS：CoreGraphics
* JPEG 压缩
* 多观看者同时观看

---

### 2️⃣ 文件管理

* 完整文件系统控制
* 分块传输（16KB）
* 可执行远程文件

---

### 3️⃣ 进程管理

* 列出系统进程
* 远程结束进程

---

### 4️⃣ 终端控制

* 类似 SSH 的交互 Shell
* 支持多终端

---

### 5️⃣ 活动窗口监控（Windows）

* 监控当前激活窗口标题
* 关键词匹配
* 触发通知
* 可截图
* 支持 Telegram / Discord 通知

---

### 6️⃣ 插件系统（WASM）

* 服务器下发插件
* 客户端加载并运行
* 插件可与 UI 通信
* 沙箱运行（但仍可执行逻辑）

---

### 7️⃣ 持久化机制（开机自启）

* Windows：注册表 Run
* Linux：systemd / autostart
* macOS：LaunchAgent

👉 **保证客户端重启后仍然存在**

---

## 九、服务端架构

### 功能

* 客户端连接管理
* 用户管理
* 权限控制（Admin / Operator / Viewer）
* 操作审计日志
* IP 封禁
* 自动脚本执行
* 客户端构建系统

---

## 十、客户端构建系统（非常关键）

服务器可以：

* **直接编译客户端程序**
* 支持 Windows / Linux / macOS
* 可配置：

  * 服务器地址
  * 是否自启动
  * 是否隐藏窗口
  * 是否混淆代码
  * 是否去除调试信息

👉 **服务端 = 客户端生成器**

---

## 十一、安全相关说明

### 已实现

* TLS
* JWT
* RBAC
* 操作审计
* 登录限速
* IP 封禁

### ⚠️ 风险点

* 默认账号：admin/admin
* 可关闭 TLS 校验
* 插件可执行任意逻辑
* 客户端无代码签名
* 支持静默执行
* 支持持久化
* 客户端拥有完整系统权限

---

# 一句话总结：Overlord 是在“做什么”？

> **Overlord 是一个功能非常完整的「集中式远程控制与管理平台」**
> 技术上属于：**C2 / RAT / RMM 的交叉形态**

---

## 更直白一点说 👇

这个系统可以：

* 在一台服务器上
* **集中管理大量电脑**
* 并且可以在**用户不在场的情况下**：

  * 看屏幕
  * 控鼠标键盘
  * 读写文件
  * 执行命令和脚本
  * 监控窗口内容
  * 长期驻留系统（开机自启）

---

## 它“像什么”？

从**能力层面**看，它非常接近：

* 🛠 **企业远程运维工具**
* 🔐 **红队 / 蓝队安全测试 C2**
* 🧪 **安全研究 / 教学平台**
* ☠️ **高级远控木马（RAT）**

👉 **区别不在技术，而在“使用场景和授权”**

---

## 对应你之前的日志

你之前发的日志正是：

* Windows 客户端
* 屏幕捕获
* 通过 WSS
* 向 Bun WebSocket 服务器
* 推送 JPEG 帧

完全符合 **Overlord Remote Desktop 模块**。

---

# 技术细节深入解析

## 问题1：远程桌面"看屏幕"是如何实现的？

### 答案：是的！通过连续截图 + JPEG压缩 + 流式传输

### 完整流程：

#### 第一步：屏幕捕获（Windows平台）

**使用 Windows GDI API：**

```
1. 获取屏幕设备上下文（DC）
2. 使用 BitBlt 函数复制屏幕像素到内存
3. 将像素数据转换为 RGBA 格式的图像
```

**关键代码位置：** `Overlord-Client/cmd/agent/capture/win_bitblt.go`

**核心函数：**
- `bitBlt()` - Windows GDI 函数，直接从屏幕DC复制像素到内存DC
- `createDIBSection()` - 创建设备无关位图，获取原始像素缓冲区
- `swapRB()` - 交换红蓝通道（BGR → RGB）

**实际操作：**
```go
// 1. 获取屏幕DC
hdcScreen := getDC(0)  // 0 = 整个屏幕

// 2. 创建内存DC和位图
hdcMem := createCompatibleDC(hdcScreen)
hbmp := createDIBSection(hdcMem, ...)

// 3. 使用 BitBlt 复制屏幕像素
bitBlt(hdcMem, 0, 0, width, height, hdcScreen, x, y, SRCCOPY|CAPTUREBLT)

// 4. 直接访问像素缓冲区
buf := unsafe.Slice((*byte)(bits), stride*height)
```

#### 第二步：智能编码（两种模式）

**模式1：完整帧（Full Frame）- JPEG压缩**

```
屏幕截图 → JPEG编码（质量95） → 发送整张图片
```

**触发条件：**
- 首次连接
- 分辨率改变
- 每5秒强制关键帧
- 变化区域超过40%

**模式2：增量帧（Block-based）- 只传输变化区域**

```
1. 将屏幕分成 64x64 像素的块
2. 对比当前帧和上一帧，检测哪些块发生了变化
3. 只对变化的块进行JPEG编码
4. 合并相邻的变化块成矩形区域
5. 只传输这些小区域的JPEG数据
```

**变化检测算法：**
```go
// 采样检测（每3个像素采样一次）
for row := 0; row < h; row += 3 {
    for col := 0; col < w; col += 3 {
        // 计算RGB差值
        dr = |current[r] - previous[r]|
        dg = |current[g] - previous[g]|
        db = |current[b] - previous[b]|

        // 如果任一通道差值 > 3，认为像素变化
        if dr > 3 || dg > 3 || db > 3 {
            changedPixels++
        }
    }
}

// 如果变化像素超过33%，认为该块变化
return changedPixels * 33 > sampledPixels
```

**数据格式（增量帧）：**
```
[宽度:2字节][高度:2字节][区域数量:2字节][保留:2字节]
[区域1: x:2 y:2 w:2 h:2 size:4 jpeg数据]
[区域2: x:2 y:2 w:2 h:2 size:4 jpeg数据]
...
```

#### 第三步：流式传输

**协议：** WebSocket (WSS) + MessagePack

**帧结构：**
```go
Frame {
    Type: "frame"
    Header: {
        Monitor: 0           // 显示器编号
        FPS: 30              // 当前帧率
        Format: "jpeg"       // 或 "blocks" / "blocks_raw"
    }
    Data: [JPEG字节数据]
}
```

**传输流程：**
```
客户端循环：
1. 捕获屏幕 → image.RGBA
2. 编码为JPEG/Blocks
3. 通过WebSocket发送
4. 等待下一帧（根据FPS控制）
```

#### 第四步：服务端转发

**服务端角色：** 中继器（Relay）

```
客户端 → 服务端 → 多个查看器
```

**服务端处理：**
```typescript
// 接收客户端的帧
client.on('frame', (frameData) => {
    // 转发给所有正在观看该客户端的查看器
    viewers.forEach(viewer => {
        viewer.send(frameData)
    })
})
```

### 性能优化技巧

#### 1. 增量传输
- 静止画面：只传输变化区域，带宽节省 **80-95%**
- 示例：桌面静止时，只有鼠标移动，只传输鼠标周围的小块

#### 2. 自适应质量
- 变化大时：降级为完整JPEG（避免大量小块）
- 变化小时：使用增量块传输

#### 3. 帧率控制
```go
// 实时计算FPS
每秒统计发送的帧数 → 动态调整
```

#### 4. 多显示器支持
```go
// 可以选择捕获哪个显示器
mons := monitorList()
mon := mons[display]  // 0, 1, 2...
```

### 实际性能数据（从日志）

```
capture: stream
  display=0
  fps≈30
  format=blocks
  size=15234
  cap=8.5ms      // 屏幕捕获耗时
  enc=12.3ms     // JPEG编码耗时
  send=5.2ms     // 网络发送耗时
  total=26ms     // 总耗时
```

**解读：**
- 每帧总耗时 26ms → 理论最大 38 FPS
- 实际运行 30 FPS
- 增量模式下，平均每帧只有 15KB（完整帧可能 200KB+）

### 与传统远程桌面对比

| 方案 | Overlord | RDP | VNC |
|------|----------|-----|-----|
| 协议 | WebSocket | 专有协议 | RFB |
| 编码 | JPEG + 增量块 | RemoteFX/H.264 | RRE/Hextile |
| 传输 | 二进制流 | 二进制流 | 二进制流 |
| 浏览器支持 | ✅ 原生 | ❌ 需插件 | ❌ 需插件 |

### 总结

**你的理解完全正确！**

```
看屏幕 = 连续截图 + JPEG压缩 + WebSocket传输
```

**核心技术：**
1. ✅ Windows BitBlt API 捕获屏幕像素
2. ✅ 转换为 RGBA 图像
3. ✅ JPEG 压缩（质量可调）
4. ✅ 增量传输优化（只传变化区域）
5. ✅ WebSocket 流式推送
6. ✅ 服务端中继到多个查看器

**本质上就是：**
> **实时屏幕录制 + 流式传输 = 远程桌面**

---
