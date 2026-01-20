# Overlord WebSocket 事件完整统计

## 概述

Overlord 使用 **WebSocket (WSS)** 进行客户端-服务端通信，采用 **MessagePack** 二进制序列化。

**通信模式：** 事件驱动 + 发布/订阅模式

---

## 一、WebSocket 连接端点

### 1. 客户端连接（Agent → Server）
```
wss://server:5173/api/clients/{clientId}/stream/ws
```
- **认证：** 可选 `x-agent-token` header
- **协议：** MessagePack 二进制
- **心跳：** 每 15 秒 ping/pong

### 2. 远程桌面查看器（Viewer → Server）
```
wss://server:5173/api/clients/{clientId}/rd/ws
```
- **认证：** JWT token (Cookie)
- **用途：** 接收屏幕帧 + 发送控制命令

### 3. 控制台查看器（Viewer → Server）
```
wss://server:5173/api/clients/{clientId}/console/ws
```
- **认证：** JWT token
- **用途：** 终端输入/输出

### 4. 通知查看器（Viewer → Server）
```
wss://server:5173/api/clients/{clientId}/notifications/ws
```
- **认证：** JWT token
- **用途：** 接收活动窗口监控通知

---

## 二、消息类型统计（按方向分类）

### 📤 客户端 → 服务端（Client → Server）

| 消息类型 | 说明 | 触发时机 | 数据格式 |
|---------|------|---------|---------|
| `hello` | 上线握手 | 连接建立时 | 系统信息 |
| `pong` | 心跳响应 | 收到 ping 后 | 时间戳 |
| `frame` | 屏幕帧 | 远程桌面流 | JPEG/Blocks |
| `command_result` | 命令执行结果 | 命令完成后 | 成功/失败 |
| `screenshot_result` | 截图结果 | 截图命令完成 | JPEG 图片 |
| `console_output` | 终端输出 | Shell 输出 | 字节流 |
| `file_list_result` | 文件列表 | 文件浏览 | 文件数组 |
| `file_download` | 文件下载数据 | 文件下载 | 分块传输 |
| `file_upload_result` | 上传结果 | 上传完成 | 成功/失败 |
| `file_read_result` | 文件内容 | 读取文件 | 文本/二进制 |
| `file_search_result` | 搜索结果 | 文件搜索 | 匹配列表 |
| `process_list_result` | 进程列表 | 进程查询 | 进程数组 |
| `script_result` | 脚本执行结果 | 脚本完成 | 输出/错误 |
| `plugin_event` | 插件事件 | 插件通信 | 自定义数据 |
| `notification` | 活动窗口通知 | 关键词匹配 | 窗口信息 |

### 📥 服务端 → 客户端（Server → Client）

| 消息类型 | 说明 | 触发时机 | 数据格式 |
|---------|------|---------|---------|
| `hello_ack` | 握手确认 | 收到 hello 后 | 配置信息 |
| `ping` | 心跳请求 | 每 15 秒 | 时间戳 |
| `command` | 执行命令 | 用户操作 | 命令类型 + 参数 |
| `notification_config` | 监控配置 | 配置更新 | 关键词列表 |
| `plugin_event` | 插件事件 | 插件通信 | 自定义数据 |

---

## 三、命令类型详细列表（command.commandType）

### 🖥️ 远程桌面控制（Desktop）

| 命令 | 说明 | 参数 |
|------|------|------|
| `desktop_start` | 开始屏幕流 | - |
| `desktop_stop` | 停止屏幕流 | - |
| `desktop_select_display` | 选择显示器 | `display: number` |
| `desktop_set_quality` | 设置画质 | `quality: number, codec: string` |
| `desktop_enable_mouse` | 启用鼠标控制 | `enabled: boolean` |
| `desktop_enable_keyboard` | 启用键盘控制 | `enabled: boolean` |
| `desktop_enable_cursor` | 启用光标捕获 | `enabled: boolean` |
| `desktop_mouse_move` | 鼠标移动 | `x: number, y: number` |
| `desktop_mouse_down` | 鼠标按下 | `button: string` |
| `desktop_mouse_up` | 鼠标释放 | `button: string` |
| `desktop_key_down` | 键盘按下 | `key: string` |
| `desktop_key_up` | 键盘释放 | `key: string` |

