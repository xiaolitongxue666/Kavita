# Kavita — VPS 部署

独立 Docker 栈：仅监听 `127.0.0.1:5000`，由 [vps_nginx](https://github.com/xiaolitongxue666/vps_nginx) 反代 **`/kavita/`**。

公网：将 `kavita` 加入 `VPS_NGINX_PUBLIC_EXPOSE`，访问  
`https://xiaolitongxue.com.cn/kavita/`（与 `/blog/`、`/freshrss/`、`/paperless/` 同模式，非独立子域名）。

## 路径

| 环境 | 路径 |
|------|------|
| VPS | `/home/ubuntu/Code/VPS/kavita` |
| Compose | `deploy/vps/` |

## 启动

```bash
cd /home/ubuntu/Code/VPS/kavita/deploy/vps
docker compose pull && docker compose up -d
```

vps_nginx：`PUBLIC_EXPOSE` 含 `kavita` 后 `sudo -E ./scripts/deploy.sh`。

首次登录后，在 Kavita UI 将 **Base URL** 设为 `/kavita/`。

## 书库

- 库类型：**Book**
- 宿主机/容器目录：`/books`（compose 挂载 `./books:/books:ro`）
- 系列布局：`books/<Series>/file.pdf|epub`

## 与 Paperless 分工

- **Paperless**：文档归档
- **Kavita**：阅读

## 约束

- 勿并入 RSS 栈。
- 探测：`curl --noproxy '*'`。
