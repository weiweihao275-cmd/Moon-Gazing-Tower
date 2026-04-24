# Moon Gazing Tower - Docker 部署指南

## 📋 部署前检查清单

- [ ] 已安装 Docker 和 Docker Compose
- [ ] 服务器有足够的磁盘空间（建议 > 10GB）
- [ ] 端口 5003、27017、6379 未被占用
- [ ] 已配置适当的防火墙规则

## 🚀 快速部署（3 步）

### 1️⃣ 克隆项目
```bash
git clone https://github.com/weiweihao275-cmd/Moon-Gazing-Tower.git
cd Moon-Gazing-Tower/backend
```

### 2️⃣ 启动服务
```bash
docker-compose up -d --build
```

### 3️⃣ 验证部署
```bash
# 查看容器状态（所有容器应该为 healthy 或 running）
docker-compose ps

# 查看应用日志
docker-compose logs -f app

# 验证 MongoDB 连接
docker-compose exec mongo mongo --eval "db.adminCommand('ping').ok"

# 验证 Redis 连接
docker-compose exec redis redis-cli ping
```

---

## 🌐 访问应用

启动完成后，使用以下信息访问：

| 项目 | 值 |
|------|-----|
| **前端/后端地址** | http://your-server-ip:5003 |
| **默认用户名** | admin |
| **默认密码** | admin123 |
| **MongoDB** | localhost:27017 |
| **Redis** | localhost:6379 |

### 首次登录后立即修改密码！

---

## 📁 项目结构

```
backend/
├── config/
│   └── config.yaml          # ⭐ 主配置文件（已配置为生产模式）
├── docker-compose.yml       # ⭐ Docker Compose 配置
├── Dockerfile               # 后端镜像构建文件
├── tools/linux/             # 扫描工具存放目录
│   ├── ksubdomain          # 子域名扫描工具
│   ├── gogo                # 端口扫描工具
│   ├── katana              # Web 爬虫工具
│   ├── nuclei              # 漏洞扫描工具
│   └── ...
├── web/                     # 前端静态文件
└── ...
```

---

## 🔧 配置说明

### 环境变量配置 (`config/config.yaml`)

**生产级配置已包含：**
- ✅ JWT 密钥：`K9mP2xL7vN4qR8sT5wY6zU1aB3cD4eF5gH6iJ7kL8mN9oP0qR1sTuVwXyZaBcDeEf`
- ✅ 运行模式：`release`（生产）
- ✅ 数据库：MongoDB 容器连接
- ✅ 缓存：Redis 容器连接
- ✅ 日志级别：`info`

**如需自定义配置，编辑 `config/config.yaml` 后重启服务：**
```bash
docker-compose restart app
```

---

## 📦 外部工具安装

项目需要以下扫描工具。根据你的使用需求，下载相应版本到 `tools/linux/` 目录：

```bash
# 创建工具目录（如果不存在）
mkdir -p tools/linux

# 示例：下载 ksubdomain（子域名扫描）
wget https://github.com/boy-hack/ksubdomain/releases/download/v2.9.5/ksubdomain_2.9.5_linux_amd64.zip
unzip ksubdomain_2.9.5_linux_amd64.zip -d tools/linux/
chmod +x tools/linux/ksubdomain

# 示例：下载 gogo（端口扫描）
wget https://github.com/chainreactors/gogo/releases/download/v2.0.0/gogo_2.0.0_linux_amd64.tar.gz
tar -xzf gogo_2.0.0_linux_amd64.tar.gz -C tools/linux/
chmod +x tools/linux/gogo
```

### 推荐工具版本

| 工具 | 版本 | 用途 |
|------|------|------|
| ksubdomain | 2.9.5+ | 子域名爆破 |
| gogo | 2.0.0+ | 端口扫描 |
| nuclei | 2.9.0+ | 漏洞扫描 |
| katana | 1.0.0+ | Web 爬虫 |

---

## 🆘 故障排查

### 问题 1：容器启动失败

```bash
# 查看详细错误日志
docker-compose logs app

# 重新构建镜像
docker-compose down -v
docker-compose up -d --build
```

### 问题 2：MongoDB 连接失败

```bash
# 检查 MongoDB 是否正常运行
docker-compose logs mongo

# 验证 MongoDB 健康状态
docker-compose ps mongo

# 重启 MongoDB
docker-compose restart mongo
```

### 问题 3：Redis 连接失败

```bash
# 检查 Redis 日志
docker-compose logs redis

# 测试 Redis 连接
docker-compose exec redis redis-cli ping
# 预期输出：PONG
```

### 问题 4：应用无法访问