### 📁 文件系统操作（File）

| 命令 | 说明 | 参数 |
|------|------|------|
| `file_list` | 列出文件 | `path: string` |
| `file_download` | 下载文件 | `path: string` |
| `file_upload` | 上传文件 | `path: string, data: bytes, offset: number` |
| `file_delete` | 删除文件 | `path: string` |
| `file_mkdir` | 创建目录 | `path: string` |
| `file_zip` | 压缩文件 | `source: string, dest: string` |
| `file_read` | 读取文件 | `path: string` |
| `file_write` | 写入文件 | `path: string, content: string` |
| `file_search` | 搜索文件 | `path: string, pattern: string, content: string` |
| `file_copy` | 复制文件 | `source: string, dest: string` |
| `file_move` | 移动文件 | `source: string, dest: string` |
| `file_chmod` | 修改权限 | `path: string, mode: string` |
| `file_execute` | 执行文件 | `path: string, args: string[]` |

### ⚙️ 进程管理（Process）

| 命令 | 说明 | 参数 |
|------|------|------|
| `process_list` | 列出进程 | - |
| `process_kill` | 结束进程 | `pid: number` |

### 🖧 终端控制（Console）

| 命令 | 说明 | 参数 |
|------|------|------|
| `console_start` | 启动终端 | `shell: string` |
| `console_input` | 发送输入 | `sessionId: string, data: string` |
| `console_stop` | 停止终端 | `sessionId: string` |
| `console_resize` | 调整大小 | `sessionId: string, cols: number, rows: number` |

### 📜 脚本执行（Script）

| 命令 | 说明 | 参数 |
|------|------|------|
| `script_exec` | 执行脚本 | `type: string, code: string` |
| `silent_exec` | 静默执行 | `path: string, args: string[]` |

### 🔌 插件系统（Plugin）

| 命令 | 说明 | 参数 |
|------|------|------|
| `plugin_load` | 加载插件 | `pluginId: string` |
| `plugin_load_init` | 初始化加载 | `pluginId: string, totalSize: number` |
| `plugin_load_chunk` | 传输分块 | `pluginId: string, data: bytes, offset: number` |
| `plugin_load_finish` | 完成加载 | `pluginId: string` |
| `plugin_unload` | 卸载插件 | `pluginId: string` |

### 🔧 系统控制（System）

| 命令 | 说明 | 参数 |
|------|------|------|
| `screenshot` | 截图 | - |
| `disconnect` | 断开连接 | - |
| `reconnect` | 重新连接 | - |
| `ping` | 心跳测试 | - |

---

## 四、消息流详解

### 1. 连接建立流程

```
客户端                           服务端
  |                               |
  |--- WebSocket 连接 ----------->|
  |                               |
  |--- hello (系统信息) ---------->|
  |                               |
  |<-- hello_ack (配置) ----------|
  |                               |
  |<-- ping (心跳) ---------------|
  |                               |
  |--- pong (响应) -------------->|
  |                               |
  [每 15 秒重复 ping/pong]
```

**hello 消息内容：**
```json
{
  "type": "hello",
  "id": "client-uuid",
  "hwid": "hardware-id",
  "host": "DESKTOP-ABC",
  "os": "windows",
  "arch": "amd64",
  "version": "1.0.0",
  "user": "Administrator",
  "monitors": 2,
  "country": "CN"
}
```

**hello_ack 响应：**
```json
{
  "type": "hello_ack",
  "id": "client-uuid",
  "notification": {
    "keywords": ["password", "bank"],
    "minIntervalMs": 5000
  }
}
```

