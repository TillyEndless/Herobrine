---
title: 【CS日志】杂·早期blog
published: 2024-10-29T00:00:00.000Z
description: ''
image: ''
tags:
  - journal
category: CS日志
draft: false
lang: ''
---
***目录***

<!-- toc -->

- [10.28 搭建博客 学mac基本配件和应用](#1028-%E6%90%AD%E5%BB%BA%E5%8D%9A%E5%AE%A2-%E5%AD%A6mac%E5%9F%BA%E6%9C%AC%E9%85%8D%E4%BB%B6%E5%92%8C%E5%BA%94%E7%94%A8)
- [10.29 配置iTerm2](#1029-%E9%85%8D%E7%BD%AEiterm2)
- [10.30 侧边栏html编写 && TuringCourse资源cloning](#1030-%E4%BE%A7%E8%BE%B9%E6%A0%8Fhtml%E7%BC%96%E5%86%99--turingcourse%E8%B5%84%E6%BA%90cloning)
  * [安装自动markdown侧边栏](#%E5%AE%89%E8%A3%85%E8%87%AA%E5%8A%A8markdown%E4%BE%A7%E8%BE%B9%E6%A0%8F)
    + [GPT回答1](#gpt%E5%9B%9E%E7%AD%941)
    + [GPT回答2：](#gpt%E5%9B%9E%E7%AD%942)
- [const { headings, content } = Astro.props;](#const--headings-content---astroprops)
    + [额外：安装nvm](#%E9%A2%9D%E5%A4%96%E5%AE%89%E8%A3%85nvm)
  * [TuringCourse资源cloning](#turingcourse%E8%B5%84%E6%BA%90cloning)
- [11.15 添加密码](#1115-%E6%B7%BB%E5%8A%A0%E5%AF%86%E7%A0%81)
- [11.28 更换url](#1128-%E6%9B%B4%E6%8D%A2url)

<!-- tocstop -->

## 10.28 搭建博客 学mac基本配件和应用
## 10.29 配置iTerm2 
Link:  
https://sourabhbajaj.com/mac-setup/
https://qiita.com/ryamate/items/075c34fcf29d0889c15a

有时候https clone 不了：用ssh git clone可以成功

留档部分配置过程：（**fzf使用shell扩展，按照文档报错**）
不知道是否可行：
![1](/media/17291428769621/1.png)
![2](/media/17291428769621/2.png)
最后采用博客方法解决：
https://blog.csdn.net/weixin_34256074/article/details/88758143

## 10.30 侧边栏html编写 && TuringCourse资源cloning

### 安装自动markdown侧边栏
#### GPT回答1
要让网站显示 Markdown 大纲，通常可以采用以下几种方法：

1. 使用 JavaScript Markdown 库

使用 JavaScript 库如 Marked、Showdown 或 markdown-it 可以解析 Markdown 文件并将其转换成 HTML。这些库非常适合在网页中嵌入 Markdown 大纲并动态生成 HTML。

步骤

	1.	安装 Markdown 解析库，例如 Marked。
	2.	编写 JavaScript 脚本，解析 Markdown 内容并插入到页面中。

示例代码：

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Markdown Outline</title>
  <script src="https://cdn.jsdelivr.net/npm/marked/marked.min.js"></script>
</head>
<body>
  <div id="content"></div>

  <script>
    // 假设你的 Markdown 内容是一个字符串
    const markdownContent = `
    # 大纲标题
    ## 第一部分
    一些内容...
    ## 第二部分
    更多内容...
    `;

    // 将 Markdown 内容转换为 HTML 并插入到页面
    document.getElementById('content').innerHTML = marked.parse(markdownContent);
  </script>
</body>
</html>

2. 使用 Static Site Generators

如果你的网页是静态网站，可以使用 Jekyll、Hugo 或 Hexo 等生成器，这些工具可以自动将 Markdown 文件渲染成 HTML。

步骤

	1.	将 Markdown 文件放在特定目录中。
	2.	配置生成器的模板文件来引用大纲，并指定如何渲染 Markdown。

3. 使用 Markdown 支持的 Web 框架

一些前端框架如 VuePress 或 Docusaurus，直接支持 Markdown，你可以将大纲作为 Markdown 文件包含在项目中，框架会自动生成目录和 HTML 页面。

4. 使用服务器端解析

如果你使用的是 Node.js 或其他支持 Markdown 的服务器环境，可以在服务器端解析 Markdown 并将结果发送给前端。例如，使用 express-markdown 或 markdown-it 将 Markdown 文件渲染成 HTML。

步骤

	1.	安装 markdown-it：npm install markdown-it
	2.	在 Node.js 中编写代码，将 Markdown 文件解析成 HTML 并提供给前端。

const express = require('express');
const fs = require('fs');
const MarkdownIt = require('markdown-it');
const md = new MarkdownIt();

const app = express();

app.get('/outline', (req, res) => {
  const markdownContent = fs.readFileSync('./outline.md', 'utf8');
  const htmlContent = md.render(markdownContent);
  res.send(htmlContent);
});

app.listen(3000, () => console.log('Server is running on port 3000'));

通过这些方法，您可以在网页上显示 Markdown 格式的大纲内容。

#### GPT回答2：
在 Astro 中显示每个 Markdown 文件的侧边大纲，你可以使用 Astro 结合 @astrojs/mdx 插件和 JavaScript 来解析 Markdown 文件，提取标题并生成一个可点击的侧边大纲。这里是一个实现方法：

1. 安装必要的插件

使用 @astrojs/mdx 插件使 Astro 支持 Markdown 解析。
```zsh
npm install @astrojs/mdx
```
![](/img/jurnal/1.png)
![](/img/jurnal/2.png)
但是发现依赖项中还是没有mdx，模块安装失败。

发现npm太拉，改为用pnpm：
```zsh
pnpm install @astrojs/mdx
```
在package.json 文件中找到依赖：
```json
"@astrojs/mdx": "^3.1.8",
```
安装成功！
完整过了一遍教程修改下面的内容，但deploy有报错：
在构建 Astro 项目时，出现了与 Rollup 相关的错误，提示无法解析导入 @astro/content
根据gpt提示修改：
```zsh
pnpm install @astro/content
```
发现并没有这个模块，最后参考了开始备份的一个gpt代码，修改这个后来生成的错误代码。启用不存在模块。
继续报错，安装remark和remark-extract-headings
```zsh
pnpm install remark
pnpm list remark

pnpm install remark-extract-headings
pnpm list remark-extract-headings
```
在 astro.config.mjs 文件中添加：
```mjs
import { defineConfig } from 'astro/config';
import mdx from '@astrojs/mdx';

export default defineConfig({
  integrations: [mdx()]
});
```
这里卡了很久，发现文件报错了，原因是有些添加的配置和原文件代码结构重复了……

2. 提取 Markdown 大纲

可以编写一个辅助函数，将 Markdown 文件的标题提取出来，生成大纲数据。例如，使用 Astro 的 remark 插件处理 Markdown 内容。

新建一个 utils/getHeadings.js 文件：
```js
import { remark } from 'remark';
import remarkExtractHeadings from 'remark-extract-headings';

export async function getHeadings(markdown) {
  const result = await remark()
    .use(remarkExtractHeadings, { level: [1, 2, 3] }) // 设置要提取的标题级别
    .process(markdown);
  return result.data.headings.map((heading) => ({
    text: heading.value,
    level: heading.depth
  }));
}
```
3. 创建 Post 页面模板

在 src/pages/posts/[slug].astro 中设置每个帖子页面模板。可以使用 Astro 的 getStaticPaths 动态生成每个帖子页面。使用 getHeadings 函数来生成当前 Markdown 文本的侧边大纲。

---
import { getHeadings } from '../../utils/getHeadings';
import { MarkdownContent } from '@astrojs/mdx';

export async function getStaticPaths() {
  const posts = await Astro.glob('../posts/*.md');
  return posts.map((post) => ({
    params: { slug: post.slug },
    props: { content: post.default, headings: await getHeadings(post.rawContent()) }
  }));
}

const { headings, content } = Astro.props;
---

<style>
  /* 简单的样式，用于展示大纲 */
  .sidebar {
    position: fixed;
    width: 200px;
    padding: 1rem;
    background-color: #f9f9f9;
    border-right: 1px solid #ddd;
  }
  .main-content {
    margin-left: 220px;
    padding: 1rem;
  }
</style>

<div class="sidebar">
  <h2>大纲</h2>
  <ul>
    {headings.map((heading) => (
      <li style={`margin-left: ${(heading.level - 1) * 10}px`}>
        <a href={`#${heading.text}`}>{heading.text}</a>
      </li>
    ))}
  </ul>
</div>

<div class="main-content">
  <MarkdownContent content={content} />
</div>

gpt代码：
```js
以下是完整的修改代码，包括将 getHeadings 添加到页面模板中以生成 Markdown 文本的侧边大纲。

完整的 [slug].astro 文件

---
import path from 'node:path'
import { getCollection, getHeadings } from 'astro:content'
import License from '@components/misc/License.astro'
import Markdown from '@components/misc/Markdown.astro'
import I18nKey from '@i18n/i18nKey'
import { i18n } from '@i18n/translation'
import MainGridLayout from '@layouts/MainGridLayout.astro'
import { getDir, getPostUrlBySlug } from '@utils/url-utils'
import { Icon } from 'astro-icon/components'
import { licenseConfig } from 'src/config'
import PostMetadata from '../../components/PostMeta.astro'
import ImageWrapper from '../../components/misc/ImageWrapper.astro'
import { profileConfig, siteConfig } from '../../config'
import { formatDateToYYYYMMDD } from '../../utils/date-utils'

export async function getStaticPaths() {
  const blogEntries = await getCollection('posts', ({ data }) => {
    return import.meta.env.PROD ? data.draft !== true : true
  })
  return blogEntries.map(entry => ({
    params: { slug: entry.slug },
    props: { entry },
  }))
}

const { entry } = Astro.props
const { Content } = await entry.render()
const { remarkPluginFrontmatter } = await entry.render()
const headings = await getHeadings(Content)

const jsonLd = {
  '@context': 'https://schema.org',
  '@type': 'BlogPosting',
  headline: entry.data.title,
  description: entry.data.description || entry.data.title,
  keywords: entry.data.tags,
  author: {
    '@type': 'Person',
    name: profileConfig.name,
    url: Astro.site,
  },
  datePublished: formatDateToYYYYMMDD(entry.data.published),
  inLanguage: (entry.data.lang ? entry.data.lang.replace('_', '-') : siteConfig.lang.replace('_', '-')),
  // TODO include cover image here
}
---
<MainGridLayout banner={entry.data.image} title={entry.data.title} description={entry.data.description} lang={entry.data.lang} setOGTypeArticle={true}>
  <script is:inline slot="head" type="application/ld+json" set:html={JSON.stringify(jsonLd)}></script>

  <div class="flex w-full rounded-[var(--radius-large)] overflow-hidden relative mb-4">
    <!-- Sidebar for Headings -->
    <aside class="sidebar w-1/4 p-4 bg-gray-100 dark:bg-gray-800">
      <nav class="headings">
        {headings.map(heading => (
          <a href={`#${heading.slug}`} class={`heading-${heading.depth} block mb-2`}>
            {heading.text}
          </a>
        ))}
      </nav>
    </aside>

    <!-- Post Content -->
    <div id="post-container" class:list={["card-base z-10 px-6 md:px-9 pt-6 pb-4 relative w-3/4"]}>
      <!-- word count and reading time -->
      <div class="flex flex-row text-black/30 dark:text-white/30 gap-5 mb-3 transition onload-animation">
        <div class="flex flex-row items-center">
          <div class="transition h-6 w-6 rounded-md bg-black/5 dark:bg-white/10 text-black/50 dark:text-white/50 flex items-center justify-center mr-2">
            <Icon name="material-symbols:notes-rounded"></Icon>
          </div>
          <div class="text-sm">{remarkPluginFrontmatter.words} {" " + i18n(I18nKey.wordsCount)}</div>
        </div>
        <div class="flex flex-row items-center">
          <div class="transition h-6 w-6 rounded-md bg-black/5 dark:bg-white/10 text-black/50 dark:text-white/50 flex items-center justify-center mr-2">
            <Icon name="material-symbols:schedule-outline-rounded"></Icon>
          </div>
          <div class="text-sm">{remarkPluginFrontmatter.minutes} {" " + i18n(I18nKey.minutesCount)}</div>
        </div>
      </div>

      <!-- title -->
      <div class="relative onload-animation">
        <div
          data-pagefind-body data-pagefind-weight="10" data-pagefind-meta="title"
          class="transition w-full block font-bold mb-3
          text-3xl md:text-[2.5rem]/[2.75rem]
          text-black/90 dark:text-white/90
          md:before:w-1 before:h-5 before:rounded-md before:bg-[var(--primary)]
          before:absolute before:top-[0.75rem] before:left-[-1.125rem]
        ">
          {entry.data.title}
        </div>
      </div>

      <!-- metadata -->
      <div class="onload-animation">
        <PostMetadata
          class="mb-5"
          published={entry.data.published}
          tags={entry.data.tags}
          category={entry.data.category}
        ></PostMetadata>
        {!entry.data.image && <div class="border-[var(--line-divider)] border-dashed border-b-[1px] mb-5"></div>}
      </div>

      <!-- cover image -->
      {entry.data.image &&
        <ImageWrapper id="post-cover" src={entry.data.image} basePath={path.join("content/posts/", getDir(entry.id))} class="mb-8 rounded-xl banner-container onload-animation"/>
      }

      <!-- Markdown Content -->
      <Markdown class="mb-6 markdown-content onload-animation">
        <Content />
      </Markdown>

      {licenseConfig.enable && <License title={entry.data.title} slug={entry.slug} pubDate={entry.data.published} class="mb-6 rounded-xl license-container onload-animation"></License>}
    </div>
  </div>

  <div class="flex flex-col md:flex-row justify-between mb-4 gap-4 overflow-hidden w-full">
    <a href={entry.data.nextSlug ? getPostUrlBySlug(entry.data.nextSlug) : "#"}
       class:list={["w-full font-bold overflow-hidden active:scale-95", {"pointer-events-none": !entry.data.nextSlug}]}>
      {entry.data.nextSlug && <div class="btn-card rounded-2xl w-full h-[3.75rem] max-w-full px-4 flex items-center justify-start gap-4" >
        <Icon name="material-symbols:chevron-left-rounded" class="text-[2rem] text-[var(--primary)]" />
        <div class="overflow-hidden transition overflow-ellipsis whitespace-nowrap max-w-[calc(100%_-_3rem)] text-black/75 dark:text-white/75">
          {entry.data.nextTitle}
        </div>
      </div>}
    </a>

    <a href={entry.data.prevSlug ? getPostUrlBySlug(entry.data.prevSlug) : "#"}
       class:list={["w-full font-bold overflow-hidden active:scale-95", {"pointer-events-none": !entry.data.prevSlug}]}>
      {entry.data.prevSlug && <div class="btn-card rounded-2xl w-full h-[3.75rem] max-w-full px-4 flex items-center justify-end gap-4">
        <div class="overflow-hidden transition overflow-ellipsis whitespace-nowrap max-w-[calc(100%_-_3rem)] text-black/75 dark:text-white/75">
          {entry.data.prevTitle}
        </div>
        <Icon name="material-symbols:chevron-right-rounded" class="text-[2rem] text-[var(--primary)]" />
      </div>}
    </a>
  </div>

</MainGridLayout>

<style is:global>
.headings {
  padding: 1rem;
  border-left: 2px solid var(--primary);
}

.heading-1 { font-size: 1.25rem; font-weight: bold; }
.heading-2 { font-size: 1rem; padding-left: 1rem; }
.heading-3 { font-size: 0.875rem; padding-left: 2rem; }

#post-container :nth-child(1) { animation-delay: calc(var(--content-delay) + 0ms) }
#post-container :nth-child(2) { animation-delay: calc(var(--content-delay) + 50ms) }
#post-container :nth-child(3) { animation-delay: calc(var(--content-delay) + 100ms) }
#post-container :nth-child(4) { animation-delay: calc(var(--content-delay) + 175ms) }
#post-container :nth-child(5) { animation-delay: calc(var(--content-delay) + 250ms) }
#post-container :nth-child(6) { animation-delay: calc(var(--content-delay) + 325ms) } 
</style>
```
说明

	•	大纲导航栏：在 <MainGridLayout> 中的 sidebar 区域渲染大纲内容，调用 headings 对象并使用标题的深度控制样式。
	•	样式：在 <style is:global> 中调整 headings 的样式，以控制不同深度标题的缩进和字体大小。


4. Markdown 内容自动生成 ID

为了让每个标题都可以被点击并跳转到对应位置，可以为标题自动生成 ID。可以在 utils/getHeadings.js 中添加这一步处理，也可以使用 CSS scroll-margin-top 让滚动停留在适当位置。

这样，侧边栏的 Markdown 大纲就会随着每个帖子内容的加载自动生成，并显示在浏览器页面的侧边。这个方法非常适合 Astro 项目中发布的每一个 Markdown 文件的内容页面展示。

#### 额外：安装nvm
要将下载的 nvm-master.zip 文件解压到 ~/.nvm 目录，您可以按照以下步骤进行操作：

1. 打开终端

如果您尚未打开终端，请打开它。

2. 导航到下载目录

使用以下命令切换到您的下载目录：

cd ~/Downloads

3. 解压 ZIP 文件

使用 unzip 命令解压文件。如果您的系统中未安装 unzip，可以通过 Homebrew 安装：

brew install unzip

然后，使用以下命令解压文件：

unzip nvm-master.zip

4. 移动解压后的文件

将解压后的文件夹移动到 ~/.nvm 目录：

mv nvm-master ~/.nvm

5. 配置环境变量

确保您的 shell 配置文件（如 ~/.bashrc 或 ~/.zshrc）中包含以下内容，以加载 nvm：

export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm

6. 重新加载配置

保存文件后，重新加载配置：

source ~/.zshrc   # 如果使用 zsh

7. 验证 nvm 是否安装成功

使用以下命令检查 nvm 是否安装成功：

nvm --version

如果一切顺利，您应该会看到 nvm 的版本号。如果有其他问题，请随时告诉我！

安装成功：
```zsh
# tilly @ tillydeMacBook-Pro in ~ [8:44:30]
$ nvm --version
0.40.1
```

### TuringCourse资源cloning
图灵 2024 级学长组资料汇总网站  

https://github.com/ZJU-Turing/TuringDoneRight

本地目录：/Users/tilly/code/reporsity/TuringDoneRight


还安装了mkdocs
```zsh
# tilly @ tillydeMacBook-Pro in ~/code/reporsity/TuringDoneRight on git:master o [16:33:53]
$ mkdocs --version
mkdocs, version 1.4.3 from /Users/tilly/Library/Python/3.9/lib/python/site-packages/mkdocs (Python 3.9)
```

## 11.15 添加密码
留档/src/content/config.ts：
```ts
import { defineCollection, z } from 'astro:content'

const postsCollection = defineCollection({
  schema: z.object({
    title: z.string(),
    published: z.date(),
    draft: z.boolean().optional().default(false),
    description: z.string().optional().default(''),
    image: z.string().optional().default(''),
    tags: z.array(z.string()).optional().default([]),
    category: z.string().optional().default(''),
    lang: z.string().optional().default(''),

    /* For internal use */
    prevTitle: z.string().default(''),
    prevSlug: z.string().default(''),
    nextTitle: z.string().default(''),
    nextSlug: z.string().default(''),
  }),
})
export const collections = {
  posts: postsCollection,
}
```
第一步修改：
```ts
import { defineCollection, z } from 'astro:content';

const postsCollection = defineCollection({
  schema: z.object({
    title: z.string(),
    published: z.date(),
    draft: z.boolean().optional().default(false),
    description: z.string().optional().default(''),
    image: z.string().optional().default(''),
    tags: z.array(z.string()).optional().default([]),
    category: z.string().optional().default(''),
    lang: z.string().optional().default(''),

    /* For internal use */
    prevTitle: z.string().default(''),
    prevSlug: z.string().default(''),
    nextTitle: z.string().default(''),
    nextSlug: z.string().default(''),

    // 新增的密码字段
    password: z.string().optional(),
  }),
});

export const collections = {
  posts: postsCollection,
};
```

## 11.28 更换url
gpt聊天已归档

