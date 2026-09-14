# ioDrive

<p align="center">
  <img src="./docs/images/logo/logo.svg" width="120" alt="ioDrive Logo">
</p>

<h3 align="center">
  轻量级 Cloudflare 文件分享系统
</h3>

<div align="center">
  本仓库是 <a href="https://github.com/devhunk/Cloudflare-ioDrive">devhunk/Cloudflare-ioDrive</a> 的 fork，
  已适配<strong>单环境部署</strong>与 Cloudflare 免费版限制。<br>
  部署、配置与已知问题请先阅读 <a href="./docs/FORK_DEPLOY.md">Fork 部署指南</a>。
</div>

<p align="center">
  基于 Cloudflare Workers + Hono + S3 兼容存储构建的高性能文件管理与分享平台
</p>

<p align="center">

![License](https://img.shields.io/github/license/AeroAL/Cloudflare-ioDrive?style=for-the-badge)
![Stars](https://img.shields.io/github/stars/AeroAL/Cloudflare-ioDrive?style=for-the-badge)
![Forks](https://img.shields.io/github/forks/AeroAL/Cloudflare-ioDrive?style=for-the-badge)
![Issues](https://img.shields.io/github/issues/AeroAL/Cloudflare-ioDrive?style=for-the-badge)

</p>

<p align="center">

<a href="#-写在前面">背景</a> ·
<a href="#-项目截图">截图</a> ·
<a href="#-功能特性">功能</a> ·
<a href="#-架构设计">架构</a> ·
<a href="#-快速开始">快速开始</a> ·
<a href="#-配置说明">配置</a> ·
<a href="#-api-文档">API</a> ·
<a href="#-部署指南">部署</a> ·
<a href="#-roadmap">Roadmap</a>

</p>

<p align="center">

**中文** | [English](./docs/README_EN.md) | [日本語](./docs/README_JA.md)

</p>

---

## 👋 写在前面

一直苦恼于找不到一个合适的网盘可供多人免登录下载上传，市面上的网盘基本都需要登录才能下载，还有严格的限速，而且没有发现一个很好的文件收集系统，因此诞生了这个项目。

ioDrive 是一个完全运行在 Cloudflare 边缘网络上的轻量级文件管理系统。你只需要一个 Cloudflare 账号，即可在几分钟内部署属于自己的文件分享平台——无需服务器、无需数据库、无需运维。

### 为什么选择 ioDrive？

- **零服务器成本**：全部运行在 Cloudflare Workers 免费额度内（每日 10 万次请求）
- **灵活存储**：支持 R2、AWS S3、MinIO 等任意 S3 兼容存储，可同时配置多个后端
- **极速访问**：Cloudflare 全球 330+ 数节点，用户可就近下载
- **开箱即用**：一条命令部署，无需配置数据库或服务器
- **安全可靠**：JWT 认证 + Turnstile 人机验证，防滥用防攻击

---

## 🚀 一键部署

<p align="center">
  <a href="https://github.com/AeroAL/Cloudflare-ioDrive/fork">
    <img src="https://img.shields.io/badge/⚡_Deploy_to_Cloudflare-F6821F?style=for-the-badge&logo=cloudflare&logoColor=white" alt="Deploy to Cloudflare" height="48">
  </a>
</p>

> Cloudflare 没有像 Vercel 那样的原生 Deploy Button，因此采用 **Fork → 配置 → 推送部署** 模式。
> 配置好仓库的 Secrets 与 Variables 后，推送到 `main` 即自动部署。

<details>
<summary><b>📖 完整部署流程（点击展开）</b></summary>

### 第一步：Fork 仓库

点击上方 **⚡ Deploy to Cloudflare** 按钮，将仓库 Fork 到你的 GitHub 账号。

### 第二步：配置 GitHub Secrets 与 Variables

进入你 Fork 后的仓库 → **Settings** → **Secrets and variables** → **Actions**。

**Secrets**（必需的 7 个）：

| Secret 名称 | 说明 |
|-------------|------|
| `CLOUDFLARE_API_TOKEN` | Cloudflare API 令牌，需 Workers 脚本 / D1 / Workers R2 存储 编辑，以及 Workers 路由 编辑、区域 读取 |
| `CLOUDFLARE_ACCOUNT_ID` | 账户 ID |
| `META_DB_ID` | D1 数据库 ID |
| `ADMIN_PASS` | 管理员密码 |
| `JWT_SECRET` | JWT 签名密钥，用 `node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"` 生成 |
| `TURNSTILE_SITE_KEY` | Turnstile 站点密钥 |
| `TURNSTILE_SECRET` | Turnstile 密钥 |

**Variables**（必需的 6 个）：

| Variable 名称 | 示例 | 说明 |
|---------------|------|------|
| `WORKER_NAME` | `iodrive` | Worker 名称 |
| `SITE_ID` | `production` | 站点标识，改动会使已登录会话失效 |
| `ADMIN_USER` | `admin` | 管理员用户名 |
| `DEPLOY_DOMAIN` | `drive.example.com` | Worker 对外域名，不带 `https://` |
| `R2_BUCKET` | `iodrive-prod` | R2 存储桶名称 |
| `R2_PUBLIC_DOMAIN` | `r2.example.com` | R2 公开访问域名，**不配置则非图片文件无法下载** |

> 可选：`CLOUDFLARE_D1_API_TOKEN`（D1 编辑权限，用于 CI 自动迁移）、`CACHE_KV_ID`（启用 KV 缓存）、
> `RATE_LIMITER_NAMESPACE_ID`（启用限流绑定）。

### 第三步：触发部署

推送到 `main` 分支即自动部署；也可在 **Actions** → **Build and Deploy to Cloudflare** → **Run workflow** 手动触发。

### 后续更新

每次推送到 `main` 都会自动执行「类型检查 → 生成配置 → 应用数据库迁移 → 上传密钥 → 部署」。

</details>

### 方式一：Setup 脚本（推荐本地部署）

交互式引导配置，自动生成配置文件，一条命令完成部署：

```bash
git clone https://github.com/AeroAL/Cloudflare-ioDrive.git
cd Cloudflare-ioDrive
chmod +x setup.sh
./setup.sh
```

脚本会自动引导你完成：
- ✅ 检查前置依赖（Node.js、npm、wrangler）
- ✅ 收集 Cloudflare 账户信息
- ✅ 配置域名和 R2 存储桶
- ✅ 配置 Turnstile 人机验证
- ✅ 生成 `wrangler.toml` 和 `.dev.vars`
- ✅ 自动创建 R2 存储桶并部署

> ⚠️ `setup.sh` 由上游维护，它会额外推送 `R2_ACCESS_KEY` / `R2_SECRET_KEY` 这两个密钥。
> 使用 R2 原生绑定时这两个字段**不会被读取**，可以忽略；见
> [Fork 部署指南](./docs/FORK_DEPLOY.md)。

### 方式二：GitHub Actions 自动部署（推荐持续部署）

> ⚠️ 使用前须先 **[Fork 本仓库](https://github.com/AeroAL/Cloudflare-ioDrive/fork)** 到你的 GitHub 账号

**第一步：配置 GitHub Secrets 与 Variables**

所需的 7 个 Secrets 与 6 个 Variables 清单见上文
[「第二步：配置 GitHub Secrets 与 Variables」](#第二步配置-github-secrets-与-variables)。

**第二步：触发部署**

推送到 `main` 分支即自动部署；也可进入仓库 → **Actions** →
**Build and Deploy to Cloudflare** → **Run workflow** 手动触发。

**后续更新**

配置完成后，每次推送到 `main` 分支都会自动触发部署（通过 `deploy.yml` 工作流）。

---

## 📸 项目截图

### 管理后台

<img src="./docs/images/screenshots/dashboard.jpg" width="100%" alt="管理后台">

### 文件上传

<img src="./docs/images/screenshots/upload.png" width="100%" alt="文件上传">
<img src="./docs/images/screenshots/upload-link-1.png" width="100%" alt="上传链接管理">

### 分享页面

<img src="./docs/images/screenshots/share-link-1.png" width="100%" alt="分享页面-桌面端">
<img src="./docs/images/screenshots/share-link-2.png" width="100%" alt="分享页面-命令下载">
<img src="./docs/images/screenshots/share.png" width="100%" alt="分享页面">
<img src="./docs/images/screenshots/share-1.png" width="100%" alt="分享页面-移动端">

---

## ✨ 功能特性

### 📁 文件管理

- **文件夹操作**：创建、浏览、面包屑导航
- **文件上传**：支持拖拽上传、点击上传，单文件与分片上传（>20MB 自动分片）
- **文件移动**：可视化的文件夹选择器，支持跨目录移动
- **批量操作**：多选文件后批量删除、批量分享、批量移动
- **文件搜索**：在当前目录下按文件名实时搜索
- **并发上传**：分片上传支持最多 6 个并发分片

### 🔗 分享系统

- **一键创建分享链接**：支持单个文件和批量生成
- **分享链接管理**：查看所有分享记录、下载次数统计
- **无广告干扰**：可选「无广告」模式（关闭 S3 备用下载）
- **安全删除**：支持单个或批量清除分享链接

### 🌍 公共上传

- **无需登录**：任何人可通过 `/upload` 页面上传文件
- **Turnstile 人机验证**：防止恶意上传和滥用
- **可限制上传目录**：通过环境变量指定公共上传路径
- **上传来源记录**：区分来自「面板」「公共上传」「上传链接」的来源

### ⏳ 上传链接

- **独立上传地址**：生成专属的 `/u/:keyId` 上传页面
- **自定义有效期**：按小时设置链接过期时间
- **指定上传目录**：每个上传链接可指定不同的目标文件夹
- **后台统一管理**：创建、查看、删除上传链接，追踪使用次数

### 📊 下载统计

每次下载详细记录：

| 字段 | 说明 |
|------|------|
| IP 地址 | 下载者 IP |
| 国家/地区 | Cloudflare IP 地理信息 |
| 浏览器 | Chrome / Safari / Firefox / Edge 等 |
| 操作系统 | Windows / macOS / iOS / Android / Linux |
| 设备类型 | 桌面端 / 移动端 / 平板 |
| 下载时间 | ISO 时间戳 |
| 下载来源 | R2 / S3 / R2+S3 |
| 分享来源 | 通过哪个分享链接下载 |
| 完成状态 | 是否完成下载（beacon 追踪） |

### 📊 上传统计

每次上传详细记录：

| 字段 | 说明 |
|------|------|
| IP 地址 | 上传者 IP |
| 国家/地区 | Cloudflare IP 地理信息 |
| 浏览器 / 操作系统 / 设备 | 完整的 UA 解析 |
| 上传来源 | `dashboard` / `public` / `upload-key` |
| 上传链接标签 | 通过哪个上传链接上传 |

### ☁️ Cloudflare 用量查询

仪表盘右上角提供「Cloudflare 用量」入口，会在新标签页打开 Cloudflare 官方 R2 控制台。账户级 R2 存储用量、10 GB-month 免费额度、操作量和账单状态以 Cloudflare 控制台显示为准；本项目不在 Worker 内把桶内对象大小冒充为当月计费用量。

如需查看当前网盘后端的对象容量诊断，可直接调用受 JWT 保护的 `GET /api/storage/quota`，但该接口不是 Cloudflare 账户级月度账单查询，也不代表真实的 GB-month 消耗。

### ☁️ 多存储后端

- **Cloudflare R2**（推荐）：零出口流量费，Cloudflare 原生存储
- **多 S3 兼容后端**：可同时配置多个 S3 兼容存储，上传自动同步
- **路径风格自动检测**：根据存储提供商自动选择 virtual-hosted 或 path-style
- **支持的存储平台**：

| 提供商 | 标识 | 说明 |
|--------|------|------|
| Cloudflare R2 | `r2` | 零出口流量费（推荐） |
| AWS S3 | `aws` | Amazon 对象存储 |
| Backblaze B2 | `b2` | 低成本云存储 |
| MinIO | `minio` | 自建 S3 兼容存储 |
| 阿里云 OSS | `alibaba` | 国内主流云存储 |
| 腾讯云 COS | `tencent` | 国内主流云存储 |
| Wasabi | `wasabi` | 无限免费出口流量 |
| DigitalOcean Spaces | `digitalocean` | 开发者友好的云存储 |
| 火山引擎 TOS | `volcengine` | 字节跳动云存储 |
| 自定义 | `custom` | 任意 S3 兼容服务 |

- **双通道下载**：分享页面同时提供所有后端的预签名下载链接
- **存储状态检测**：一键检测各存储后端的连通性、响应时间和文件数量
- **多后端文件浏览**：在控制台中切换不同存储后端，独立浏览各后端中的文件
- **动态存储管理**：通过控制台添加、编辑、删除存储后端，无需修改环境变量

### 🌙 用户体验

- **深色模式**：一键切换，偏好持久化到 localStorage
- **响应式布局**：适配桌面端、平板、手机
- **移动端侧边栏**：底部导航栏 + 可折叠侧边菜单
- **上传进度条**：实时显示上传进度（单文件与分片均支持）
- **剪贴板复制**：一键复制分享链接
- **Curl / Aria2 命令**：分享页面自动生成命令行下载指令
- **可配置管理员账号**：通过控制台修改管理员用户名和密码，密码使用带随机盐的 PBKDF2-SHA256 存储

### 🗄️ D1 元数据库

- **D1 元数据库**：`META_DB` binding 为必需配置，用于账号、分享、日志、上传密钥与分片会话
- **单表 key-value 设计**：参考 ImgBed 的简化模式，所有元数据（分享、下载日志、上传日志、上传链接、配置、多段上传会话、审核日志）统一存到 `kv` 表
- **显式初始化**：部署前执行 `database/migrations/`，避免在请求链路动态建表
- **故障可见**：D1 故障直接返回错误，避免静默回退造成元数据分叉

### 🔌 WebDAV 网盘挂载

- **完整读写**：支持 `OPTIONS / PROPFIND / GET / PUT / DELETE / MKCOL / MOVE / COPY / PROPPATCH` 九个方法
- **HTTP Basic 鉴权**：通过 `WEBDAV_USER` / `WEBDAV_PASS`（用 `wrangler secret put` 注入）配置
- **挂载方式**：
  - Windows 资源管理器：`net use Z: https://<host>/dav /user:user pass`
  - macOS Finder：「前往」→「连接服务器」→ `https://<host>/dav`
  - RaiDrive / Cyberduck / Mountain Duck：填入 URL + 用户名密码
- **复用现有逻辑**：WebDAV 内部用短期 JWT（5 分钟）调 `POST /api/upload/single`、`DELETE /api/files/{key}` 等 API，自动复用上传同步 + 日志 + 审核
- **路径安全**：拒绝 `_` 前缀（内部文件）、`..` 反向穿越、反斜杠、URL 双重编码
- **demo 域拦截**：`demo.iodevo.com` 调 WebDAV 一律 403（该主机名硬编码在 `src/demo-mode.ts`；
  对自建部署无影响，除非你恰好也用它）

### 🎲 随机图片 API

- **端点签名**：`GET /random?dir=uploads/photos&content=image&orientation=auto&type=img&form=text`
- **三种返回形式**：
  - `type=img` → 302 跳转到图片
  - `type=url` → JSON 完整 URL
  - `form=text` → 纯文本 URL
  - 默认 → JSON 相对路径
- **Workers Cache 缓存**：目录索引按 `dir` 缓存 24h，重复访问毫秒级返回
- **白名单控制**：`RANDOM_ALLOWED_DIRS` CSV 配置允许的目录，留空 = 不限制
- **方向过滤**：`orientation=auto` 时按 User-Agent 推断（mobile → portrait，desktop → landscape）

### 🛡️ 内容审核

- **默认关闭**：不配 `_config/moderation` 时完全跳过，对性能零影响
- **写后审（异步）**：上传成功 → `c.executionCtx.waitUntil(moderateAndCleanup)` → 不阻塞响应
- **多 Provider**：
  - `moderatecontent`：调用 `https://api.moderatecontent.com/moderate/`
  - `nsfwjs`：调用自部署的 NSFWJS 实例
- **8 秒超时**：`AbortSignal.timeout(8000)` 防止 API 挂起
- **命中规则**：adult（默认阈值 0.9）→ 物理删除文件 + 写审核日志；racy（0.7）→ 仅记录
- **管理员 UI**：在 dashboard 侧边栏「审核日志」标签查看 / 清空 / 测试 Provider
- **豁免规则**：超过 `maxSize`（默认 20MB）或 contentType 不在白名单（`image/jpeg/png/webp/gif`）→ 跳过

---

## 🏗️ 架构设计

```text
┌──────────────────────────┐
                          │     Cloudflare Worker     │
                          │     (Hono Framework)      │
                          │                           │
        ┌─────────────────┼───────────────────────────┼─────────────────┐
        │                 │                           │                 │
   ┌────▼────┐      ┌─────▼──────┐           ┌───────▼──────┐   ┌──────▼──────┐
   │  JWT    │      │   File     │           │   Share      │   │   Upload    │
   │  Auth   │      │   CRUD     │           │   Service    │   │   Service   │
   └─────────┘      └─────┬──────┘           └───────┬──────┘   └──────┬──────┘
                          │                          │                 │
                          └──────────┬───────────────┘                 │
                                     │                                 │
                              ┌──────▼──────┐                   ┌──────▼──────┐
                              │  Cloudflare │                   │  Cloudflare │
                              │     R2      │                   │  Turnstile  │
                              └──────┬──────┘                   └─────────────┘
                                     │
                              ┌──────▼──────┐
                              │  Optional   │
                              │  S3 Sync    │
                              └─────────────┘
```

### 数据存储设计

ioDrive 不依赖传统数据库，所有元数据以 JSON 文件形式存储在对象存储中（R2 或任意 S3 兼容存储）：

| 路径前缀 | 内容 | 说明 |
|----------|------|------|
| `uploads/` | 用户文件 | 所有上传的文件 |
| `_shares/` | 分享记录 | `{token}.json` 存储分享元数据 |
| `_dl_logs/` | 下载日志 | `{source}_{ts}_{rand}.json` |
| `_ul_logs/` | 上传日志 | `{source}_{ts}_{rand}.json` |
| `_upload_keys/` | 上传链接 | `{id}.json` 存储上传链接配置 |
| `_s3/` | S3 多部分元数据 | 临时存储 S3 分片上传的 uploadId |

### 下载流程

```text
用户访问 /s/:token
       │
       ▼
  展示文件信息 + Turnstile 验证
       │
       ▼
  用户完成 Turnstile 验证
       │
       ▼
  POST /api/download/token
       │
       ├──► 生成 R2 预签名 URL（5 分钟有效）
       ├──► 生成 S3 预签名 URL（5 分钟有效，如已配置）
       └──► 记录下载日志
       │
       ▼
  用户点击下载链接
       │
       ▼
  sendBeacon 上报下载完成状态
```

### 上传流程

```text
Dashboard 上传                   公共上传 / 上传链接
     │                                  │
     ├── JWT 认证                       ├── Turnstile 验证
     │                                  ├── 上传链接验证（可选）
     │                                  │
     └──────────┬───────────────────────┘
                │
         ┌──────▼──────┐
         │ 文件大小判断  │
         └──────┬──────┘
                │
      ┌─────────┴─────────┐
      │                   │
   ≤ 20MB              > 20MB
      │                   │
  R2 PutObject      R2 CreateMultipartUpload
  + S3 PutObject    │
      │             ├── 并发上传分片（最多 6 个）
      │             │   R2 uploadPart + S3 uploadPart
      │             │
      │             R2 CompleteMultipartUpload
      │             + S3 CompleteMultipartUpload
      │                   │
      └─────────┬─────────┘
                │
          记录上传日志
```

---

## 🧰 技术栈

<p align="center">

<img src="https://skillicons.dev/icons?i=ts" height="48" alt="TypeScript"/>
<img src="https://hono.dev/images/logo-large.png" height="48" alt="Hono"/>
<img src="https://www.vectorlogo.zone/logos/cloudflare/cloudflare-icon.svg" height="48" alt="Cloudflare"/>

</p>

| 技术 | 版本 | 用途 |
|------|------|------|
| [TypeScript](https://www.typescriptlang.org/) | ^5.0 | 类型安全的开发语言 |
| [Hono](https://hono.dev/) | ^4.0 | 轻量级 Web 框架，专为边缘计算优化 |
| [Cloudflare Workers](https://workers.cloudflare.com/) | - | Serverless 运行时，全球边缘部署 |
| [Cloudflare R2](https://www.cloudflare.com/r2/) | - | 对象存储（可选），零出口流量费 |
| [S3 兼容存储](https://aws.amazon.com/s3/) | - | 主存储后端，支持 AWS S3、MinIO、B2 等 |
| [Cloudflare Turnstile](https://www.cloudflare.com/products/turnstile/) | - | 隐私友好的人机验证 |
| [jose](https://github.com/panva/jose) | ^6.0 | JWT 签名与验证（HS256） |
| [Wrangler](https://developers.cloudflare.com/workers/wrangler/) | ^4.0 | Cloudflare CLI 开发部署工具 |

### 为什么选择 Hono？

- 专为 Cloudflare Workers 设计，极小的运行时体积
- 类 Express 的 API，学习成本低
- 内置 CORS、中间件等开箱即用
- 支持 JSX 和模板字符串渲染 HTML

### 为什么用 D1 存元数据？

> 注：早期版本把元数据以 JSON 文件存进 R2，现已迁移到 D1。以下为当前设计。

- **D1 是必需绑定**（`META_DB`），单表 key-value 模式，存储账号、分享、日志、上传链接、分片会话等
- 相比在 R2 里读写 JSON 文件，D1 提供索引与原子计数（如上传链接已用次数），也避免列表操作的额外开销
- 文件本体仍在 R2 / S3 兼容存储中，D1 只放元数据

---

## 🚀 快速开始

### 前置要求

- [Node.js](https://nodejs.org/) >= 18
- [Cloudflare 账号](https://dash.cloudflare.com/sign-up)
- Cloudflare 账号中已开通 **Workers** 服务
- 至少一个 S3 兼容存储服务（AWS S3、Cloudflare R2、MinIO、Backblaze B2、阿里云 OSS、腾讯云 COS 等）
- 一个域名（托管在 Cloudflare DNS 上）

### 第一步：克隆项目

```bash
git clone https://github.com/AeroAL/Cloudflare-ioDrive.git
cd Cloudflare-ioDrive
npm install
```

### 第二步：配置 Cloudflare 资源

#### 创建 R2 存储桶

1. 登录 [Cloudflare Dashboard](https://dash.cloudflare.com/)
2. 进入 **R2** → **创建存储桶**
3. 填写存储桶名称（如 `iodrive`）
4. 记录存储桶名称，稍后填入配置

#### 获取 R2 API 密钥

1. 进入 **R2** → **管理 R2 API 令牌**
2. 创建 API 令牌，权限选择「管理员读/写」
3. 记录 **Access Key ID** 和 **Secret Access Key**

#### 获取 Account ID

1. 进入 Cloudflare Dashboard 首页
2. 右侧「API」区域复制 **Account ID**

#### 获取 Turnstile 密钥

1. 进入 **Turnstile** 页面
2. 添加站点，选择「托管」模式
3. 记录 **Site Key** 和 **Secret Key**

### 第三步：配置环境变量

复制示例配置文件：

```bash
cp wrangler.toml.example wrangler.toml
```

编辑 `wrangler.toml`，填入你的配置：

```toml
name = "iodrive"
main = "src/index.ts"
compatibility_date = "2026-09-12"
compatibility_flags = ["nodejs_compat"]
routes = [{ pattern = "YOUR_DOMAIN/*", zone_name = "YOUR_ZONE" }]

[vars]
SITE_ID = "production"
ADMIN_USER = "admin"
APP_DOMAIN = "YOUR_WORKER_DOMAIN"
R2_PUBLIC_DOMAIN = "YOUR_R2_PUBLIC_DOMAIN"
R2_BUCKET = "YOUR_R2_BUCKET"
R2_ACCOUNT_ID = "YOUR_R2_ACCOUNT_ID"
TURNSTILE_SITE_KEY = "YOUR_TURNSTILE_SITE_KEY"

# 可选 S3 同步
S3_ENDPOINT = "YOUR_S3_ENDPOINT"
S3_BUCKET = "YOUR_S3_BUCKET"
S3_REGION = "YOUR_S3_REGION"
PUBLIC_UPLOAD_PATH = "uploads/public/"

[[r2_buckets]]
binding = "DRIVE"
bucket_name = "YOUR_R2_BUCKET"
```

### 第四步：配置密钥

通过 `wrangler secret` 设置敏感信息：

```bash
# 必需密钥
wrangler secret put ADMIN_PASS      # 管理员密码
wrangler secret put JWT_SECRET      # JWT 签名密钥（建议使用随机生成的 64 位字符串）
wrangler secret put TURNSTILE_SECRET # Turnstile 密钥
wrangler secret put R2_ACCESS_KEY   # R2 Access Key
wrangler secret put R2_SECRET_KEY   # R2 Secret Key

# 可选 S3 密钥（如需 S3 同步）
wrangler secret put S3_ACCESS_KEY
wrangler secret put S3_SECRET_KEY
```

> 💡 **生成 JWT_SECRET**：`node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"`

### 第五步：本地开发

创建 `.dev.vars` 文件用于本地开发（此文件已加入 `.gitignore`）：

```bash
# .dev.vars
ADMIN_PASS=your_admin_password
JWT_SECRET=your_jwt_secret
TURNSTILE_SECRET=your_turnstile_secret
R2_ACCESS_KEY=your_r2_access_key
R2_SECRET_KEY=your_r2_secret_key
S3_ACCESS_KEY=your_s3_access_key     # 可选
S3_SECRET_KEY=your_s3_secret_key     # 可选
```

启动本地开发服务器：

```bash
npm run dev
```

### 第六步：部署

```bash
npm run deploy
```

部署完成后访问 `https://YOUR_DOMAIN` 即可使用。

---

## ⚙️ 配置说明

### 环境变量完整列表

#### 必需变量（wrangler.toml vars）

| 变量 | 说明 | 示例 |
|------|------|------|
| `SITE_ID` | 部署实例唯一标识，用于缓存和 JWT 隔离 | `production` |
| `APP_DOMAIN` | Worker 对外域名，WebDAV 内部回调使用；不含协议 | `drive.example.com` |
| `ADMIN_USER` | 管理员用户名 | `admin` |
| `R2_PUBLIC_DOMAIN` | R2 公开访问域名 | `r2.example.com` |
| `R2_BUCKET` | R2 存储桶名称 | `iodrive` |
| `R2_ACCOUNT_ID` | Cloudflare 账户 ID | `YOUR_ACCOUNT_ID` |
| `TURNSTILE_SITE_KEY` | Turnstile 站点密钥（公开） | `YOUR_TURNSTILE_SITE_KEY` |

#### 必需密钥（wrangler secret）

| 变量 | 说明 |
|------|------|
| `ADMIN_PASS` | 管理员密码 |
| `JWT_SECRET` | JWT HS256 签名密钥（建议 64 位随机字符串） |
| `TURNSTILE_SECRET` | Turnstile 密钥（用于服务端验证） |
| `R2_ACCESS_KEY` | ⚠️ **残留字段，源码未读取**。使用 R2 原生绑定时无需配置 |
| `R2_SECRET_KEY` | ⚠️ **残留字段，源码未读取**。同上 |

#### 可选变量

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `STORAGE_CONFIG` | 多后端存储配置（JSON 数组） | - |
| `S3_CREDENTIALS` | 多后端存储密钥（JSON 对象，通过 wrangler secret 设置） | - |
| `S3_ENDPOINT` | 单个 S3 端点（向后兼容，推荐使用 STORAGE_CONFIG） | - |
| `S3_BUCKET` | S3 存储桶名称（向后兼容） | - |
| `S3_REGION` | S3 区域（向后兼容） | - |
| `S3_ACCESS_KEY` | S3 Access Key（向后兼容） | - |
| `S3_SECRET_KEY` | S3 Secret Key（向后兼容） | - |
| `PUBLIC_UPLOAD_PATH` | 公共上传默认路径 | `uploads/public/` |
| `PUBLIC_UPLOAD_MAX_BYTES` | 公共分片上传上限（字节） | `2147483648` |
| `WEBDAV_ENABLED` | WebDAV 总开关（`true` / `false`） | `false` |
| `WEBDAV_USER` | WebDAV HTTP Basic 用户名（建议用 `wrangler secret put` 注入） | - |
| `WEBDAV_PASS` | WebDAV HTTP Basic 密码（建议用 `wrangler secret put` 注入） | - |
| `RANDOM_ENABLED` | 随机图片 API 总开关 | `false` |
| `RANDOM_ALLOWED_DIRS` | 随机 API 允许的目录 CSV（留空 = 不限制） | 空 |
| `DEMO_MODE` | 只读演示开关；仅演示 Worker 设置为 `true` | `false` |

#### D1 元数据库（必需）

```toml
[[d1_databases]]
binding = "META_DB"
database_name = "iodrive-meta"
database_id = "<your-d1-uuid>"
migrations_dir = "database/migrations"
```

部署前应用版本化迁移：

```bash
npx wrangler d1 migrations apply META_DB --remote
```

### 路由配置

```toml
# 生产环境
routes = [{ pattern = "drive.example.com/*", zone_name = "example.com" }]

# 如果不需要自定义域名，也可以使用 workers.dev 子域名
# 删除 routes 配置即可自动分配 <worker-name>.<subdomain>.workers.dev
```

### R2 存储桶绑定

```toml
[[r2_buckets]]
binding = "DRIVE"       # 代码中使用的绑定名，不可修改
bucket_name = "iodrive" # 你的 R2 存储桶名称
```

> ⚠️ **重要：`bucket_name` 必须与 `[vars]` 中的 `R2_BUCKET` 完全一致。**
> 如果两者不一致（例如拼写错误），Worker 会绑定到错误的桶，导致文件列表显示为空或数据丢失。请务必仔细核对。

### KV 缓存配置

```toml
[[kv_namespaces]]
binding = "CACHE_KV"
id = "your-kv-namespace-id"
```

> ⚠️ **多实例部署注意：** 共享同一个 KV 命名空间时，每个实例必须设置不同的 `SITE_ID`。更稳妥的做法仍是为每个实例分配独立的 KV 命名空间。

### CORS 配置

默认允许所有来源访问 `/api/*` 路径。如需限制，可在 `src/index.ts` 中修改：

```typescript
app.use('/api/*', cors({
  origin: 'https://your-domain.com',
  allowMethods: ['GET', 'POST', 'DELETE'],
}));
```

---

## 📁 项目结构

```
drive/
├── .github/
│   └── workflows/
│       ├── deploy.yml                # 单环境部署（推送到 main 自动触发）
│       └── deploy-button.yml         # 参数化手动部署（workflow_dispatch）
├── database/
│   └── migrations/
│       └── 0001_init.sql             # D1 初始版本化迁移
├── docs/
│   ├── FORK_DEPLOY.md                # Fork 部署指南（含免费版限制与已知问题）
│   ├── MAINTENANCE.md                # 维护、迁移与发布约定
│   ├── prototypes/
│   │   └── pen.html                  # 历史界面原型
│   ├── README_EN.md                  # 英文文档
│   ├── README_JA.md                  # 日文文档
│   └── images/
│       ├── logo/
│       │   └── logo.svg              # 项目 Logo
│       └── screenshots/
│           ├── dashboard.jpg         # 管理后台截图
│           ├── upload.png            # 上传截图
│           ├── upload-link-1.png     # 上传链接截图
│           ├── share.png             # 分享截图
│           ├── share-1.png           # 分享截图-移动端
│           ├── share-link-1.png      # 分享链接截图-桌面端
│           └── share-link-2.png      # 分享链接截图-命令下载
├── src/
│   ├── index.ts                      # 🚀 应用入口：路由注册、页面路由、SEO
│   ├── demo-mode.ts                  # 🔒 演示环境边界与只读判定
│   ├── jwt-scope.ts                  # 🔐 JWT 签发方与环境作用域
│   ├── auth.ts                       # 🔐 JWT 认证：登录、JWT 签发与验证中间件、频率限制
│   ├── files.ts                      # 📁 文件 CRUD：列表、创建文件夹、删除、批量删除、移动
│   ├── upload.ts                     # 📤 仪表盘上传：单文件、分片上传（init/part/complete/abort）
│   ├── upload-public.ts              # 🌍 公共上传：Turnstile 验证 + 可选上传链接
│   ├── upload-keys.ts                # 🔑 上传链接管理：创建、列表、删除、验证
│   ├── upload-logs.ts                # 📊 上传日志：列表、清空、删除、日志写入
│   ├── upload-utils.ts               # 🛠 上传工具：MIME 类型映射、唯一文件名生成
│   ├── download.ts                   # ⬇️ 下载服务：预签名 URL 生成、下载日志、beacon 追踪
│   ├── share.ts                      # 🔗 分享服务：创建、列表、删除、批量分享、公开信息查询
│   ├── s3-upload.ts                  # ☁️ S3 上传：AWS Signature V4 实现（单文件 + 分片）
│   ├── storage-engine.ts             # 🧰 存储引擎抽象：R2 / S3 统一接口
│   ├── storage-config.ts             # 🗄 多后端存储配置管理
│   ├── metadata-store.ts             # 🗄 D1 元数据存储实现
│   ├── moderation.ts                 # 🛡 内容审核 Provider 接口 + ModerateContent / NSFWJS 实现
│   ├── moderation-admin.ts           # 🛡 审核配置 / 日志 / 测试 API
│   ├── random.ts                     # 🎲 随机图片 API（带 Workers Cache 缓存）
│   ├── webdav.ts                     # 🔌 WebDAV 服务（完整 RW + HTTP Basic 鉴权）
│   ├── webdav-xml.ts                 # 🔌 WebDAV PROPFIND XML 生成
│   ├── turnstile.ts                  # 🛡 Turnstile 验证：服务端 token 校验
│   ├── ua-parser.ts                  # 🔍 UA 解析：浏览器、操作系统、设备类型识别
│   ├── types.ts                      # 📐 类型定义：Env、JwtPayload、FileMeta、ShareRecord、ModerationConfig 等
│   └── html/
│       ├── dashboard.ts              # 管理后台 SPA（含审核日志共六个视图）
│       ├── login.ts                  # 登录页面
│       ├── share.ts                  # 公开分享下载页面
│       ├── upload-key.ts             # 上传链接页面（限时上传）
│       ├── public-upload.ts          # 公共上传页面（无需登录）
│       └── demo.ts                   # 演示站 Landing Page
├── wrangler.toml                     # 生产环境部署配置
├── wrangler.toml.example             # 配置文件模板
├── tsconfig.json                     # TypeScript 配置
├── package.json                      # 项目依赖与脚本
├── SECURITY.md                       # 安全报告与部署约束
├── LICENSE                           # GPL-3.0 许可证
└── README.md                         # 项目文档
```

---

## 📡 API 文档

所有 API 路径以 `/api` 为前缀，支持 CORS 跨域访问。

### 认证说明

- **JWT 认证**：在请求头中携带 `Authorization: Bearer <token>`
- **Turnstile 验证**：在请求体中携带 `turnstile` 字段（token 由客户端 Turnstile widget 生成）
- JWT 有效期：24 小时

---

### 认证 API

#### `POST /api/auth/login`

登录获取 JWT Token。

**请求体：**

```json
{
  "username": "admin",
  "password": "your_password",
  "turnstile": "turnstile_token"
}
```

**成功响应：**

```json
{
  "token": "eyJhbGciOiJIUzI1NiJ9..."
}
```

**错误响应：**

- `400` - 缺少人机验证
- `401` - 用户名或密码错误
- `403` - 人机验证失败
- `429` - 登录尝试过多（5 次失败后锁定 5 分钟）

---

### 文件管理 API（需要 JWT）

#### `GET /api/files?prefix=uploads/`

列出指定目录下的文件和文件夹。

**响应：**

```json
{
  "files": [
    {
      "key": "uploads/example.pdf",
      "name": "example.pdf",
      "size": 1024000,
      "uploaded": "2025-01-01T00:00:00.000Z",
      "contentType": "application/pdf"
    }
  ],
  "folders": [{ "name": "docs", "path": "uploads/docs/" }],
  "currentPath": "uploads/",
  "ancestors": []
}
```

#### `GET /api/files/folders`

递归获取所有文件夹列表（用于移动文件选择器）。

#### `GET /api/files/:key`

获取单个文件元数据。

#### `POST /api/files/folder`

创建文件夹。

**请求体：**

```json
{ "path": "uploads/new-folder/" }
```

#### `DELETE /api/files/:key`

删除文件或文件夹（文件夹会递归删除其下所有文件）。

#### `POST /api/files/batch-delete`

批量删除文件或文件夹。

**请求体：**

```json
{ "keys": ["uploads/file1.pdf", "uploads/folder/"] }
```

#### `POST /api/files/move`

移动文件到目标文件夹。

**请求体：**

```json
{
  "keys": ["uploads/file1.pdf"],
  "targetPath": "uploads/docs/"
}
```

---

### 仪表盘上传 API（需要 JWT）

#### `POST /api/upload/single`

单文件上传（≤ 20MB）。使用 `multipart/form-data`。

**表单字段：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `file` | File | 上传的文件 |
| `path` | string | 目标路径（默认 `uploads/`） |

#### `POST /api/upload/init`

初始化分片上传（> 20MB）。

**请求体：**

```json
{
  "filename": "large-file.zip",
  "size": 104857600,
  "path": "uploads/"
}
```

**响应：**

```json
{
  "uploadId": "abc123...",
  "key": "uploads/large-file.zip"
}
```

#### `POST /api/upload/part`

上传分片。使用 `multipart/form-data`。

**表单字段：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `uploadId` | string | 上传会话 ID |
| `key` | string | 文件 key |
| `partNumber` | number | 分片编号（从 1 开始） |
| `chunk` | File | 分片数据 |

#### `POST /api/upload/complete`

完成分片上传。

**请求体：**

```json
{
  "uploadId": "abc123...",
  "key": "uploads/large-file.zip",
  "parts": [
    { "partNumber": 1, "etag": "abc" },
    { "partNumber": 2, "etag": "def" }
  ]
}
```

#### `POST /api/upload/abort`

取消分片上传。

**请求体：**

```json
{
  "uploadId": "abc123...",
  "key": "uploads/large-file.zip"
}
```

---

### 公共上传 API（需要 Turnstile）

接口与仪表盘上传相同，路径前缀为 `/api/upload-public`，额外需要 `turnstile` 字段。

#### `POST /api/upload-public/single`

公共单文件上传。表单额外字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| `turnstile` | string | Turnstile 验证 token |
| `uploadKeyId` | string | 可选，上传链接 ID |

#### `POST /api/upload-public/init`

公共分片上传初始化。请求体额外字段：

```json
{
  "filename": "file.zip",
  "size": 104857600,
  "turnstile": "turnstile_token",
  "uploadKeyId": "optional_key_id"
}
```

---

### 下载 API

#### `GET /api/download/presign/:key`（需要 JWT）

生成 R2 预签名下载链接（仪表盘内使用），同时记录下载日志。

#### `POST /api/download/token`（需要 Turnstile）

通过分享链接生成预签名下载链接。

**请求体：**

```json
{
  "shareToken": "abc123...",
  "turnstile": "turnstile_token"
}
```

**响应：**

```json
{
  "r2Url": "https://bucket.r2.cloudflarestorage.com/...",
  "s3Url": "https://bucket.s3.amazonaws.com/...",
  "logKey": "_dl_logs/...",
  "name": "example.pdf",
  "size": 1024000
}
```

#### `POST /api/download/beacon`

下载完成追踪（浏览器通过 `sendBeacon` 调用）。

**请求体：**

```json
{
  "logKey": "_dl_logs/abc123_1234567890_abc123.json",
  "event": "complete"
}
```

---

### 下载日志 API（需要 JWT）

#### `GET /api/download/logs`

获取下载日志列表（最近 500 条）。

#### `DELETE /api/download/logs`

清空所有下载日志。

#### `DELETE /api/download/logs/:logKey`

删除单条下载日志。

---

### 上传日志 API（需要 JWT）

#### `GET /api/upload-logs/logs`

获取上传日志列表（最近 500 条）。

#### `DELETE /api/upload-logs/logs`

清空所有上传日志。

#### `DELETE /api/upload-logs/logs/:logKey`

删除单条上传日志。

---

### 分享 API

#### `POST /api/share`（需要 JWT）

创建分享链接。

**请求体：**

```json
{
  "key": "uploads/example.pdf",
  "name": "example.pdf",
  "noAd": false
}
```

#### `POST /api/share/batch`（需要 JWT）

批量创建分享链接。

**请求体：**

```json
{ "keys": ["uploads/file1.pdf", "uploads/file2.pdf"] }
```

#### `GET /api/share`（需要 JWT）

获取所有分享记录。

#### `DELETE /api/share/:token`（需要 JWT）

删除指定分享链接。

#### `GET /api/share/info/:token`（公开）

获取分享文件信息（无需认证）。

---

### 上传链接 API

#### `POST /api/upload-keys`（需要 JWT）

创建上传链接。

**请求体：**

```json
{
  "label": "项目文件收集",
  "path": "uploads/project-a/",
  "expiresHours": 72
}
```

#### `GET /api/upload-keys`（需要 JWT）

获取所有上传链接列表。

#### `DELETE /api/upload-keys/:id`（需要 JWT）

删除指定上传链接。

#### `GET /api/upload-keys/validate/:id`（公开）

验证上传链接是否有效。

---

### 随机图片 API（公开）

#### `GET /random`

**Query 参数：**

| 参数 | 说明 | 默认 |
|------|------|------|
| `dir` | 目录（必须在 `RANDOM_ALLOWED_DIRS` 白名单中） | 空（根目录） |
| `content` | contentType 包含过滤 | `image` |
| `orientation` | `all` / `auto` / `landscape` / `portrait` / `square` | `all` |
| `type` | `img`（302 跳转） / `url`（JSON 完整 URL） / 空（JSON 相对路径） | 空 |
| `form` | `text`（纯文本 URL） | 空 |

**示例：**

```bash
# 默认 JSON
curl https://drive.example.com/random?dir=uploads/photos

# 纯文本 URL
curl https://drive.example.com/random?dir=uploads/photos&form=text

# 302 跳转到图片
curl -L https://drive.example.com/random?dir=uploads/photos&type=img
```

#### `POST /api/random/refresh`（需要 JWT）

清空 Workers Cache 中指定目录的随机索引缓存。

```json
{ "dirs": ["uploads/photos", "uploads/wallpapers"] }
```

### 内容审核 API（需要 JWT）

#### `GET /api/moderation/config`

读取审核配置（apiKey 返回脱敏值）。

#### `PUT /api/moderation/config`

写入审核配置。

```json
{
  "enabled": true,
  "provider": "moderatecontent",
  "apiKey": "your-key",
  "thresholds": { "adult": 0.9, "racy": 0.7 },
  "fileTypes": ["image/jpeg", "image/png", "image/webp", "image/gif"],
  "maxSize": 20971520
}
```

#### `POST /api/moderation/test`

测试 Provider（不影响真实数据）。

```json
{ "url": "https://example.com/test.jpg" }
```

#### `GET /api/moderation/logs`

读取审核日志列表（按时间倒序，最多 500 条）。

#### `DELETE /api/moderation/logs`

清空所有审核日志。

#### `DELETE /api/moderation/logs/:id`

删除单条审核日志。

### 存储配置 API（需要 JWT）

用于在控制台中动态管理存储后端，以及提供当前存储后端的容量诊断接口。Cloudflare 账户级 R2 月度用量和账单请使用仪表盘右上角的官方入口查询。

| 方法 | 路径 | 说明 |
|------|------|------|
| `GET` | `/api/storage/providers` | 获取支持的存储提供商预设 |
| `GET` | `/api/storage/backends` | 获取已配置的后端列表（密钥脱敏） |
| `POST` | `/api/storage/backends` | 新增后端 |
| `PUT` | `/api/storage/backends/:name` | 修改后端 |
| `DELETE` | `/api/storage/backends/:name` | 删除后端 |
| `POST` | `/api/storage/test` | 测试连接 |
| `POST` | `/api/storage/status` | 检测指定后端状态 |
| `GET` | `/api/storage/quota` | 获取当前后端对象容量诊断（本 fork 新增） |

#### `GET /api/storage/quota`

返回当前存储后端的对象容量诊断。该接口通过列举对象累加大小和数量，**不查询 Cloudflare 账户级 R2 月度用量，也不代表 10 GB-month 的实际计费消耗**。Cloudflare 官方用量和账单请使用仪表盘右上角的 Cloudflare 入口。

**查询参数：**

- `refresh=1`（可选）：跳过缓存，强制重新扫描当前存储后端

**成功响应：**

```json
{
  "usedBytes": 1234567,
  "objectCount": 42,
  "limitBytes": 10737418240,
  "remainingBytes": 10736183773,
  "usedPercent": 0.000115,
  "truncated": false,
  "backend": "r2",
  "scannedAt": "2026-09-14T06:07:47.895Z",
  "cached": false
}
```

- `truncated`：为 `true` 时表示对象数超过 10 万，统计不完整
- `limitBytes`：当前后端容量诊断使用的应用自定义比较值，来自 `QUOTA_LIMIT_BYTES`；未设置时使用内部默认值，仅用于诊断展示，不是 Cloudflare 账户配额
- 绑定了 `CACHE_KV` 时结果缓存 120 秒；`refresh=1` 可绕过

### WebDAV

`/dav/*` 路径（无需 JWT，使用 HTTP Basic）：

| 方法 | 用途 |
|------|------|
| `OPTIONS` | 返回 `DAV: 1, 2` 与 Allow 头 |
| `PROPFIND` | 列出目录 / 文件元数据，207 Multi-Status XML |
| `GET` | 文件流式下载 / 目录 HTML 列表 |
| `PUT` | 上传文件（复用 `/api/upload/single`，自动同步 + 日志） |
| `DELETE` | 删除（复用 `/api/files/{key}`） |
| `MKCOL` | 创建目录（`/` 结尾） |
| `MOVE` | 移动 / 重命名（复用 `/api/files/move`） |
| `COPY` | 复制 |
| `PROPPATCH` | 200 OK no-op |

**挂载示例：**

```bash
# Windows 资源管理器
net use Z: https://drive.example.com/dav /user:webdav_user webdav_pass

# macOS Finder
# 「前往」→「连接服务器」→ https://drive.example.com/dav

# Linux (davfs2)
mount -t davfs https://drive.example.com/dav /mnt/iodrive
```

---

## 🚢 部署指南

### 方式一：手动部署

```bash
# 安装依赖
npm install

# 类型检查
npx tsc --noEmit

# 部署到 Cloudflare Workers
npm run deploy
```

### 方式二：GitHub Actions 自动部署

> ⚠️ **前置步骤**：使用此方式前，你须先 **Fork 本仓库** 到你的 GitHub 账号下。否则将无权配置 Secrets，无法触发自动部署。

本 fork 的 CI/CD 流水线（`.github/workflows/deploy.yml`）已改造为**单环境部署**：

- **推送到 `main` 分支** → 类型检查通过后，依次生成配置、创建/复用 R2 存储桶、应用 D1 迁移、上传密钥、部署
- **创建 PR 到 `main`** → 仅运行类型检查和 dry-run 构建
- **手动触发** → Actions 页面 **Run workflow**

> 上游版本的 `deploy.yml` 会额外部署一个只读演示环境，且硬编码了上游作者的域名与 D1 数据库 ID，
> 在 fork 中必然失败。本 fork 已移除该任务。

#### 配置 GitHub Actions

1. Fork 本仓库后，在你自己的仓库中进入 **Settings** → **Secrets and variables** → **Actions**。

   **必需的 Secrets：**

   - `CLOUDFLARE_API_TOKEN`：Cloudflare API 令牌，需 Workers 脚本 / D1 / Workers R2 存储 编辑，
     以及 Workers 路由 编辑、区域 读取
   - `CLOUDFLARE_ACCOUNT_ID`：Cloudflare 账户 ID
   - `META_DB_ID`：D1 数据库 ID
   - `ADMIN_PASS`：管理员密码
   - `JWT_SECRET`：JWT 签名密钥
   - `TURNSTILE_SITE_KEY` / `TURNSTILE_SECRET`：Turnstile 人机验证密钥

   **必需的 Variables：**

   - `WORKER_NAME`：Worker 名称（例 `iodrive`）
   - `SITE_ID`：站点标识（例 `production`）；改动会使已登录会话失效
   - `ADMIN_USER`：管理员用户名（例 `admin`）
   - `DEPLOY_DOMAIN`：Worker 对外域名（例 `drive.example.com`），不带 `https://`
   - `R2_BUCKET`：R2 存储桶名称
   - `R2_PUBLIC_DOMAIN`：R2 公开访问域名（例 `r2.example.com`）；
     **不配置则非图片文件无法下载**

   **可选：** `CLOUDFLARE_D1_API_TOKEN`（D1 编辑权限，用于 CI 自动迁移）、
   `CACHE_KV_ID`（启用 KV 缓存）、`RATE_LIMITER_NAMESPACE_ID`（启用限流绑定）

2. 创建 Cloudflare API Token：
   
   - 进入 [Cloudflare Dashboard](https://dash.cloudflare.com/) → **我的个人资料** → **API 令牌**
   - 创建令牌 → **Create Custom Token**，按下表添加权限
   - 账户级：`Workers 脚本:编辑`、`D1:编辑`、`Workers R2 存储:编辑`
   - 区域级：`Workers 路由:编辑`、`区域:读取`（区域选你的域名）

3. 获取 Cloudflare Account ID：
   
   - 进入 [Cloudflare Dashboard](https://dash.cloudflare.com/) → 选择你的域名 → **概览** 页面右侧可以看到 **账户 ID**

> 完整的资源创建步骤、验收清单与免费版已知限制，见
> [Fork 部署指南](./docs/FORK_DEPLOY.md)。

### 演示环境（可选，本 fork 未部署）

上游的官方 Demo 是无账号、只读的模拟数据环境，通过 `DEMO_MODE=true` 显式启用，拒绝所有写请求，
并使用独立的 `SITE_ID`、D1 与 R2。本 fork 的 CI 不部署演示环境；`src/demo-mode.ts` 相关代码
仍然保留，如需自建请参考 `wrangler.toml.example`，并务必使用独立资源且不要注入生产密钥。

### 部署后检查

- [ ] 访问你的域名，确认管理后台正常加载
- [ ] 使用配置的管理员账号登录
- [ ] 测试文件上传、分享、下载流程
- [ ] **上传一个非图片文件（如 .zip）并下载**——验证 `R2_PUBLIC_DOMAIN` 已正确配置
- [ ] 测试公共上传页面 `/upload`
- [ ] 检查 R2 存储桶中的文件是否正常写入

---

## 🛣️ Roadmap

### ✅ 已完成

- [x] 文件上传（单文件 + 分片上传）
- [x] 文件夹管理（创建、导航、面包屑）
- [x] 文件移动与批量操作
- [x] 分享链接（单个 + 批量）
- [x] 下载日志（IP、国家、浏览器、OS、设备类型）
- [x] 上传日志（来源追踪）
- [x] 上传链接（限时 + 指定目录）
- [x] 公共上传（Turnstile 验证）
- [x] R2 + S3 双存储支持
- [x] 深色模式
- [x] 响应式布局 / 移动端适配
- [x] Curl / Aria2 命令行下载支持
- [x] 多后端存储状态检测
- [x] 多后端文件浏览切换
- [x] 可配置管理员账号密码
- [x] 图床功能（一键复制 Markdown / BBCode）

### 🚧 计划中

- [ ] 文件预览（图片、视频、文档在线预览）
- [ ] 回收站（软删除 + 恢复机制）
- [ ] 文件标签与分类
- [ ] 多用户系统（多管理员 + 权限控制）
- [ ] OAuth 登录（GitHub / Google）
- [ ] API Token（程序化访问）
- [ ] 文件直链管理
- [ ] Webhook 通知（上传/下载事件通知）
- [ ] 国际化 i18n（English / 日本語）

---

## 🤝 贡献指南

欢迎各种形式的贡献！无论是提 Bug、建议新功能还是提交 PR。

### 提交 Issue

- 使用清晰的标题描述问题
- 提供复现步骤和环境信息
- 提交前先搜索是否已有相同问题

### 提交 Pull Request

1. Fork 本仓库
2. 创建功能分支：`git checkout -b feature/amazing-feature`
3. 提交更改：`git commit -m 'feat: add amazing feature'`
4. 推送分支：`git push origin feature/amazing-feature`
5. 提交 Pull Request

### 开发注意事项

- 运行 `npx tsc --noEmit` 确保类型检查通过
- HTML 模板在 `src/html/` 目录中，使用 TypeScript 模板字符串
- API 路由在 `src/` 根目录，按功能模块拆分
- 遵循现有代码风格（中文注释，功能模块化）

---

## ⭐ Star History

如果这个项目对你有帮助，欢迎给上游项目点一个 Star ⭐

[![Star History Chart](https://api.star-history.com/svg?repos=devhunk/Cloudflare-ioDrive&type=Date)](https://star-history.com/#devhunk/Cloudflare-ioDrive&Date)

---

## 📬 联系方式

* **上游作者**: [devhunk](https://github.com/devhunk)（原 MareixHunk）
* **GitHub**: [devhunk/Cloudflare-ioDrive](https://github.com/devhunk/Cloudflare-ioDrive)

## 📜 License

GPL-3.0 License。本仓库为 [devhunk/Cloudflare-ioDrive](https://github.com/devhunk/Cloudflare-ioDrive)
的 fork，同样以 GPL-3.0 分发；版权归原作者所有。

---

​感谢LINUXDO社区开发者对本项目的支持​ [**LINUX.DO**](https://linux.do)

<p align="center">
  <sub>Built with ❤️ on Cloudflare's edge network</sub>
</p>