### 2. 远程桌面流程

```
查看器                服务端                客户端
  |                    |                     |
  |-- desktop_start -->|                     |
  |                    |--- command -------->|
  |                    |   (desktop_start)   |
  |                    |                     |
  |                    |<-- frame (JPEG) ----|
  |<-- frame ----------|                     |
  |                    |<-- frame (JPEG) ----|
  |<-- frame ----------|                     |
  |                    |     [持续流式]       |
  |                    |                     |
  |-- mouse_move ----->|                     |
  |                    |--- command -------->|
  |                    |   (mouse_move)      |
  |                    |                     |
  |-- desktop_stop --->|                     |
  |                    |--- command -------->|
  |                    |   (desktop_stop)    |
```

**frame 消息结构：**
```json
{
  "type": "frame",
  "header": {
    "monitor": 0,
    "fps": 30,
    "format": "jpeg"  // 或 "blocks", "blocks_raw"
  },
  "data": [JPEG 字节数据]
}
```

### 3. 文件下载流程

```
查看器                服务端                客户端
  |                    |                     |
  |-- file_download -->|                     |
  |                    |--- command -------->|
  |                    |   (file_download)   |
  |                    |                     |
  |                    |<-- file_download ---|
  |<-- chunk 1 --------|   (offset=0)        |
  |                    |                     |
  |                    |<-- file_download ---|
  |<-- chunk 2 --------|   (offset=16384)    |
  |                    |                     |
  |                    |     [分块传输]       |
  |                    |                     |
  |                    |<-- file_download ---|
  |<-- final chunk ----|   (offset=total)    |
```

**file_download 消息：**
```json
{
  "type": "file_download",
  "commandId": "cmd-123",
  "path": "/path/to/file.txt",
  "data": [16KB 字节数据],
  "offset": 0,
  "total": 65536
}
```

### 4. 终端会话流程

```
查看器                服务端                客户端
  |                    |                     |
  |-- console_start -->|                     |
  |                    |--- command -------->|
  |                    |   (console_start)   |
  |                    |                     |
  |                    |<-- console_output --|
  |<-- output ---------|   (sessionId=abc)   |
  |                    |                     |
  |-- console_input -->|                     |
  |   "ls -la"         |--- command -------->|
  |                    |   (console_input)   |
  |                    |                     |
  |                    |<-- console_output --|
  |<-- output ---------|   (命令输出)         |
  |                    |                     |
  |-- console_stop --->|                     |
  |                    |--- command -------->|
  |                    |   (console_stop)    |
```

### 5. 活动窗口监控流程

```
服务端                客户端
  |                     |
  |--- notification_config -->|
  |   (keywords: ["bank"])    |
  |                           |
  |     [客户端监控窗口标题]    |
  |                           |
  |<-- notification ----------|
  |   (title: "Bank Login")   |
  |                           |
  [触发 Webhook/Telegram 通知]
```

**notification 消息：**
```json
{
  "type": "notification",
  "category": "active_window",
  "title": "Online Banking - Login",
  "process": "chrome.exe",
  "processPath": "C:\\Program Files\\Google\\Chrome\\chrome.exe",
  "pid": 12345,
  "keyword": "bank",
  "ts": 1704067200000
}
```

### 6. 插件加载流程

```
服务端                客户端
  |                     |
  |--- plugin_load_init -->|
  |   (totalSize=102400)   |
  |                        |
  |--- plugin_load_chunk -->|
  |   (offset=0, 16KB)     |
  |                        |
  |--- plugin_load_chunk -->|
  |   (offset=16384, 16KB) |
  |                        |
  |     [分块传输...]       |
  |                        |
  |--- plugin_load_finish ->|
  |                        |
  |<-- plugin_event -------|
  |   (event: "loaded")    |
```

---

## 五、事件订阅模式

