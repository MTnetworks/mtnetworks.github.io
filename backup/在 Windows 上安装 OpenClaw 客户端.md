快速安装（PowerShell）
以管理员身份打开 PowerShell，运行：
```
iwr -useb https://openclaw.ai/install.ps1 | iex
```
这会：

检查并安装 Node.js 22+
安装 OpenClaw
启动配置向导
安装后配置
安装完成后，运行：
```
openclaw dashboard
```
这会打开浏览器界面，你可以：

配置频道（Webhook、WhatsApp、Telegram 等）
配对节点（让服务器和你现在的电脑连接）
在你的 Windows PowerShell 中运行：
```
$env:OPENCLAW_ALLOW_INSECURE_PRIVATE_WS=1
$env:OPENCLAW_GATEWAY_TOKEN="XXXXc1cb464f68b113f96eec3ade1f585454d"
openclaw node run --host 192.168.9.64 --port 18789 --display-name "我的Windows"
```