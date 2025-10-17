# Screenshot-to-Code 使用说明

## 📦 快速启动（构建完成后）

### 1. 拉取镜像

```bash
# 登录阿里云镜像仓库
docker login --username=goto@1958399857494618 registry.cn-hangzhou.aliyuncs.com

# 拉取后端镜像
docker pull registry.cn-hangzhou.aliyuncs.com/gyq_personal/screenshot-to-code-backend:latest

# 拉取前端镜像
docker pull registry.cn-hangzhou.aliyuncs.com/gyq_personal/screenshot-to-code-frontend:latest
```

### 2. 创建配置文件

**创建 `.env` 文件**（必需）：

```bash
cat > .env << 'EOF'
# OpenAI API Key（用于 GPT-4o 模型）
OPENAI_API_KEY=sk-your-openai-key-here

# Anthropic API Key（用于 Claude Sonnet 3.7 模型）
ANTHROPIC_API_KEY=your-anthropic-key-here

# 后端端口（可选，默认 7001）
BACKEND_PORT=7001
EOF
```

**创建 `docker-compose.yml` 文件**：

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
# 启动服务（后台运行）
docker-compose up -d

# 查看运行状态
docker-compose ps

# 查看日志
docker-compose logs -f
```

### 4. 访问应用

打开浏览器访问：**http://localhost:5173**

---

## 🤖 如何选择和切换 AI 模型

### 方法1：在 Web UI 中选择（推荐）

1. 打开 http://localhost:5173
2. 找到 **"Select Model"** 下拉框（通常在界面上方）
3. 选择你想使用的模型：
   - **Claude Sonnet 3.7** ⭐ 推荐（准确度最高：70.31%）
   - **GPT-4o** ⭐ 推荐（速度快）
   - **Claude Sonnet 3.5**（备选）
4. 上传截图，点击 **Generate**

### 方法2：通过 API Key 限制可用模型

如果只配置一个 API Key，系统只能使用对应的模型：

```bash
# 编辑 .env 文件
vim .env

# 只使用 Claude 模型
ANTHROPIC_API_KEY=your-key
# OPENAI_API_KEY=  # 不配置或注释掉

# 或只使用 GPT-4o 模型
OPENAI_API_KEY=sk-your-key
# ANTHROPIC_API_KEY=  # 不配置或注释掉

# 保存后重启
docker-compose restart
```

---

## 🔑 获取 API Keys

### OpenAI API Key

1. 访问：https://platform.openai.com/api-keys
2. 登录账号
3. 点击 **"Create new secret key"**
4. 复制 API Key（格式：`sk-xxxxxxxx...`）
5. 确保账号有 **GPT-4o** 访问权限

### Anthropic API Key

1. 访问：https://console.anthropic.com/
2. 登录账号
3. 点击 **"Get API keys"** 或 **"Create Key"**
4. 复制 API Key（格式：`sk-ant-xxxxxxxx...`）
5. Claude Sonnet 3.7 是默认推荐模型

---

## 🎨 使用应用

### 1. 上传截图

支持三种方式：
- **拖拽**：直接将图片拖到浏览器窗口
- **点击上传**：点击上传按钮选择文件
- **粘贴**：截图后按 `Ctrl+V`（Mac: `Cmd+V`）

支持格式：PNG、JPG、JPEG、WEBP

### 2. 选择输出格式

在界面上选择代码格式：
- **HTML + Tailwind** ⭐ 最流行
- **HTML + CSS** - 传统 CSS
- **React + Tailwind** - React 组件
- **Vue + Tailwind** - Vue 组件
- **Bootstrap** - Bootstrap 框架
- **Ionic + Tailwind** - 移动端
- **SVG** - 矢量图

### 3. 生成代码

1. 点击 **"Generate Code"** 按钮
2. 等待 10-30 秒（视模型和复杂度）
3. 查看生成的代码和预览

### 4. 优化结果

如果效果不满意：
- 点击 **"Regenerate"** 重新生成
- 使用 **"Refine"** 微调
- 添加更详细的描述

### 5. 下载代码

- 点击 **"Download"** 下载完整代码
- 或直接复制代码到编辑器

---

## 🔧 修改配置

### 更改 API Keys

```bash
# 1. 编辑 .env 文件
vim .env

# 2. 修改 API Key
OPENAI_API_KEY=sk-new-key
ANTHROPIC_API_KEY=new-key

# 3. 重启服务
docker-compose restart

# 4. 验证（查看后端日志）
docker-compose logs backend | tail -20
```

### 临时配置（Web UI）

1. 打开 http://localhost:5173
2. 点击右上角 **齿轮图标 ⚙️**
3. 在设置中输入 API Keys
4. 点击保存

**注意**：Web UI 配置仅在当前浏览器会话有效。

### 配置 OpenAI 代理

如果无法直接访问 OpenAI API：

```bash
# 在 .env 中添加
OPENAI_BASE_URL=https://your-proxy.com/v1
```

---

## 📊 模型对比

| 模型 | 准确度 | 速度 | 代码完整度 | 推荐场景 |
|------|--------|------|-----------|----------|
| **Claude Sonnet 3.7** | ⭐⭐⭐⭐⭐ 70.31% | 中等 | 高（少占位符） | 复杂布局，精确还原 |
| **GPT-4o** | ⭐⭐⭐⭐ ~68% | 快 | 中 | 快速原型，简单页面 |
| **Claude Sonnet 3.5** | ⭐⭐⭐ 65% | 中等 | 中 | 成本敏感项目 |

**建议**：
- 首选 **Claude Sonnet 3.7**（效果最佳）
- 追求速度选 **GPT-4o**
- 两个都配置，对比效果后选择

---

## 🛠️ 常用命令

### 查看状态

```bash
# 查看容器状态
docker-compose ps

