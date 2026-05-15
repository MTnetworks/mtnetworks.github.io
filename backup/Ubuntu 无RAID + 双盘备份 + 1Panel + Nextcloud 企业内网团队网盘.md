# ======================================================
# Ubuntu 无RAID + 双盘备份 + 1Panel + Nextcloud 企业内网团队网盘
# 最终稳定版（彻底告别软RAID掉盘、重启崩溃、数据丢失）
# ======================================================

# 一、服务器环境说明
1. 系统：Ubuntu Server 24.04（无桌面纯净版）
2. 存储：无RAID，双盘独立使用（sdb=主盘，sdc=自动备份盘）
3. 架构：系统盘sda独立，数据盘sdb/sdc不组RAID
4. 管理面板：1Panel（仅维护、不反向代理、不装OpenResty）
5. 数据库：独立MariaDB容器（禁用SQLite）
6. 网盘：Nextcloud Docker部署 端口8080 无SSL
7. 共享方式：SMB本地目录 + Windows WebDAV挂载
8. 数据安全：主盘disk1 → 每天自动备份到disk2
9. 使用人数：10人以内团队

# 二、整体部署完整流程
1. 安装Ubuntu Server纯净系统
2. 双盘独立挂载（sdb→disk1主盘，sdc→disk2备份盘）
3. 系统初始化、静态IP、禁止本机Nginx/Apache
4. 安装1Panel（仅维护）
5. Docker部署MariaDB独立数据库
6. Docker部署Nextcloud 8080端口
7. 固化Nextcloud优化配置
8. SMB与Nextcloud权限互通
9. 每5分钟自动文件扫描
10. Windows WebDAV注册表修复（免重复输密码）
11. 每天凌晨自动备份disk1→disk2（数据双保险）

# 三、分步详细教程 + 全程避坑
## 【1】Ubuntu 24.04 系统安装
1. 必须安装：Server 无桌面纯净版
2. 务必勾选：安装OpenSSH
3. 磁盘默认自动分区，不手动分区
4. 设置静态IP：192.168.1.100 禁止DHCP

避坑要点：
1. 禁止装Ubuntu桌面版
2. 必须固定静态IP
3. 禁止随意升级系统内核

## 【2】双盘独立挂载（无RAID，最稳定）
### 2.1 硬盘分配
sdb → /data/disk1  主存储（Nextcloud+SMB全部数据）
sdc → /data/disk2  自动备份盘（每天同步disk1）

### 2.2 永久挂载命令
mkdir -p /data/disk1
mkdir -p /data/disk2

echo "UUID=8dc3ea4b-e4e8-4495-aefe-def4a58048ea /data/disk1 ext4 defaults 0 0" >> /etc/fstab
echo "UUID=a4e7f246-7c04-47c0-878b-4c0b6a73ea3d /data/disk2 ext4 defaults 0 0" >> /etc/fstab

systemctl daemon-reload
mount -a

### 2.3 无RAID核心优势
1. 重启永不掉盘、不崩溃
2. 坏一块盘，另一盘数据完整
3. 维护简单、不折腾阵列
4. 数据安全 = 主盘 + 自动定时备份

## 【3】系统基础初始化
apt update && apt upgrade -y
apt autoremove -y

# 严禁安装：nginx apache2 openssl
# 避免端口冲突，Docker专用

## 【4】1Panel 安装与使用规范
# 一键安装
curl -sSL https://resource.1panel.dev/install.sh | sh

# 1Panel 作用（必须保留）
1. Docker容器可视化管理
2. 网页文件管理、权限修改
3. 定时任务、服务器监控
4. 一键备份数据

# 禁止操作
1. 不装OpenResty
2. 不做反向代理、SSL
3. 不在应用商店装Nextcloud、数据库

## 【5】全新重装 MariaDB + Nextcloud 命令
# 第一步：清空旧环境
docker rm -f nextcloud nextcloud_db
docker system prune -a
rm -rf /data/nextcloud /data/mariadb
rm -rf /data/disk1/nextcloud /data/disk1/mariadb

