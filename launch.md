# YesPlayMusic 启动指南

> 仓库：[xhit-ux/YesPlayMusic](https://github.com/xhit-ux/YesPlayMusic/tree/feature/friends-messaging)
> 分支：`feature/friends-messaging`

---

## 📋 目录

- [环境要求](#环境要求)
- [克隆项目](#克隆项目)
- [安装依赖](#安装依赖)
- [配置环境变量](#配置环境变量)
- [启动方式](#启动方式)
  - [🌐 网页端开发](#-网页端开发)
  - [🖥 Electron 桌面端开发](#-electron-桌面端开发)
  - [🐳 Docker 部署](#-docker-部署)
- [构建与打包](#构建与打包)
  - [网页版生产构建](#网页版生产构建)
  - [Electron 安装包构建](#electron-安装包构建)
- [代码质量工具](#代码质量工具)
- [项目结构](#项目结构)
- [常见问题](#常见问题)

---

## 环境要求

| 依赖       | 版本  | 说明                     |
| ---------- | ----- | ------------------------ |
| Node.js    | 16.x  | 推荐 16.13.1+            |
| Yarn       | 1.x   | `npm install -g yarn`    |
| Git        | 最新版 | 需支持 `--recursive`    |
| Python 3   | —     | Electron 构建原生依赖时需要 |
| make / g++ | —     | Electron 构建原生依赖时需要 |

---

## 克隆项目

```bash
git clone --recursive -b feature/friends-messaging https://github.com/xhit-ux/YesPlayMusic.git
cd YesPlayMusic
```

---

## 安装依赖

```bash
yarn install
```

> 首次安装耗时较长，取决于网络环境。如遇 Electron 下载卡住，可配置国内镜像：
>
> ```bash
> yarn config set electron_mirror https://npmmirror.com/mirrors/electron/
> yarn config set registry https://registry.npmmirror.com
> ```

---

## 配置环境变量

```bash
cp .env.example .env
```

编辑 `.env` 文件：

```env
# ============================================================
# 网易云 API 地址
# 本地开发 → http://localhost:3000
# 反向代理 → /api
# ============================================================
VUE_APP_NETEASE_API_URL=http://localhost:3000

# Electron 打包后的 API 地址
VUE_APP_ELECTRON_API_URL=/api

# Electron 开发模式下的 API 地址
VUE_APP_ELECTRON_API_URL_DEV=http://127.0.0.1:10754

# Last.fm API（可选，用于歌曲信息展示）
VUE_APP_LASTFM_API_KEY=09c55292403d961aa517ff7f5e8a3d9c
VUE_APP_LASTFM_API_SHARED_SECRET=307c9fda32b3904e53654baff215cb67

# 开发服务器端口
DEV_SERVER_PORT=20201
```

---

## 启动方式

### 🌐 网页端开发

需要**同时运行两个服务**：前端开发服务器 + 网易云 API。

```bash
# 终端 1：启动网易云 API 服务（默认监听 localhost:3000）
yarn netease_api:run
```

```bash
# 终端 2：启动前端开发服务器（默认监听 localhost:8080）
yarn serve
```

启动完成后访问：**http://localhost:8080**

> 热重载已开启，修改代码后浏览器会自动刷新。

---

### 🖥 Electron 桌面端开发

同样需要同时运行 API 服务：

```bash
# 终端 1：启动网易云 API
yarn netease_api:run
```

```bash
# 终端 2：启动 Electron 开发模式
yarn electron:serve
```

> Electron 窗口会自动打开，支持热重载。

---

### 🐳 Docker 部署

#### 方式一：单独运行

```bash
# 构建镜像
docker build -t yesplaymusic .

# 启动容器（映射到 80 端口）
docker run -d --name YesPlayMusic -p 80:80 yesplaymusic
```

访问：**http://localhost**

#### 方式二：Docker Compose（推荐）

包含 YesPlayMusic + UnblockNeteaseMusic，自动解锁灰色歌曲：

```bash
docker-compose up -d
```

服务说明：

| 容器                    | 端口 | 说明                    |
| ----------------------- | ---- | ----------------------- |
| YesPlayMusic            | 80   | 前端 + 内置 API         |
| UnblockNeteaseMusic     | 80/443 | 解锁灰色歌曲（音源代理）|

查看日志：

```bash
docker-compose logs -f
```

停止服务：

```bash
docker-compose down
```

---

## 构建与打包

### 网页版生产构建

```bash
yarn build
```

产物输出到 `/dist` 目录，使用 Nginx 等 Web 服务器部署即可。

Nginx 参考配置：

```nginx
server {
    listen 80;
    server_name your-domain.com;
    root /path/to/dist;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location /api {
        proxy_pass http://localhost:3000;
    }
}
```

---

### Electron 安装包构建

```bash
# 全平台构建（macOS + Windows + Linux）
yarn electron:build-all

# 单平台构建
yarn electron:build-mac       # macOS
yarn electron:build-win       # Windows
yarn electron:build-linux     # Linux

# 指定架构构建
yarn electron:build --windows nsis:ia32    # Windows 32位
yarn electron:build --windows nsis:arm64   # Windows ARM
yarn electron:build --linux deb:armv7l     # Debian armv7l（树莓派等）
yarn electron:build --macos dir:arm64      # macOS ARM

# 发布构建（自动上传到发布平台）
yarn electron:publish
```

产物输出到 `/dist_electron` 目录。

---

## 代码质量工具

```bash
# 代码检查
yarn lint

# 代码格式化（Prettier）
yarn prettier
```

---

## 项目结构

```
YesPlayMusic/
├── src/                    # 前端源码（Vue.js）
├── public/                 # 静态资源
├── build/                  # Electron 构建资源（图标等）
├── docker/                 # Docker 配置文件（nginx.conf）
├── images/                 # README 截图等
├── .env.example            # 环境变量模板
├── vue.config.js           # Vue CLI 配置
├── babel.config.js         # Babel 配置
├── jsconfig.json           # JS 路径别名配置
├── package.json            # 项目依赖与脚本
├── yarn.lock               # 依赖锁定文件
├── Dockerfile              # Docker 镜像定义
├── docker-compose.yml      # Docker Compose 编排
├── devenv.nix              # Nix 开发环境配置
├── install-replit.sh       # Replit 一键安装脚本
└── vercel.example.json     # Vercel 部署配置示例
```

---

## 常见问题

### Q: `yarn install` 报错 node-gyp 相关错误？

安装系统构建工具：

```bash
# macOS
xcode-select --install

# Ubuntu / Debian
sudo apt install python3 make g++

# Windows
npm install -g windows-build-tools
```

### Q: API 启动失败，端口被占用？

```bash
# 查看占用 3000 端口的进程
lsof -i :3000

# 杀掉进程后重新启动
yarn netease_api:run
```

### Q: 网页端打开后空白？

1. 确认 `.env` 中 `VUE_APP_NETEASE_API_URL` 填写正确
2. 确认 API 服务已启动（终端 1 运行 `yarn netease_api:run`）
3. 浏览器打开 DevTools 查看 Network 面板的请求状态

### Q: Electron 构建下载慢？

配置国内镜像源：

```bash
yarn config set electron_mirror https://npmmirror.com/mirrors/electron/
yarn config set registry https://registry.npmmirror.com
sed -i 's/registry.yarnpkg.com/registry.npmmirror.com/g' yarn.lock
sed -i 's/registry.npmjs.org/registry.npmmirror.com/g' yarn.lock
```

### Q: Docker 部署后无法播放音乐？

确保 `docker-compose.yml` 中 `UnblockNeteaseMusic` 服务正常运行：

```bash
docker-compose ps
docker-compose logs UnblockNeteaseMusic
```

### Q: Replit 上如何运行？

```bash
# 在 Replit Shell 中执行
bash <(curl -s -L https://raw.githubusercontent.com/qier222/YesPlayMusic/main/install-replit.sh)
```

> Replit 个人版内存限制 1G，构建可能失败，重试即可。

---

## 快速开始（TL;DR）

```bash
# 1. 克隆
git clone --recursive -b feature/friends-messaging https://github.com/xhit-ux/YesPlayMusic.git
cd YesPlayMusic

# 2. 安装
yarn install

# 3. 配置
cp .env.example .env

# 4. 启动 API（终端 1）
yarn netease_api:run

# 5. 启动前端（终端 2）
yarn serve

# 6. 打开浏览器
# → http://localhost:8080
```