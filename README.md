> 本 README.md 使用 AI 生成。

# 枫的博客

这是一个基于 [Hexo](https://hexo.io/) 静态博客生成器的个人博客项目，使用 [Butterfly](https://github.com/jerryc127/hexo-theme-butterfly) 主题。

## 项目结构

- `source/_posts/` - 博客文章
- `themes/butterfly/` - Butterfly 主题
- `public/` - 生成的静态文件
- `.github/workflows/` - GitHub Actions 工作流

## 安装

确保你已经安装了 [Node.js](https://nodejs.org/) 和 [pnpm](https://pnpm.io/)。

```bash
# 克隆仓库
git clone https://github.com/your-username/your-repo.git
cd your-repo

# 安装依赖
pnpm install
```

## 开发

启动本地开发服务器：

```bash
pnpm run server
```

服务器将在 `http://localhost:4000` 启动。

## 构建

生成静态文件：

```bash
pnpm run build
```

清理缓存：

```bash
pnpm run clean
```

## 部署

项目使用 GitHub Actions 自动部署到 GitHub Pages。

推送代码到 `main` 分支时，会自动触发部署工作流，将生成的静态文件推送到 `FoXLinne/blog` 仓库的 `main` 分支。

## 配置

- 主配置文件：`_config.yml`
- 主题配置文件：`themes/butterfly/_config.yml`

## 许可证

[MIT License](LICENSE) （如果有许可证文件）