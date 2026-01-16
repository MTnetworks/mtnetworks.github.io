在目录下
创建docker-compose.yml文件
# 1. 拉取镜像（可能需要一些时间）
sudo docker compose pull
# 2. 启动所有服务（在后台运行）
sudo docker compose up -d
# 3. 查看启动日志，观察是否有错误
sudo docker compose logs -f --tail=50
# 4.查看服务状态
sudo docker compose ps
# 5停止服务
sudo docker compose down
# 6重启服务
sudo docker compose restart
# 7查看所有容器的实时日志
sudo docker compose logs -f
# 8查看特定服务（如 pgvecto_upgrade）的日志（这是诊断的关键）
sudo docker compose logs pgvecto_upgrade
# 9更新 Immich（先拉取新镜像，再重新启动）
sudo docker compose pull
sudo docker compose up -d

# 删除 Docker Compose 版应用
# 1. 进入部署目录（请确认你的路径）
cd /mnt/DATA/Apps/Immich
# 2. 【关键】停止并删除所有容器、网络，同时删除所有关联的命名卷（-v 参数）
sudo docker compose down -v
# 3. 【可选但建议】删除从 Docker Hub 拉取的镜像，释放磁盘空间
sudo docker image prune -a
# 4. 【可选】最后，如果你也想删除主机上存放所有数据、配置的物理目录（谨慎操作！）
#    执行前请绝对确认此目录 `/mnt/DATA/Apps/Immich` 下没有其他重要数据！
#    sudo rm -rf /mnt/DATA/Apps/Immich