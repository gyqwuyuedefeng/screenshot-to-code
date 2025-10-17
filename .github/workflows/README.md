# Screenshot-to-Code Docker 构建 Workflow

这个 GitHub Actions workflow 用于自动构建和推送 screenshot-to-code 项目的 Docker 镜像到阿里云容器镜像仓库。

## 项目架构

screenshot-to-code 是一个前后端分离的项目：
- **Backend**: FastAPI (Python) + Poetry，端口 7001
- **Frontend**: React + Vite，端口 5173

## 触发条件

Workflow 会在以下情况下自动触发：

1. **手动触发**: 在 GitHub Actions 页面手动运行
2. **推送 tag**: 推送格式为 `v*` 的标签（如 `v1.0.0`）
3. **推送到分支**: 推送到 `main` 或 `dev` 分支
4. **文件变更**: 以下文件发生变更时：
   - `backend/**`
   - `frontend/**`
   - `.github/workflows/docker-build.yml`
   - `docker-compose.yml`

## 配置 GitHub Secrets

在使用前，需要在 GitHub 仓库设置以下 Secrets：

### 必需的 Secrets

| Secret 名称 | 说明 | 示例 |
|------------|------|------|
| `ALIYUN_DOCKER_REGISTRY` | 阿里云容器镜像仓库地址 | `registry.cn-hangzhou.aliyuncs.com` |
| `ALIYUN_DOCKER_USER` | 阿里云容器镜像仓库用户名 | `your-username` |
| `ALIYUN_DOCKER_TOKEN` | 阿里云容器镜像仓库密码/Token | `your-password` |

### 如何配置 Secrets

1. 进入 GitHub 仓库
2. 点击 `Settings` > `Secrets and variables` > `Actions`
3. 点击 `New repository secret`
4. 添加上述三个 secrets

## 构建产物

### 镜像命名规则

- **Backend 镜像**: `{ALIYUN_DOCKER_REGISTRY}/gyq_personal/screenshot-to-code-backend`
- **Frontend 镜像**: `{ALIYUN_DOCKER_REGISTRY}/gyq_personal/screenshot-to-code-frontend`

例如，如果你的 `ALIYUN_DOCKER_REGISTRY` 是 `registry.cn-hangzhou.aliyuncs.com`，则镜像地址为：
- `registry.cn-hangzhou.aliyuncs.com/gyq_personal/screenshot-to-code-backend`
- `registry.cn-hangzhou.aliyuncs.com/gyq_personal/screenshot-to-code-frontend`

### 标签策略

每次构建会生成两个标签：
- `latest`: 最新版本
- `{VERSION}`: 具体版本号
  - Tag 触发: 使用 tag 名称（如 `v1.0.0`）
  - 分支触发: 使用格式 `1.0-SNAPSHOT-{commit-sha}`

## 使用构建的镜像

### 1. 拉取镜像

```bash
# 拉取最新版本
docker pull registry.cn-hangzhou.aliyuncs.com/gyq_personal/screenshot-to-code-backend:latest
docker pull registry.cn-hangzhou.aliyuncs.com/gyq_personal/screenshot-to-code-frontend:latest

# 拉取指定版本
docker pull registry.cn-hangzhou.aliyuncs.com/gyq_personal/screenshot-to-code-backend:v1.0.0
docker pull registry.cn-hangzhou.aliyuncs.com/gyq_personal/screenshot-to-code-frontend:v1.0.0
```

### 2. 使用 docker-compose 启动

创建 `.env` 文件：

```bash
# 必需：配置 API Keys
OPENAI_API_KEY=sk-your-openai-key
ANTHROPIC_API_KEY=your-anthropic-key

# 可选：配置后端端口
BACKEND_PORT=7001
```

创建 `docker-compose.yml`：

