# Astro Theme Pure 开发文档

## 项目概述

这是一个基于 Astro 框架的静态博客主题，名为 **Astro Theme Pure**。它是一个轻量、快速且功能强大的博客和文档主题，专为二次开发和使用而设计。

### 核心特性

- **高性能**：基于 Astro 的静态站点生成，加载速度极快
- **响应式设计**：完美适配各种设备尺寸
- **全站搜索**：集成 Pagefind 实现客户端搜索
- **SEO 友好**：自动生成 sitemap、RSS 订阅
- **组件丰富**：提供大量可复用的 UI 组件
- **易于定制**：通过配置文件轻松自定义主题

## 技术栈

### 核心框架
- **Astro** `^5.12.0` - 主框架，负责静态站点生成
- **TypeScript** `^5.8.3` - 类型安全的 JavaScript 超集
- **UnoCSS** - 原子化 CSS 引擎（替代 Tailwind CSS）

### 样式系统
- **UnoCSS PresetMini** - 基础样式预设
- **UnoCSS Typography** - 排版样式预设
- **CSS 变量** - 主题颜色系统

### 内容管理
- **Astro Content Collections** - 内容集合管理
- **Markdown/MDX** - 内容格式支持
- **Zod** - 内容 schema 验证

### 构建工具
- **Bun** - 包管理器和运行时（推荐）
- **Vite** - 构建工具（Astro 内置）
- **Sharp** - 图像处理优化

### 开发工具
- **ESLint** - 代码检查
- **Prettier** - 代码格式化
- **Astro Check** - TypeScript 类型检查

## 项目结构

```
pengJiang0930.github.io/
├── .github/                    # GitHub 配置
├── packages/                   # 本地包（monorepo）
│   └── pure/                   # 核心主题包
│       ├── components/         # 主题组件
│       ├── libs/               # 工具库
│       ├── plugins/            # 插件
│       ├── schemas/            # 内容 schema
│       ├── scripts/            # CLI 脚本
│       ├── types/              # TypeScript 类型
│       └── utils/              # 工具函数
├── public/                     # 静态资源
│   ├── favicon/                # 网站图标
│   ├── fonts/                  # 字体文件
│   ├── icons/                  # 图标
│   ├── images/                 # 图片资源
│   └── scripts/                # 客户端脚本
├── src/                        # 源代码
│   ├── assets/                 # 静态资源（构建时处理）
│   │   ├── styles/             # 全局样式
│   │   └── images/             # 图片资源
│   ├── components/             # 页面组件
│   │   ├── about/              # 关于页面组件
│   │   ├── home/               # 首页组件
│   │   ├── links/              # 链接页面组件
│   │   ├── projects/           # 项目页面组件
│   │   └── waline/             # 评论系统组件
│   ├── content/                # 内容目录
│   │   ├── blog/               # 博客文章
│   │   └── docs/               # 文档内容
│   ├── layouts/                # 布局组件
│   ├── pages/                  # 页面路由
│   │   ├── blog/               # 博客页面
│   │   ├── archives/           # 归档页面
│   │   ├── tags/               # 标签页面
│   │   └── search/             # 搜索页面
│   ├── plugins/                # 自定义插件
│   └── site.config.ts          # 站点配置
├── astro.config.ts             # Astro 配置
├── tsconfig.json               # TypeScript 配置
├── uno.config.ts               # UnoCSS 配置
└── package.json                # 项目配置
```

## 核心配置

### 1. 站点配置 (`src/site.config.ts`)

这是主要的配置文件，包含以下部分：

#### 主题配置 (`theme`)
```typescript
export const theme: ThemeUserConfig = {
  title: "嘭嘭's Blog",           // 网站标题
  author: 'Peng Jiang',                // 作者名称
  description: 'Stay hungry, stay foolish', // 网站描述
  favicon: '/favicon/favicon.ico',     // 网站图标
  locale: {
    lang: 'en-US',                     // 语言设置
    dateLocale: 'en-US',               // 日期格式
    dateOptions: { ... }               // 日期选项
  },
  logo: {
    src: 'public/images/gdjd.jpg',     // Logo 图片
    alt: 'Avatar'
  },
  header: {
    menu: [                            // 导航菜单
      { title: 'Blog', link: '/blog' },
      { title: 'About', link: '/' }
    ]
  },
  footer: {
    year: `© ${new Date().getFullYear()}`,
    links: [ ... ],                    // 底部链接
    credits: true,                     // 显示主题信息
    social: { github: '...' }          // 社交链接
  },
  content: {
    externalLinks: { ... },            // 外部链接样式
    blogPageSize: 8,                   // 每页文章数
    share: ['weibo', 'x', 'bluesky']   // 分享按钮
  }
}
```

