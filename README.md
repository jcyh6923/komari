# komari
## 当前镜像版本 v1.2.3

## 快速开始

```bash
docker run -d \
  --name komari \
  --restart unless-stopped \
  -p 25774:25774 \
  -p 8443:8443 \
  -v ./komari-data:/app/data \
  # 【必需】GitHub 备份配置
  -e GH_BACKUP_USER="your_github_username" \
  -e GH_REPO="your_private_repo_name" \
  -e GH_PAT="your_github_personal_access_token" \
  -e GH_EMAIL="your_github_email@example.com" \
  # 【必需】面板登录
  -e ADMIN_USERNAME="yourusername" \
  -e ADMIN_PASSWORD="yourpassword" \
  # 【可选】备份配置
  # -e BACKUP_TIME="0 20 * * *" \
  # -e BACKUP_DAYS="10" \
  # 【可选】节点订阅（VLESS/VMESS）
  # -e UUID="your-uuid-here" \
  # -e ARGO_DOMAIN="your-domain.com" \
  # -e CF_IP="your-cf-ip" \
  # -e CADDY_PROXY_PORT="8443" \
  # -e SUB_NAME="komari" \
  # 【可选】Cloudflare 隧道
  # -e KOMARI_ENABLE_CLOUDFLARED="true" \
  # -e KOMARI_CLOUDFLARED_TOKEN="eyJxxxxx" \
  ghcr.io/jyucoeng/komari:latest
```

## 必需的环境变量

### GitHub 备份

- `GH_BACKUP_USER` - GitHub 用户名
- `GH_REPO` - 备份仓库名（私有）
- `GH_PAT` - GitHub Personal Access Token（需要 repo 权限）
- `GH_EMAIL` - Git 提交邮箱

### 面板登录

- `ADMIN_USERNAME` - 面板用户名
- `ADMIN_PASSWORD` - 面板密码

## 可选的环境变量

### 备份配置

- `BACKUP_TIME` - Cron 表达式，默认 `0 20 * * *`（UTC 20:00）
- `BACKUP_DAYS` - 保留备份天数，默认 `10`

### 原镜像变量

- `KOMARI_PORT` - 内部服务端口，默认 `25774`
- `ADMIN_USERNAME` - 登录用户名
- `ADMIN_PASSWORD` - 登录密码
- `KOMARI_ENABLE_CLOUDFLARED` - 是否启用 CF 隧道（true/false）
- `KOMARI_CLOUDFLARED_TOKEN` - 隧道 Token

### 节点订阅（可选）

- `UUID` - 生成 VLESS/VMESS 订阅链接（未设置时跳过）
- `ARGO_DOMAIN` - 服务器域名（未设置时自动获取公网 IP）
- `CF_IP` - CDN 优选 IP，默认 `ip.sb`
- `CADDY_PROXY_PORT` - 反代端口，默认 `8443`
- `SUB_NAME` - 订阅名称，默认 `komari`

## 使用节点订阅

启用节点订阅需要设置 `UUID`：

```bash
docker run -d \
  --name komari \
  --restart unless-stopped \
  -p 25774:25774 \
  -v ./komari-data:/app/data \
  # 【必需】GitHub 备份配置
  -e GH_BACKUP_USER="your_github_username" \
  -e GH_REPO="your_private_repo_name" \
  -e GH_PAT="your_github_personal_access_token" \
  -e GH_EMAIL="your_github_email@example.com" \
  # 【必需】面板登录
  -e ADMIN_USERNAME="yourusername" \
  -e ADMIN_PASSWORD="yourpassword" \
  # 【必需】节点订阅
  -e UUID="your-uuid-here" \
  # 【可选】节点订阅配置（如果不设置 ARGO_DOMAIN 则需暴露 8443 端口）
  -e ARGO_DOMAIN="your-domain.com" \
  # -e CF_IP="your-cf-ip" \
  # -e CADDY_PROXY_PORT="8443" \
  # -p 8443:8443 \
  # -e SUB_NAME="komari" \
  # 【可选】备份配置
  # -e BACKUP_TIME="0 20 * * *" \
  # -e BACKUP_DAYS="10" \
  # 【可选】Cloudflare 隧道
  # -e KOMARI_ENABLE_CLOUDFLARED="true" \
  # -e KOMARI_CLOUDFLARED_TOKEN="eyJxxxxx" \
  ghcr.io/jyucoeng/komari:latest
```

**端口暴露规则**：
- 如果设置了 `ARGO_DOMAIN`，无需暴露 `CADDY_PROXY_PORT`
- 如果未设置 `ARGO_DOMAIN`，需暴露 `CADDY_PROXY_PORT`：`-p 8443:8443`

**访问订阅**：
- 设置了域名：`https://ARGO_DOMAIN:443/UUID`
- 未设置域名：`https://public-ip:CADDY_PROXY_PORT/UUID`

## 备份和还原

### 自动备份

根据 `BACKUP_TIME` 环境变量自动定时备份，备份数据包括面板配置、主题设置、服务器列表等。

### 手动操作

```bash
# 手动备份
docker exec komari /app/komari_bak.sh bak

# 手动还原
docker stop komari
docker exec komari /app/komari_bak.sh res
docker start komari
```

## 进程管理

使用 Supervisor 管理后台进程（cron、komari、caddy）。如果某个进程意外退出会自动重启。

## 节点订阅工作原理

1. 容器启动时检查 `UUID`
2. 如果设置了 UUID，生成 Caddyfile 并启动 Caddy 反代
3. 调用 sub_link.sh 生成 VLESS 和 VMESS 订阅链接
4. 订阅链接保存到 `/tmp/list.log`
5. 客户端可通过 Caddy 反代访问订阅文件

**支持的协议**：
- VLESS（WebSocket + TLS）
- VMESS（WebSocket + TLS）

## 使用 Docker Compose

```bash
docker compose up -d
```

## 原始项目

- https://github.com/yutian81/komari-backup
