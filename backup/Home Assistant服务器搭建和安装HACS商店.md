### Home Assistant服务器搭建
搭建 Home Assistant 服务器是一个非常有趣且实用的项目，它能让你彻底掌控自己的智能家居，实现高度自动化和隐私保护。
**第一步**：选择**硬件平台这是最关键**的第一步，决定了你后续的体验和扩展性。以下是主流选择，按推荐程度排序：
1. 专业单板计算机（最推荐）Raspberry Pi 4/5 (树莓派)
2. 小型专用服务器Intel NUC / 类似迷你PC
3. 成品 NAS 设备群晖 / 威联通 NAS
4. 直接购买官方设备Home Assistant Yellow / Green：Home Assistant 官方推出的硬件

**第二步**：Home Assistant安装方式： 提供了三种不同的安装方式，对应不同的用户群体。

- 方法一（**推荐**）：从U盘启动Ubuntu并安装Home Assistant操作系统它也适用于带有内置硬盘的笔记本电脑和台式电脑。请务必选择“试用Ubuntu”。这样就会在U盘上运行Ubuntu系统。在应用程序中，搜索并打开“磁盘”，然后开始恢复 HAOS 映像：在“磁盘”选项卡的左侧，选择要安装 HAOS 的内部磁盘设备。在屏幕顶部，选择三个点。菜单并选择“恢复磁盘映像…”。https://www.home-assistant.io/installation/generic-x86-64/

- 方式二：Home Assistant Operating System（最推荐，尤其新手）这是官方最推荐的安装方式。它会直接在你的硬件上安装一个专为 Home Assistant 优化的完整操作系统。优点：最简单、最稳定、管理最方便，支持所有功能和插件（Add-ons）。缺点：独占整个机器。适用平台：树莓派、迷你PC、虚拟机等。安装流程（以树莓派为例）：下载 Home Assistant OS 镜像文件（针对你的树莓派型号）。使用 Balena Etcher 工具将镜像写入你的 SSD（或 SD 卡）。将 SSD 插入树莓派，连接网线和电源。开机等待约 20-30 分钟，然后在浏览器中输入 `http://homeassistant.local:8123` 即可访问。

- 方式三：Home Assistant Container（使用 Docker）如果你已经有在运行 Docker 的服务器（如 NAS、Linux 系统），这是最灵活的方式。优点：资源隔离，不污染主机环境，易于备份和迁移。缺点：需要一定的 Docker 知识，高级功能（如 Add-ons）需要手动配置。安装流程：在主机上安装 Docker 和 Docker Compose。创建一个 `docker-compose.yml` 文件。运行 `docker-compose up -d` 启动容器。

### Home Assistant怎么开启SSH
对于 Home Assistant OS（最常用的方法）在 Home Assistant OS 中，官方提供了一个非常方便的 SSH 服务器插件。这是最推荐、最安全的方法。
步骤如下：

1. **进入 Home Assistant 管理界面**
打开你的 Home Assistant 网页。
2. **安装「Terminal & SSH」插件**

-  在左侧边栏点击 **「设置」**。
-  选择 **「加载项」**，
- 在右下角点击 **「加载项商店」**。
- 在商店中搜索 "SSH"，找到官方提供的 **「Terminal & SSH」** 插件并点击它。
-  点击 **「安装」** 按钮。

**3.配置插件**
安装完成后，不要立即启动。先点击 **「配置」** 标签页。
你需要修改配置文件，一个典型的配置如下：yaml

```
sessions: {}
packages: []
username: "你的用户名"  # 例如 "root" 或 "hass"
password: "你的强密码"  # 请务必设置一个复杂的密码！
authorized_keys: []
sftp: false

```
`username`: 默认为 `root`，你也可以改成其他名字。
`password`: **这是最重要的**，请务必设置一个非常强壮的密码，因为这会暴露你的系统到网络上。
`authorized_keys`: 如果你使用 SSH 密钥对登录（更安全），可以将你的公钥内容填写在这里，格式如 `["ssh-rsa AAAAB3...you@host"]`。如果使用密钥，可以将 `password` 字段留空或删除。
`sftp`: 如果需要使用 SFTP 传输文件，可以设置为 `true`。

 **4.启动插件并设置自启动**

- 保存配置后，切换到 **「信息」** 标签页。
- 点击 **「启动」** 按钮。
- 启动成功后，建议将 **「自动启动」** 和 **「监视」** 选项打开，这样 Home Assistant 重启后 SSH 服务会自动运行。

5.**连接测试**

