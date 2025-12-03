# Dockerfile构建镜像基础知识
理解 Dockerfile 是构建 Docker 镜像的核心。它是一种**文本格式的配置文件**，通过一系列指令告诉 Docker 如何一步步地组装出一个镜像。你可以把它想象成一个“自动化制作镜像的菜谱”。
下面这张图清晰地展示了从 Dockerfile 到最终容器运行的完整流程与核心概念：

<img width="2327" height="2653" alt="Image" src="https://github.com/user-attachments/assets/2ecbd552-899d-484d-b33a-4f69c16f9ec2" />

## 📝 Dockerfile 核心指令详解

- 指令  |  用途与说明  | 常用示例
- FROM | （必须的起点） 指定基础镜像，所有操作都基于它。 | FROM ubuntu:20.04
- RUN | （构建时执行） 在镜像内执行Shell命令，常用于安装软件、配置环境。 | RUN apt-get update && apt-get install -y python3
- COPY | （复制文件） 从构建上下文复制本地文件或目录到镜像内。 | COPY ./app /usr/src/app
- ADD | 功能类似COPY，但源可以是URL或自动解压tar包。多数场景推荐用COPY。 | ADD https://example.com/file.tar.gz /tmp/
- WORKDIR | （设置工作目录） 设置后续指令（如RUN, CMD）的当前工作目录，若不存在则创建。 | WORKDIR /app
- EXPOSE | （声明端口） 说明容器运行时打算监听的网络端口，仅起文档作用，不自动映射。 | EXPOSE 8080
- ENV | （设置环境变量） 定义的环境变量在构建和容器运行时都可用。 | ENV NODE_ENV production
- CMD | （容器默认命令） 指定容器启动时默认执行的命令。一个Dockerfile只能有一条CMD，多条则仅最后一条生效。 | CMD ["python", "app.py"]
- ENTRYPOINT | 类似CMD，但更“固执”。通常与CMD搭配使用，ENTRYPOINT定义可执行文件，CMD作为其默认参数。 | ENTRYPOINT ["nginx"] + CMD ["-g", "daemon off;"]

## 🛠️ 构建镜像的完整流程
让我们通过一个简单的例子，将上述知识串联起来。假设你有一个简单的Python应用。

**1. 创建 Dockerfile**
   在项目根目录创建一个名为 `Dockerfile` 的文件（**无后缀名**），内容如下：
``` 
dockerfile
 
# 使用官方Python运行时作为基础镜像
FROM python:3.9-slim
# 设置工作目录
WORKDIR /app
# 将当前目录下的所有文件复制到容器的/app目录下
COPY . /app
# 安装应用依赖
RUN pip install --no-cache-dir -r requirements.txt
# 声明容器运行时监听的端口
EXPOSE 8080
# 定义环境变量
ENV NAME World
# 容器启动时运行的命令
CMD ["python", "app.py"]
```

**2. 准备构建上下文**
   确保 `requirements.txt` 和 `app.py` 等文件与 Dockerfile 在同一目录。建议创建一个 `.dockerignore` 文件来排除不必要的文件（如`__pycache__`, `.git`），这能显著加快构建速度和减小镜像体积。

**3. 执行构建命令**

- **在命令行中**，进入 Dockerfile 所在目录，运行：

```
docker build -t my-python-app:v1 .
```
`-t` 用于给镜像打标签，最后的 `.` 代表**当前目录**是构建上下文。

- **在桌面版 Docker (Docker Desktop) 中**：
- 打开 Docker Desktop。
- 在左侧导航栏或主界面找到 “Images”。
- 通常会有一个 “Build” 或 “Build from Dockerfile” 的按钮。
- 选择你的 Dockerfile 所在目录，并填写镜像标签，然后点击构建。
## 💡 最佳实践与要点提示

- **理解镜像分层与缓存**：Dockerfile 中的**每条指令都会创建一个只读的镜像层**。
`docker build` 时，Docker 会**缓存**这些层。如果 Dockerfile 和某层之前的指令未变，就会直接使用缓存，这能极大加快重建速度。因此，**将不常变的指令（如安装依赖）放在前面，常变的指令（如复制源代码）放在后面**是优化关键。
- `COPY` vs `ADD`：除非需要自动解压或从URL获取，否则一律使用 `COPY`，因为它行为更明确、更简单。
  - `CMD` vs `ENTRYPOINT`：
  - `CMD` 更灵活，容易被 `docker run` 后面的命令覆盖。适用于指定默认的可变命令。
- `ENTRYPOINT` 更固定，适合将容器设为“可执行程序”，`CMD` 的内容作为其参数。
- **使用** `.dockerignore`：这非常重要，可以避免将本地调试文件、日志、git历史等打包进镜像，保证镜像精简和安全。
- **选择合适的基础镜像**：优先选择官方镜像，并选择带有 `-alpine`、`-slim` 等标签的轻量版本，可以大幅减小最终镜像体积。掌握了这些基础知识，你就能自己编写 Dockerfile 来创建定制镜像了。

## 上传镜像至hub.docker.com步骤
**上传步骤**（以命令行操作为例）
通过命令行完成“打标签 → 推送”两步是可靠的方法。下面以你的用户名为 myusername，本地镜像名为 myapp，标签为 latest 为例：

**第一步：为镜像打上正确的标签**
这相当于为你的本地镜像创建一个符合Docker Hub规则的“邮寄地址”。
```
docker tag 本地镜像名:标签 你的DockerHub用户名/仓库名:标签
# 例如：
docker tag myapp:latest myusername/myapp:latest
```
**第二步：推送已正确标记的镜像**
现在推送的地址指向了你自己的仓库。
```
docker push 你的DockerHub用户名/仓库名:标签
# 例如：
docker push myusername/myapp:latest
```