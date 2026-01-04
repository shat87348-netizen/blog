# Cloudflare Pages 部署指南

## 前置准备

### 1. 获取 Cloudflare API Token

1. 登录 [Cloudflare Dashboard](https://dash.cloudflare.com/)
2. 进入 **My Profile** > **API Tokens**
3. 点击 **Create Token**
4. 使用 **Edit Cloudflare Workers** 模板，或者自定义权限：
   - Account: Cloudflare Pages:Edit
   - Zone: 如果需要自定义域名，添加 Zone:Read 权限
5. 复制生成的 API Token

### 2. 获取 Account ID

1. 在 Cloudflare Dashboard 右侧边栏可以看到 **Account ID**
2. 复制 Account ID

### 3. 在 GitHub 仓库中配置 Secrets

1. 进入 GitHub 仓库
2. 点击 **Settings** > **Secrets and variables** > **Actions**
3. 点击 **New repository secret**，添加以下 secrets：
   - `CLOUDFLARE_API_TOKEN`: 你的 Cloudflare API Token
   - `CLOUDFLARE_ACCOUNT_ID`: 你的 Cloudflare Account ID

## 首次部署

### 方式一：使用 GitHub Actions（推荐）

1. 推送代码到 `main` 分支
2. GitHub Actions 会自动触发构建和部署
3. 部署完成后，在 Cloudflare Pages 控制台可以看到部署状态

### 方式二：在 Cloudflare Pages 中连接 GitHub

1. 登录 [Cloudflare Pages](https://dash.cloudflare.com/?to=/:account/pages)
2. 点击 **Create a project** > **Connect to Git**
3. 选择你的 GitHub 仓库
4. 配置构建设置：
   - **Framework preset**: Jekyll
   - **Build command**: `bundle exec jekyll build`
   - **Build output directory**: `_site`
   - **Root directory**: `/` (留空)
5. 点击 **Save and Deploy**

## 自动部署

配置完成后，每次推送到 `main` 分支都会自动触发部署。

## 自定义域名

1. 在 Cloudflare Pages 项目设置中
2. 点击 **Custom domains**
3. 添加你的自定义域名
4. 按照提示配置 DNS 记录

## 环境变量

如果需要设置环境变量（如 `JEKYLL_ENV=production`），可以在 Cloudflare Pages 项目设置中添加。

## 故障排查

### 构建失败

1. 检查 GitHub Actions 日志
2. 确认 Ruby 和 Node.js 版本兼容
3. 检查 `Gemfile` 和 `package.json` 是否正确

### 部署失败

1. 检查 Cloudflare API Token 和 Account ID 是否正确
2. 确认项目名称 `zhangshuming-blog` 是否已创建
3. 检查 Cloudflare Pages 控制台的部署日志