#### 集成配置 (`integ`)
```typescript
export const integ: IntegrationUserConfig = {
  links: {
    logbook: [ ... ],                  // 友链日志
    applyTip: [ ... ]                  // 友链申请信息
  },
  pagefind: true,                      // 启用搜索
  quote: {
    server: 'https://v1.hitokoto.cn/?c=i', // 随机语录 API
    target: `(data) => data.hitokoto || 'Error'`
  },
  typography: {
    class: 'prose text-base text-muted-foreground',
    blockquoteStyle: 'italic',
    inlineCodeBlockStyle: 'modern'
  },
  mediumZoom: {
    enable: true,                      // 启用图片缩放
    selector: '.prose .zoomable'
  },
  waline: {
    enable: false,                     // 评论系统
    server: 'https://...',
    emoji: ['bmoji', 'weibo']
  }
}
```

### 2. Astro 配置 (`astro.config.ts`)

```typescript
export default defineConfig({
  site: 'https://pengJiang0930.github.io',
  trailingSlash: 'never',
  output: 'static',
  image: {
    responsiveStyles: true,
    service: { entrypoint: 'astro/assets/services/sharp' }
  },
  integrations: [
    AstroPureIntegration(config)       // 主题集成
  ],
  prefetch: true,
  markdown: {
    remarkPlugins: [remarkMath],       // 数学公式支持
    rehypePlugins: [ ... ],            // 标题 ID、自动链接等
    shikiConfig: {
      themes: {
        light: 'github-light',
        dark: 'github-dark'
      },
      transformers: [ ... ]            // 代码块增强
    }
  }
})
```

### 3. UnoCSS 配置 (`uno.config.ts`)

```typescript
export default defineConfig({
  presets: [
    presetMini(),                      // 基础样式
    presetTypography(typographyConfig) // 排版样式
  ],
  theme: {
    colors: themeColors                // 主题颜色系统
  },
  safelist: [ ... ]                    // 安全列表
})
```

## 内容管理

### 博客文章

博客文章位于 `src/content/blog/` 目录，支持 Markdown 和 MDX 格式。

#### 文章 Frontmatter

```yaml
---
title: "文章标题"                    # 必填，最大 60 字符
description: "文章描述"              # 必填，最大 160 字符
publishDate: 2024-01-01              # 必填，发布日期
updatedDate: 2024-01-02              # 可选，更新日期
heroImage:                           # 可选，封面图片
  src: ./image.jpg
  alt: "图片描述"
  width: 800
  height: 400
tags: ["tag1", "tag2"]               # 标签
language: "zh-CN"                    # 可选，语言
draft: false                         # 是否为草稿
comment: true                        # 是否启用评论
---
```

#### 文章组织

```
src/content/blog/
├── 文章标题/
│   ├── index.md
│   └── images/
│       └── cover.jpg
└── 另一篇文章.md
```

### 文档内容

文档位于 `src/content/docs/` 目录，用于创建结构化文档。

#### 文档 Frontmatter

```yaml
---
title: "文档标题"
description: "文档描述"
publishDate: 2024-01-01
updatedDate: 2024-01-02
tags: ["tag1", "tag2"]
draft: false
order: 1                             # 排序权重
---
```

## 布局系统

### 可用布局

1. **BaseLayout** - 基础布局，包含头部和底部
2. **BlogPost** - 博客文章布局
3. **CommonPage** - 通用页面布局
4. **ContentLayout** - 内容页面布局
5. **IndividualPage** - 独立页面布局

### 使用示例

```astro
---
import BaseLayout from '@/layouts/BaseLayout.astro'
---

<BaseLayout meta={{ title: '页面标题', description: '页面描述' }}>
  <main>
    <!-- 页面内容 -->
  </main>
</BaseLayout>
```

## 组件系统

### 主题组件

主题提供丰富的组件，分为以下几类：

#### 基础组件 (`astro-pure/components/basic`)
- `Header` - 页面头部
- `Footer` - 页面底部
- `ThemeProvider` - 主题切换

#### 页面组件 (`astro-pure/components/pages`)
- `PostPreview` - 文章预览
- `Paginator` - 分页器

#### 用户组件 (`astro-pure/user`)
- `Button` - 按钮
- `Card` - 卡片
- `Icon` - 图标
- `Label` - 标签

#### 高级组件 (`astro-pure/advanced`)
- `Quote` - 随机语录
- `GithubCard` - GitHub 卡片
- `LinkPreview` - 链接预览
- `QRCode` - 二维码

### 自定义组件

在 `src/components/` 目录下创建自定义组件：

```astro
---
// src/components/MyComponent.astro
interface Props {
  title: string
  description?: string
}

const { title, description } = Astro.props
---

<div class="my-component">
  <h2>{title}</h2>
  {description && <p>{description}</p>}
</div>

<style>
  .my-component {
    /* 组件样式 */
  }
</style>
```

## 样式系统

### 颜色主题

使用 CSS 变量定义颜色系统：

