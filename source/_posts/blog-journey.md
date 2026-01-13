---
title: 从零搭建个人博客
date: 2026-01-13 16:00:00
tags:
  - Hexo
  - 博客搭建
  - Netlify
  - 技术分享
categories:
  - 技术
cover: 
description: 记录我从零开始搭建个人博客的完整历程，包括技术选型、踩坑经验和最终的解决方案。
---

## 前言：为什么我要搭建博客

说实话，我一开始并没有想过要搭建博客。

直到有一天，我在调试一个棘手的 bug 时，突然意识到——这个问题我三个月前就遇到过，当时花了整整两天才解决，但现在我完全想不起来当时是怎么做的了。那一刻，我决定要有一个属于自己的地方来记录这些东西。

市面上有很多现成的博客平台：CSDN、掘金、知乎专栏……但作为一个喜欢折腾的程序员，我更想拥有一个**完全可控**的个人站点。于是，搭建个人博客这件事就被正式提上了日程。

---

## 第一章：技术选型的纠结

### 静态站点生成器之战

既然决定自己搭建，第一个问题就是：用什么技术？

我对比了市面上主流的几个静态站点生成器：

| 框架 | 语言 | 特点 | 我的评价 |
|------|------|------|----------|
| **Hexo** | Node.js | 成熟稳定，主题丰富，中文社区活跃 | ⭐⭐⭐⭐⭐ |
| Hugo | Go | 构建速度极快，但主题相对较少 | ⭐⭐⭐⭐ |
| Jekyll | Ruby | GitHub Pages 原生支持，但 Ruby 环境配置较麻烦 | ⭐⭐⭐ |
| VuePress | Vue | 适合文档站点，对博客支持一般 | ⭐⭐⭐ |

最终我选择了 **Hexo**。原因很简单：

1. **Node.js 环境我已经有了**——作为前端开发者，这几乎是零成本
2. **中文社区活跃**——遇到问题基本都能找到答案
3. **Butterfly 主题太好看了**——颜值即正义

### 主题选择：一见钟情的 Butterfly

