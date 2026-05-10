# 快速开始指南

## 5 分钟快速上手

### 1. 环境准备

确保已安装：
- Node.js 18.0.0+
- Bun（推荐）或 npm

### 2. 安装项目

```bash
# 克隆项目
git clone https://github.com/pengJiang0930/pengJiang0930.github.io.git
cd pengJiang0930.github.io

# 安装依赖
bun install
```

### 3. 启动开发

```bash
# 启动开发服务器
bun dev
```

访问 `http://localhost:4321` 查看效果。

### 4. 创建第一篇文章

```bash
# 使用 CLI 创建
bun new

# 或手动创建
# 在 src/content/blog/ 目录下创建 .md 文件
```

文章模板：

```markdown
---
title: "我的第一篇文章"
description: "这是文章描述"
publishDate: 2024-01-01
tags: ["blog", "first-post"]
---

# 文章内容

这是文章正文。
```

### 5. 自定义配置

编辑 `src/site.config.ts`：

```typescript
export const theme: ThemeUserConfig = {
  title: "我的博客",
  author: '我的名字',
  description: '我的博客描述',
  // ... 其他配置
}
```

### 6. 构建部署

```bash
# 构建生产版本
bun run build

# 预览构建结果
bun preview
```

## 常用命令速查

| 命令 | 说明 |
|------|------|
| `bun dev` | 启动开发服务器 |
| `bun run build` | 构建生产版本 |
| `bun preview` | 预览生产版本 |
| `bun new` | 创建新文章 |
| `bun lint` | 代码检查 |
| `bun format` | 代码格式化 |
| `bun check` | TypeScript 检查 |
| `bun clean` | 清理构建文件 |

## 目录速查

| 目录 | 说明 |
|------|------|
| `src/content/blog/` | 博客文章 |
| `src/content/docs/` | 文档内容 |
| `src/pages/` | 页面路由 |
| `src/components/` | 自定义组件 |
| `src/layouts/` | 布局组件 |
| `src/assets/` | 静态资源 |
| `public/` | 公共资源 |
| `packages/pure/` | 主题核心包 |

## 配置文件速查

| 文件 | 说明 |
|------|------|
| `src/site.config.ts` | 站点配置 |
| `astro.config.ts` | Astro 配置 |
| `tsconfig.json` | TypeScript 配置 |
| `uno.config.ts` | UnoCSS 配置 |

## 快速自定义

### 修改网站标题

编辑 `src/site.config.ts`：

```typescript
title: "我的新标题"
```

### 修改导航菜单

编辑 `src/site.config.ts`：

```typescript
header: {
  menu: [
    { title: '首页', link: '/' },
    { title: '博客', link: '/blog' },
    { title: '关于', link: '/about' }
  ]
}
```

### 修改主题颜色

编辑 `src/assets/styles/app.css`：

```css
:root {
  --primary: 200 29% 45%;  /* 修改主色调 */
}
```

### 添加新页面

创建 `src/pages/my-page.astro`：

```astro
---
import BaseLayout from '@/layouts/BaseLayout.astro'
---

<BaseLayout meta={{ title: '我的页面' }}>
  <main>
    <h1>我的自定义页面</h1>
  </main>
</BaseLayout>
```

## 常见问题

### Q: 如何添加评论系统？

A: 编辑 `src/site.config.ts`，配置 `waline` 集成。

### Q: 如何添加数学公式？

A: 直接在 Markdown 中使用 LaTeX 语法，项目已集成 KaTeX。

### Q: 如何修改 Logo？

A: 替换 `public/images/` 中的图片，并在 `src/site.config.ts` 中更新 `logo` 配置。

### Q: 如何添加搜索功能？

A: 搜索功能已默认启用，使用 Pagefind 实现。

### Q: 如何部署到 GitHub Pages？

A: 推送代码到 GitHub，在仓库设置中启用 GitHub Pages。

## 获取帮助

- 查看完整文档：[DEVELOPMENT.md](./DEVELOPMENT.md)
- Astro 官方文档：https://docs.astro.build/
- UnoCSS 官方文档：https://unocss.dev/
- 项目 GitHub：https://github.com/cworld1/astro-theme-pure