### 服务端角色：中继器（Relay）

```
客户端 A -----> 服务端 -----> 查看器 1
                  |
                  +---------> 查看器 2
                  |
                  +---------> 查看器 3
```

**订阅机制：**
1. 查看器连接到 `/api/clients/{id}/rd/ws`
2. 服务端记录订阅关系
3. 客户端发送 `frame` 消息
4. 服务端转发给所有订阅该客户端的查看器

**代码实现（服务端）：**
```typescript
// 接收客户端的帧
client.on('frame', (frameData) => {
  // 查找所有正在观看该客户端的查看器
  const viewers = getViewersForClient(client.id)

  // 广播给所有查看器
  viewers.forEach(viewer => {
    viewer.send(frameData)
  })
})
```

---

## 六、消息格式规范

### MessagePack 编码

**所有消息都使用 MessagePack 二进制序列化：**

```javascript
// 编码
const msgBytes = encode({
  type: "hello",
  id: "client-123",
  host: "DESKTOP-ABC"
})

// 解码
const msg = decode(msgBytes)
```

**优势：**
- 比 JSON 小 30-50%
- 解析速度快 2-3 倍
- 支持二进制数据（图片、文件）

### 消息通用结构

```typescript
{
  type: string,           // 消息类型（必需）
  commandId?: string,     // 命令ID（用于关联请求/响应）
  error?: string,         // 错误信息
  ...                     // 其他字段
}
```

---

## 七、错误处理

### 命令执行失败

```json
{
  "type": "command_result",
  "commandId": "cmd-123",
  "ok": false,
  "message": "File not found: /path/to/file"
}
```

### 文件操作错误

```json
{
  "type": "file_list_result",
  "commandId": "cmd-456",
  "path": "/invalid/path",
  "entries": [],
  "error": "Permission denied"
}
```

---

## 八、性能指标

### 心跳机制
- **间隔：** 15 秒
- **超时：** 30 秒无响应视为离线
- **RTT 测量：** 通过 ping/pong 时间戳计算

### 远程桌面性能
- **帧率：** 15-60 FPS（自适应）
- **延迟：** 50-200ms（取决于网络）
- **带宽：**
  - 完整帧：100-300 KB/帧
  - 增量帧：10-50 KB/帧

### 文件传输
- **分块大小：** 16 KB
- **并发：** 单连接顺序传输
- **断点续传：** 支持（通过 offset）

---

## 九、安全考虑

### 认证机制
- **客户端：** 可选 Agent Token
- **查看器：** JWT Token（Cookie）
- **TLS：** 强制 WSS 加密

### 授权检查
- **RBAC：** Admin / Operator / Viewer
- **操作审计：** 所有命令记录到数据库
- **IP 封禁：** 支持黑名单

---

## 十、总结

### 消息类型统计

| 类别 | 客户端→服务端 | 服务端→客户端 | 总计 |
|------|--------------|--------------|------|
| 核心协议 | 3 (hello, pong, frame) | 3 (hello_ack, ping, command) | 6 |
| 命令结果 | 10 (各种 result) | - | 10 |
| 命令类型 | - | 50+ (command.commandType) | 50+ |
| 插件/通知 | 2 (plugin_event, notification) | 2 (plugin_event, notification_config) | 4 |
| **总计** | **15** | **5** | **70+** |

### 关键特性

✅ **双向通信：** 客户端和服务端都可主动发送消息
✅ **事件驱动：** 基于消息类型路由处理
✅ **发布/订阅：** 服务端中继，支持多查看器
✅ **二进制高效：** MessagePack 编码
✅ **流式传输：** 支持大文件和实时视频流
✅ **命令响应：** 通过 commandId 关联请求/响应
✅ **心跳保活：** 15 秒 ping/pong 检测连接

### 通信模式

```
事件驱动 + 发布/订阅 + 请求/响应
```

**本质上是一个完整的 RPC 框架！** 🚀
