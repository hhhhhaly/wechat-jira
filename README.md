# 微信需求采集 + Jira 同步项目启动说明

## 下载完整压缩包

[v0.1.0 脱敏源码包](https://github.com/hhhhhaly/wechat-jira/releases/download/v0.1.0/wechat_jira_github_clean_20261013.zip)

> 该包不包含 `.env`、微信解密数据库、微信 key、SQLite 业务数据库和日志。启动前请按下方说明创建配置文件。

这个包包含两个可启动部分：

1. `wechat_requirement_collector/`：微信需求采集、Obsidian/Jira 同步服务  
2. `JIRA-main/`：本地 Jira 桥接后端和管理前端  
3. `obisidian-ticket/`：示例/当前 Obsidian 工单 Vault

服务地址：

| 服务 | 地址 |
|---|---|
| 微信需求采集器 | http://127.0.0.1:8000 |
| Jira 桥接后端 API | http://127.0.0.1:8311 |
| Jira 管理前端 | http://127.0.0.1:8312 |

> 注意：`.env` / `.env.local` 内可能包含本机 Jira API、数据库、微信或模型密钥配置，请不要外发包。

---

## 1. 环境要求

- macOS
- Python 3.12：用于 `wechat_requirement_collector`
- Python 3.9+：用于 `JIRA-main/backend`
- Node.js 20+ / npm：用于 `JIRA-main/frontend`
- SQLite：Python 自带即可
- 如需采集本机微信数据，需要已登录微信 4.x 并完成 wxkey 初始化

---

## 2. 启动微信需求采集器

```bash
cd wechat_requirement_collector

# 第一次运行：创建虚拟环境并安装依赖
python3.12 -m venv .venv
.venv/bin/pip install -r requirements.txt
```

配置文件：

```text
.env
```

如果包内已经带 `.env`，可以直接使用；如果没有：

```bash
cp .env.example .env
# 然后编辑 .env，填入本机微信、Obsidian、Jira、模型等配置
```

普通启动：

```bash
.venv/bin/python -m uvicorn app.main:app --host 127.0.0.1 --port 8000
```

访问：

```text
http://127.0.0.1:8000
```

如果需要采集本机微信 4.x 数据，先初始化 wxkey：

```bash
./setup_wxkey_mac.command
```

然后使用一键启动脚本，它会刷新解密数据库并启动采集器：

```bash
./start_mac.sh
```

---

## 3. 启动 Jira 桥接后端

```bash
cd JIRA-main/backend

# 第一次运行：创建虚拟环境并安装依赖
python3 -m venv venv
venv/bin/pip install -r requirements.txt
```

Jira 配置主要在：

```text
JIRA-main/.env
```

需要确认这些变量：

```text
JIRA_URL=https://orbischina.atlassian.net
JIRA_EMAIL=...
JIRA_API_TOKEN=...
DATABASE_URL=...
```

启动：

```bash
venv/bin/python -m uvicorn main:app --host 127.0.0.1 --port 8311
```

检查 API：

```text
http://127.0.0.1:8311/openapi.json
```

---

## 4. 启动 Jira 管理前端

```bash
cd JIRA-main/frontend

# 第一次运行：安装依赖
npm install
```

启动：

```bash
npm run dev -- -p 8312
```

访问：

```text
http://127.0.0.1:8312
```

---

## 5. 推荐启动顺序

```text
1. Jira 桥接后端：8311
2. 微信需求采集器：8000
3. Jira 管理前端：8312
```

采集器本身可以先独立启动；只有同步 Jira、更新 ORBIS Consultants、读取 Jira 工单时才依赖 8311。

---

## 6. Obsidian Vault 路径

包内带有：

```text
obisidian-ticket/orbis-wechat
```

当前 `.env` 中使用的是本机绝对路径：

```text
OBSIDIAN_VAULT_DIR=/Users/milu/Downloads/obisidian-ticket/orbis-wechat
```

如果你解压到其他目录，请把 `.env` 里的 `OBSIDIAN_VAULT_DIR` 改成解压后的实际路径，例如：

```text
OBSIDIAN_VAULT_DIR=/path/to/obisidian-ticket/orbis-wechat
```

---

## 7. 数据库

采集器 SQLite 数据库在：

```text
wechat_requirement_collector/wechat_requirements.db
```

程序启动时会自动创建/补齐表结构。  
如果包内已经带有数据库，则会保留当前需求、群配置、映射和状态记录。

---

## 8. 测试

在 `wechat_requirement_collector` 下运行：

```bash
PYTHONPATH=. .venv/bin/pytest -q
```

---

## 9. 常见问题

### 8000 端口被占用

```bash
lsof -iTCP:8000 -sTCP:LISTEN
kill <PID>
```

### 8311 端口被占用

```bash
lsof -iTCP:8311 -sTCP:LISTEN
kill <PID>
```

### 8312 端口被占用

```bash
lsof -iTCP:8312 -sTCP:LISTEN
kill <PID>
```

### 微信采集状态显示未连接

确认：

1. 本机微信已登录；
2. 已运行 `setup_wxkey_mac.command`；
3. `storage/wxkey/keys.json` 存在；
4. `storage/wxkey/decrypted/` 下有解密数据库；
5. 使用 `./start_mac.sh` 启动。

---

## 10. 核心功能入口

- 微信群选择：`http://127.0.0.1:8000`
- 需求列表：`http://127.0.0.1:8000`
- Jira 工作人员账号库：`http://127.0.0.1:8000`
- 状态回填按钮：`http://127.0.0.1:8000`
- 本地 Jira 管理前端：`http://127.0.0.1:8312`
