# 终身学习

基于 [Astro](https://astro.build) 构建的个人技术博客，记录一个终身学习者的技术成长：AI 实践、工程落地与认知升级。

## 特性

- **Markdown / MDX 写作** + Shiki 代码高亮
- **SEO 友好**：静态 HTML 生成 + Sitemap + RSS
- **Pagefind 搜索**（免后端模糊搜索）
- **Giscus 评论**（预留，基于 GitHub Discussions）
- **零部署成本**：纯静态，可部署到 Cloudflare Pages / Vercel / GitHub Pages

## 快速开始

```bash
npm install
npm run dev          # 开发预览
npm run build:search  # 构建 + 生成搜索索引
```

详细配置请查看 [SETUP.md](./SETUP.md)