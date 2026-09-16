# knowledge-base-keycloak

Knowledge Base 的 Keycloak **自定义镜像**与本地开发编排。

本仓库只负责：

- Keycloak 26 + [Cloudflare Turnstile SPI](https://github.com/zymlabs/keycloak-cloudflare-turnstile-provider)
- `kb` 主题（登录页 / Admin Console favicon）
- 推送 `v*` 标签时 CI 构建并发布到 GHCR

**不含** realm 导入、用户 bootstrap、CSP / Turnstile flow 等运行时配置（由部署环境或 Admin Console 处理）。

## 本地开发

```bash
cp .env.example .env   # 可选
docker compose up -d --build
```

Admin Console: http://localhost:8180

## 生产镜像

| 项 | 值 |
| --- | --- |
| Registry | `ghcr.io` |
| 镜像 | `ghcr.io/lennylin-lab/knowledge-base-keycloak` |
| 标签 | Git 标签 `v*`（如 `v1.0.0`） |

推送标签触发 [Release workflow](.github/workflows/release.yml)：构建镜像 → 推送 GHCR → 创建 GitHub Release。

```bash
git tag v1.0.0
git push origin v1.0.0
```

拉取：

```bash
docker pull ghcr.io/lennylin-lab/knowledge-base-keycloak:v1.0.0
```

## Turnstile（运行时配置）

镜像已预装 SPI 与 `kb` 主题，认证流 / CSP / Turnstile 密钥在 **Admin Console** 或部署脚本中配置。

单页登录（用户名/密码 + Turnstile 同一页）要点：

1. Realm **Themes** → Login theme = **`kb`**
2. **Authentication** → 复制 `browser` 为 `browser-turnstile`
3. 在 **`browser-turnstile forms`** 子 flow 内只保留 **Cloudflare Turnstile**（Required，Implementation = **Custom Theme**）
4. 不要把 Turnstile 放在 flow 顶层 Required，也不要放在 Username Password Form 之后
5. Realm **Security defenses** → CSP 需允许 `https://challenges.cloudflare.com`（Keycloak 26 不支持 `KC_SPI_*` 环境变量）

Dev 测试密钥：<https://developers.cloudflare.com/turnstile/troubleshooting/testing/>

## 目录结构

```text
Dockerfile              # KC 26 + Turnstile SPI + kb theme
docker-compose.yml      # dev: start-dev, :8180
assets/favicon.svg      # favicon 源文件
themes/kb/              # login（parent=cloudflare-turnstile）+ admin
scripts/build-theme-icons.sh   # 本地生成 favicon（dev volume 挂载用）
```

改图标后：

```bash
./scripts/build-theme-icons.sh
docker compose up -d --build
```