```css
:root {
  --background: 210 33% 99%;
  --foreground: 240 10% 3.9%;
  --primary: 200 29% 45%;
  --primary-foreground: 0 0% 92.5%;
  /* ... 更多颜色变量 */
}

.dark {
  --background: 240 20.54% 5.2%;
  --foreground: 0 0% 98%;
  --primary: 195 95% 85%;
  /* ... 暗色主题变量 */
}
```

### UnoCSS 类名

使用 UnoCSS 原子化类名：

```html
<div class="flex items-center gap-4 p-4 bg-card rounded-lg">
  <span class="text-lg font-semibold text-foreground">标题</span>
  <p class="text-muted-foreground">描述文本</p>
</div>
```

### 自定义样式

在 `src/assets/styles/app.css` 中添加全局样式：

```css
/* 自定义字体 */
@font-face {
  font-family: 'CustomFont';
  src: url('/fonts/CustomFont.woff2') format('woff2');
}

/* 自定义组件样式 */
.my-custom-component {
  background: hsl(var(--card));
  color: hsl(var(--card-foreground));
  border-radius: var(--radius);
}
```

## 开发指南

### 环境要求

- **Node.js** 18.0.0+
- **Bun**（推荐）或 npm/yarn/pnpm

### 安装和运行

```bash
# 克隆项目
git clone https://github.com/pengJiang0930/pengJiang0930.github.io.git
cd pengJiang0930.github.io

# 安装依赖
bun install

# 启动开发服务器
bun dev

# 构建生产版本
bun run build

# 预览生产版本
bun preview

# 创建新文章
bun new
```

### 开发流程

1. **创建新文章**
   ```bash
   bun new
   # 或手动创建 src/content/blog/my-post.md
   ```

2. **添加新页面**
   在 `src/pages/` 目录下创建 `.astro` 文件：
   ```astro
   ---
   // src/pages/my-page.astro
   import BaseLayout from '@/layouts/BaseLayout.astro'
   ---
   
   <BaseLayout meta={{ title: 'My Page' }}>
     <main>
       <h1>My Custom Page</h1>
     </main>
   </BaseLayout>
   ```

3. **创建自定义组件**
   在 `src/components/` 目录下创建组件文件。

4. **修改主题配置**
   编辑 `src/site.config.ts` 文件。

5. **自定义样式**
   编辑 `src/assets/styles/app.css` 或使用 UnoCSS 类名。

### 代码规范

项目使用 ESLint 和 Prettier 进行代码规范：

```bash
# 检查代码规范
bun lint

# 格式化代码
bun format

# 类型检查
bun check

# 一键修复
bun yijiansilian
```

## 部署

### GitHub Pages

项目已配置为 GitHub Pages 部署：

1. 推送代码到 GitHub
2. 在仓库设置中启用 GitHub Pages
3. 选择 `gh-pages` 分支或 GitHub Actions

### Vercel

项目已配置 Vercel 部署：

1. 连接 GitHub 仓库到 Vercel
2. Vercel 会自动检测 Astro 项目
3. 自动部署和域名配置

### 其他平台

构建命令：
```bash
bun run build
```

输出目录：`dist/`

## 常见问题

### 如何添加新页面？

在 `src/pages/` 目录下创建 `.astro` 文件，Astro 会自动生成对应路由。

### 如何修改主题颜色？

编辑 `src/assets/styles/app.css` 中的 CSS 变量。

### 如何添加评论系统？

在 `src/site.config.ts` 中配置 `waline` 集成。

### 如何自定义导航菜单？

编辑 `src/site.config.ts` 中的 `header.menu` 配置。

### 如何添加数学公式支持？

项目已集成 KaTeX，直接在 Markdown 中使用 LaTeX 语法：

```markdown
行内公式：$E = mc^2$

块级公式：
$$
\sum_{i=1}^{n} i = \frac{n(n+1)}{2}
$$
```

## 扩展开发

### 创建自定义集成

```typescript
// src/integrations/my-integration.ts
import type { AstroIntegration } from 'astro'

export default function myIntegration(): AstroIntegration {
  return {
    name: 'my-integration',
    hooks: {
      'astro:config:setup': ({ updateConfig }) => {
        // 配置修改
      },
      'astro:build:done': ({ dir }) => {
        // 构建后处理
      }
    }
  }
}
```

### 创建自定义插件

```typescript
// src/plugins/my-plugin.ts
import type { Plugin } from 'unified'

export default function myPlugin(): Plugin {
  return (tree) => {
    // AST 处理
  }
}
```

## 相关资源

- [Astro 官方文档](https://docs.astro.build/)
- [UnoCSS 官方文档](https://unocss.dev/)
- [Astro Theme Pure 文档](https://astro-pure.js.org/docs)
- [项目 GitHub 仓库](https://github.com/cworld1/astro-theme-pure)

## 许可证

本项目基于 Apache 2.0 许可证。