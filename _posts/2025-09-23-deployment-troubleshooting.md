---
title: Next.js 项目部署到 Cloudflare Pages 问题排查指南
date: 2025-09-23
categories: [其他]
tags: [Next.js, Cloudflare Pages, GitHub Actions, 部署问题]
---

# Next.js 项目部署到 Cloudflare Pages 问题排查指南

本文档记录了将 Next.js 15 项目部署到 Cloudflare Pages 过程中遇到的各种问题及其解决方案。

## 项目信息

- **框架**: Next.js 15.4.6
- **React**: 19.1.0
- **部署平台**: Cloudflare Pages
- **CI/CD**: GitHub Actions
- **构建工具**: @cloudflare/next-on-pages@1.13.16

---

## 问题 1: package-lock.json 与 package.json 不同步

### 错误信息

```
npm error `npm ci` can only install packages when your package.json and package-lock.json or npm-shrinkwrap.json are in sync.
npm error Missing: @cloudflare/next-on-pages@1.13.16 from lock file
npm error Missing: vercel@47.0.4 from lock file
npm error Missing: wrangler@4.54.0 from lock file
```

### 问题原因

- `package.json` 中添加了新依赖，但 `package-lock.json` 未更新
- Cloudflare 构建环境使用 `npm ci` 命令，要求 lock 文件与 `package.json` 完全同步

### 解决方案

**方案 1: 删除旧的 package-lock.json（推荐）**

让 Cloudflare 构建环境自动重新生成：

```bash
git rm package-lock.json
git commit -m "删除旧的 package-lock.json，让 Cloudflare 重新生成"
git push
```

**方案 2: 在本地更新 package-lock.json**

```bash
npm install
git add package-lock.json
git commit -m "更新 package-lock.json"
git push
```

### 注意事项

- 确保本地 Node.js 版本与 Cloudflare 构建环境一致（推荐 Node.js 20+）
- 如果本地环境有问题，优先使用方案 1

---

## 问题 2: wrangler.toml 配置无效

### 错误信息

```
A wrangler.toml file was found but it does not appear to be valid. 
Did you mean to use wrangler.toml to configure Pages? 
If so, then make sure the file is valid and contains the `pages_build_output_dir` property.
```

### 问题原因

- `wrangler.toml` 文件缺少 `pages_build_output_dir` 属性
- Cloudflare Pages 项目需要明确指定构建输出目录

### 解决方案

在 `wrangler.toml` 中添加 `pages_build_output_dir` 配置：

```toml
name = "metro1"
compatibility_date = "2024-01-01"
pages_build_output_dir = ".vercel/output/static"
```

### 完整配置示例

```toml
name = "metro1"
compatibility_date = "2024-01-01"
pages_build_output_dir = ".vercel/output/static"
compatibility_flags = ["nodejs_compat"]
```

---

## 问题 3: GitHub Actions 缺少依赖锁文件

### 错误信息

```
Error: Dependencies lock file is not found in /home/runner/work/metro1/metro1. 
Supported file patterns: package-lock.json,npm-shrinkwrap.json,yarn.lock
```

### 问题原因

- `actions/setup-node@v4` 配置了 `cache: 'npm'`，需要 lock 文件
- 但项目中删除了 `package-lock.json` 以解决 Cloudflare 部署问题

### 解决方案

**修改 GitHub Actions 工作流配置**

移除 npm 缓存配置，直接使用 `npm install`：

```yaml
- name: Setup Node.js
  uses: actions/setup-node@v4
  with:
    node-version: '20'
    # 不使用缓存，避免需要 lock 文件

- name: Install dependencies
  run: npm install
```

**或者创建最小化的 package-lock.json**

```json
{
  "name": "next-map",
  "version": "0.1.0",
  "lockfileVersion": 3,
  "requires": true,
  "packages": {}
}
```

GitHub Actions 运行 `npm install` 时会自动更新该文件。

---

## 问题 4: wrangler.toml 包含 Pages 不支持的配置

### 错误信息

```
✘ [ERROR] Running configuration file validation for Pages:
    - Configuration file for Pages projects does not support "build"
▲ [WARNING] Unexpected fields found in build field: "upload"
```

### 问题原因

- `wrangler.toml` 中包含了 `[build]` 和 `[build.upload]` 配置块
- 这些配置是用于 Cloudflare Workers 项目的，不是 Pages 项目
- Cloudflare Pages 项目不支持这些配置

### 解决方案

**移除 Pages 项目不支持的配置**

删除 `[build]` 和 `[build.upload]` 配置块：

```toml
# 错误配置（Workers 项目使用）
[build]
command = "npm run build && npm run build:cloudflare"
cwd = "."

[build.upload]
format = "service-worker"
dir = ".vercel/output/static"

# 正确配置（Pages 项目）
name = "metro1"
compatibility_date = "2024-01-01"
pages_build_output_dir = ".vercel/output/static"
compatibility_flags = ["nodejs_compat"]
```

### 说明

- **构建命令**：在 GitHub Actions 工作流中执行，不需要在 `wrangler.toml` 中配置
- **部署目录**：通过 `cloudflare/pages-action` 的 `directory` 参数指定
- **Pages 项目**：会自动处理静态文件和 Worker 脚本

---

## 问题 5: 缺少 Node.js 兼容性标志导致运行时错误

### 错误信息

```
▲ [WARNING] The package "node:buffer" wasn't found on the file system but is built into node.
  Your Worker may throw errors at runtime unless you enable the "nodejs_compat" compatibility flag.
▲ [WARNING] The package "node:async_hooks" wasn't found on the file system but is built into node.
  Your Worker may throw errors at runtime unless you enable the "nodejs_compat" compatibility flag.
```

