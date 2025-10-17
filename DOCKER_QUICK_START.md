# Screenshot-to-Code Docker 快速启动指南

## 前置条件

1. 已在 GitHub 仓库配置 Secrets：
   - `ALIYUN_DOCKER_USER`
   - `ALIYUN_DOCKER_TOKEN`

2. 已推送代码触发 GitHub Actions 构建

## 方式一：使用 docker-compose（推荐）

### 1. 创建 `.env` 文件

```bash
# 在项目根目录创建 .env 文件
cat > .env << 'EOF'
# 必需：配置至少一个 AI 模型的 API Key
OPENAI_API_KEY=sk-your-openai-api-key-here
ANTHROPIC_API_KEY=your-anthropic-api-key-here

# 可选：配置后端端口（默认 7001）
BACKEND_PORT=7001
EOF
```

### 2. 创建 `docker-compose.yml`

```bash
cat > docker-compose.yml << 'EOF'
version: '3.9'

services:
  backend:
    image: registry.cn-hangzhou.aliyuncs.com/gyq_personal/screenshot-to-code-backend:latest
    container_name: screenshot-backend
    env_file:
      - .env
    ports:
      - "${BACKEND_PORT:-7001}:${BACKEND_PORT:-7001}"
    command: poetry run uvicorn main:app --host 0.0.0.0 --port ${BACKEND_PORT:-7001}
    restart: unless-stopped

  frontend:
    image: registry.cn-hangzhou.aliyuncs.com/gyq_personal/screenshot-to-code-frontend:latest
    container_name: screenshot-frontend
    ports:
      - "5173:5173"
    depends_on:
      - backend
    restart: unless-stopped
EOF
```

### 3. 启动服务

```bash
# 拉取最新镜像
docker-compose pull

# 启动服务
docker-compose up -d

# 查看日志
docker-compose logs -f

# 停止服务
docker-compose down
```

### 4. 访问应用

打开浏览器访问：http://localhost:5173

## 方式二：单独运行容器

### 1. 启动 Backend

```bash
docker run -d \
  --name screenshot-backend \
  --restart unless-stopped \
  -p 7001:7001 \
  -e OPENAI_API_KEY=sk-your-key \
  -e ANTHROPIC_API_KEY=your-key \
  registry.cn-hangzhou.aliyuncs.com/gyq_personal/screenshot-to-code-backend:latest \
  poetry run uvicorn main:app --host 0.0.0.0 --port 7001
```

### 2. 启动 Frontend

```bash
docker run -d \
  --name screenshot-frontend \
  --restart unless-stopped \
  -p 5173:5173 \
  --link screenshot-backend:backend \
  registry.cn-hangzhou.aliyuncs.com/gyq_personal/screenshot-to-code-frontend:latest
```

### 3. 查看日志

```bash
# Backend 日志
docker logs -f screenshot-backend

# Frontend 日志
docker logs -f screenshot-frontend
```

## API Keys 获取

### OpenAI API Key（推荐 GPT-4o）

1. 访问：https://platform.openai.com/api-keys
2. 点击 "Create new secret key"
3. 复制 API Key（格式：`sk-xxxxx`）
4. 确保账号有 GPT-4o 访问权限

### Anthropic API Key（推荐 Claude Sonnet 3.7）

1. 访问：https://console.anthropic.com/
2. 点击 "Get API keys"
3. 创建新的 API Key
4. 复制 API Key

注意：推荐同时配置两个 API Key，以便对比效果。

## 常用命令

### 查看运行状态

```bash
docker ps | grep screenshot
```

### 重启服务

```bash
docker restart screenshot-backend screenshot-frontend
```

### 更新镜像

```bash
# 停止容器
docker stop screenshot-backend screenshot-frontend

# 删除旧容器
docker rm screenshot-backend screenshot-frontend

# 拉取最新镜像
docker pull registry.cn-hangzhou.aliyuncs.com/gyq_personal/screenshot-to-code-backend:latest
docker pull registry.cn-hangzhou.aliyuncs.com/gyq_personal/screenshot-to-code-frontend:latest

# 重新启动（使用上面的启动命令）
```

### 清理资源

```bash
# 停止并删除容器
docker-compose down

# 删除镜像
docker rmi registry.cn-hangzhou.aliyuncs.com/gyq_personal/screenshot-to-code-backend:latest
docker rmi registry.cn-hangzhou.aliyuncs.com/gyq_personal/screenshot-to-code-frontend:latest
```

## 功能测试

### 1. 上传截图

- 打开 http://localhost:5173
- 点击上传按钮或拖拽截图到页面
- 支持的格式：PNG, JPG, JPEG

### 2. 选择输出格式

- HTML + Tailwind
- HTML + CSS
- React + Tailwind
- Vue + Tailwind
- Bootstrap
- Ionic + Tailwind
- SVG

### 3. 选择 AI 模型

- **Claude Sonnet 3.7**（推荐，准确度最高 70.31%）
- GPT-4o（也推荐）

### 4. 生成代码

- 点击 "Generate" 按钮
- 等待 AI 生成代码（通常 10-30 秒）
- 查看生成的代码和预览效果

### 5. 下载代码

- 点击 "Download" 按钮下载完整代码
- 或直接复制代码到编辑器

## 故障排查

### Backend 无法启动

检查日志：
```bash
docker logs screenshot-backend
```

常见问题：
- API Key 未配置或无效
- 端口 7001 被占用

### Frontend 无法访问 Backend

检查网络连接：
```bash
docker exec screenshot-frontend ping backend
```

解决方案：
- 确保 Backend 先启动
- 检查 docker-compose 中的 `depends_on` 配置

### 生成失败

可能原因：
- API Key 额度不足
- 网络连接问题
- 上传的图片格式不支持

### 端口冲突

如果端口被占用，修改 `.env` 文件：
```bash
BACKEND_PORT=7002  # 改为其他端口
```

然后修改 `docker-compose.yml` 中的端口映射。

## 性能优化建议

### 1. 使用特定版本镜像

```yaml
backend:
  image: registry.cn-hangzhou.aliyuncs.com/gyq_personal/screenshot-to-code-backend:v1.0.0
```

### 2. 限制资源使用

```yaml
backend:
  deploy:
    resources:
      limits:
        cpus: '2'
        memory: 2G
```

### 3. 配置日志驱动

```yaml
backend:
  logging:
    driver: "json-file"
    options:
      max-size: "10m"
      max-file: "3"
```

## 镜像版本说明

- `latest`: 最新版本（跟随 main/dev 分支）
- `v1.0.0`: 稳定版本（使用 git tag 发布）
- `1.0-SNAPSHOT-abc123`: 开发版本（commit sha）

## 支持与反馈

如有问题，请查看：
1. GitHub Actions 构建日志
2. Docker 容器日志
3. 原项目文档：https://github.com/abi/screenshot-to-code

## 许可证

遵循原项目 screenshot-to-code 的许可证。
