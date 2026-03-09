---
title: Hugo创建博客
subtitle: 使用FixIt主题
date: 2026-03-10T01:18:18+08:00
slug: fed42e9
draft: false
author:
  name:
  link:
  email:
  avatar:
description:
keywords:
comment: false
weight: 0
tags:
  - hugo
categories:
  - 博客搭建

# See details front matter: https://fixit.lruihao.cn/documentation/content-management/introduction/#front-matter
---

使用hugo+FixIt创建博客的总结

<!--more-->

## 前期准备

1）下载 hugo： https://github.com/gohugoio/hugo  选择合适版本，建议下载 `extended `版本

2）下载 git： https://git-scm.com/ 选择合适的版本下载安装

## 创建博客及使用主题

> hugo 参考文档： https://www.gohugo.io/

在hugo 所在文件夹地址栏输入 `cmd` 打开终端，可通过以下命令创建一个带有FixIt主题的Hugo网站。

```
hugo new site /path/to/site			//可选位置
cd my-blog
git init
git submodule add https://github.com/hugo-fixit/FixIt.git themes/FixIt			//可使用其他主题
echo "theme = 'FixIt'" >> hugo.toml				//添加主题至配置文件
hugo server
```

{{<admonition>}}

如果没有设置 hugo 全局环境变量需要将 `hugo.exe` 文件粘贴至生成的博客文件夹才能运行 `hugo` 命令，建议还是设置环境变量。

{{</admonition>}}

使用 `hugo server -D` 命令启动服务并且实时预览，浏览器输入 `http://localhost:1313` 访问，<kbd>ctrl</kbd>+<kbd>C</kbd> 终止服务

**使用FixIt主题必要配置**

```
[markup]
_merge = "shallow"

[outputs]
_merge = "shallow"

[taxonomies]
_merge = "shallow"
```

以上配置表示继承 FixIt 主题的 `markup`，`outputs` 和 `taxonomies` 配置。

{{<admonition tip>}}

具体配置可参考官方文档[hugo配置篇](https://fixit.lruihao.cn/zh-cn/documentation/getting-started/configuration/)

{{</admonition>}}

**Fly fish**

底部小鱼游动动画，参考[菠菜眾長](https://lruihao.cn/projects/hugo-fixit/cmpt-flyfish/)文章

```
git submodule add https://github.com/hugo-fixit/cmpt-flyfish.git themes/cmpt-flyfish	//作为git子模块安装
```

通过 FixIt 主题在 `layouts/_partials/custom.html` 文件中开放的 [自定义块](https://fixit.lruihao.cn/references/blocks/) 将 `cmpt-flyfish.html` 注入到 `custom-assets` 中，你需要填写以下必要配置：

```
[params]
  [params.customPartials]
    # ... other partials
    assets = [ "inject/cmpt-flyfish.html" ]
    # ... other partials
```

配置小鱼主题色，并启用动画

```
  [params.flyfish]
    enable = true
    light = "rgb(0 119 190 / 10%)"
    dark = "rgb(255 255 255 / 10%)"
```

### 添加内容

Hugo 在 `content/posts` 目录中创建了xxx文件

```
hugo new content posts/xxx.md
```

打开后创建的内容`Front matter`中的`draft`值为`true`，默认情况下，Hugo 在构建网站时不会发布草稿内容，详细了解 [草稿、未来和过期内容](https://gohugo.io/getting-started/usage/#draft-future-and-expired-content)。

保存文件后可通过以下任一命令启动hugo服务器和查看（包含草稿内容）：

```
hugo server --buildDrafts
hugo server -D
hugo server -D --disableFastRender
```

非常建议添加 `--disableFastRender` 参数来实时预览正在编辑的文章页面。



## 部署

### 







### SSH

如果远程仓库地址使用SSH协议，则推送前要进行身份认证，建议配置SSH密钥，一劳永逸。

> 也可以切换为https协议

**1.**查看是否有SSH密钥，可看当前用户文件下是否有`.ssh`文件以及是否有`id_rsa`和`id_rsa.pub`，如果没有，则生成：

```Linux
ssh-keygen -t rsa -b 4096 -C "你的邮箱@example.com"
```

> [!TIP]
> 建议安装`git`后使用`git bash`操作

**2.**执行完成并生成后，需要启动SSH代理并添加私钥：

1. 在 Git Bash 中启动 SSH 代理，执行：

   ```
   eval "$(ssh-agent -s)"	
   ```

​		执行成功会输出类似 `Agent pid 1234` 的内容。

2. 把生成的私钥添加到代理中：

   ```
   ssh-add ~/.ssh/id_rsa
   ```

   执行成功会提示 `Identity added: ...`，说明添加完成。

{{<admonition tip>}}
可将 `OpenSSH Authentication Agent` 服务设置开机自启，自动加载SSH代理。
{{</admonition >}}

**3.**复制公钥，可使用命令：

```
cat ~/.ssh/id_rsa.pub | clip
```

**4.**添加公钥至GitHub账号

**5.**测试是否成功：

```
ssh -T git@github.com
```

若出现以下类似内容则成功

```
Hi 你的GitHub用户名! You've successfully authenticated, but GitHub does not provide shell access.
```

**6.**如果已经有`ssh`密钥，首先最好了解已有的密钥用途，可根据需求重新生成对`GitHub`连接的专用密钥：

```
ssh-keygen -t rsa -b 4096 -C "GitHub邮箱@example.com" -f ~/.ssh/id_rsa_github
```

之后可进行复制公钥至GitHub操作

**7.** **关键：**在`.ssh`文件夹中创建配置文件`config`，记事本打开并填写以下内容：

```
# GitHub 专用配置
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_rsa_github
    PreferredAuthentications publickey
```

只要是连接 `github.com`，就强制使用 `id_rsa_github` 这个私钥，和旧密钥完全隔离开。

{{<admonition note>}}

开启SSH服务自启动，**只是让 SSH 代理程序在后台运行**，不用每次重启电脑后手动执行 `eval "$(ssh-agent -s)"` 启动服务，不会自动把私钥加载到代理缓存里，**如果设置了私钥密码**，则在第一次推送前需要进入博客目录手动执行一次加载

```
ssh-add ~/.ssh/id_rsa_github
```

执行后私钥会暂时存至代理的内存，如果没有设置则不需要。

{{</admonition>}}