### 问题原因

- Next.js 应用使用了 Node.js 内置模块（如 `node:buffer`、`node:async_hooks`）
- Cloudflare Workers 默认不支持 Node.js 内置模块
- 需要启用 `nodejs_compat` 兼容性标志

### 解决方案

**在 wrangler.toml 中添加兼容性标志**

```toml
name = "metro1"
compatibility_date = "2024-01-01"
pages_build_output_dir = ".vercel/output/static"
compatibility_flags = ["nodejs_compat"]
```

**或者在 Cloudflare Dashboard 中设置**

1. 登录 [Cloudflare Dashboard](https://dash.cloudflare.com/)
2. 进入 **Workers & Pages** > **Pages** > **metro1**
3. 进入 **Settings** > **Functions**
4. 在 **Compatibility Flags** 中添加 `nodejs_compat`

### 注意事项

- `nodejs_compat` 标志会增加 Worker 的启动时间
- 只有在使用 Node.js 内置模块时才需要启用
- 推荐在 `wrangler.toml` 中配置，便于版本控制

---

## 问题 6: 部署目录不正确导致界面问题

### 问题现象

- 部署成功，但页面无法正常显示
- JavaScript 文件加载失败
- Worker 脚本未正确部署

### 问题原因

- GitHub Actions 部署目录设置为 `.vercel/output/static`
- 但 `_worker.js`、`_routes.json` 等文件在 `.vercel/output` 目录下
- 只部署 `static` 目录会导致 Worker 脚本缺失

### 解决方案

**修改 GitHub Actions 部署目录**

```yaml
- name: Deploy to Cloudflare Pages
  uses: cloudflare/pages-action@v1
  with:
    apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
    accountId: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
    projectName: metro1
    directory: .vercel/output  # 使用整个 output 目录
    gitHubToken: ${{ secrets.GITHUB_TOKEN }}
```

### 目录结构说明

```
.vercel/output/
├── _worker.js          # Worker 脚本（必需）
├── _routes.json        # 路由配置（必需）
├── _headers            # HTTP 头配置（可选）
└── static/             # 静态文件目录
    ├── index.html
    ├── _next/
    └── ...
```

### 重要提示

- **必须部署整个 `.vercel/output` 目录**，不能只部署 `static` 子目录
- `_worker.js` 负责处理动态路由和 API 请求
- `_routes.json` 定义了哪些请求由 Worker 处理

---

## 完整配置文件参考

### wrangler.toml

```toml
name = "metro1"
compatibility_date = "2024-01-01"
pages_build_output_dir = ".vercel/output/static"
compatibility_flags = ["nodejs_compat"]
```

### .github/workflows/deploy.yml

```yaml
name: Deploy to Cloudflare Pages

on:
  push:
    branches:
      - main
      - master
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      deployments: write
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm install

      - name: Build Next.js
        run: npm run build

      - name: Build for Cloudflare Pages
        run: npm run build:cloudflare

      - name: Deploy to Cloudflare Pages
        uses: cloudflare/pages-action@v1
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          accountId: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
          projectName: metro1
          directory: .vercel/output
          gitHubToken: ${{ secrets.GITHUB_TOKEN }}
```

### package.json

```json
{
  "scripts": {
    "build": "next build",
    "build:cloudflare": "npx @cloudflare/next-on-pages"
  },
  "devDependencies": {
    "@cloudflare/next-on-pages": "^1.13.16"
  }
}
```

---

## 部署检查清单

在部署前，请确认以下配置：

- [ ] `wrangler.toml` 包含 `pages_build_output_dir` 配置
- [ ] `wrangler.toml` 不包含 `[build]` 配置块（Pages 项目不支持）
- [ ] `wrangler.toml` 包含 `compatibility_flags = ["nodejs_compat"]`（如果使用 Node.js 内置模块）
- [ ] GitHub Actions 部署目录设置为 `.vercel/output`（不是 `.vercel/output/static`）
- [ ] GitHub Secrets 中配置了 `CLOUDFLARE_API_TOKEN` 和 `CLOUDFLARE_ACCOUNT_ID`
- [ ] `package.json` 和 `package-lock.json` 同步（或删除 lock 文件让 Cloudflare 重新生成）

---

## 常见错误速查表

| 错误信息 | 原因 | 解决方案 |
|---------|------|---------|
| `package-lock.json` 不同步 | lock 文件过期 | 删除或更新 lock 文件 |
| `pages_build_output_dir` 缺失 | wrangler.toml 配置不完整 | 添加 `pages_build_output_dir` |
| `Configuration file does not support "build"` | 使用了 Workers 配置 | 删除 `[build]` 配置块 |
| `node:buffer wasn't found` | 缺少 Node.js 兼容性 | 添加 `nodejs_compat` 标志 |
| 页面无法显示 | 部署目录错误 | 使用 `.vercel/output` 而不是 `static` |

---

## 参考资源

- [Cloudflare Pages 文档](https://developers.cloudflare.com/pages/)
- [@cloudflare/next-on-pages 文档](https://github.com/cloudflare/next-on-pages)
- [Wrangler 配置参考](https://developers.cloudflare.com/workers/wrangler/configuration/)
- [Node.js 兼容性](https://developers.cloudflare.com/workers/runtime-apis/nodejs/)

---

## 更新日志

- **2025-09-23**: 初始版本，记录所有部署问题及解决方案

