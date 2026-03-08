---
title: 【施工中】一次 Hexo 博客从 Git 托管到云端自动构建的踩坑日记
id: 05-github-action-learning
date: 2026-03-07 07:00:00
updated: 2026-03-08 03:45:12
tags:
  - 博客
  - Git
  - AI
  - 远程
  - 终端
categories:
  - 技术研究与分享
description: 记录了一次在搭建 Hexo 博客自动化部署的折腾过程，包括解决环境报错、主题仓库的 Git Submodule
  残留导致无法生成网页，以及在 iOS 端使用 lg2 进行远程仓库管理的权鉴配置相关的问题。
cover: /images/thumb/5.jpg
---

{% note info flat %}
本篇文章的撰写、构建与发布均使用该工作流完成。
{% endnote %}

# 写在开头

**工欲善其事，必先利其器。**

我自己的理解就是，在正式开始沉下心写点东西之前，如果发布流程不够顺畅，总会让人产生一种抗拒心理。

于是，为了能让自己写作的时候更省心点，我决定将博客的部署流程进行自动化。

---

# 实践前的准备

## 确定需求

静态博客搭建的工作流其实非常繁琐：撰写 Markdown 文件、本地生成静态页面、最后推送到服务器或托管平台部署，这其中有些步骤强依赖 `Node.js` 环境以及 `Git` 命令行工具。

如果我的主力写作工具是电脑，出门在外想继续使用其他设备随时随地写作，单独的 Markdown 文件还好说，但像这种组织化的网页源文件的多设备同步会非常让人头疼。而当使用场景切换到移动端时，受限于 iOS 系统的底层机制，本地环境的缺失成了最大的痛点。

基于以上，我确定了接下来的三个需求：

1. 多设备写作：需要确保文章内容能够在不同设备之间同步。

2. 远程管理：将除了文章内容以外的博客源文件作为一个仓库进行管理，在不同设备上都能够方便的进行编辑和推送。

3. 自动化部署：能够在推送内容后自动生成静态网页，并部署到指定平台。

解决思路也很明确：使用 `GitHub` 仓库来进行多设备间的同步，把编译、生成和部署的繁琐步骤全部交由云端服务自动完成。

这样，在本地就只需要专注于写作即可了。

## 确立方案

既然明确了核心需求是将源码云端同步与自动部署相结合，`GitHub Actions` 自然成为了首选。

整体的工作流设计如下：

首先，建立一个仓库专门存放 `Hexo` 的源码。多设备之间通过支持 `Git` 的文本编辑器客户端来拉取、编辑和推送源码文件（包括其中最重要的 Markdown 文章），解决跨设备同步的问题。

同时，因为我使用 `GitHub Pages` 来托管静态网页，所以之前已经准备好了另一个用于存储生成网页内容的仓库。

当 `GitHub` 监听到源码仓库有新的 `Push` 操作后，便会自动触发  `GitHub Actions` 工作流：在云端配置 `Node.js` 环境、安装相关依赖，最后执行生成指令，将生成的静态网页推送至用于展示的 `GitHub Pages` 公开仓库中。

理论上的逻辑十分闭环，既满足了多端写作，又实现了自动化发布。但实际操作起来，云端环境的配置以及 `Git` 仓库的管理还是踩了不少坑。

---

# 踩坑记录

## 环境配置报错

### 第一步 配置 `deploy.yml` 文件

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

### 第二步 触发工作流并遇到环境报错

成功使用 `Git` 将代码推送到仓库后，`GitHub Actions` 工作流自动开始执行，但很快就遇到了环境报错的问题。在执行到 `Generate Website` 这一步，也就是运行 `npx hexo g` 时，控制台抛出了如下的致命错误：

```bash
> npx hexo generate

FATAL 
Error [ERR_REQUIRE_ESM]: require() of ES Module /home/runner/work/blog-source/blog-source/node_modules/strip-ansi/index.js from /home/runner/work/blog-source/blog-source/node_modules/hexo/dist/plugins/console/list/common.js not supported.
Instead change the require of index.js in /home/runner/work/blog-source/blog-source/node_modules/hexo/dist/plugins/console/list/common.js to a dynamic import() which is available in all CommonJS modules.
    at Object.<anonymous> (/home/runner/work/blog-source/blog-source/node_modules/hexo/dist/plugins/console/list/common.js:7:38)
...

Error: Process completed with exit code 2.

```

从报错信息 `Error [ERR_REQUIRE_ESM]` 可以明确看出，`Hexo` 的组件是基于传统的 `CommonJS` 规范，使用 `require()` 语法去加载模块的。

当试图去加载 `strip-ansi` 这个依赖包时，在默认情况下， `npm` 会去自动拉取最新的版本。

由于前端生态开始使用 `ES Module` 规范，`strip-ansi` 等依赖包放弃了对 `CommonJS` 的支持，从而导致无法正确加载。

### 第三步 解决报错

最直接的解决方式就是强制锁定这些包的版本，让它们回退到支持 `CommonJS` 的老版本。

`strip-ansi` 的 6.x 版本和 `string-width` 的 4.x 版本刚好是它们支持 `require()` 导入的最后一个大版本。

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

但当我访问博客时，发现页面只有一片空白。

## 踩坑二：主题仓库的 `Git Submodule` 残留导致无法生成网页

网页加载异常，准确来说，网页内容是完全空白的。

查看托管的网页仓库，发现所有生成好的 `.html` 文件均为大小为 0kb 的空文件，也就是说，生成过程虽然正常的跑完，但没有正确生成文件。

开始排查`GitHub Actions` 的构建日志，发现了大量如下报错：

**报错原因分析：**

这其实是使用 Git 托管 Hexo 源码时极易踩中的一个机制问题：**Git Submodule（子模块）残留**。

在配置 Hexo 时，我们通常是通过 `git clone` 命令将主题下载到 `themes/` 目录下。克隆下来的主题文件夹内部自带了一个隐藏的 `.git` 目录。因此，在外层的博客主仓库看来，这个主题目录并不是一个普通的文件夹，而是一个独立的 Git 子模块。

当我把整个博客源码 `git push` 到我的私有仓库时，Git 默认并不会把子模块里面的具体文件一起推送上去，它只会在主仓库里记录该子模块的一个提交节点（Commit Hash）。导致的结果就是，云端的 GitHub Actions 在拉取代码时，只拿到了一个名为主题的空目录，里面没有任何源文件，Hexo 自然无法渲染出网页。

**解决办法：**

我的解决办法非常直接：把主题目录下的 `.git` 隐藏文件夹直接删除，让它从一个 Git 子模块彻底变成一个普通的本地文件夹，随主仓库一起提交。

具体的操作命令如下（假设使用的主题名称为 `your-theme`）：

```bash
# 1. 进入主题目录
cd themes/your-theme

# 2. 删除隐藏的 .git 文件夹
rm -rf .git

# 3. 返回博客根目录
cd ../..

# 4. 清除 Git 对该子模块的缓存记录（关键步骤）
git rm —cached themes/your-theme

# 5. 重新将主题文件夹作为一个普通目录添加到主仓库暂存区
git add themes/your-theme

# 6. 提交修改并推送
git commit -m ”fix: 移除主题 submodule 关联，转为普通文件夹“
git push origin main

```