```yaml
version: '3.9'

services:
  backend:
    image: registry.cn-hangzhou.aliyuncs.com/gyq_personal/screenshot-to-code-backend:latest
    env_file:
      - .env
    ports:
      - "${BACKEND_PORT:-7001}:${BACKEND_PORT:-7001}"
    command: poetry run uvicorn main:app --host 0.0.0.0 --port ${BACKEND_PORT:-7001}

  frontend:
    image: registry.cn-hangzhou.aliyuncs.com/gyq_personal/screenshot-to-code-frontend:latest
    ports:
      - "5173:5173"
    depends_on:
      - backend
```

启动服务：

```bash
docker-compose up -d
```

访问 http://localhost:5173 使用应用。

### 3. 单独启动容器

```bash
# 启动 Backend
docker run -d \
  --name screenshot-backend \
  -p 7001:7001 \
  -e OPENAI_API_KEY=sk-your-key \
  -e ANTHROPIC_API_KEY=your-key \
  registry.cn-hangzhou.aliyuncs.com/gyq_personal/screenshot-to-code-backend:latest \
  poetry run uvicorn main:app --host 0.0.0.0 --port 7001

# 启动 Frontend
docker run -d \
  --name screenshot-frontend \
  -p 5173:5173 \
  --link screenshot-backend:backend \
  registry.cn-hangzhou.aliyuncs.com/gyq_personal/screenshot-to-code-frontend:latest
```

## API Keys 获取方式

### OpenAI API Key

1. 访问 https://platform.openai.com/api-keys
2. 创建新的 API Key
3. 确保账号有 GPT-4o 访问权限

### Anthropic API Key

1. 访问 https://console.anthropic.com/
2. 创建新的 API Key
3. 推荐使用 Claude Sonnet 3.7（效果最佳）

## Workflow 功能特性

### 磁盘空间优化

- 构建前清理大型预装软件（.NET、Android SDK 等）
- 使用 Docker 缓存优化
- 构建后自动清理

### 构建优化

- 使用 Docker Buildx 多平台构建
- GitHub Actions 缓存加速
- 120 分钟超时保护

### 错误处理

- 自动检查必需的 Secrets
- 构建失败时尝试手动推送
- 详细的资源监控和日志

## 本地测试 Workflow

在推送前，可以本地测试 Docker 构建：

```bash
# 测试 Backend 构建
cd backend
docker build -t screenshot-backend:test .

# 测试 Frontend 构建
cd frontend
docker build -t screenshot-frontend:test .

# 测试完整 docker-compose
docker-compose build
```

## 故障排查

### 构建失败

1. 检查 Secrets 是否正确配置
2. 查看 Actions 日志中的错误信息
3. 检查阿里云镜像仓库权限

### 镜像无法拉取

1. 确认已登录阿里云镜像仓库：
   ```bash
   docker login registry.cn-hangzhou.aliyuncs.com
   ```
2. 检查镜像名称和标签是否正确
3. 确认镜像已成功推送

### 应用无法启动

1. 检查 `.env` 文件中的 API Keys 是否有效
2. 查看容器日志：
   ```bash
   docker logs screenshot-backend
   docker logs screenshot-frontend
   ```
3. 确认端口未被占用

## 版本发布流程

### 开发版本发布

```bash
git add .
git commit -m "feat: add new feature"
git push origin dev
```

### 正式版本发布

```bash
# 创建并推送 tag
git tag -a v1.0.0 -m "Release version 1.0.0"
git push origin v1.0.0
```

## 支持的 AI 模型

- **Claude Sonnet 3.7** - 推荐，效果最佳（准确度 70.31%）
- **GPT-4o** - 同样推荐
- **DALL-E 3 / Flux Schnell** - 用于图像生成

## 输出格式支持

- HTML + Tailwind
- HTML + CSS
- React + Tailwind
- Vue + Tailwind
- Bootstrap
- Ionic + Tailwind
- SVG

## 性能监控

每次构建会输出：
- CPU 和内存信息
- 磁盘空间使用情况
- Docker 镜像大小
- 构建时间

## 相关链接

- 原项目: https://github.com/abi/screenshot-to-code
- 托管版本: https://screenshottocode.com
- 问题反馈: 在本仓库提交 Issue

## 许可证

遵循原项目 screenshot-to-code 的许可证。
