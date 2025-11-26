### Truenas上安装独立的HomeAssistantOS系统
因TrueNAS商店的HomeAssistant有功能上的缺陷，所以需独立部署HAOS系统

**重要提醒** 在创建虚拟机之前先创建**2块数据集**（Dataset）格式为**Zvol**，分别存储下载的镜像文件和系统盘。并提前下载好[Ubuntu Desktop](https://ubuntu.com/download/desktop)镜像文件。

**1. 创建虚拟机**

- 进入 **虚拟化 → 虚拟机**
- 点击 **添加**
- 配置：
  **虚拟机基本配置**
  -   名称: Homeassistant
  -   描述: 独立 Home Assistant 实例
  -   **硬盘选项：**选择创建好的系统镜像盘
  -   系统和配置: 选择Linux和合适的内存和 CPU
  -   **引导设备**: 选择**[Ubuntu Desktop instructions](https://ubuntu.com/download/desktop).ISO**文件
  -   **网卡选择**：**不要选VirtIO**
  -   保存配置后不要急着启动
      **添加硬盘**
     找到虚拟机列表
    1.点设备
    2.添加设备
    3.添加硬盘
   4.添加之前创建好的 存放镜像文件的硬盘

**2.通过[Ubuntu Desktop试用版](https://ubuntu.com/download/desktop)进行安装Homeassistant系统**

1. 提前将Ubuntu Desktop镜像文件ISO放入共享文件里
2. 修改设备类型：将CD-ROM文件修改为Ubuntu Desktop镜像文件ISO
3. 系统随后启动Ubuntu。
4. 把你的系统连接到网络，确保它有互联网接入。
5. 在Ubuntu中，打开浏览器并打开当前文档页面，这样你可以按照步骤作。
6. 从那里开始，[下载HAOS镜像](https://github.com/home-assistant/operating-system/releases/download/16.3/haos_generic-x86-64-16.3.img.xz).
7. 在Ubuntu中，左下角选择“显示应用程序”。
8. 在应用程序中搜索并打开磁盘，开始恢复HAOS映像：
9. 在磁盘中，左侧选择你想安装HAOS的内部磁盘设备。
10. 在屏幕顶部，选择三点菜单，选择“恢复磁盘映像......”。恢复磁盘映像：选择三点菜单
11. 选择你刚下载的图片。恢复磁盘映像：选择映像
12. 选择开始恢复......。恢复磁盘镜像：开始恢复
13. 请选择恢复确认。恢复磁盘镜像：选择恢复
14. 重启得到home-assistant管理地址