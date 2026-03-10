---
title: Hugo创建博客
subtitle: 使用FixIt主题
date: 2026-03-10T01:18:18+08:00
slug: fed42e9
draft: false
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

### 先决条件

1. 本地项目正常
2. 创建`GitHub`账户
3. 创建`cloud flare`账户
4. 安装`git`

### 部署过程

1. 确保本地项目可用，主题子模块配置正常，如果作为`git`子模块要确认根目录生成`.gitmodules`文件且包含主题配置。在项目根目录创建`.gitignore`文件说明无需提交的内容：

   ```
   # Hugo 构建产物与缓存
   /public/
   /resources/_gen/
   .hugo_build.lock
   hugo_stats.json
   assets/jsconfig.json
   
   # 系统与编辑器临时文件
   .DS_Store
   Thumbs.db
   .vscode/
   .idea/
   
   # 依赖与日志文件
   node_modules/
   .npmrc
   .pnpm-debug.log
   npm-debug.log
   yarn-error.log
   .stash/
   
   # Hugo 可执行文件
   hugo.exe
   hugo.darwin
   hugo.linux
   ```

2. `GitHub`创建空白仓库，**注意保持仓库空白，不要添加`README`、`.gitignore`等**

3. 本地项目初始化`git`并关联远程仓库

   ```
   git remote add origin https://github.com/<你的GitHub用户名>/<仓库名>.git
   ```

   切换到主分支`main`：

   ```
   git branch -M main
   ```

   将所有文件加入 Git 暂存区：

   ```
   git add .
   ```

   提交代码，填写提交信息：

   ```
   git commit -m "init: 初始化 Hugo FixIt 博客项目，添加 Cloudflare 部署配置"
   ```

   推送到 GitHub 主分支：

   ```
   git push -u origin main
   ```

   查看仓库是否上传成功，如果出现`SSH`连接问题参考下方操作

4. 如果希望使用`workers`配置，参考[hugo官方部署文档](https://gohugo.io/host-and-deploy/host-on-cloudflare/#procedure)

5. 本项目使用`pages`方案配置，登录`Cloud flare`找到添加`pages`，按提示操作，构建配置填写

   ```
   生产分支：main
   构建命令：git submodule update --init --recursive && hugo build --gc --minify --baseURL $CF_PAGES_URL
   构建输出目录：public
   环境变量：添加 HUGO_VERSION，值为你的本地 Hugo 版本
   ```

   对于构建命令

   ```
   git submodule update --init --recursive && hugo build --gc --minify --baseURL $CF_PAGES_URL
   ```

   前一部分执行成功后才执行后半部分，前一部分是确保 Cloudflare 构建环境里，`themes/FixIt` 文件夹是**完整、版本正确、包含所有嵌套依赖**的，后一部分中，`--gc`为垃圾回收，清理旧缓存和残留文件，避免 `public` 目录堆积垃圾；`--minify`是为压缩所有静态资源（HTML/CSS/JS），减小体积，提升访问速度；`--baseURL` 强制覆盖 `hugo.toml` 里的 `baseURL` 配置，用命令行里的值代替；`$CF_PAGES_URL`是**Cloudflare Pages 提供的内置环境变量**，会自动填充为当前部署的完整 URL，可实现动态适配，解决页面部署后空白的问题。



### SSH

如果远程仓库地址使用SSH协议，则推送前要进行身份认证，建议配置SSH密钥，一劳永逸。

> 也可以切换为https协议

**1.** 查看是否有SSH密钥，可看当前用户文件下是否有`.ssh`文件以及是否有`id_rsa`和`id_rsa.pub`，如果没有，则生成：

```Linux
ssh-keygen -t rsa -b 4096 -C "你的邮箱@example.com"
```

> [!TIP]
> 建议安装`git`后使用`git bash`操作

**2.** 执行完成并生成后，需要启动SSH代理并添加私钥：

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

**3.** 复制公钥，可使用命令：

```
cat ~/.ssh/id_rsa.pub | clip
```

**4.** 添加公钥至GitHub账号

**5.** 测试是否成功：

```
ssh -T git@github.com
```

若出现以下类似内容则成功

```
Hi 你的GitHub用户名! You've successfully authenticated, but GitHub does not provide shell access.
```

**6.** 如果已经有`ssh`密钥，首先最好了解已有的密钥用途，可根据需求重新生成对`GitHub`连接的专用密钥：

```
ssh-keygen -t rsa -b 4096 -C "GitHub邮箱@example.com" -f ~/.ssh/id_rsa_github
```

之后可进行复制公钥至GitHub操作

**7.**  **关键：** 在`.ssh`文件夹中创建配置文件`config`，记事本打开并填写以下内容：

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



### 其他问题

1. 部署成功后某些功能未显示，控制台报错：

   ```
   Refused to execute script from 'https://xxx/lib/cell-tooltip/cell-tooltip.umd.cjs' because its MIME type ('application/node') is not executable, and strict MIME type checking is enabled.
   ```

   这是一个 **Cloudflare Pages MIME 类型识别错误**导致的连锁反应，Cloudflare Pages 把 FixIt 主题的 `cell-tooltip.umd.cjs` 脚本文件错误识别成了 `application/node`（Node.js 应用文件），而不是可执行的 `application/javascript`，浏览器出于安全考虑拒绝执行该脚本。因为 `cell-tooltip` 脚本加载失败，`window.CellTooltip` 变成了 `undefined`，FixIt 主题的核心 JS（`theme.min.js`）试图调用它时崩溃，导致依赖该核心 JS 的所有功能（目录、返回顶部、评论等）全部失效。

   **解决：**

   保留并正常使用 `cell-tooltip` 组件，通过 Cloudflare Pages 的 `_headers` 文件强制修正 `.cjs` 文件的 MIME 类型。

   - 创建`_headers`文件

     - 在项目根目录（和 `hugo.toml` 同级）创建一个名为 `_headers` 的文件（**没有后缀，就叫 `_headers`**），填入以下内容：

     ```
     # 强制修正 .cjs 文件的 MIME 类型为可执行 JavaScript
     /*.cjs
       Content-Type: application/javascript
     ```

   - 确保 `_headers` 文件被 Hugo 复制到部署目录

     - 打开 `hugo.toml`，添加或修改 `staticDir` 配置，确保 `_headers` 被包含在部署产物中：

     ```
     # 将根目录的 _headers 等静态文件包含进去
     staticDir = ["static", "assets"]
     
     # 或者使用 mounts 配置（更灵活）
     [module]
       [[module.mounts]]
         source = "static"
         target = "static"
       [[module.mounts]]
         source = "assets"
         target = "assets"
       [[module.mounts]]
         # 关键：将根目录的 _headers 挂载到部署根目录
         source = "_headers"
         target = "_headers"
     ```

     

2. 







### 参考

[hugo官方部署文档](https://gohugo.io/host-and-deploy/host-on-cloudflare/#procedure)
