# Overlord 操作指南

本文档提供 Overlord 系统的部署、启用、禁用和重启操作说明。

---

## 目录

- [部署方式](#部署方式)
  - [Docker 部署（推荐）](#docker-部署推荐)
  - [源码部署](#源码部署)
  - [系统服务部署](#系统服务部署)
- [启用服务](#启用服务)
- [禁用服务](#禁用服务)
- [重启服务](#重启服务)
- [状态检查](#状态检查)
- [日志查看](#日志查看)
- [故障排查](#故障排查)

---

## 部署方式

### Docker 部署（推荐）

#### 快速部署

```bash
# 1. 克隆项目
git clone <repository-url>
cd Overlord

# 2. 启动服务（使用默认配置）
docker compose up -d

# 3. 访问服务
# URL: https://localhost:5173
# 默认账号: admin / admin
# 首次登录会强制修改密码
```

#### 生产环境部署

```bash
# 1. 创建环境变量文件
cat > .env << EOF
# 服务端口
PORT=5173
HOST=0.0.0.0

# 管理员账号（首次登录后强制修改）
OVERLORD_USER=admin
OVERLORD_PASS=YourStrongPassword123!

# JWT 密钥（至少32字符）
JWT_SECRET=$(openssl rand -base64 32)

# 运行环境
NODE_ENV=production

# TLS 证书路径（可选，不设置则自动生成自签名证书）
# OVERLORD_TLS_CERT=/path/to/server.crt
# OVERLORD_TLS_KEY=/path/to/server.key
# OVERLORD_TLS_CA=/path/to/ca.crt
EOF

# 2. 使用自定义配置启动
docker compose up -d

# 3. 查看日志确认启动成功
docker compose logs -f overlord-server
```

#### 使用自定义镜像

```bash
# 修改 docker-compose.yml 中的镜像地址
# image: ${DOCKER_IMAGE:-ghcr.io/pulsarv2/overlord:latest}

# 或通过环境变量指定
DOCKER_IMAGE=your-registry/overlord:v1.0.0 docker compose up -d
```

---

### 源码部署

#### 前置要求

- **服务端**: Bun 1.0+ (https://bun.sh)
- **客户端**: Go 1.21+
- **系统**: Linux/macOS/Windows

#### 服务端部署

```bash
# 1. 进入服务端目录
cd Overlord-Server

# 2. 安装依赖
bun install

# 3. 生成 TLS 证书（可选）
cd ..
./generate-certs.sh  # Linux/macOS
# 或
generate-certs.bat   # Windows

# 4. 配置环境变量
export PORT=5173
export HOST=0.0.0.0
export OVERLORD_USER=admin
export OVERLORD_PASS=admin
export JWT_SECRET=$(openssl rand -base64 32)

# 5. 启动服务

# 开发模式（热重载）
bun run dev

# 生产模式
bun run start

# 或使用启动脚本
cd ..
./start-prod.sh      # Linux/macOS
# 或
start-prod.bat       # Windows
```

#### 客户端部署

```bash
# 1. 进入客户端目录
cd Overlord-Client

# 2. 配置环境变量
export OVERLORD_SERVER=wss://your-server:5173/ws
export OVERLORD_TLS_INSECURE_SKIP_VERIFY=false  # 生产环境设为 false
export OVERLORD_TLS_CA=/path/to/ca.crt          # 可选

# 3. 运行客户端

# 开发模式
go run ./cmd/agent

# 编译后运行
go build -o overlord-agent ./cmd/agent
./overlord-agent

# 或使用启动脚本
cd ..
./start-dev-client.sh  # Linux/macOS
```

---

### 系统服务部署

#### Linux (systemd)

```bash
# 1. 创建服务文件
sudo tee /etc/systemd/system/overlord-server.service > /dev/null << EOF
[Unit]
Description=Overlord Server
After=network.target

[Service]
Type=simple
User=overlord
Group=overlord
WorkingDirectory=/opt/overlord/Overlord-Server
Environment="PORT=5173"
Environment="HOST=0.0.0.0"
Environment="OVERLORD_USER=admin"
Environment="OVERLORD_PASS=admin"
Environment="JWT_SECRET=your-jwt-secret-here"
Environment="NODE_ENV=production"
ExecStart=/usr/local/bin/bun run start
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF

# 2. 创建用户和目录
sudo useradd -r -s /bin/false overlord
sudo mkdir -p /opt/overlord
sudo cp -r Overlord-Server /opt/overlord/
sudo chown -R overlord:overlord /opt/overlord

# 3. 重载 systemd 配置
sudo systemctl daemon-reload

# 4. 启用并启动服务
sudo systemctl enable overlord-server
sudo systemctl start overlord-server

# 5. 查看状态
sudo systemctl status overlord-server
```

#### Windows (NSSM)

```powershell
# 1. 下载 NSSM (Non-Sucking Service Manager)
# https://nssm.cc/download

# 2. 安装服务
nssm install OverlordServer "C:\Program Files\bun\bun.exe"
nssm set OverlordServer AppDirectory "C:\overlord\Overlord-Server"
nssm set OverlordServer AppParameters "run start"
nssm set OverlordServer AppEnvironmentExtra "PORT=5173" "HOST=0.0.0.0" "OVERLORD_USER=admin" "OVERLORD_PASS=admin"

# 3. 启动服务
nssm start OverlordServer

# 4. 查看状态
nssm status OverlordServer
```

---

## 启用服务

### Docker 方式

```bash
# 启动服务（如果已停止）
docker compose start

# 或完整启动（包括创建容器）
docker compose up -d

# 启动特定服务
docker compose start overlord-server

# 查看启动日志
docker compose logs -f overlord-server
```

### systemd 方式

```bash
# 启动服务
sudo systemctl start overlord-server

# 设置开机自启
sudo systemctl enable overlord-server

# 同时启动并设置自启
sudo systemctl enable --now overlord-server

# 查看状态
sudo systemctl status overlord-server
```

### 手动启动

```bash
# 服务端
cd Overlord-Server
bun run start

# 或使用脚本
./start-prod.sh      # Linux/macOS
start-prod.bat       # Windows

# 客户端
cd Overlord-Client
go run ./cmd/agent

# 或使用脚本
./start-dev-client.sh  # Linux/macOS
```

---

## 禁用服务

### Docker 方式

```bash
# 停止服务（保留容器）
docker compose stop

# 停止并删除容器（保留数据卷）
docker compose down

# 停止并删除所有内容（包括数据卷）
docker compose down -v

# 停止特定服务
docker compose stop overlord-server
```

### systemd 方式

```bash
# 停止服务
sudo systemctl stop overlord-server

# 禁用开机自启
sudo systemctl disable overlord-server

# 同时停止并禁用自启
sudo systemctl disable --now overlord-server

# 完全卸载服务
sudo systemctl stop overlord-server
sudo systemctl disable overlord-server
sudo rm /etc/systemd/system/overlord-server.service
sudo systemctl daemon-reload
```

### 手动停止

```bash
# 查找进程
ps aux | grep -E "bun|overlord"

# 停止进程
kill <PID>

# 强制停止
kill -9 <PID>

# 或使用 pkill
pkill -f "bun.*Overlord-Server"
pkill -f "overlord-agent"
```

---

## 重启服务

### Docker 方式

```bash
# 重启服务
docker compose restart

# 重启特定服务
docker compose restart overlord-server

# 重新构建并重启
docker compose up -d --build

# 强制重新创建容器
docker compose up -d --force-recreate
```

### systemd 方式

```bash
# 重启服务
sudo systemctl restart overlord-server

# 重新加载配置（不中断服务）
sudo systemctl reload overlord-server

# 重新加载 systemd 配置文件
sudo systemctl daemon-reload
sudo systemctl restart overlord-server
```

### 手动重启

```bash
# 方法1: 停止后启动
pkill -f "bun.*Overlord-Server"
cd Overlord-Server && bun run start

# 方法2: 使用脚本
./start-prod.sh      # Linux/macOS
start-prod.bat       # Windows
```

---

## 状态检查

### Docker 方式

```bash
# 查看容器状态
docker compose ps

# 查看详细信息
docker compose ps -a

# 查看资源使用
docker stats overlord-server

# 健康检查
docker inspect overlord-server | grep -A 10 Health

# 测试服务可用性
curl -k https://localhost:5173/health
```

### systemd 方式

```bash
# 查看服务状态
sudo systemctl status overlord-server

# 查看详细状态
sudo systemctl status overlord-server -l

# 检查是否启用
sudo systemctl is-enabled overlord-server

# 检查是否运行
sudo systemctl is-active overlord-server
```

### 手动检查

```bash
# 检查进程
ps aux | grep -E "bun|overlord"

# 检查端口
netstat -tlnp | grep 5173
# 或
ss -tlnp | grep 5173
# 或
lsof -i :5173

# 测试连接
curl -k https://localhost:5173/health

# 测试 WebSocket
wscat -c wss://localhost:5173/ws --no-check
```

---

## 日志查看

### Docker 方式

```bash
# 查看实时日志
docker compose logs -f

# 查看特定服务日志
docker compose logs -f overlord-server

# 查看最近 100 行
docker compose logs --tail=100 overlord-server

# 查看带时间戳的日志
docker compose logs -f -t overlord-server

# 导出日志
docker compose logs > overlord.log
```

### systemd 方式

```bash
# 查看实时日志
sudo journalctl -u overlord-server -f

# 查看最近 100 行
sudo journalctl -u overlord-server -n 100

# 查看今天的日志
sudo journalctl -u overlord-server --since today

# 查看特定时间范围
sudo journalctl -u overlord-server --since "2024-01-01" --until "2024-01-02"

# 导出日志
sudo journalctl -u overlord-server > overlord.log
```

### 应用日志

```bash
# 服务端日志位置
# Docker: /app/data/logs/
# 源码: Overlord-Server/data/logs/

# 查看应用日志
tail -f Overlord-Server/data/logs/combined.log
tail -f Overlord-Server/data/logs/error.log

# 查看审计日志
sqlite3 Overlord-Server/data/overlord.db "SELECT * FROM audit_logs ORDER BY timestamp DESC LIMIT 100;"
```

---

## 故障排查

### 常见问题

#### 1. 服务无法启动

```bash
# 检查端口占用
sudo lsof -i :5173
sudo netstat -tlnp | grep 5173

# 检查配置文件
cat Overlord-Server/config.json

# 检查环境变量
env | grep OVERLORD

# 查看详细错误
docker compose logs overlord-server
# 或
sudo journalctl -u overlord-server -n 50
```

#### 2. 客户端无法连接

```bash
# 检查服务端是否运行
curl -k https://server-ip:5173/health

# 检查防火墙
sudo ufw status
sudo firewall-cmd --list-all

# 检查 TLS 证书
openssl s_client -connect server-ip:5173 -showcerts

# 测试 WebSocket 连接
wscat -c wss://server-ip:5173/ws --no-check
```

#### 3. 数据库错误

```bash
# 检查数据库文件
ls -lh Overlord-Server/data/overlord.db

# 检查数据库完整性
sqlite3 Overlord-Server/data/overlord.db "PRAGMA integrity_check;"

# 备份数据库
cp Overlord-Server/data/overlord.db Overlord-Server/data/overlord.db.backup

# 查看数据库表
sqlite3 Overlord-Server/data/overlord.db ".tables"
```

#### 4. 性能问题

```bash
# 查看资源使用
docker stats overlord-server
# 或
top -p $(pgrep -f "bun.*Overlord-Server")

# 查看连接数
netstat -an | grep :5173 | wc -l

# 查看数据库大小
du -sh Overlord-Server/data/overlord.db

# 清理旧日志
find Overlord-Server/data/logs/ -name "*.log" -mtime +30 -delete
```

### 紧急恢复

```bash
# 1. 停止服务
docker compose down
# 或
sudo systemctl stop overlord-server

# 2. 备份数据
cp -r Overlord-Server/data Overlord-Server/data.backup

# 3. 重置配置
rm Overlord-Server/config.json
cp Overlord-Server/config.json.example Overlord-Server/config.json

# 4. 重新生成证书
./generate-certs.sh

# 5. 重启服务
docker compose up -d
# 或
sudo systemctl start overlord-server
```

---

## 维护建议

### 定期备份

```bash
# 创建备份脚本
cat > backup-overlord.sh << 'EOF'
#!/bin/bash
BACKUP_DIR="/backup/overlord/$(date +%Y%m%d_%H%M%S)"
mkdir -p "$BACKUP_DIR"

# 备份数据库
cp Overlord-Server/data/overlord.db "$BACKUP_DIR/"

# 备份配置
cp Overlord-Server/config.json "$BACKUP_DIR/"

# 备份证书
cp -r Overlord-Server/certs "$BACKUP_DIR/"

# 压缩备份
tar -czf "$BACKUP_DIR.tar.gz" -C /backup/overlord "$(basename $BACKUP_DIR)"
rm -rf "$BACKUP_DIR"

echo "Backup completed: $BACKUP_DIR.tar.gz"
EOF

chmod +x backup-overlord.sh

# 添加到 crontab（每天凌晨2点备份）
(crontab -l 2>/dev/null; echo "0 2 * * * /path/to/backup-overlord.sh") | crontab -
```

### 更新服务

```bash
# Docker 方式
docker compose pull
docker compose up -d

# 源码方式
git pull
cd Overlord-Server && bun install
sudo systemctl restart overlord-server
```

### 监控告警

```bash
# 使用 systemd 监控
sudo systemctl status overlord-server

# 配置邮件告警
sudo tee /etc/systemd/system/overlord-server.service.d/notify.conf > /dev/null << EOF
[Service]
OnFailure=status-email@%n.service
EOF

sudo systemctl daemon-reload
```

---

## 安全建议

1. **修改默认密码**: 首次登录后立即修改 admin 密码
2. **使用强 JWT 密钥**: 至少 32 字符的随机字符串
3. **启用 TLS**: 生产环境必须使用有效的 TLS 证书
4. **配置防火墙**: 仅允许必要的 IP 访问
5. **定期更新**: 及时更新到最新版本
6. **审计日志**: 定期检查审计日志
7. **备份数据**: 定期备份数据库和配置文件
8. **限制权限**: 使用最小权限原则配置用户角色

---

## 相关文档

- [README.md](README.md) - 项目介绍
- [DOCKER.md](DOCKER.md) - Docker 部署详细指南
- [PLUGINS.md](PLUGINS.md) - 插件开发文档

---

## 技术支持

如遇问题，请查看：
- GitHub Issues: <repository-url>/issues
- Telegram 群组: <telegram-link>
- 项目文档: <docs-url>
