---
title: 一次 Hexo 博客从 Git 托管到云端自动构建的踩坑日记
id: 05-github-action-learning
date: 2026-03-07 07:00:00
updated: 2026-03-09 04:01:44
tags:
  - 博客
  - Git
  - AI
  - 远程
  - 终端
categories:
  - 技术研究与分享
description: 记录了一次在搭建 Hexo 博客自动化部署的折腾过程，包括解决环境报错、主题仓库的 Git 子模块 残留导致无法生成网页相关的问题。
cover: /images/thumb/5.jpg
---

{% note info flat %}
本篇文章的撰写、构建与发布均使用该工作流完成。
{% endnote %}

# 写在开头

**工欲善其事，必先利其器。**

我自己的理解就是，在正式开始沉下心写点东西之前，如果发布流程不够顺畅，总会让人产生一种抗拒心理。

于是，为了能让自己写作的时候更省心点，我决定实现博客部署流程的自动化。

---

# 实践前的准备

## 明确需求

`Hexo` 驱动的静态博客搭建过程大致可分为如下几步：

1. 撰写 Markdown 文件

2. 本地生成静态页面

3. 推送到服务器或托管平台部署

**这其中有些步骤强依赖 `Node.js` 环境以及 `Git` 命令行工具。**

如果我的主力写作工具是电脑，出门在外想继续使用其他设备随时随地写作，单独的 Markdown 文件还好说，但像这种组织化的网页源文件的多设备同步会非常让人头疼。而当使用场景切换到移动端时，受限于 iOS 系统的底层机制，本地环境的缺失成了最大的痛点。

---

基于以上，我确定了接下来的三个需求：

- 将整个博客源文件作为一个仓库进行管理，做到支持版本控制。

- 确保文章内容能够在不同设备之间进行同步。

- 使用仓库的最新版本来自动化生成静态网页，并部署到指定平台上。

繁琐的步骤全部交由云端服务自动完成，这样，在本地就只需要专注于写作即可了。

## 确立方案

既然明确了核心需求是将源码云端同步与自动部署相结合，我使用 `GitHub` 仓库来进行多设备间的同步，并`GitHub Actions` 工作流来执行编译、生成和部署的步骤。

我不想买服务器，托管网页就用 `GitHub Pages` 好了。

于是，就这样开始了搭建工作。

---

首先，建立一个仓库专门存放 `Hexo` 的源码。多设备之间通过支持 `Git` 的文本编辑器客户端来拉取、编辑和推送源码文件（包括其中最重要的 Markdown 文章），解决跨设备同步的问题。

同时，因为我使用 `GitHub Pages` 来托管静态网页，所以之前已经准备好了另一个用于存储生成网页内容的仓库。

当 `GitHub` 监听到源码仓库有新的 `Push` 操作后，便会自动触发  `GitHub Actions` 工作流：在云端配置 `Node.js` 环境、安装相关依赖，最后执行生成指令，将生成的静态网页推送至用于展示的 `GitHub Pages` 公开仓库中。

理论上的逻辑十分闭环，既满足了多端写作，又实现了自动化发布。但实际操作起来，云端环境的配置以及 `Git` 仓库的管理还是踩了不少坑。

---

# 踩坑记录

## 问题一 环境配置报错

### 1. 配置工作流

借助 AI 的帮助，我编写了如下的 `deploy.yml` 工作流文件：

```yaml
name: Hexo Deploy

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Setup Node
        uses: actions/setup-node@v3
        with:
          node-version: 20

      - name: Install Dependencies
        run: |
          npm install

      - name: Generate Website
        run: npx hexo g

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          personal_token: ${{ secrets.GH_TOKEN }}
          external_repository: FoXLinne/blog
          publish_dir: ./public
          publish_branch: main
```
接着使用 `Git` 将代码推送到仓库。

### 2. 问题排查

工作流自动开始执行，但很快就遇到了环境报错的问题。

排查构建日志日志，在执行到 `Generate Website` 这一步时，输出了如下错误：

```bash
> npx hexo g

FATAL 
Error [ERR_REQUIRE_ESM]: require() of ES Module /home/runner/work/blog-source/blog-source/node_modules/strip-ansi/index.js from /home/runner/work/blog-source/blog-source/node_modules/hexo/dist/plugins/console/list/common.js not supported.
Instead change the require of index.js in /home/runner/work/blog-source/blog-source/node_modules/hexo/dist/plugins/console/list/common.js to a dynamic import() which is available in all CommonJS modules.
    at Object.<anonymous> (/home/runner/work/blog-source/blog-source/node_modules/hexo/dist/plugins/console/list/common.js:7:38)
...

Error: Process completed with exit code 2.

```

从报错信息 `Error [ERR_REQUIRE_ESM]` 可以明确看出，在加载 `strip-ansi` 依赖包时，环境不支持 `ES Module` 的语法。

