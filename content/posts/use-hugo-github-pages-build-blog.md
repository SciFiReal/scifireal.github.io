---
title: "使用Hugo在Github Pages上搭建个人博客"

date: 2026-10-02T08:59:47+08:00

draft: false

categories: ["建站教程"]

tags: ["hugo","Github"]

summary: "Hugo静态博客部署Github.io实操教程，包含常见命令与避坑要点"

toc: true
---

## 1.了解Github静态网站的搭建方式

### 1.1 什么是Github Pages

Github Page是Github提供的免费网页托管服务。把网页文件放到Github仓库后，就能生成xxx.github.io网址，其他人可以访问，不用自己买服务器。静态网站的优点是打开快、安全、维护简单；缺点是交互能力弱，适合写博客，记笔记。

### 1.2 什么是Hugo

Hugo是一款开源免费的静态网站生成工具，用Go语言所写的，速度很快。简单理解就是你只需要写Markdown笔记文章，Hugo会自动把文章与主题模板，一次性转换成一堆html网页文件。不用写HTML、CSS前端代码，写完直接生成网页，之后上传到Github Pages就能访问。

其它静态网站构建工具：

> 1. Hexo：基于Node.js，国内用户最多，中文教程、博客主题非常丰富。缺点是文章多的时候构建速度较慢。
> 2. VuePress/VitePress：VuePress是基于Vue，VitePress是它的升级版，速度更快。可以在Markdown嵌入Vue组件，适合写技术文档。
> 3. Jekyll：Github Pages原生支持。缺点是Ruby环境安装麻烦，构建慢。
> 4. 其它的还有MkDocs，Docusaurus。

如何选择呢合适的工具呢？个人博客，最求简单快速使用Hugo；中文资源多，折腾主题使用Hexo；写技术文档、想页面交互使用VitePress；其它方式不建议使用。

---

## 2. 部署个人博客

### 2.1 安装（Windows）

（1）安装Hugo

- Windowns的cmd或powershell下用winget一键安装Hugo，缺点是会安装到C盘

  ```bash
  winget install Hugo.Hugo.Extended
  ```

- 下载zip压缩包，拿到hugo.exe进行安装，下载位置https://github.com/gohugoio/hugo/releasesHugo.

> 验证是否安装成功：终端输入`hugo version` 出现`hugo v0.1xx.x ... extended`一大串即是成功

（2）本地创建站点

- 在自定义文件目录下创建站点

  ```bash
  D:\Computer\blog>hugo new site my-blog
  ```

- 初始化与安装主题（PaperMod）

  ```bash
  D:\Computer\blog\my-blog>git init  // 初始化
  D:\Computer\blog\my-blog>git submodule add   // 添加主题https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
  ```

（3）修改配置

修改根目录hugo.toml

```text
baseURL = 'https://yourname.github.io/' # 先占位，第二阶段部署时改成真实地址
locale = 'zh-cn'
title = '我的博客'
theme = 'PaperMod'
```

（4）写一篇笔记

```bash
D:\Computer\blog\my-blog>hugo new content posts/hello-world.md
```

hello-world.md默认构造如下：

```text
date = '2026-09-16T15:48:44+08:00' 
draft = false  # 注意：draft默认是true是草稿模式，正式文章要改为false
title = 'Hello World'
---
内容区
```

（5）启动预览，浏览器打开

```bash
D:\Computer\blog\my-blog>hugo server -D
```

---

### 2.2 部署到GitHub Pages

整体的流程：本地写文章 → git push 上传代码 → GitHub Actions 自动构建 → 别人访问
https://你的用户名.github.io

（1）注册/登录Github

（2）创建仓库

1. Repository name：你的小写用户名.github.io，例如yougli.github.io（仓库名必须是你的用户名，要全部小写）
2. 选择public（免费仓库托管需要公开仓库）
3. 其它选项不要勾选

（3）配置git

（4）修改baseURL

修改根目录hugo.toml下的baseURL：

```text
baseURL ='https://你的用户名.github.io' # https://yougli.github.io
```

（5）创建自动构建配置

GitHub上托管静态文件，需要用Hugo进行编译上去，这一步需要GitHub Actions 免费构建工具

1. 创建文件目录和文件

   ```bash
   D:\Computer\blog\my-blog>mkdir .github\workflows
   D:\Computer\blog\my-blog>notepad .github\workflows\hugo.yml
   ```

2. 在hugo.yml添加以下内容

   ```yaml
   name: Deploy Hugo site to GitHub Pages
   
   on:
     push:
       branches:
         - main
     workflow_dispatch:
   
   permissions:
     contents: read
     pages: write
     id-token: write
   
   jobs:
     build:
       runs-on: ubuntu-latest
       steps:
         - name: Checkout
           uses: actions/checkout@v4
           with:
             submodules: recursive
             fetch-depth: 0
         - name: Setup Hugo
           uses: peaceiris/actions-hugo@v3
           with:
             hugo-version: 'latest'
             extended: true
         - name: Build
           run: hugo --minify
         - name: Upload artifact
           uses: actions/upload-pages-artifact@v3
           with:
             path: ./public
   
     deploy:
       environment:
         name: github-pages
         url: ${{ steps.deployment.outputs.page_url }}
       runs-on: ubuntu-latest
       needs: build
       steps:
         - name: Deploy to GitHub Pages
           id: deployment
           uses: actions/deploy-pages@v4
   ```

   > `submodules: recursive` 会在构建时自动拉取 PaperMod 主题，这就是第一阶段用 `git submodule add` 安装主题的原因

（6）推送到GitHub

在D:\Computer\blog\my-blog目录下使用git命令

```bash
git init
git add .
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/你的用户名/你的用户名(小写).github.io.git
git push -u origin main
```

第一次执行 `git push` 时会弹出浏览器窗口要求登录 GitHub 授权

（7）开启GitHub Pages

打开个人博客的仓库，点击Settings，左侧栏Pages，在Build and deployment 下，把 Source 选择为 GitHub Actions

（8）查看结果

点击顶部Actions标签页，看到一条正在运行的Deploy Hugo site...任务，变绿色后，浏览器访问

```text
https://你的用户名.github.io   # https://yougli.github.io
```

你的个人博客正式上线了

> 草稿预览与发布控制
>
> 本地写完.md笔记后，在`D:\Computer\blog\my-blog>hugo server -D`进行本地预览，`Ctrl +C`终止预览，预览之后再推送。
>
> 记得正式推送要把`draft: true`改为`draft: false`

