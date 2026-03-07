> [!NOTE]
> 本仓库为个人用仓库，托管博客源文件，`README.md` 文件使用 AI 生成。

# 枫的博客

一个由 [Hexo](https://hexo.io/) 驱动的个人博客项目文件，使用 [Butterfly](https://github.com/jerryc127/hexo-theme-butterfly) 主题。

## 项目结构

- `source/_posts/` - 博客文章
- `themes/butterfly/` - Butterfly 主题
- `public/` - 生成的静态文件
- `.github/workflows/` - GitHub Actions 工作流

## 安装

确保你已经安装了 [Node.js](https://nodejs.org/) 和 [pnpm](https://pnpm.io/)。

```bash
# 克隆仓库
git clone https://github.com/FoXLinne/blog-source.git
cd blog-source

# 安装依赖
pnpm install
```

## 配置

- 主配置文件：`_config.yml`
- 主题配置文件：`_config.butterfly.yml`

## 开发

启动本地开发服务器：

```bash
hexo server
```

服务器将在 `http://localhost:4000` 启动。

## 构建

生成静态文件：

```bash
hexo generate
```

清理缓存：

```bash
hexo clean
```

## 部署

项目使用 GitHub Actions 自动部署到 GitHub Pages。

推送代码到 `main` 分支时，会自动触发部署工作流，将生成的静态文件推送到 `FoXLinne/blog` 仓库的 `main` 分支。




