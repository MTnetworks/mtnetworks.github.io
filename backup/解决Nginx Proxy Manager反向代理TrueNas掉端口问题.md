TrueNas  SCALE上安装了Nginx Proxy Manager反向代理，在对truenas自身进行反向代理时发现会丢端口情况，由于是家庭宽带公网IP对80和443端口封禁无法使用，改为了其他端口如8443进行https访问，现在的情况是每次输完域名加端口后，如（https://nas.xx.com:8443）会默认去掉8443端口跳转到https://nas.xx.com/ui 这个地址，把8443这个端口丢弃了，如果我强行把8443这个端口加在网址后面如：https://nas.xx.com:8443/ui 这样的又可以访问，这通常是由于后端服务（TrueNAS Web UI）返回的重定向响应中包含了绝对 URL，出现这种情况后我们的解决方案是：
**方案一：在 NPM 中修改 Host 头（推荐，最简单）**
这是最直接且副作用最小的办法。核心原理是让 NPM 在转发请求时，告诉 TrueNAS 后端：“你看到的访问域名和端口是 nas.xx.com:8443”。这样 TrueNAS 生成的跳转地址就会包含正确的端口。

1. **登录 Nginx Proxy Manager 后台**。
2. 找到你为 TrueNAS 配置的 Proxy Host，点击 **Edit**（编辑）。
3. 切换到 **Advanced**（高级）选项卡。
4. 在 **Custom Nginx Configuration**（自定义 Nginx 配置）文本框中，粘贴以下内容：
```
location / {
    proxy_set_header Host $host:8443;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    
    #--- 新增 WebSocket 支持配置 ---
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_pass https://{你的TrueNAS内网IP}:443;
}
```
**请务必将** {你的TrueNAS内网IP} **替换为实际的 IP 地址**（例如 192.168.1.100）。
6. **关键点**：proxy_set_header Host $host:8443; 这一行强制将端口信息传递给后端，解决了端口丢失的根本原因。
7. 点击 **Save** 保存。建议清除浏览器缓存后，直接访问 https://nas.xx.com:8443 测试。