```bash
# 检查端口是否正确映射
docker-compose ps

# 测试网络连接
curl -I http://localhost:5003

# 检查防火墙规则
sudo ufw allow 5003/tcp
```

### 问题 5：磁盘空间不足

```bash
# 查看 Docker 数据卷使用情况
docker system df

# 清理未使用的卷
docker volume prune

# 查看容器大小
docker ps -a --format '{{.Names}}\t{{.Size}}'
```

---

## 💾 数据备份与恢复

### 备份 MongoDB

```bash
# 备份整个数据库
docker-compose exec mongo mongodump --archive=/data/db/moongazing_backup.archive

# 从容器复制备份文件到本地
docker cp moongazing_mongo:/data/db/moongazing_backup.archive ./backup/

# 定时备份（每日 2 点）
# 在 crontab 中添加
0 2 * * * cd /path/to/backend && docker-compose exec -T mongo mongodump --archive=/data/db/backup_$(date +\%Y\%m\%d).archive
```

### 恢复 MongoDB

```bash
# 复制备份文件到容器
docker cp ./backup/moongazing_backup.archive moongazing_mongo:/data/db/

# 恢复数据库
docker-compose exec mongo mongorestore --archive=/data/db/moongazing_backup.archive
```

### 备份 Redis

```bash
# Redis 已配置 AOF 持久化，数据自动保存到
# docker_redis_data 卷中

# 手动触发 Redis 快照
docker-compose exec redis redis-cli bgsave

# 备份数据卷
docker cp moongazing_redis:/data ./backup/redis_data_$(date +%Y%m%d)
```

---

## 🔐 安全建议

### 立即完成（部署后）

1. **修改默认密码**
   ```bash
   # 登录应用后，进入用户设置修改 admin 用户密码
   ```

2. **更新 JWT 密钥**（如果在生产环境）
   - 编辑 `config/config.yaml`
   - 修改 `jwt.secret` 为安全的随机字符串
   - 重启应用：`docker-compose restart app`

3. **配置 HTTPS**
   - 使用 Nginx 做反向代理
   - 获取 SSL 证书（Let's Encrypt）

### 防火墙配置

```bash
# 仅允许特定 IP 访问
sudo ufw allow from 192.168.1.0/24 to any port 5003

# 或仅允许本地访问
sudo ufw allow from 127.0.0.1 to any port 5003
```

### 网络隔离

```bash
# 移除端口映射，仅通过 Nginx 反向代理访问
# 编辑 docker-compose.yml，删除 ports 配置
# 使用容器网络通信
```

---

## 📊 性能优化

### 增加扫描并发数

编辑 `config/config.yaml`：
```yaml
scanner:
  worker_count: 20        # 默认 10，可根据服务器配置增加
  timeout: 300            # 扫描超时时间（秒）
  retry_count: 3
  retry_delay: 5
```

### 增加日志轮转

```yaml
log:
  max_size: 200           # 单个日志文件大小（MB）
  max_backups: 10         # 保留日志备份数
  max_age: 30             # 日志保留天数
```

### 资源限制（可选）

编辑 `docker-compose.yml`，为各个服务添加资源限制：

```yaml
services:
  app:
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 2G
        reservations:
          cpus: '1'
          memory: 1G
```

---

## 📝 常用命令

```bash
# 启动所有服务
docker-compose up -d

# 停止所有服务（保留数据）
docker-compose stop

# 停止并删除所有容器和网络（保留数据卷）
docker-compose down

# 完全清除（删除所有数据）⚠️ 谨慎使用
docker-compose down -v

# 查看日志
docker-compose logs -f          # 所有容器
docker-compose logs -f app      # 应用容器
docker-compose logs -f mongo    # MongoDB
docker-compose logs -f redis    # Redis

# 进入容器 shell
docker-compose exec app sh      # 应用容器
docker-compose exec mongo bash  # MongoDB

# 查看容器资源使用
docker stats

# 重建镜像
docker-compose build --no-cache
```

---

## 📞 获取帮助

- **项目仓库**：https://github.com/weiweihao275-cmd/Moon-Gazing-Tower
- **提交问题**：https://github.com/weiweihao275-cmd/Moon-Gazing-Tower/issues
- **README**：查看项目根目录的 README.md

---

## ✅ 部署成功标志

如果你看到以下现象，说明部署成功了：

1. ✅ `docker-compose ps` 显示所有容器都在运行
2. ✅ MongoDB 容器状态为 `healthy`
3. ✅ 能访问 http://your-server-ip:5003
4. ✅ 能用 admin/admin123 登录
5. ✅ `docker-compose logs app` 中没有错误信息

---

**部署日期**: 2026-04-24  
**配置版本**: 1.0 (Production)
