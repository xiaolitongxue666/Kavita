# Kavita — VPS 部署

独立 Docker 栈：仅监听 `127.0.0.1:5000`，由 [vps_nginx](https://github.com/xiaolitongxue666/vps_nginx) 反代 **`/kavita/`**。

公网：将 `kavita` 加入 `VPS_NGINX_PUBLIC_EXPOSE`，访问  
`https://xiaolitongxue.com.cn/kavita/`（与 `/blog/`、`/freshrss/`、`/paperless/` 同模式）。

## 路径

| 环境 | 路径 |
|------|------|
| 本地 fork | `e:/Code/my_code/Vps/kavita`（工作区） |
| VPS | `/home/ubuntu/Code/VPS/kavita` |
| Compose | `deploy/vps/` |
| 凭证备忘（勿提交） | `deploy/vps/admin-credentials.txt` |

## 启动

```bash
cd /home/ubuntu/Code/VPS/kavita/deploy/vps
docker compose pull && docker compose up -d
```

vps_nginx：`PUBLIC_EXPOSE` 含 `kavita` 后 `sudo -E ./scripts/deploy.sh`。

**Base URL** 必须为 `/kavita/`（可写 `config/appsettings.json` 或 Admin UI）。Nginx **不要** rewrite 剥前缀。

## 书库

- 库类型：**Book**；容器路径 `/books`（`./books:/books:ro`）
- 布局：`books/<Series>/file.epub|pdf`
- **精读 / 沉浸式翻译：优先 EPUB**（HTML DOM）。PDF 用 pdf.js，目录或可译、**正文通常不可译**。
- 勿把 Paperless `originals/` 哈希目录当 library 根。
- 换 EPUB 后：替换文件 → `POST /api/Library/scan?libraryId=<id>`（或 UI 扫描）→ 浏览器强刷。

## 密码

`admin-credentials.txt` 只是备忘，**不**被容器读取。改密请用 Kavita 账号设置；应急可停容器后改 SQLite `AspNetUsers.PasswordHash`（ASP.NET Identity V3），再同步 txt。

## 与 Paperless 分工

| Paperless | Kavita |
|-----------|--------|
| 归档 / OCR / 搜索 | 阅读进度 / EPUB 双语 |

## 约束

- 镜像：`jvmilazz0/kavita:latest`；`mem_limit: 512m`
- 勿并入 RSS 栈；探针：`curl --noproxy '*'`