# 查看实时日志
docker-compose logs -f

# 仅查看后端日志
docker-compose logs -f backend

# 仅查看前端日志
docker-compose logs -f frontend
```

### 重启服务

```bash
# 重启所有服务
docker-compose restart

# 仅重启后端
docker-compose restart backend

# 仅重启前端
docker-compose restart frontend
```

### 更新镜像

```bash
# 拉取最新镜像
docker-compose pull

# 重新创建容器
docker-compose up -d --force-recreate

# 查看镜像版本
docker images | grep screenshot-to-code
```

### 停止服务

```bash
# 停止服务
docker-compose stop

# 停止并删除容器
docker-compose down

# 停止并删除所有数据
docker-compose down -v
```

---

## ❓ 常见问题

### 1. API Key 无效

**症状**：生成失败，提示 API Key 错误

**解决**：
```bash
# 检查 API Key 格式
cat .env | grep API_KEY

# OpenAI 格式：sk-proj-xxxxxx 或 sk-xxxxxx
# Anthropic 格式：sk-ant-xxxxxx

# 验证 API Key 有效性
# OpenAI: 访问 https://platform.openai.com/account/usage
# Anthropic: 访问 https://console.anthropic.com/settings/keys

# 更新后重启
docker-compose restart
```

### 2. 无法访问服务

**症状**：浏览器无法打开 http://localhost:5173

**解决**：
```bash
# 检查容器状态
docker-compose ps

# 检查端口占用
netstat -tuln | grep 5173
netstat -tuln | grep 7001

# 查看详细日志
docker-compose logs

# 重启服务
docker-compose restart
```

### 3. 生成速度慢

**原因**：
- API 服务器响应慢
- 网络延迟
- 图片过大

**解决**：
- 切换到 GPT-4o（速度更快）
- 压缩图片（建议 < 2MB）
- 配置代理加速访问

### 4. 前端无法连接后端

**解决**：
```bash
# 检查后端健康
curl http://localhost:7001/

# 测试网络连通
docker exec screenshot-frontend ping backend

# 查看后端日志
docker-compose logs backend

# 重启服务
docker-compose restart
```

### 5. 模型选择不生效

**解决**：
```bash
# 1. 确认对应的 API Key 已配置
cat .env

# 2. 重启服务
docker-compose restart

# 3. 清除浏览器缓存
# 按 Ctrl+Shift+R 强制刷新页面

# 4. 查看后端日志确认模型
docker-compose logs backend | grep -i "model\|claude\|gpt"
```

---

## 🔒 安全建议

### 保护 API Keys

```bash
# 设置文件权限
chmod 600 .env

# 添加到 .gitignore
echo ".env" >> .gitignore

# 定期轮换 API Keys
```

### 限制外部访问

如果部署在公网服务器，修改 docker-compose.yml：

```yaml
services:
  frontend:
    ports:
      - "127.0.0.1:5173:5173"  # 仅本地访问
  backend:
    ports:
      - "127.0.0.1:7001:7001"  # 仅本地访问
```

---

## 📈 性能优化

### 使用固定版本

生产环境使用固定版本镜像：

```yaml
backend:
  image: registry.cn-hangzhou.aliyuncs.com/gyq_personal/screenshot-to-code-backend:v1.0.0
```

### 限制资源使用

```yaml
services:
  backend:
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 2G
```

### 配置日志轮转

```yaml
services:
  backend:
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

---

## 📚 快速参考

### 默认端口

| 服务 | 端口 | 用途 |
|------|-----|------|
| Frontend | 5173 | Web UI 访问 |
| Backend | 7001 | API 服务 |

### 重要 URL

| 用途 | URL |
|------|-----|
| Web 应用 | http://localhost:5173 |
| API 文档 | http://localhost:7001/docs |
| 健康检查 | http://localhost:7001/ |

### 镜像地址

```
Backend:  registry.cn-hangzhou.aliyuncs.com/gyq_personal/screenshot-to-code-backend:latest
Frontend: registry.cn-hangzhou.aliyuncs.com/gyq_personal/screenshot-to-code-frontend:latest
```

---

## 💡 使用技巧

1. **同时配置两个 API Key**，对比不同模型效果
2. **压缩图片**再上传，提高生成速度
3. **添加详细描述**，获得更准确的结果
4. **使用 Regenerate**，尝试不同的生成结果
5. **保存好的结果**，避免重复生成

---

## 📞 获取帮助

1. 查看构建日志：GitHub Actions 页面
2. 查看运行日志：`docker-compose logs -f`
3. 原项目文档：https://github.com/abi/screenshot-to-code
4. 提交 Issue 到项目仓库

---

**最后更新**：2025-10
**版本**：v1.0
