### 前期准备工作
TrueNAS Scale 下载地址：https://www.truenas.com/download-truenas-scale/
下载文件说明：您下载的文件将是一个 .iso 格式的镜像文件，文件名类似于：TrueNAS-SCALE-24.10.0.iso 这个文件用于制作安装盘。
安装前的准备工作
**制作启动盘：**
您需要一个至少 8GB 的 U 盘。
使用刻录工具将下载的 .iso 文件写入 U 盘。
**推荐刻录工具：**
Rufus (Windows)： https://rufus.ie/
balenaEtcher (Windows/macOS/Linux)： https://www.balena.io/etcher/
硬件要求：
内存：最低 8GB，强烈推荐 16GB 或以上，尤其是计划使用 ZFS 或运行较多应用时。
存储：至少一块硬盘用于安装系统（推荐 16GB 以上的 SSD 以获得最佳性能），外加多块硬盘用于组建存储池。
启动方式：确保您的服务器/电脑 BIOS/UEFI 已设置为从您制作的 U 盘启动。
**重要提示**：TrueNAS 系统会独占安装它的整个磁盘。该磁盘将无法再用于存储数据。
数据存储盘和系统安装盘是分开的。您用 U 盘安装系统到另一块 SSD 上，然后用其他的硬盘来存放您的电影、文档等数据。
### 总结步骤
1. 访问 https://www.truenas.com/download-truenas-scale/ 下载 ISO 文件。
2. 使用 Rufus 或 balenaEtcher 将 ISO 文件写入 U 盘，制作成安装盘。
3. 将安装盘插入您打算用作 NAS 的主机，并设置从 U 盘启动。
4. 按照屏幕提示进行安装（过程通常很简单，主要是选择系统安装的磁盘和设置管理员密码）。
5. 安装完成后，重启并拔掉 U 盘，系统会正常启动。
6. 根据屏幕上显示的 IP 地址，在您同一局域网内的电脑浏览器上访问该 IP，即可进入 TrueNAS 强大的 Web 管理界面。

### 🛠️ 创建数据集
**创建数据集 (Create Dataset)**
数据集（Dataset）是在存储池内创建的逻辑分区，帮你更好地管理和配置不同类别的数据。

- **操作路径**：在TrueNAS Web管理界面，进入 Storage > Pools。
- **如何操作**：找到你已创建的存储池，点击池右边的"三点菜单"图标，然后选择 Add Dataset。
- **关键设置**：为新数据集命名（例如 Documents），其他高级设置（如压缩、加密）初期可保持默认。

**配置用户与权限**

为日常使用创建独立用户并分配合适的权限，是保证系统安全的重要一环。

- **创建用户**：进入 Credentials > Local Users，点击 Add。填写用户名、密码等基本信息。可以勾选"Microsoft Account"选项以更好地兼容SMB共享。

- **设置权限**：进入 Storage > Pools，找到特定数据集，点击其右边的"三点菜单"，选择 Edit Permissions。将"用户"和"组"设置为刚创建的用户，并勾选"Apply User"和"Apply Group"。

**设置文件共享 (File Sharing)**

TrueNAS支持多种共享协议，其中SMB/CIFS最适合在Windows局域网中使用。

- **创建SMB共享**：进入 Sharing > Windows Shares (SMB)，点击 Add。
- **指定路径**：在"Path"中选择你刚才创建的数据集。
- **启用服务**：共享创建后，系统可能会提示你启动SMB服务，记得选择"自动启动"。

完成设置后，你可以在局域网其他电脑的文件资源管理器地址栏输入\\<TrueNAS的IP地址>来访问共享文件夹。
**配置网络与服务**

- **设置静态IP**：在TrueNAS控制台菜单或Web界面的 Network 部分，为你的NAS设置一个静态IP地址。这能确保设备的网络地址固定不变。
- **检查服务**：进入 Services 列表，确保你使用的共享服务（如SMB）已设置为"自动启动"。

### 如何配置共享权限

<img width="567" height="1821" alt="Image" src="https://github.com/user-attachments/assets/c1a11071-dce8-4147-998c-72b20cf403f9" />

**步骤一：（强烈推荐）创建本地用户**
为了系统安全，不建议一直使用 root 账户。为日常使用创建一个独立的用户。

1. **进入 Credentials（凭证） > Local Users（本地用户）。**
2. 点击 **Add（添加）**。
3. **填写用户信息**：
**Full Name（全名）**：可选，如 John Doe。
**Username（用户名）**：必填，这是登录共享时用的名字，如 john。
**Password（密码） 和 Confirmation（确认密码）**：设置一个强密码。
**勾选 “Microsoft Account”**：这个选项很重要**，确保它与 SMB 协议兼容。
4. 点击 **Save（保存）**。

**步骤二：配置 SMB 共享**
这是核心步骤，告诉 TrueNAS 将哪个目录共享到网络上。

1.  **进入 Sharing（共享） > Windows Shares (SMB)。**
2. 点击右上角的 **ADD（添加）**。
3. 在 **Basic（基本）** 配置中：

- **Path（路径）**：点击下拉菜单，浏览并选择你在 步骤一 中创建的数据集（例如 /mnt/your-pool/documents）。
- **Name（名称）**：这是共享在网络上显示的名称。可以和数据集名称相同，也可以不同（例如 Public Docs）。
- **Description（描述）**：可选，用于说明这个共享的用途。

4. 其他设置保持默认即可，点击 **Save（保存）**。

**步骤三：设置权限（确保用户可以访问）**
现在你需要确保在 步骤二 创建的用户对这个数据集有访问权限。

1. 返回 **Storage（存储） > Pools（存储池）**。
2. 找到你用于共享的那个数据集，点击右边的 ... 按钮，然后选择 **Edit Permissions（编辑权限）**。
3. 在弹出的权限编辑器窗口中：

- **User（用户）**：选择你创建的用户（如 john）。
- **Group（组）**：可以选择 users。
- **权限**：根据你的需要设置。对于简单的读写访问，可以勾选 **Read（读）、Write（写） 和 Execute（执行）**。
- **重要**：勾选 **Apply User（应用用户） 和 Apply Group（应用组）** 选项。这将把权限递归地应用到该数据集内的所有现有文件和文件夹。

4.点击 **Save（保存）**。