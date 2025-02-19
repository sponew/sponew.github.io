---
date: '2025-02-11T15:25:46+08:00'
draft: false
title: 'Oh My Zsh'
tags: ["Oh My Zsh", "Bash", "Shell", "Server", "Terminal", "Linux", "MacOS"]
---

# Oh my zsh 使用



## zsh shell

Zsh（Z Shell）是一个功能强大的命令行 shell，具有丰富的自定义选项、自动补全、插件支持和增强的脚本功能，常用于替代 bash 提供更高效的用户体验。



## 查看本地已有的 shell

```shell
cat /etc/shells
# List of acceptable shells for chpass(1).
# Ftpd will not allow users to connect who are not using
# one of these shells.

/bin/bash
/bin/csh
/bin/dash
/bin/ksh
/bin/sh
/bin/tcsh
/bin/zsh
```



## 使用 zsh 作为默认 shell

```shell
chsh -s /bin/zsh
```



## 安装

如果本地没有`zsh`，执行以下脚本安装

### 自动安装

```shell
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

### 手动安装

下载 oh-my-zsh

```shell
git clone --depth=1 https://github.com/ohmyzsh/ohmyzsh.git ~/.oh-my-zsh
```

备份原有 ~/.zshrc（如果有）

```shell
cp ~/.zshrc ~/.zshrc.bak
```

从模板创建 zsh 配置文件

```shell
cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
```



## 更改主题

主题样式 [这里](https://github.com/ohmyzsh/ohmyzsh/wiki/Themes) 查看。

```shell
vi ~/.zshrc
# 找到
ZSH_THEME="robbyrussell"
# 例如修改为
ZSH_THEME="ys"
```

快速修改：

```shell
sed -i '/^ZSH_THEME=.*/c ZSH_THEME="ys"' ~/.zshrc
```

注：主题文件在 `~/.oh-my-zsh/themes` 目录

> 修改配置后，通过 `source ~/.zshrc` 或者退出重新登录使配置生效 (sysin)。



##  别名 alias

```zsh
cat ~/.oh-my-zsh/plugins/git/git.plugin.zsh
......

alias g='git'

alias ga='git add'
alias gaa='git add --all'
alias gapa='git add --patch'
alias gau='git add --update'
alias gav='git add --verbose'
alias gap='git apply'
alias gapt='git apply --3way'

......
```

自定义别名，在 `~/.zshrc` 中，最下面直接写即可。

```shell
# Example aliases
# alias zshconfig="mate ~/.zshrc"
# alias ohmyzsh="mate ~/.oh-my-zsh"
alias ll='ls -lahFT'
# 直接命令加入
echo 'alias ll="ls -lahFT"' >> ~/.zshrc
```

更强大的 alias 命令，比如下面命令，当你在 zsh 环境下输入 [hello.py](http://hello.py/) 即可直接用 vim 打开文件编辑，一个 tgz 的文件即可自动解压缩。

```shell
alias -s py=vim
alias -s conf=vim
alias -s tgz='tar zxvf'
```



## 命令自动补全

### 内置自动补全功能

**默认 oh-my-zsh 命令自动补全功能如下**：

- 自动列出目录

  输入 cd 按 tab 键，目录将自动列出，在按 tab 可以切换

- 自动目录名简写补全

  要访问 `/usr/local/bin` 这个长路径，只需要 `cd /u/l/b` 按 tab 键自动补全

- 自动大小写更正 (sysin)

  要访问 Desktop 文件夹，只需要 `cd de` 按 tab 键自动补全，或者查看 [README.md](http://readme.md/)，只需要 `cat rea` 自动更正补全

- 自动命令补全

  输入 `kubectl` 按 tab 键即可看到可用命令

- 自动补全命令参数

  输入 `kill` 按 tab 键会自动显示出进程的 process id

**小技巧**：

可以忽略 `cd` 命令，输入 `..` 或者 `...` 和当前目录名都可以跳转。

上述功能不需要额外的插件。




## 6. plugins

### [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting)

```shell
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

编辑 `/.zshrc` ，找到 `plugins=(git)` 这一行，修改为：

```shell
plugins=(
    git
    # other plugins...
    zsh-syntax-highlighting
)
```

*修改配置后，通过* `source ~/.zshrc` *或者退出重新登录使配置生效 (sysin)。*



### [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions)

```shell
git clone --depth=1 https://github.com/zsh-users/zsh-autosuggestions.git ${ZSH_CUSTOM:-${ZSH:-~/.oh-my-zsh}/custom}/plugins/zsh-autosuggestions
```

编辑 `/.zshrc` ，找到 `plugins=(git)` 这一行，修改为：

```
plugins=(
    git
    # other plugins...
    zsh-autosuggestions
)
```

*修改配置后，通过* `source ~/.zshrc` *或者退出重新登录使配置生效 (sysin)。*



### 我的`plugins`

```shell
plugins=(
        evalcache git git-extras debian tmux screen history extract colorize web-search docker
        zsh-autosuggestions
        zsh-syntax-highlighting
)
```

 