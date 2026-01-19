# Docker 部署指南

## 快速开始

```bash
# 克隆项目
git clone https://github.com/Li112233ning/gemini_business.git
cd gemini-business2api

# 创建环境配置
cp .env.example .env
# 编辑 .env 设置 ADMIN_KEY

# 构建并启动
docker-compose up -d gemini-api
```

## 服务说明

| 服务 | 说明 | 启动命令 |
|------|------|----------|
| `gemini-api` | API 主服务 | `docker-compose up -d gemini-api` |
| `register` | 账号注册 | `docker-compose --profile register run register` |
| `keeper` | 账号守护 | `docker-compose --profile keeper up -d keeper` |
| **一键启动** | API + 守护 | `docker-compose --profile all up -d` |

## API 服务

```bash
# 启动 API 服务
docker-compose up -d gemini-api

# 查看日志
docker-compose logs -f gemini-api

# 停止服务
docker-compose down
```

访问管理面板：`http://localhost:7860`

## 账号注册

手动注册指定数量的账号：

```bash
# 注册 1 个账号（默认）
docker-compose --profile register run register

# 注册 10 个账号
docker-compose --profile register run -e TOTAL_ACCOUNTS=10 register
```

### 配置项

编辑 `docker-compose.yml` 中的 `register` 服务：

```yaml
environment:
  - TOTAL_ACCOUNTS=1           # 注册数量
  - MAIL_API=https://...       # 邮箱 API 地址
  - MAIL_KEY=your-key          # 邮箱 API 密钥
```

## 账号守护

自动检测账号有效性，保持可用账号数量：

```bash
# 启动守护服务
docker-compose --profile keeper up -d keeper

# 查看日志
docker-compose logs -f keeper

# 停止守护服务
docker-compose --profile keeper down
```

### 配置项

编辑 `docker-compose.yml` 中的 `keeper` 服务：

```yaml
environment:
  - MIN_ACCOUNTS=5       # 最少保持账号数
  - CHECK_INTERVAL=3600  # 检测间隔（秒）
  - MAIL_API=https://... # 邮箱 API 地址
  - MAIL_KEY=your-key    # 邮箱 API 密钥
```

### 守护逻辑

1. 每隔 `CHECK_INTERVAL` 秒检测所有账号
2. 删除失效账号（HTTP 401/403）
3. 可用账号 < `MIN_ACCOUNTS` 时自动注册补充

## 一键启动

同时启动 API 服务和账号守护：

```bash
docker-compose --profile all up -d
```

查看所有服务状态：

```bash
docker-compose --profile all ps
```

停止所有服务：

```bash
docker-compose --profile all down
```

## 配置说明

### 环境变量（.env）

```ini
# 管理员密钥（必需）
ADMIN_KEY=your-admin-key

# API 密钥（可选）
API_KEY=your-api-key

# 路径前缀（可选，用于隐藏端点）
PATH_PREFIX=your-random-prefix
```

## API 使用示例

```bash
curl http://localhost:7860/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer your-api-key" \
  -d '{
    "model": "gemini-2.5-flash",
    "messages": [{"role": "user", "content": "Hello"}]
  }'
```

## 支持的模型

- `gemini-auto`
- `gemini-2.5-flash`
- `gemini-2.5-pro`
- `gemini-3-flash-preview`
- `gemini-3-pro-preview`

## 目录结构

```
gemini-business2api/
├── main.py              # API 主程序
├── Dockerfile           # Docker 镜像
├── docker-compose.yml   # Docker Compose 配置
├── .env.example         # 环境变量示例
├── data/                # 数据目录（持久化）
│   ├── accounts/        # 账号配置
│   └── settings.yaml    # 系统设置
└── script/
    ├── register_accounts.py  # 账号注册脚本
    └── account_keeper.py     # 账号守护脚本
```
