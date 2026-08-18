# Project Memory (Compact)

1) 仓库：fork `xiaolitongxue666/Kavita`，分支 `develop` 跟踪 `origin/develop`；本地 `e:/Code/my_code/Vps/kavita`，VPS `/home/ubuntu/Code/VPS/kavita`。
2) VPS 部署：`deploy/vps/` Docker 栈，镜像 `jvmilazz0/kavita:latest`，loopback `127.0.0.1:5000`，nginx 反代 `https://xiaolitongxue.com.cn/kavita/`，BaseUrl 必须 `/kavita/`。
3) 书库：`deploy/vps/books/<Series>/file.epub|pdf`（容器 `/books` 只读挂载）；EPUB 优先（精读/沉浸式翻译）；`.gitignore` 忽略 `books/**`（保留 `.gitkeep`），本地与 VPS 需手动同步。
4) 已入库：《毛泽东选集.epub》（2026-08-18，源 M0rtzz/Selected-Works-of-MaoTseTung，GitHub 需走代理 127.0.0.1:7890）。
5) 运维：kavita.db 属主 root（容器以 root 运行）→ 宿主写库需 `sudo sqlite3`；WAL 模式可不停容器写。
6) 无 UI 凭证时：API key 明文存 `AppUserAuthKey.Key`（6–32 字母数字，Provider=1），可插入临时 key 调 `POST /api/Library/scan?libraryId=1&force=true`（Books 库 id=1），验证用 `GET /api/Opds/{key}/libraries/1`；key 经 nginx 公网可达，用完删除。
7) 换书流程：替换文件 → 扫描 API（或 UI）→ 浏览器强刷；探针 `curl --noproxy '*'`。
8) 本地忽略：`.codegraph/`（CodeGraph 索引）、`undefined/`（node-compile-cache 意外产物，已清理并加入 .gitignore）。
9) 凭证备忘 `deploy/vps/admin-credentials.txt`（勿提交，当前不存在）。
