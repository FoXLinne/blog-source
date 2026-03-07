---
title: 在 Windows 上安装 zsh 并将其设置为默认 Shell
id: 03-zshonwindows
date: 2024-04-10 12:00:00
updated: 2024-04-11 12:00:00
tags:
  - Windows
  - 终端
  - Git
categories:
  - 技术研究与分享
discription: 如何在 Windows 上安装 zsh 并将其设置为 Windows Terminal 与 SSH 下的默认 Shell。
cover: /images/thumb/3.jpg
---

# 背景

> `zsh` 作为目前比较强大的终端，支持丰富的主题与插件，可以协助用户更加便捷的使用操作系统，而目前在 macOS 各种 Linux 发行版下安装并配置 `zsh` 比较容易，本文将教大家如何在 Windows 下安装配置 `zsh`，使用 `Oh My Zsh` 配置插件，安装主题，并将其设置为 `Windows 终端` 与 SSH 连接时的默认 Shell。

# 安装 `Git`

> `Git for Windows` 提供了一个仿真环境，可以让用户在 Windows 的命令行中执行与 Linux 相同的部分命令，方便快捷。可以通过以下几种方式安装 `Git for Windows` ：

## 官网下载

下载完毕后，安装时请勾选 Add a Git Bash Profile to Windows Terminal ，`Git Bash` 的标签页将会自动添加到 Windows Terminal 中，以便我们后续进行进一步的配置。

## 使用包管理器 Scoop 安装

```CMD
scoop install git
```

下载完毕后，在 Windows Terminal 中手动添加 Git Bash 标签页，并将其设为默认（可选）。

# 安装 `zsh`

点击访问下载地址，进入后，点击 File 旁边的链接，即可下载。
下载完成后得到如图所示的扩展名为 .zst 的压缩包，可以使用 7-Zip 等软件解压，我这里使用 Bandizip 解压。

将解压好的文件移动到 Git 的根目录下，合并同名文件夹，得到的文件夹结构如图所示：

此时在 Windows Terminal 中打开 Git Bash ，输入 zsh ，出现下图则说明安装成功。

选择 0 ，创建完毕 `~/.zshrc` 文件后成功进入 `zsh` 。

# 安装 Oh My Zsh

> 刚安装好的 zsh 还不够完善，Oh My Zsh 是基于 zsh 的命令行拓展工具，提供了主题配置与插件机制，并内置一些便捷的操作，此处我们安装 Oh My Zsh 来改善 zsh 的使用体验。

> 可以使用 curl 或者 wget 两种方式来安装，任选一种即可。

## curl

```sh
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

## wget

```sh
sh -c "$(wget -O- https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh )"
```

# 将 zsh 设置为默认的 Shell

要让每次打开终端时都使用 zsh，需要让 Git Bash 在启动时自动运行 zsh，因此，我们需要在 bash 的配置文件中增加一些内容。

```bash
nano ~/.bashrc
```

同样使用 Nano 编辑器打开 bash 的配置文件，在最后加入以下内容。

```nano
if[ -t 1 ]; then
execzsh
fi
```

至此，在 Windows 上安装 zsh 以及安装之后的其他配置工作都已经大功告成，接下来就可以使用 zsh 代替默认的命令行工具，提高工作效率了。