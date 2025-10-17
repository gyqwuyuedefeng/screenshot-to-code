# Screenshot-to-Code 部署与使用指南

## 📦 部署步骤

### 方式一：使用 Docker Compose（推荐）

#### 1. 创建工作目录

```bash
mkdir screenshot-to-code-deploy
cd screenshot-to-code-deploy
```

#### 2. 创建 `.env` 配置文件

```bash
cat > .env << 'EOF'
# ========================================
# AI 模型 API Keys（必需配置至少一个）
# ========================================

# OpenAI API Key（推荐 GPT-4o）
OPENAI_API_KEY=sk-your-openai-api-key-here

# Anthropic API Key（推荐 Claude Sonnet 3.7，效果最佳）
ANTHROPIC_API_KEY=your-anthropic-api-key-here

# ========================================
# 可选配置
# ========================================

# 后端端口（默认 7001）
BACKEND_PORT=7001

# OpenAI 代理地址（如果需要代理访问）
# OPENAI_BASE_URL=https://your-proxy.com/v1

# 图像生成模型（DALL-E 3 或 Replicate）
# IMAGE_GENERATION_MODEL=dall-e-3
EOF
```

#### 3. 创建 `docker-compose.yml`

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
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:${BACKEND_PORT:-7001}/"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

  frontend:
    image: registry.cn-hangzhou.aliyuncs.com/gyq_personal/screenshot-to-code-frontend:latest
    container_name: screenshot-frontend
    ports:
      - "5173:5173"
    depends_on:
      backend:
        condition: service_healthy
    restart: unless-stopped
    environment:
      - VITE_WS_BACKEND_URL=ws://localhost:${BACKEND_PORT:-7001}
      - VITE_HTTP_BACKEND_URL=http://localhost:${BACKEND_PORT:-7001}

volumes:
  backend-data:
EOF
```

#### 4. 启动服务

```bash
# 拉取最新镜像
docker-compose pull

# 启动服务（后台运行）
docker-compose up -d

# 查看日志
docker-compose logs -f

# 查看运行状态
docker-compose ps
```

#### 5. 访问应用

打开浏览器访问：**http://localhost:5173**

---

## 🔑 修改 API Key

### 方式1：修改 .env 文件（推荐）

```bash
# 1. 编辑 .env 文件
vim .env  # 或使用其他编辑器

# 2. 修改 API Key
OPENAI_API_KEY=sk-your-new-key
ANTHROPIC_API_KEY=your-new-key

# 3. 保存后重启服务
docker-compose restart

# 4. 验证新配置是否生效
docker-compose logs backend | grep -i "api"
```

### 方式2：通过 Web UI 配置（临时生效）

1. 打开应用：http://localhost:5173
2. 点击右上角的 **设置图标（⚙️）**
3. 在设置面板中输入 API Keys：
   - **OpenAI API Key**
   - **Anthropic API Key**
4. 点击保存

**注意**：通过 Web UI 配置的 API Key 仅在当前浏览器会话中有效，刷新页面后需要重新配置。

### 方式3：环境变量注入（适合 Docker 单容器）

```bash
# 停止现有容器
docker stop screenshot-backend

# 使用新的 API Key 启动
docker run -d \
  --name screenshot-backend \
  -p 7001:7001 \
  -e OPENAI_API_KEY=sk-new-key \
  -e ANTHROPIC_API_KEY=new-anthropic-key \
  registry.cn-hangzhou.aliyuncs.com/gyq_personal/screenshot-to-code-backend:latest \
  poetry run uvicorn main:app --host 0.0.0.0 --port 7001
```

---

## 🚀 使用指南

### 1. 上传截图

支持三种方式：

**方式A：拖拽上传**
- 直接将截图拖拽到浏览器窗口

**方式B：点击上传**
- 点击上传按钮
- 选择本地图片文件

**方式C：粘贴上传**
- 截图后使用 `Ctrl+V`（或 `Cmd+V`）粘贴

支持格式：PNG, JPG, JPEG, WEBP

### 2. 选择 AI 模型

在界面上选择 AI 模型：

- **Claude Sonnet 3.7** ⭐ 推荐，准确度最高（70.31%）
- **GPT-4o** - 也推荐，速度快
- **Claude Sonnet 3.5** - 备选方案

### 3. 选择输出格式

支持多种代码格式：

- **HTML + Tailwind** - 最流行
- **HTML + CSS** - 纯 CSS 样式
- **React + Tailwind** - React 组件
- **Vue + Tailwind** - Vue 组件
- **Bootstrap** - Bootstrap 框架
- **Ionic + Tailwind** - 移动端框架
- **SVG** - 矢量图形

### 4. 生成代码

1. 点击 **"Generate Code"** 按钮
2. 等待 AI 生成（通常 10-30 秒）
3. 查看生成的代码和实时预览

### 5. 优化和调整

如果生成结果不满意：

- 点击 **"Regenerate"** 重新生成
- 使用 **"Refine"** 功能微调
- 在输入框中添加额外的描述或要求

### 6. 下载代码

- 点击 **"Download"** 按钮
- 或直接复制代码到编辑器

---

## 🔧 高级配置

### 配置 OpenAI 代理

如果无法直接访问 OpenAI API，可以配置代理：

```bash
# 编辑 .env 文件
OPENAI_BASE_URL=https://your-proxy-domain.com/v1
```

常见代理服务：
- Cloudflare Workers 代理
- API2D（中国）
- OpenAI-Forward

### 配置图像生成

```bash
# 使用 DALL-E 3（需要 OpenAI API Key）
IMAGE_GENERATION_MODEL=dall-e-3