- 打开你的 SSH 客户端（如 Windows 下的 PuTTY、macOS/Linux 下的 Terminal）。
- 连接地址是你的 Home Assistant 的 IP 地址，端口是 22。
- 使用你设置的用户名和密码（或密钥）进行连接。

**示例命令（在终端中）**：
```
ssh root@你的_home_assistant_ip

```
输入密码后，你应该就能成功登录到 Home Assistant OS 的系统命令行了。

Home Assistant商店默认仓库地址:https://github.com/esphome/home-assistant-addon

### HACS如何安装
**前提条件**

1. **运行中的 Home Assistant**：确保你的 Home Assistant 实例已启动并可以访问。
2. **GitHub 账户**：你需要一个 GitHub 账户，HACS 需要使用它进行认证。
3. **文件管理能力**：你需要能够访问并管理 Home Assistant 的文件系统。推荐使用官方加载项 **「File editor」 或 「Samba share」**。

**安装步骤（通过终端命令 - 推荐方法）**
这是 HACS 官方推荐的、最快捷的安装方式。
**第一步：安装 File editor 加载项（如果尚未安装）**
为了能操作文件，最好先安装一个文件编辑器。

1. 进入 Home Assistant：**设置 -> 加载项 -> 加载项商店**。
2. 搜索并安装 "**File editor**" （通常由 "Home Assistant Community Add-ons" 提供，如果没有，可以安装 "Studio Code Server"）。
3. 安装后启动它，并设置自动启动。

**第二步：使用 File editor 的终端执行安装命令**

1. 打开 **File editor** 的 Web UI。
2. 在 File editor 的界面上，**寻找一个终端/命令行图标或标签页**（通常位于界面底部或侧边栏，可能叫做 "Terminal", "Console", ">" 符号）。点击它打开一个命令行窗口。
(如果你的 File editor 没有终端，请跳至下方的「备用方法」)
3. 在终端中，逐行输入并执行以下命令：
```
wget -O - https://get.hacs.xyz | bash -
```
这个命令会从 HACS 官方服务器下载安装脚本并自动执行。
4. 等待脚本运行完成。当看到类似 "**Installation complete!**" 的成功提示时，即可关闭窗口。
**第三步：重启 Home Assistant 核心**

1. 进入 Home Assistant：**设置 -> 系统 -> 硬件**。
2. 点击右上角的 "**重启**" 按钮，选择 "**重新启动 Home Assistant 核心**"。
3. 等待 Home Assistant 完全重启。

### 备用方法（手动安装）
如果上述命令方法因网络问题失败，或者你的环境无法使用终端，可以采用手动安装。
**第一步：下载 HACS**
1.用电脑浏览器访问 HACS 的 GitHub 发布页面：
https://github.com/hacs/integration/releases/latest
2.下载名为 hacs.zip 的文件。
**第二步：解压并放置文件**

1. 通过 **File editor** 或 Samba share 连接到你的 Home Assistant 配置文件目录（通常是 /config）。
2. 确认是否存在 custom_components 文件夹。**如果没有，请创建一个**。
3. 将下载的 hacs.zip 文件解压。你会得到一个名为 hacs 的文件夹。
4. 将这个 hacs 文件夹**完整地**放入 /config/custom_components/ 目录下。

- 最终路径应该是：/config/custom_components/hacs/

**第三步：重启 Home Assistant 核心**
方法与上面相同，进入系统设置并重启核心。

**配置与使用 HACS**
无论使用哪种安装方法，重启后都需要进行配置。
**第一步：添加 HACS 集成**

1. 重启完成后，再次进入 Home Assistant。
2. 进入：设置 -> 设备与服务。
3. 点击右下角的 "添加集成" 按钮。
4. 在搜索框中输入 "HACS"。

- 如果搜索不到，请稍等几分钟再试，或者再次彻底重启 Home Assistant。

5. 点击找到的 HACS 集成，并按照屏幕提示进行操作。
6. 这个过程会引导你前往 GitHub 进行认证。你需要：
- 登录你的 GitHub 账户。
- 授权 HACS 访问（这通常是安全的）。
- 你会得到一个设备验证码，将其复制并粘贴回 Home Assistant 的配置界面中。

**第二步：开始使用 HACS**

1. 配置成功后，你的 Home Assistant 左侧边栏会出现一个新的 "HACS" 菜单项。
2. 点击进入 HACS，你就可以开始浏览和安装：

- **集成**：各种智能家居设备的自定义集成。
- **前端**：主题、Lovelace 卡片、插件等。
- **自动化**：预制的自动化脚本。
- **模板**：设备配置模板。