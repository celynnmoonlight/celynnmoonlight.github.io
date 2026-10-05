# 部署到 GitHub Pages

本项目已经配置好了 GitHub Pages 的 GitHub Actions 部署流程。部署入口是 `.github/workflows/deploy.yml`，网站内容推送到 `main` 分支后会自动构建并发布。

## 一次性配置

1. 将项目推送到 GitHub 仓库，并确认默认分支是 `main`。
2. 打开仓库的 **Settings → Pages**。
3. 在 **Build and deployment → Source** 中选择 **GitHub Actions**。
4. 打开 **Settings → Actions → General**，确认 Actions 没有被禁用。
5. 在 **Settings → Actions → General → Workflow permissions** 中选择 **Read and write permissions**（如果组织策略不允许，至少确保 Pages 部署所需权限可用）。

项目的 `hugoblox.yaml` 已设置：

```yaml
deploy:
  host: 'github-pages'
```

因此 `deploy.yml` 会使用 GitHub Pages 部署，而不是 Netlify 或其他平台。

## 发布网站

修改内容后提交并推送到 `main`：

```powershell
git add .
git commit -m "update website"
git push origin main
```

推送后，打开仓库的 **Actions** 页面，找到 **Deploy website to GitHub Pages** 工作流。它会依次完成：

1. 读取 `hugoblox.yaml` 中的 Hugo 版本（当前为 `0.167.0`）。
2. 安装 Node.js、pnpm、Hugo Extended 和项目依赖。
3. 使用 Hugo 构建到 `public/`。
4. 使用 Pagefind 生成站内搜索索引。
5. 将构建结果发布到 GitHub Pages。

工作流成功后，在 **Settings → Pages** 页面可以看到网站地址。通常是：

```text
https://<你的用户名>.github.io/<仓库名>/
```

如果仓库名是 `<你的用户名>.github.io`，地址通常是：

```text
https://<你的用户名>.github.io/
```

## 手动重新部署

不修改代码也可以重新发布：

1. 打开仓库的 **Actions** 页面。
2. 选择 **Deploy website to GitHub Pages**。
3. 点击 **Run workflow**。
4. 选择 `main` 分支并运行。

## 本地检查后再推送

本地需要安装 Hugo Extended、Go、Node.js 和 pnpm。项目指定的 pnpm 版本在 `package.json` 中。

```powershell
pnpm install
hugo server --disableFastRender
```

浏览器打开 Hugo 输出的本地地址（通常是 `http://localhost:1313/`）。生产构建可以运行：

```powershell
hugo --minify
pnpm run pagefind
```

## 自定义域名（可选）

如果要使用自己的域名：

1. 在仓库 **Settings → Pages → Custom domain** 填写域名并保存。
2. 在域名 DNS 中添加 GitHub Pages 要求的记录。
3. 等待 DNS 生效并启用 **Enforce HTTPS**。

不要把 `public/` 目录提交到仓库；它由 GitHub Actions 每次构建生成。

## 常见问题

- **Actions 没有运行**：确认代码确实推送到了 `main`，并检查仓库的 Actions 是否启用。
- **Pages 显示没有部署源**：回到 **Settings → Pages**，将 Source 设置为 **GitHub Actions**。
- **构建失败**：先查看 Actions 中 `Build` 步骤的日志，重点检查 Hugo 模板、Markdown、Go Modules 和 pnpm 依赖错误。
- **页面打开但资源路径错误**：确认访问的是 GitHub Pages 显示的完整地址，并让工作流使用 `configure-pages` 生成的 base URL；本项目已配置该步骤。
