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
- 布局：`books/<Series>/file.epub|pdf`，已入库示例：`books/毛泽东选集/毛泽东选集.epub`（源：[M0rtzz/Selected-Works-of-MaoTseTung](https://github.com/M0rtzz/Selected-Works-of-MaoTseTung)）
- **精读 / 沉浸式翻译：优先 EPUB**（HTML DOM）。PDF 用 pdf.js，目录或可译、**正文通常不可译**。
- 勿把 Paperless `originals/` 哈希目录当 library 根。
- 换 EPUB 后：替换文件 → `POST /api/Library/scan?libraryId=<id>`（或 UI 扫描）→ 浏览器强刷。

### 扫描 / 验证（无 UI 凭证时）

Kavita API 需 API key（`Kavita-APIKey` header 或 `?apiKey=` query）。admin 若未生成过 key，可直接在数据库插入临时 key（明文，6–32 位字母数字，存 `AppUserAuthKey.Key`）：

```bash
# 容器以 root 运行 → kavita.db 属主 root，宿主写库需 sudo；WAL 模式，无需停容器
cd /home/ubuntu/Code/VPS/kavita/deploy/vps/config
KEY=$(openssl rand -hex 16 | head -c 32)
sudo sqlite3 kavita.db "INSERT INTO AppUserAuthKey (Key, Name, CreatedAtUtc, Provider, AppUserId) VALUES ('$KEY', 'scan-tmp', '2026-08-18 01:50:00', 1, 1);"
# 触发全量扫描
curl --noproxy '*' -X POST "http://127.0.0.1:5000/api/Library/scan?libraryId=1&apiKey=$KEY&force=true"
# 验证入库（OPDS，无需登录）
curl --noproxy '*' -s "http://127.0.0.1:5000/api/Opds/$KEY/libraries/1" | grep -oE '<title>[^<]*</title>'
# 用完删除临时 key
sudo sqlite3 kavita.db "DELETE FROM AppUserAuthKey WHERE Name='scan-tmp';"
```

`Provider=1` 即用户 API key；`libraryId` 见 `Library` 表（Books 库为 1）。此 key 经 nginx 公网可达，用完务必删除。

## 密码

`admin-credentials.txt` 只是备忘，**不**被容器读取。改密请用 Kavita 账号设置；应急可停容器后改 SQLite `AspNetUsers.PasswordHash`（ASP.NET Identity V3），再同步 txt。

## 与 Paperless 分工

| Paperless | Kavita |
|-----------|--------|
| 归档 / OCR / 搜索 | 阅读进度 / EPUB 双语 |

## 约束

- 镜像：`jvmilazz0/kavita:latest`；`mem_limit: 512m`
- 勿并入 RSS 栈；探针：`curl --noproxy '*'`
