# Maintenance Guide

## Source of truth

- Runtime code: `src/`
- Cloudflare configuration template: `wrangler.toml.example`
- Deployment: `.github/workflows/deploy.yml` — **本 fork 已改造为单环境部署**，配置全来自
  Secrets 与 Variables，不再硬编码域名与 D1 ID。详见 `docs/FORK_DEPLOY.md`。
- Database schema history: `database/migrations/`

Do not commit generated `wrangler.toml` or `dist/`.

## Required checks

```bash
npm ci
npm run check
```

`npm run check` performs TypeScript checking and a Wrangler dry-run build. It is not a behavioral test suite.

## Environment invariants

| Environment | SITE_ID | DEMO_MODE | Secrets | Writes |
| --- | --- | --- | --- | --- |
| Production | unique production value | unset | production-only | enabled |
| Demo (可选，本 fork 未部署) | unique demo value | `true` | none | blocked |

JWTs are scoped to `SITE_ID`. Changing `SITE_ID` invalidates existing sessions.

`APP_DOMAIN` must be the Worker hostname without a scheme. WebDAV uses it for
internal authenticated callbacks. `PUBLIC_DOMAIN` and `R2_PUBLIC_DOMAIN` are
reserved for direct object-storage domains. Only `R2_PUBLIC_DOMAIN` is read by the
download path; without it, non-image downloads fail with HTTP 500. When upgrading an
older deployment, move a Worker hostname from `PUBLIC_DOMAIN` to `APP_DOMAIN`; keep
`PUBLIC_DOMAIN` only when it is an actual object-storage domain. Public multipart
uploads are capped by `PUBLIC_UPLOAD_MAX_BYTES` (2 GiB by default), bound to a
24-hour capability, and must use 20 MiB parts except for the final part. New upload
initialization also opportunistically aborts up to four expired multipart sessions.

## Cloudflare Free plan constraints

Workers Free allows **10 ms CPU per request**. Two upstream defaults exceeded it and
were fixed in this fork — see `docs/FORK_DEPLOY.md` §8b for measurements:

- `PASSWORD_ITERATIONS` in `src/auth.ts` (210,000 → 5,000; ~79 ms → ~2 ms CPU)
- The storage-capacity diagnostic endpoint, which must use `StorageEngine.sumUsage()` rather than
  `list()` (the latter calls `toISOString()` per object: ~54 ms at 100k objects)

Also note: external subrequests are capped at 50/request, so batch deletes against an
S3 backend should stay under ~50 files; and WebDAV PUT/COPY buffer whole objects in
memory with no size guard.

## Database changes

Create a new numbered SQL file instead of editing an applied migration. CI applies migrations when `CLOUDFLARE_D1_API_TOKEN` is configured with D1 Edit permission; otherwise apply them manually before deploying code:

```bash
npx wrangler d1 migrations apply META_DB --remote
```

Log tables (`_ul_logs/`, `_dl_logs/`, moderation logs) are **never pruned automatically**.
Clear them from the dashboard periodically; the free D1 database cap is 500 MB.

## Branches

`main` is the deployment source. The remote `demo` branch is legacy and does not deploy; delete it only after confirming that no historical work is needed.

## Release verification

- Production root loads and unauthenticated admin APIs return 401.
- Admin login works (Turnstile is required on the login endpoint).
- Upload a **non-image** file and download it — this proves `R2_PUBLIC_DOMAIN` is set.
- The dashboard opens the Cloudflare R2 usage console from the top-right link; it does not display a local quota card.
- If a Demo is deployed: dashboard loads without credentials, API writes return 403,
  and Demo uses separate D1/R2 resources and a different `SITE_ID`.