# 或使用 Replicate Flux Schnell（需要 Replicate API Key）
IMAGE_GENERATION_MODEL=flux-schnell
REPLICATE_API_TOKEN=your-replicate-token
```

### 配置后端端口

```bash
# 修改 .env
BACKEND_PORT=8001

# 修改 docker-compose.yml
ports:
  - "8001:8001"

# 重启服务
docker-compose up -d
```

---

## 📊 监控和日志

### 查看实时日志

```bash
# 查看所有服务日志
docker-compose logs -f

# 仅查看后端日志
docker-compose logs -f backend

# 仅查看前端日志
docker-compose logs -f frontend

# 查看最近 100 行日志
docker-compose logs --tail=100
```

### 查看服务状态

```bash
# 查看容器状态
docker-compose ps

# 查看资源使用
docker stats screenshot-backend screenshot-frontend

# 查看健康检查
docker inspect screenshot-backend | grep -A 10 Health
```

### 查看 API 调用情况

```bash
# 查看后端日志中的 API 调用
docker-compose logs backend | grep -i "openai\|anthropic"
```

---

## 🛠️ 常用命令

### 服务管理

```bash
# 启动服务
docker-compose up -d

# 停止服务
docker-compose stop

# 重启服务
docker-compose restart

# 停止并删除容器
docker-compose down

# 停止并删除容器、网络、卷
docker-compose down -v
```

### 更新镜像

```bash
# 拉取最新镜像
docker-compose pull

# 重新创建容器
docker-compose up -d --force-recreate

# 或者一步到位
docker-compose pull && docker-compose up -d --force-recreate
```

### 调试命令

```bash
# 进入后端容器
docker exec -it screenshot-backend /bin/bash

# 进入前端容器
docker exec -it screenshot-frontend /bin/sh

# 测试后端 API
curl http://localhost:7001/

# 测试网络连接
docker exec screenshot-frontend ping backend
```

---

## ❓ 常见问题

### 1. API Key 无效

**症状**：生成失败，提示 API Key 错误

**解决方案**：
```bash
# 检查 .env 文件中的 API Key
cat .env | grep API_KEY

# 验证 API Key 是否正确
# OpenAI API Key 格式：sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
# Anthropic API Key 格式：sk-ant-xxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# 更新后重启
docker-compose restart
```

### 2. 无法访问服务

**症状**：浏览器无法打开 http://localhost:5173

**解决方案**：
```bash
# 检查容器状态
docker-compose ps

# 检查端口占用
netstat -tuln | grep 5173
netstat -tuln | grep 7001

# 查看日志
docker-compose logs frontend
docker-compose logs backend

# 重启服务
docker-compose restart
```

### 3. 生成速度慢

**原因**：
- API 服务器响应慢
- 网络延迟
- 图片过大

**解决方案**：
- 使用 GPT-4o（速度更快）
- 压缩图片大小（建议小于 2MB）
- 配置代理服务器

### 4. 前端无法连接后端

**症状**：前端界面加载正常，但无法生成代码

**解决方案**：
```bash
# 检查后端健康状态
curl http://localhost:7001/

# 检查前端配置
docker exec screenshot-frontend env | grep BACKEND

# 确保网络连通
docker exec screenshot-frontend ping backend

# 重启服务
docker-compose restart
```

### 5. 内存不足

**症状**：容器频繁重启或 OOM（Out of Memory）

**解决方案**：
```yaml
# 在 docker-compose.yml 中限制资源
services:
  backend:
    deploy:
      resources:
        limits:
          memory: 2G
        reservations:
          memory: 512M
```

---

## 🔐 安全建议

### 1. 保护 API Keys

```bash
# 确保 .env 文件权限正确
chmod 600 .env

# 添加到 .gitignore
echo ".env" >> .gitignore
```

### 2. 限制访问

如果部署在公网服务器：

```yaml
# 仅允许本地访问
services:
  frontend:
    ports:
      - "127.0.0.1:5173:5173"
  backend:
    ports:
      - "127.0.0.1:7001:7001"
```

### 3. 使用环境隔离

生产环境建议使用反向代理（Nginx）+ HTTPS。

---

## 📈 性能优化

### 1. 使用固定版本镜像

```yaml
# 生产环境使用固定版本
backend:
  image: registry.cn-hangzhou.aliyuncs.com/gyq_personal/screenshot-to-code-backend:v1.0.0
```

### 2. 配置日志轮转

```yaml
services:
  backend:
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

### 3. 开启健康检查

已在示例的 `docker-compose.yml` 中包含。

---

## 📞 支持

如有问题：
1. 查看日志：`docker-compose logs -f`
2. 检查 GitHub Actions 构建状态
3. 查看原项目文档：https://github.com/abi/screenshot-to-code
4. 提交 Issue 到项目仓库

---

## 📝 快速参考

### API Keys 获取

| 服务 | 获取地址 | 说明 |
|------|---------|------|
| OpenAI | https://platform.openai.com/api-keys | 需要 GPT-4o 权限 |
| Anthropic | https://console.anthropic.com/ | Claude Sonnet 3.7 |

### 默认端口

| 服务 | 端口 | 说明 |
|------|-----|------|
| Frontend | 5173 | Web UI 访问地址 |
| Backend | 7001 | API 服务端口 |

### 常用 URL

| 用途 | URL |
|------|-----|
| Web 应用 | http://localhost:5173 |
| API 根路径 | http://localhost:7001/ |
| API 文档 | http://localhost:7001/docs |