当试图去加载 `strip-ansi` 这个依赖包时，在默认情况下， `npm` 会自动拉取最新的版本，而最新版本的 `strip-ansi` 使用 `ES Module`，与 `Hexo` 的加载方式不兼容，导致了这个错误的发生。

> `Hexo` 的组件是基于传统的 `CommonJS` 规范，使用 `require()` 语法去加载模块的，由于前端生态开始使用 `ES Module` 规范，`strip-ansi` 等依赖包放弃了对 `CommonJS` 的支持。

### 3. 解决办法

**强制锁定这些包的版本，让 `npm` 拉取支持 `CommonJS` 的老版本。**

> `strip-ansi` 的 6.x 版本和 `string-width` 的 4.x 版本是支持 `require()` 导入的最后一个大版本。

在项目根目录的 `package.json` 文件中添加如下内容：

```JSON
{
  ...
  "overrides": {
    "strip-ansi": "6.0.1",
    "string-width": "4.2.3"
  }
}
```

重新提交代码并触发 `GitHub Actions`，环境报错终于消失了，相应的 `GitHub Pages` 托管仓库也显示成功将网页上线。

## 问题二：主题仓库的 `Git` 子模块残留导致无法生成网页

### 1. 紧接着到来的是

**当我访问博客时，网页并没有正常加载。**

**准确来说，网页内容是完全空白的。**


### 2. 问题排查

查看托管的网页仓库，发现所有生成好的 `.html` 文件均为大小为 `0KB` 的空文件，也就是说，工作流跑完了，但没有正确生成文件。

开始排查`GitHub Actions` 中的构建日志：

```bash
> npx hexo g

INFO  Validating config
INFO  Start processing
INFO  Files loaded in 120 ms
WARN  No layout: categories/index.html
WARN  No layout: about/index.html
WARN  No layout: aplayer/index.html
WARN  No layout: link/index.html
WARN  No layout: tags/index.html
...
```

可以看出，在生成过程中，`Hexo` 无法找到主题文件夹中的 `layout`，导致无法正确渲染出网页内容。

打开 `GitHub` 并检查仓库，发现 `/theme/butterfly` 目录的文件夹图标上有一个小箭头，同时也无法直接点进去查看文件。

在配置 `Hexo` 的时候通过 `git clone` 命令将主题仓库下载下来。主题文件夹内部自带了一个隐藏的 `.git` 目录，这样就导致了这个文件夹被 `Git` 识别成了一个 `Submodule`。

> 当把整个博客源码 `Push` 到仓库时，`GitHub` 只会把这个子模块当成一个指向外部仓库的链接来处理，而不会把其中的文件内容一并上传。

导致的结果就是，云端的 `GitHub Actions` 在拉取代码时，只拿到了一个名为主题的空目录，里面没有任何源文件，`Hexo` 自然无法渲染出网页。


### 3. 解决办法

**非常简单，把主题目录下的 `.git` 隐藏文件夹删除，让它从一个 `Git` 子模块变成普通文件夹，再次提交即可。**

```bash
> rm -rf .git

> cd ..

> git rm --cached themes/butterfly

> git add themes/butterfly

> git commit -m "fix: 移除主题 submodule 关联，转为普通文件夹"

> git push origin main
```

> 同时根据 Wiki 得知，Butterfly 主题依赖于
`hexo-renderer-pug` 和 `hexo-renderer-stylus` 这两个渲染器，所以在 `deploy.yml` 中的 `Install Dependencies` 这一步也要添加安装这两个依赖的命令：

```yaml
- name: Install Dependencies
  run: |
    npm install
    npm install hexo-renderer-pug hexo-renderer-stylus --save
```

顺带拟写一个 Readme 文件用于介绍网页。

位了防止 Markdown 文件被渲染成 HTML，在 `Generate Website` 步骤后添加：

```yaml
  - name: Add a README.md
    run: cp ./readme-deploy.md ./public/readme.md
```

---

再次提交，并触发 `GitHub Actions`。

```bash
> npx hexo g

INFO  Validating config
INFO  
  ===================================================================
      #####  #    # ##### ##### ###### #####  ###### #      #   #
      #    # #    #   #     #   #      #    # #      #       # #
      #####  #    #   #     #   #####  #    # #####  #        #
      #    # #    #   #     #   #      #####  #      #        #
      #    # #    #   #     #   #      #   #  #      #        #
      #####   ####    #     #   ###### #    # #      ######   #
                            5.5.4
  ===================================================================
INFO  Start processing
INFO  Files loaded in 3.33 s
INFO  Generated: categories/index.html
...
```

这次终于可以在构建日志中看到熟悉的 Butterfly 图标了，网页也均正常生成了。

---

# 写在最后

虽然这个工作流的搭建过程比较曲折，但最终还是跑起来了，后续在使用过程中如果遇到什么问题，也会继续更新这篇文章的。

再会。

写于 2026年3月9日 凌晨 03:30