# 第二步：创建数据目录
mkdir -p /data/disk1/nextcloud
mkdir -p /data/disk1/mariadb

# 第三步：部署数据库
docker run -d \
--name nextcloud_db \
--restart always \
-v /data/disk1/mariadb:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=root123 \
-e MYSQL_DATABASE=nextcloud \
-e MYSQL_USER=nextcloud \
-e MYSQL_PASSWORD=nextcloud123 \
mariadb:10.11

# 第四步：部署Nextcloud
docker run -d \
--name nextcloud \
--restart always \
-p 8080:80 \
-v /data/disk1/nextcloud:/var/www/html \
--link nextcloud_db:db \
nextcloud

# 第五步：修复权限
chown -R 33:33 /data/disk1/nextcloud
chmod -R 755 /data/disk1/nextcloud

# 访问地址
http://192.168.1.100:8080

# 安装填写信息
数据库地址：db:3306
数据库名：nextcloud
账号：nextcloud
密码：nextcloud123
取消安装推荐应用

## 【6】Nextcloud 安装后必做固化配置
# 关闭新用户演示文件夹
docker exec --user www-data nextcloud php occ config:system:set skeletondirectory --value=""

# 信任域名
docker exec --user www-data nextcloud php occ config:system:set trusted_domains 0 --value=192.168.1.100
docker exec --user www-data nextcloud php occ config:system:set overwrite.cli.url --value=http://192.168.1.100:8080

# 手动全盘扫描
docker exec --user www-data nextcloud php occ files:scan --all

## 【7】SMB + Nextcloud 完美互通（实时可见）
# 权限修复（SMB上传看不到就执行）
chown -R 33:33 /data/disk1/nextcloud

# 每5分钟自动扫描（SMB文件自动显示）
(crontab -l 2>/dev/null; echo "*/5 * * * * docker exec --user www-data nextcloud php occ files:scan --all > /dev/null 2>&1") | crontab -

## 【8】Windows WebDAV 修复（免重复输密码）
# 新建文本文档，保存为 WebDAV修复.reg 编码ANSI
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WebClient\Parameters]
"BasicAuthLevel"=dword:00000002
"FileSizeLimitInBytes"=dword:ffffffff
"AuthForwardServerList"="http://192.168.1.100:8080"

# 挂载地址
http://192.168.1.100:8080/remote.php/dav/files/你的用户名

# ======================================================
# 数据安全：每天自动备份 disk1 → disk2
# ======================================================
# 每天凌晨2点自动完整同步，删文件也同步清理
(crontab -l 2>/dev/null; echo "0 2 * * * rsync -av --delete /data/disk1/ /data/disk2/ > /dev/null 2>&1") | crontab -

# ======================================================
# 日常维护极简快捷命令（直接复制即用）
# ======================================================
# 1 修复SMB权限
chown -R 33:33 /data/disk1/nextcloud

# 2 手动扫描所有文件
docker exec --user www-data nextcloud php occ files:scan --all

# 3 重启Nextcloud
docker restart nextcloud

# 4 重启数据库
docker restart nextcloud_db

# 5 查看磁盘占用
df -h

# 6 清理Docker垃圾
docker system prune -a

# 7 查看自动任务
crontab -l

# ======================================================
# 常见故障快速排查
# ======================================================
1. Nextcloud打不开 → docker restart nextcloud
2. 数据库连接失败 → docker restart nextcloud_db
3. SMB文件看不到 → 权限+扫描
4. WebDAV弹窗密码 → 导入注册表+重启电脑
5. 磁盘满 → docker system prune -a
6. 数据丢失风险 → 依靠disk2自动备份恢复

# ======================================================
# 永久禁止操作（踩坑黑名单）
# ======================================================
1. ❌ 不更新Linux内核
2. ❌ 不安装Nginx/Apache
3. ❌ 不装OpenResty、不折腾SSL
4. ❌ 不组软RAID、不碰mdadm
5. ❌ 不用SQLite做多用户环境
6. ❌ 不随意修改8080端口
7. ❌ 不暴力关机、断电

