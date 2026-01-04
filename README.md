# zhangshuming技术博客

## 功能特性

### 文章分页功能
- 每页显示10篇文章
- 支持分类筛选和标签筛选
- 智能分页导航，最多显示5个页码按钮
- 响应式设计，支持移动端

### 分页控件特性
- 显示当前页信息（如：显示第1-10篇，共25篇）
- 上一页/下一页按钮
- 页码按钮（当前页高亮显示）
- 省略号显示（当页码过多时）
- 第一页和最后一页快速跳转

### 使用方法
1. 在文章列表页面，选择分类（前端、后端、其他、工具）
2. 可选择标签进行进一步筛选
3. 使用分页控件浏览不同页面的文章
4. 点击页码按钮或上一页/下一页按钮进行导航

### 技术实现
- 使用JavaScript实现客户端分页
- 支持动态筛选和分页
- 响应式CSS样式
- 平滑滚动到文章列表顶部

## 开发说明
- 分页大小：`postsPerPage = 10`
- 最大显示页码：5个
- 支持分类和标签组合筛选
- 自动重置分页状态

## 文件结构
- `_layouts/post.html` - 主要布局文件，包含分页逻辑
- `posts-data.json` - 文章数据源
- `css/style.scss` - 样式文件

## 部署说明

### Cloudflare Pages 部署

项目使用 GitHub Actions 自动构建并部署到 Cloudflare Pages。

#### 前置准备

1. **获取 Cloudflare API Token**
   - 登录 [Cloudflare Dashboard](https://dash.cloudflare.com/)
   - 进入 **My Profile** > **API Tokens** > **Create Token**
   - 使用 **Edit Cloudflare Workers** 模板或自定义权限（Account: Cloudflare Pages:Edit）

2. **获取 Account ID**
   - 在 Cloudflare Dashboard 右侧边栏可以看到 **Account ID**

3. **配置 GitHub Secrets**
   - 进入 GitHub 仓库 **Settings** > **Secrets and variables** > **Actions**
   - 添加以下 secrets：
     - `CLOUDFLARE_API_TOKEN`: Cloudflare API Token
     - `CLOUDFLARE_ACCOUNT_ID`: Cloudflare Account ID

#### 自动部署

- 推送代码到 `main` 分支会自动触发构建和部署
- 也可以在 GitHub Actions 中手动触发工作流

详细部署指南请参考 [cloudflare-pages.md](./cloudflare-pages.md)

### 本地开发

```bash
# 安装依赖
gem install bundler
bundle install
npm install

# 启动本地服务器
bundle exec jekyll serve

# 构建静态站点
bundle exec jekyll build
```
