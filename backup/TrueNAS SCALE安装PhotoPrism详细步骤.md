### 第一步：规划与创建存储数据集
在TrueNAS存储池中创建专门的数据集，用于存放数据库和照片数据，便于管理和备份。
进入 **存储** > **数据集**。
点击 **添加**，创建数据集（名称可自定义）：photoprism：建议在此数据集下再创建三个子数据集。
分别用于：import：临时存放从Web界面“导入”的照片目录；storage：存放PhotoPrism的配置、缓存和索引文件；originals：存放你的原始照片和视频文件。
记下这些数据集的 挂载点路径（通常为 /mnt/你的存储池/数据集名称），后续配置会用到。
### 第二步：通过应用市场搜索PhotoPrism开始安装
### 第三步：基本设置
**Site URL（站点URL）**：暂时可以留空，或填写你打算访问的地址（如 http://你的TrueNAS-IP:2342）。
**管理员密码***：设置一个强密码，用于登录PhotoPrism的Web管理界面（用户名默认是admin）。务必牢记。
用户ID / 群组ID：保持默认的 568 即可，这是PhotoPrism应用在容器内运行的身份标识。
### 第四步：添加数据库变量（### **重点**### ）
点击：**Additional Environment Variables** 添加
**名称**：PHOTOPRISM_DATABASE_DRIVER
**值**：sqlite
### 第五步：网络配置
WebUI端口：
**Port Bind Mode**：保持默认的 Publish port on the host for external access（在主机上发布端口以供外部访问）。
**Port Number***：2342，或更改为任意未被占用的端口。
**Host IPs**：留空表示绑定到主机的所有IP地址。
其他选项（Host Network, Certificate ID, DNS Options）保持默认或留空。
### 第六步：存储配置
系统为你自动创建了三个ixVolume，这是合理的，但你需要明确它们的用途：

- **Photoprism Import Storage**：临时存放从Web界面“导入”的照片，导入完成后文件会被移动到Originals。可使用自动创建的数据集。
**类型**：Host Path
**Host Path**：/mnt/DATA/photoprism/import

- **Photoprism Storage**：存放PhotoPrism的配置文件、缓存和索引数据库（SQLite文件，但我们将用MariaDB替代）。建议你在这里选择“添加”，手动挂载一个你预先创建好的、专门用于storage的数据集，便于管理和备份。
**类型**：Host Path
**Host Path**：/mnt/DATA/photoprism/storage

- **Photoprism Originals Storage**：存放所有原始照片和视频文件的核心位置。强烈建议手动挂载到你预先创建的originals数据集。
**类型**：Host Path
**Host Path**：/mnt/DATA/photoprism/originals
操作建议：对于后两项，点击“添加”，选择类型为“存储卷”，然后从列表中选择你之前为PhotoPrism创建的数据集（例如 photoprism/storage 和 photoprism/originals），并填写正确的容器内挂载路径（/photoprism/storage 和 /photoprism/originals）。
### 第七步：安装
配置好cpu和内存后点安装即可

### 第八步：配置在 TrueNAS 中创建专用的 SMB 共享数据集（推荐）
为了解决核心需求（从Windows方便上传，且PhotoPrism能读取），我推荐一个更清晰、安全的一劳永逸的方案：创建一个专用的、高权限的“照片管理”用户。
以下是详细操作流程图，它能帮你理清用户、权限和应用间的完整关系：

<img width="2055" height="3516" alt="Image" src="https://github.com/user-attachments/assets/4e40f324-4b3a-4a8e-904d-fe0f9b09d868" />

具体操作命令/步骤：
1. 创建专用用户：
进入 “**凭据**” > “**本地用户**” > “**添加**”。
用户名：例如 photo_admin。
密码：设置一个强密码。
**主要组**：**选择 apps**（这一步至关重要，它让该用户创建的文件天然对容器可读）。
其他保持默认，点击 “提交”。
2. **配置数据集SMB共享权限（使用ACL管理器）**：
**为 apps 组添加 读取、执行 和 遍历目录 权限**。
编辑或创建SMB共享时，在 “**ACL编辑**” 中找到 “访问控制列表” 设置，**添加 apps 组**。这能确保通过SMB创建的文件继承正确的组所有权。
完成后，你将拥有一个完美的流程：
在Windows上：你用 photo_admin 账户登录SMB，拥有完全的上传和管理权限。
在PhotoPrism中：容器（属apps组）能无缝读取你上传的所有照片并为其建立索引。
所有新创建的文件，其组所有者都会是 apps，从根本上杜绝了权限问题。
这个方案结构清晰，安全可控，是兼顾便利性与系统安全的推荐做法。