选定 Hexo 后，下一步就是挑主题。我花了一整个晚上在 Hexo 主题库里翻来翻去，直到遇见了 [Butterfly](https://butterfly.js.org/)。

第一眼看到 Butterfly 的演示站，我就知道：**就是它了。**

它的设计既不过于花哨，又不会显得单调乏味。更重要的是，它的配置项非常丰富，几乎所有你能想到的功能都可以通过配置文件来开关——这意味着我不需要去改动主题的源码。

---

## 第二章：动手搭建

### 初始化项目

有了技术选型，接下来就是实际动手了。整个过程比我想象中顺利：

```bash
# 安装 Hexo CLI
npm install -g hexo-cli

# 创建博客项目
hexo init my-blog
cd my-blog

# 安装依赖
npm install

# 本地预览
hexo server
```

打开 `http://localhost:4000`，默认的 Landscape 主题就跑起来了。虽然丑了点，但至少证明环境没问题。

### 安装 Butterfly 主题

接下来是安装 Butterfly。我直接参考了官方文档：

```bash
# 通过 npm 安装
npm install hexo-theme-butterfly
```

然后修改根目录的 `_config.yml`：

```yaml
theme: butterfly
```

为了方便管理主题配置，我在根目录创建了 `_config.butterfly.yml` 文件。这样做的好处是：**主题更新时不会覆盖我的配置**。

重新运行 `hexo server`，熟悉的 Butterfly 界面出现了。我长舒一口气——到目前为止，一切顺利。

---

## 第三章：踩坑实录

然而，接下来的事情告诉我：**过早的乐观是要付出代价的。**

### 坑一：Node.js 版本兼容问题

第一个坑来得猝不及防。当我尝试运行 `hexo generate` 时，控制台报了一堆红色的错误：

```
FATAL Something's wrong. Maybe you can find the solution here: 
https://hexo.io/docs/troubleshooting.html
TypeError: Cannot read properties of undefined (reading 'split')
```

我一脸懵逼地查了半天，最后发现问题出在 **Node.js 版本上**。我本地用的是 Node.js 22，而 Hexo 的某些插件还没有完全兼容最新版本。

**解决方案：** 使用 nvm 切换到 Node.js 20 LTS 版本：

```bash
nvm install 20
nvm use 20
```

问题立刻解决。这让我意识到一个重要的教训：**新不一定好，稳定才是王道。**

### 坑二：中文路径导致的怪异问题

第二个坑更加隐蔽。我的博客项目放在一个包含中文的路径下：

```
C:\Users\wangh\Desktop\workspace\lqbz\工作台\my-blog
```

结果就是，某些静态资源加载失败，控制台报 404 错误。

排查了很久，最后发现是 `hexo-asset-image` 插件在处理中文路径时出了问题。更讽刺的是，这个问题在本地预览时不会出现，只有在生成静态文件后才会暴露。

**解决方案：** 虽然最"正确"的做法是把项目移到纯英文路径下，但我比较懒，选择了换用 `hexo-asset-link` 插件。幸运的是，这个插件对中文路径的处理更加友好。

### 坑三：图片路径的困惑

第三个坑和图片有关。Hexo 的图片引用方式有好几种，我一开始没搞清楚：

- 放在 `source/images/` 下，用 `/images/xxx.jpg` 引用
- 开启 `post_asset_folder`，和文章同名的文件夹放图片
- 直接用外链

我最终选择了最简单粗暴的方案：**统一放在 `source/img/` 目录下**。虽然不够"优雅"，但胜在简单可靠，我懒得折腾那些花里胡哨的相对路径了。

---

## 第四章：部署上线

博客在本地跑通了，下一步是让全世界都能访问到它。

### 为什么选择 Netlify

部署方案我考虑过几个：

1. **GitHub Pages**：免费，但国内访问速度堪忧
2. **Vercel**：速度快，但免费版有带宽限制
3. **Netlify**：免费额度够用，有国内 CDN 节点，配置简单

最终选择了 **Netlify**。决定性因素是：它的配置真的太简单了。

### 配置 Netlify

首先，我在项目根目录创建了 `netlify.toml` 配置文件：

```toml
[build]
  publish = "public"
  command = "npm run build"

[build.environment]
  NODE_VERSION = "20"

[[headers]]
  for = "/*"
  [headers.values]
    X-Frame-Options = "DENY"
    X-XSS-Protection = "1; mode=block"
```

这个配置文件告诉 Netlify：

- 发布 `public` 目录（Hexo 生成的静态文件默认在这里）
- 构建命令是 `npm run build`
- 使用 Node.js 20（记住之前踩的坑！）
- 顺便加了点安全相关的 HTTP 头

然后，把代码推到 GitHub，在 Netlify 控制台关联仓库，点击部署——一切就这么简单地完成了。

博客成功上线：**<https://iridescent-flan-30c407.netlify.app/>**

（这个随机生成的域名虽然很魔性，但暂时够用了）

### 自动化部署的魔力

Netlify 最让我惊喜的是它的**自动化部署**机制：

1. 我在本地写完文章
2. `git add . && git commit -m "新文章" && git push`
3. Netlify 自动检测到代码变更，触发构建
4. 1-2 分钟后，新文章就上线了

整个流程行云流水，我甚至不需要打开 Netlify 的控制台。这种"推送即部署"的体验，真的让人上瘾。

---

## 第五章：持续优化

博客上线只是开始，接下来是漫长的优化过程。

### 优化一：配置 Butterfly 主题

Butterfly 的配置项多得令人发指，我花了大概一个下午来调整。这里分享几个我觉得比较重要的配置：

```yaml
# 导航菜单
menu:
  首页: / || fas fa-home
  归档: /archives/ || fas fa-archive
  标签: /tags/ || fas fa-tags
  分类: /categories/ || fas fa-folder-open
  关于: /about/ || fas fa-heart

# 侧边栏作者卡片
aside:
  card_author:
    enable: true
    description: "记录成长，分享知识"
    button:
      enable: true
      icon: fab fa-github
      text: Follow Me
      link: https://github.com/xxxxx
```

### 优化二：添加必要的页面

除了文章，博客还需要一些固定页面。我创建了：

```bash
# 关于页面
hexo new page about

# 标签页面
hexo new page tags

# 分类页面
hexo new page categories
```

每个页面都需要在 Front-matter 里设置正确的 `type`，否则会显示空白。这又是一个小坑。

### 优化三：写作流程的规范化

为了让以后写文章更加顺畅，我整理了一套标准的发布流程：

```bash
# 1. 创建新文章
hexo new "文章标题"

# 2. 编辑文章（用 VS Code 打开 source/_posts/文章标题.md）

# 3. 本地预览
hexo clean && hexo server

# 4. 确认无误后发布
git add .
git commit -m "发布文章：文章标题"
git push

# 5. 等待 1-2 分钟，Netlify 自动部署完成
```

我甚至把这套流程写成了一个完整的《博客使用指南》，以防哪天自己忘了怎么操作。

---

## 第六章：回顾与感悟

从决定搭建博客到正式上线，大概花了我一个周末的时间。虽然中间踩了不少坑，但整体来说，这个过程是**痛并快乐**的。

### 我学到了什么

1. **技术选型要考虑生态**：Hexo 的成熟生态帮我省了很多事
2. **版本兼容性很重要**：Node.js 版本问题让我印象深刻
3. **简单的方案往往是最好的**：与其追求"优雅"，不如追求"能用"
4. **自动化是效率的关键**：Netlify 的自动部署让发布文章变得毫无负担

### 下一步计划

博客搭好了，但这只是开始。接下来我打算：

- [ ] 绑定自定义域名（那个随机域名实在太丑了）
- [ ] 添加评论功能（考虑用 Gitalk 或 Waline）
- [ ] 优化 SEO（让搜索引擎能找到我）
- [ ] 坚持写下去（最难的其实是这一条）

---

## 结语

如果你也在考虑搭建自己的博客，我的建议是：**别想太多，先动手试试。**

技术栈不重要，主题不重要，域名也不重要。重要的是你开始写了，开始记录了。

博客的价值不在于有多少人阅读，而在于它帮助你**思考、总结、成长**。

至于那些技术细节？

放心，踩坑的时候你自然会学会的。😄

---

*如果这篇文章对你有帮助，欢迎留言交流！*
