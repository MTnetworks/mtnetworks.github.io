### Truenas上安装独立的HomeAssistantOS系统
因TrueNAS商店的HomeAssistant有功能上的缺陷，所以需独立部署HAOS系统

**安装思路** 由于HAOS官方文件是xxx.img.xz后缀结尾的，不能直接在虚拟机上进行引导安装。我们需要借助悟空大神的一个系统制作工具将img.xz格式的文件转为iso进行引导安装，具体操作如下。

在创建虚拟机之前先创建**块数据集**（Dataset）格式为**Zvol**，用于安装系统使用。
首先我们来的悟空的镜像安装器项目地址：https://github.com/wukongdaily/img-installer  可以fork到自己的仓库下面，然后来到刚fork项目的**Actions**,找到**Build HAOS Inastaller ISO**这个工作流，点击**Run workflow**，输入HAOS官网固件下载地址(扩展名 .img.gz/.img.xz/.img.zip)，然后点运行，稍等几分钟后由.img.xz的固件就转换成了iso文件了。然后下载到本地上传只NAS文件夹里。

**创建虚拟机**

- 进入 **虚拟化 → 虚拟机**
- 点击 **添加**
- 配置：
  **虚拟机基本配置**
  -   名称: Homeassistant
  -   描述: 独立 Home Assistant 实例
  -   **硬盘选项：**选择创建好的系统镜像盘
  -   系统和配置: 选择Linux和合适的内存和 CPU
  -   **引导设备**: 选择刚刚上传的HAOS.ISO**文件
  -   **网卡选择**：**不要选VirtIO**
  -   保存配置启动
-  启动后:在安装器所在系统里输入 ddd 命令 方可调出安装菜单。
<img width="1370" height="548" alt="Image" src="https://github.com/user-attachments/assets/348619bf-04d4-4c33-9fd2-c91d2481a4df" />

- 接下来就是选择镜像名称和安装硬盘位置，输入大写的YES进行安装，安装完后删除CR-ROM在启动系统。
<img width="485" height="351" alt="Image" src="https://github.com/user-attachments/assets/0c0ff32f-850f-4a76-b202-26b7f80e6192" />

- 重启得到home-assistant管理地址