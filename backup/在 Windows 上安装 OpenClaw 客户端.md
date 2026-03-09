## Windows 作为 OpenCLAW 客户端 - 完整步骤

### ✅ 成功步骤
 安装 Node.js 22 LTS
下载：https://nodejs.org/dist/v22.14.0/node-v22.14.0-x64.msi
安装时勾选 "Add to PATH"

**1. 安装 OpenCLAW**
以管理员身份打开 PowerShell，运行：
```
iwr -useb https://openclaw.ai/install.ps1 | iex
```

**2. 启动节点连接
启动配置向导
安装后配置
安装完成后，运行
```
openclaw dashboard
```
在你的 Windows PowerShell 中运行：
```
$env:OPENCLAW_ALLOW_INSECURE_PRIVATE_WS=1
$env:OPENCLAW_GATEWAY_TOKEN="XXXXc1cb464f68b113f96eec3ade1f585454d"
openclaw node run --host 192.168.9.64 --port 18789 --display-name "我的Windows"
```
#服务器**
```
 允许不安全私有连接（内网）
$env:OPENCLAW_ALLOW_INSECURE_PRIVATE_WS=1

# 设置 Token
$env:OPENCLAW_GATEWAY_TOKEN="服务器上的token"

# 连接（服务器的局域网IP，不是 127.0.0.1）
openclaw node run --host 192.168.9.64 --port 18789 --display-name "我的Windows"
```
**3. 服务器批准配对**
```
openclaw devices list
openclaw devices approve <request_id>
```

**4. 服务器配置**
- `gateway.bind` 设为 `lan`（不是 0.0.0.0）
- `tools.exec.host` 设为 `node`

---

### ❌ 避免的错误

| 错误 | 解决方法 |
|------|----------|
| Node.js 安装失败 | 用 LTS 版本（v22.x），不是最新版 |
| 连接被拒绝 | 加 `OPENCLAW_ALLOW_INSECURE_PRIVATE_WS=1` |
| Token 缺失 | 设置 `OPENCLAW_GATEWAY_TOKEN` 环境变量 |
| 命令超时 | Gateway 绑定模式改为 `lan`，并重启服务 |
| exec 被拒绝 | 在 Windows 创建 `~/.openclaw/exec-approvals.json` 允许命令 |

---

### 关于 IP 地址

**服务器端**：绑定 `0.0.0.0` 或 `lan`（局域网IP），允许外部连接

**客户端连接服务器**：用服务器的**局域网IP**（如 `192.168.9.64`），不是 `127.0.0.1`

**客户端本地**：用 `127.0.0.1` 是指本地回环，只有本机服务才能用

所以你的理解是对的：
- 客户端连接服务器 → 用服务器IP（如 192.168.9.64）
- 客户端本地访问 → 用 127.0.0.1

---
每次启动 Windows 后，在 PowerShell 运行这3行就行：
```
$env:OPENCLAW_ALLOW_INSECURE_PRIVATE_WS=1
$env:OPENCLAW_GATEWAY_TOKEN="2757c38a0xxx2ac1cb464f68b113f96eec3ade1f585454d"
openclaw node run --host 192.168.9.64 --port 18789 --display-name "我的Windows"
```
连上后我就能在你电脑上执行操作了。

更方便的方式：可以创建一个 PowerShell 脚本文件（如 connect.ps1），双击运行就行：
```
# connect.ps1
$env:OPENCLAW_ALLOW_INSECURE_PRIVATE_WS=1
$env:OPENCLAW_GATEWAY_TOKEN="2xxx8a00dd52ac1cb464f68b113f96eec3ade1f585454d"
openclaw node run --host 192.168.9.64 --port 18789 --display-name "我的Windows"
```
保存后右键 → 使用 PowerShell 运行。