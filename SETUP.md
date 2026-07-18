# 博客上线配置指南

> 如果你是从脚手架开始，这份文档告诉你“改哪两个地方就能上线”。

---

## 第 1 步：修改站点地址

打开 `astro.config.mjs`，把 `site` 改成你的真实域名（必填， sitemap 和 RSS 依赖它）：

```js
// astro.config.mjs
export default defineConfig({
  site: 'https://your-domain.com',  // ← 改成你的域名
  ...
});
```

---

## 第 2 步：开启评论（ Giscus）

### 2.1 准备 GitHub Discussions

1. 在你的博客**仓库页** → Settings → Features 中开启 **Discussions**
2. 在 [giscus.app](https://giscus.app) 填写以下信息：
   - **Repository**: 你的 `owner/repo`
   - **Discussion Category**: 选 Announcements 或 General
   - 勾选你需要的评论功能

### 2.2 填入配置

打开 `src/components/Comments.astro`，填入 4 个值：

```astro
const REPO = {
  dataRepo: '',       // ← 改成 "owner/repo"
  categoryId: '',      // ← 填 giscus.app 生成的 Discussion Category ID
  categoryTerm: '',    // ← 填 giscus.app 生成的 Category Term
  repoId: '',         // ← 填 giscus.app 生成的 Repository ID
};
```

填完后评论自动显示在每篇文章底部。

---

## 第 3 步：开启搜索（ Pagefind）

搜索已经配置好！只需两步：

1. 构建时使用 `build:search` 代替 `build`：
   ```bash
   npm run build:search   # = astro build && pagefind --site dist
   ```
2. 导航栏已有“搜索”入口，点进去即可使用。

---

## 第 4 步：部署

### 方案 A：Cloudflare Pages（推荐）

1. 登录 [Cloudflare Dashboard](https://dash.cloudflare.com/) → Pages → Create a project
2. 连接 GitHub 仓库，选择 `ai-engineer-blog`
3. 构建命令畕罕 `npm run build`
4. 输出目录 `dist`
5. 保存 → Cloudflare 自动构建部署

### 方案 B：Vercel

1. [vercel.com/import](https://vercel.com/import) → Import Git Repository
2. 选择 `ai-engineer-blog`
3. Framework Preset: Astro
4. Output Directory: `dist`
5. Deploy

### 方案 C：GitHub Pages

1. 仓库 Settings → Pages → Source → GitHub Actions
2. 在 `.github/workflows/` 下创建 CI 文件（可用 [astro-build-action](https://github.com/marketplace/actions/astro-build-action)）

---

## 后续可能的扩展

- 换主题色 / 暗色模式
- 插件系统 (Utterances / Disqus 替代 Giscus)
- KaTeX 数学公式
- 标签归档页
- OGP / Twitter Card 预览图