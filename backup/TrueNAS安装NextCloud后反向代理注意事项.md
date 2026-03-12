truenas上装了nextcloud和NPM反向代理后：现在手机客户端通过公网访问能看到文件列表，点击文件或图片时一直转圈。
下面是既能公网正常、又能本地 IP 直连的最终配置。
**一、先把你nextcloud的config.php 关键配置改成这样**
```
'overwriteprotocol' => 'https',
'overwritehost' => '你的域名:端口',     //反向代理监听端口
'overwrite.cli.url' => 'https://你的域名:端口',      //反向代理监听端口

// 关键：加下面这行，允许本地IP不走HTTPS
'allow_local_remote_servers' => true,
'trusted_domains' => 
array (
  0 => '192.168.x.x',       // 你的NAS本地IP
  1 => '你的域名',
),
'trusted_proxies' => 
array (
  0 => '127.0.0.1',
  1 => '192.168.x.x',       // NAS本机IP
),
'forwarded_for_headers' => 
array (
  0 => 'HTTP_X_FORWARDED_FOR',
),
```
把 192.168.x.x 换成你 TrueNAS 本机 IP。
**二、NPM（Nginx Proxy Manager）里必须做的设置**
进入你的 Nextcloud 代理主机
SSL 选项卡
开启 SSL
开启 Force SSL（强制 HTTPS）
开启 Websockets Support
Advanced → Custom Nginx Configuration 粘贴：
```
location / {
    proxy_pass http://192.168.x.x:端口;   //内网访问端口
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_max_temp_file_size 0;
    client_max_body_size 0;
    proxy_connect_timeout 300s;
    proxy_send_timeout 300s;
    proxy_read_timeout 300s;
    proxy_buffering off;
    proxy_request_buffering off;
}
```
同样替换成你的 Nextcloud 本地 IP 和端口。
**三、这样改完后效果**
公网手机 / 外网：https 域名 → 正常看文件、预览、不转圈
局域网内：
http:// 本地 IP: 端口 → 能正常打开
https:// 域名 → 也能正常打开
两个都能用，互不冲突