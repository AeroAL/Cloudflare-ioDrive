# Fork 部署指南（ioDrive on your own Cloudflare）

本文档说明如何把本仓库部署到你自己的 Cloudflare 账号与自有域名。本仓库是
[devhunk/Cloudflare-ioDrive](https://github.com/devhunk/Cloudflare-ioDrive) 的 fork，
已做以下改动以支持独立部署：

- `.github/workflows/deploy.yml` 重写为**单环境、全参数化**部署（上游版本硬编码了作者的
  `drive.iodevo.com` / `zone_name` / R2 桶名，以及一个属于作者账号的 demo D1 UUID）。
- `package.json` 的 `wrangler` 从 `^4.107.0` 升到 `^4.131.1`，并同步
  `@cloudflare/workers-types` 到 `^5.20260911.1`。原因：4.107.0 内置的 workerd 只支持到
  compatibility date `2026-07-08`，而本项目使用 `2026-09-12`，导致 `wrangler dev` 无法启动。

---

## 1. 前置条件

- 一台装有 Node ≥ 22 的机器（本项目 `engines` 要求）
- Cloudflare 账号，且你的域名已托管在该账号（NS 已指向 Cloudflare）
- 一个 GitHub 账号（用于存放 fork 与运行 Actions）

## 2. 创建 Cloudflare 资源

### 2.1 D1 数据库

```bash
npx wrangler d1 create iodrive-meta
```

记下输出中的 `database_id`（形如 `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`），后面作为
`META_DB_ID` secret 使用。

> 表结构由 `database/migrations/0001_init.sql` 定义，CI 会自动执行迁移，无需手工建表。

### 2.2 R2 存储桶

```bash
npx wrangler r2 bucket create iodrive-prod
```

### 2.3 R2 公开访问域名（**必需**）

这是最容易踩的坑：本项目的下载实现**不使用签名 URL**。后台点「下载」时调用
`GET /api/download/presign/:key`，该接口只有在设置了 `R2_PUBLIC_DOMAIN` 时才会拼出
`https://<R2_PUBLIC_DOMAIN>/<key>`；否则会直接返回 **HTTP 500**
`存储凭证未配置（需要 R2 或 S3 兼容后端）`，而后台前端对该 500 是**静默失败**（点了没反应）。
分享页则会显示「生成下载链接失败」。

另外 `GET /f/:key` 只白名单图片（jpg/jpeg/png/gif/webp/bmp/ico），非图片会返回 403。
所以**不配公开域名的话，这个网盘只能看图，无法下载任意文件**。

在 Cloudflare Dashboard → R2 → 你的桶 → Settings → Public access 中二选一：

- **绑定自定义域名**（推荐，例 `r2.example.com`，需该域名在同一账号）：无速率限制，可用缓存。
- **启用 r2.dev 子域**：得到 `pub-xxxx.r2.dev`，仅供测试，有速率限制，不建议生产使用。

把选定的主机名记为 `R2_PUBLIC_DOMAIN`（**只填主机名，不带 `https://`**）。

> 注意：该文件按路径公开可访问，分享保护主要依赖 key 的不可猜测性，请知悉这一设计取舍。

### 2.4 Turnstile

在 Dashboard → Turnstile 新建一个站点，添加你的 Worker 域名，取得：

- **Site Key** → 仓库 secret `TURNSTILE_SITE_KEY`（会写入 `[vars]`）
- **Secret Key** → 仓库 secret `TURNSTILE_SECRET`

Turnstile 对**公共上传页 `/upload` 与分享下载是强校验**：未配置时 `/upload` 提交会返回
400 `缺少人机验证`，分享页下载也会失败。仅后台登录能优雅降级。因此生产部署必须配置。

### 2.5 API Token

在 Dashboard → My Profile → API Tokens 创建 Token，权限至少：

| 范围 | 权限 |
| --- | --- |
| Account | Workers Scripts: Edit |
| Account | D1: Edit |
| Account | R2: Edit（CI 会自动建桶，若桶已存在可省略） |
| Account | Account Settings: Read |
| Zone（你的域名） | Workers Routes: Edit |
| Zone（你的域名） | Zone: Read |

另外记下 **Account ID**（Dashboard 右侧栏可见）。

## 3. 配置仓库 Secrets 与 Variables

在 fork 仓库 → Settings → Secrets and variables → Actions：

### Secrets

| 名称 | 必需 | 说明 |
| --- | --- | --- |
| `CLOUDFLARE_API_TOKEN` | ✅ | 上面创建的 Token |
| `CLOUDFLARE_ACCOUNT_ID` | ✅ | 你的 Account ID |
| `META_DB_ID` | ✅ | 2.1 创建的 D1 database_id |
| `ADMIN_PASS` | ✅ | 后台管理员密码 |
| `JWT_SECRET` | ✅ | 随机 32 字节 hex，见下方生成命令 |
| `TURNSTILE_SITE_KEY` | ✅ | 2.4 的 Site Key |
| `TURNSTILE_SECRET` | ✅ | 2.4 的 Secret Key |
| `CLOUDFLARE_D1_API_TOKEN` | 可选 | 用于自动迁移；不填则回退用 `CLOUDFLARE_API_TOKEN` |
| `CACHE_KV_ID` | 可选 | 填了就启用 KV 缓存加速 |

生成 `JWT_SECRET`：

```bash
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

### Variables（注意是 Variables，不是 Secrets）

| 名称 | 必需 | 示例 | 说明 |
| --- | --- | --- | --- |
| `DEPLOY_DOMAIN` | ✅ | `drive.example.com` | Worker 对外域名，不带 `https://`。同时用作 `APP_DOMAIN`（WebDAV 内部回调依赖） |
| `R2_BUCKET` | ✅ | `iodrive-prod` | R2 桶名 |
| `R2_PUBLIC_DOMAIN` | ✅ | `r2.example.com` | 2.3 的公开域名 |
| `WORKER_NAME` | 可选 | `iodrive` | 默认 `iodrive` |
| `SITE_ID` | 可选 | `production` | 默认 `production`；决定 JWT 与缓存隔离，改动会使已登录会话失效 |
| `ADMIN_USER` | 可选 | `admin` | 默认 `admin` |
| `RATE_LIMITER_NAMESPACE_ID` | 可选 | `1001` | 填了才启用 Workers 限流绑定 |

## 4. 触发部署

- 推送到 `main`，或在 Actions 页面手动 `Run workflow`。
- 流程：`quality`（类型检查 + dry-run 构建）→ `deploy`（生成 `wrangler.toml` → 确保 R2 桶
  → 应用 D1 迁移 → 上传 secrets → `wrangler deploy`）。
- 部署使用 `routes = [{ pattern = "<DEPLOY_DOMAIN>", custom_domain = true }]`，由 wrangler
  自动创建 DNS 记录与证书，**无需预先手工添加 DNS**。

## 5. 验收清单

- [ ] `https://<DEPLOY_DOMAIN>` 打开后台登录页
- [ ] 用 `ADMIN_USER` / `ADMIN_PASS` 登录成功
- [ ] 上传一个**非图片**文件（如 .zip / .pdf），点下载能成功
      （这一条专门验证 `R2_PUBLIC_DOMAIN` 已生效）
- [ ] 上传一张图片，`/f/<key>` 能直接打开
- [ ] 创建分享链接，在分享页通过 Turnstile 后可下载
- [ ] `/upload` 公共上传页通过 Turnstile 后可上传

## 6. 本地开发

```bash
cp .dev.vars.example .dev.vars   # 填入 ADMIN_PASS / JWT_SECRET 等
cp wrangler.toml.example wrangler.toml   # 本地可把 database_id 填占位 UUID
npx wrangler d1 migrations apply META_DB --local
npm run dev
```

`wrangler.toml` 与 `.dev.vars` 已在 `.gitignore` 中，不会被提交。

## 7. 已知上游行为与注意事项

| 现象 | 说明 |
| --- | --- |
| 非图片下载返回 500 / 点下载无反应 | 未配置 `R2_PUBLIC_DOMAIN`。见 2.3 |
| `R2_ACCESS_KEY` / `R2_SECRET_KEY` | **残留字段，源码未读取**。使用 R2 原生绑定无需任何 S3 密钥，不必配置 |
| 控制台「R2 内置存储」卡片不显示 | 上游 bug：前端读 `r2Available`，但后端从不返回该字段。仅外观问题 |
| `STORAGE_CONFIG` 的 `primary` / `sync` 无效 | 上传路径未使用这两个标志；在 R2 绑定模式下**所有**已配置的 S3 后端都会被双写 |
| S3 同步失败 | 仅 `console.error`，无重试/回填，镜像可能静默漂移 |
| `PUBLIC_DOMAIN` | 只影响 imgbed/gallery/random 等公开 URL 辅助；**下载路径只读 `R2_PUBLIC_DOMAIN`**。两者设为同值最稳妥 |

## 8. 与上游同步

```bash
git remote add upstream https://github.com/devhunk/Cloudflare-ioDrive.git
git fetch upstream
git merge upstream/main
```

同步时预计会产生冲突的文件：`.github/workflows/deploy.yml`（已重写）、`package.json`
（wrangler 版本已升）。其余文件应能正常合并。

> GPL-3.0：本 fork 需同样以 GPL-3.0 分发。
