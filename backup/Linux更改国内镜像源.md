**Linux各发行版镜像源更换指南：**
在Linux系统使用过程中，由于默认软件源服务器位于海外，国内用户常遇到下载速度慢、更新失败等问题。通过更换国内镜像源可显著提升软件包下载和系统更新效率。本文将详细介绍CentOS/RHEL、Ubuntu/Debian、Arch Linux等主流发行版的镜像源更换方法。[清华大学源](https://mirrors.tuna.tsinghua.edu.cn/)
### Ubuntu/Debian系统镜像源更换
**Ubuntu更换阿里云源**
备份源列表：
```
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
```
备份原始源列表文件，防止配置错误时无法恢复。

修改源文件：
```
sudo nano /etc/apt/sources.list
```
替换为阿里云源（以Ubuntu 22.04为例）：
```
deb http://mirrors.aliyun.com/ubuntu/ jammy main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ jammy-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ jammy-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ jammy-backports main restricted universe multiverse
```
配置文件包含主仓库、安全更新、常规更新、候选版本和反向兼容五个部分。

更新软件包列表：
```
sudo apt update
```
更新软件包列表，使系统识别新的镜像源配置。

**Debian更换清华源**
```
sudo sed -i 's/deb.debian.org/mirrors.tuna.tsinghua.edu.cn/g' /etc/apt/sources.list
sudo apt update
```
通过sed命令批量替换源URL，实现快速切换。

### CentOS/RHEL系统镜像源更换
**手动配置阿里云镜像源**
备份原有配置：
```
sudo cp -r /etc/yum.repos.d/ /etc/yum.repos.d.backup
sudo rm -rf /etc/yum.repos.d/CentOS-*
```
该步骤通过复制整个目录实现完整备份，删除默认官方仓库文件为后续配置做准备。

创建阿里云repo文件：
```
sudo vi /etc/yum.repos.d/CentOS-Base-Aliyun.repo
```
粘贴以下内容（以CentOS 7为例）：
```
name=CentOS-7 - Base - Aliyun
baseurl=https://mirrors.aliyun.com/centos/7/os/$basearch/
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7

[updates]
name=CentOS-7 - Updates - Aliyun
baseurl=https://mirrors.aliyun.com/centos/7/updates/$basearch/
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7

[extras]
name=CentOS-7 - Extras - Aliyun
baseurl=https://mirrors.aliyun.com/centos/7/extras/$basearch/
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7
```
配置文件包含基础仓库、更新仓库和扩展仓库三个主要部分，每个部分指定了镜像URL和GPG密钥验证信息。

更新缓存：
```
sudo yum clean all
sudo yum makecache
```
清理旧缓存并生成新缓存，确保系统使用新的镜像源配置。

使用一键脚本
```
sudo sed -e 's|^mirrorlist=|#mirrorlist=|g' \
         -e 's|^#baseurl=http://mirror.centos.org|baseurl=https://mirrors.aliyun.com|g' \
         -i.bak /etc/yum.repos.d/CentOS-*.repo
sudo yum makecache
```
该脚本通过sed命令批量修改repo文件中的镜像URL，实现快速切换。



### Arch Linux系统镜像源更换
编辑镜像列表文件：
```
sudo nano /etc/pacman.d/mirrorlist
```
在文件顶部添加清华源：
```
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
```
Arch Linux的Pacman包管理器按照mirrorlist中Server的顺序选择镜像，将清华源放在顶部可优先使用。

更新缓存：
```
sudo pacman -Syyu
```
两次y参数确保从损坏镜像切换到正常镜像时不会出现问题。

通用注意事项
GPG密钥错误处理
```
sudo rpm --import https://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7  # CentOS
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys XXXXXX      # Ubuntu
```
当出现GPG密钥验证失败时，手动导入对应镜像源的GPG密钥。

缓存损坏修复
```
sudo rm -rf /var/cache/yum  # CentOS
sudo rm -rf /var/lib/apt/lists/*  # Ubuntu/Debian
sudo yum makecache || sudo apt update
```
彻底清理缓存后重建，解决因缓存损坏导致的更新失败问题。

### 其他注意事项
**1. 手动选择最快镜像（以 Ubuntu 为例）**
```
# 安装 apt-fast（自动选择最快镜像）
sudo add-apt-repository ppa:apt-fast/stable
sudo apt update && sudo apt install apt-fast -y

# 配置 apt-fast 使用 mirrors.aliyun.com
echo 'MIRRORS=( "http://mirrors.aliyun.com/ubuntu/" )' | sudo tee /etc/apt-fast.conf
```
**2. 部分软件单独配置源（以 Docker 为例）**
```
# 添加 Docker 官方源（Ubuntu/Debian）
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
**3. 验证源配置是否生效**
```
# 查看当前使用的镜像源（Debian/Ubuntu）
cat /etc/apt/sources.list

# 查看已配置的 repo（CentOS）
yum repolist
```
### 常见问题解决
**1.GPG 密钥验证失败**
```
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 密钥ID
```
**2.更新时出现 404 错误**
检查源配置中是否包含不兼容的版本（如 Ubuntu 22.04 使用了 20.04 的源）。

**3.恢复默认源**
删除自定义配置，重新下载官方配置：
```
sudo rm /etc/apt/sources.list  # Debian/Ubuntu
sudo wget https://mirrors.aliyun.com/repo/Centos-7.repo -O /etc/yum.repos.d/CentOS-Base.repo  # CentOS
```
镜像源地址汇总
发行版	推荐镜像源	配置文档链接
CentOS	阿里云/腾讯云/清华源	清华源帮助
Ubuntu	阿里云/网易/清华源	阿里云Ubuntu源
Debian	清华源/中科大源	中科大Debian源
Arch Linux	清华源/中科大源	清华Arch源
建议优先选择地理位置近、延迟低的镜像源，国内用户推荐使用阿里云、清华源等国内镜像站。更换后务必测试yum update或apt update能否正常执行，确保配置生效。