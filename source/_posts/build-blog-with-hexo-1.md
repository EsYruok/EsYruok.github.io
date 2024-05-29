---
title: GitHub Pages + Hexo + NexT 博客搭建 (搭建篇)
date: '2024/1/1 00:12:25'
tags:
  - hexo
abbrlink: 6b2ed3a
---
记录一下小白使用 GitHub Pages + Hexo + NexT 搭建博客的方法与使用方式, 作为笔记方便不熟悉时进行查阅. 由于作者不懂前端, 所以DIY 程度有限, 仅限于一些基本的配置, 本博客就是搭建的成果可以作为参考.    <!--more-->
使用的环境是 ArchLinux, 其他版本的系统除了基本工具安装外步骤应该相同, 也可以进一步查阅官方文档.  
[Hexo文档](https://hexo.io/zh-cn/docs/) | [NexT文档](https://theme-next.js.org/docs/)

## 站点搭建
1. 安装Git/Nodejs/npm.  
    ```sh
    sudo pacman -S git nodejs npm 
    ```
2. 安装 Hexo.  
    ```sh
    sudo npm install -g hexo-cli
    ```
3. 创建 Hexo 站点.  
    ```sh
    hexo init hexo-site
    cd hexo-site
    npm install
    ```
    **hexo-site** 目录中会出现一个 **\_config.yml** 文件, 后面提到的 **Hexo 配置文件** 就是指这个文件.  
4. 生成静态文件
    ```sh
    hexo generate
    ```
    > 简写 hexo g
5. 启动本地服务器
    ```sh
    hexo server 
    ```
    > 简写 hexo s  

    控制台会显示一个本地的 http 地址来让你预览站点.  
    > Options:  
    > --safe 安全模式, 不加载插件  
    > --debug 在终端中显示调试信息  
    > --silent 简介模式, 隐藏终端信息  
    > --draft 显示草稿  

Ok! 一个毛坯站点就这样完成了.  

## 文章管理
1. 新建文章
    ```sh
    hexo new "title"
    ```
    **new** 命令格式为 **hexo new [layout] \<title>**, **layout** 为可选参数, hexo 默认有三种方案 **post / draft / page**.  
    >- post  
    代表你要创建一个要发布的文章, 命令直接在 **source/\_post/** 下建立一个 **\<title>.md** 文件.  
    >- draft  
    代表你要创建一个草稿, 命令在 **source/\_draft/** 下建立一个 **\<title>.md** 文件.
    >- page  
    方案代表要创建一个页面,命令在 **source/** 下创建一个 **\<title>** 目录, 并在目录中产生一个 **index.md** 文件.  

    如果不使用 layout 参数, 则默认使用在 **_config.yml** 配置文件中 **default_layout** 项所设置的方案, 初始状态为 **post**.  
    **\<title>** 如果包含空格需要用引号括起来 **"Hello World"**.  
    接下来就可以在新建的文件中使用 Markdown 来书写文章内容了. 如果想删除文章直接删除对应的文件即可.  
2. 发表草稿
    ```sh
    hexo publish <filename>
    ```
    **publish** 操作是将你的文章从 **\_draft** 文件夹移动到 **\_post** 文件夹.  
3. Front-matter  
    使用 **Hexo new** 创建的文章会根据模板携带 **Front-matter**. 指的是文件最上方以 **---** 分隔的区域, 用于指定当前文件的一些变量.  
    ```markdown 
    title: Hello World
    data: 2013/9/13 12:45:11
    ```
    它们会对应一些 Hexo 功能的使用, 比如创建页面时需要使用 **updated** 来展示更新时间, **tag** 和 **categories** 为文章添加标签和分类属性以便参与标签分类的统计与分类展示.  
2. 修改 Hexo 配置文件  
    修改 **_config.yml** 文件选择使用的主题.  
    ```
    theme:next
    ```
3. 主题配置文件  
    官方文档建议使用 **Alternate Theme Config** 进行配置, 大概意思就是除了主题模块中所携带的配置文件外, 还可以在另一个约定好的位置有另一份配置文件, 这样可以避免主题升级时将配置文件覆盖. 要求这个文件与 Hexo 配置文件在同一级目录, 并且文件名必须是 **_config.next.yml**. 我们可以使用下列命令来生成配置文件.  
    ```
    # pwd : hexo-site/
    cp node_modules/hexo-theme-next/_config.yml _config.next.yml
    ```
4. 升级NexT主题
    ```
    cd hexo-site
    npm install hexo-theme-next@latest
    ```
  
## 站点部署
  这里我们除了使用 GitHub Pages 作为部署工具外, 还将使用 Github 作为多端同步的手段.  
  多端同步的思路大致是, 使用 GitHub 的 master 分支来存放源代码, 使用 page 分支来作为 Pages 的部署分支. 用 master 分支编写文章, page 分支部署站点.   
1. 创建 GitHub 仓库  
    仓库名必须是 **<username>.github.io** 其中 username 是指 GitHub 的用户名. 仓库必须是 Public 或者您是 Github Pro.  
2. 提交源代码到 master  
    ```
    cd hexo-site
    git init
    git branch -M master
    git add .gitginore
    git commit -m "init site source"
    git remote add origin git@github.com:Username/Username.github.io.git
    git checkout -b page 
    rm .gitginore
    git commit -m "clean page repo"
    git push origin master
    git push origin page
    ```
    稍微有一点繁琐, 因为我们需要两个分支, 而 Git 不能推送一个空的仓库上去, 也不能创建空的分支, 所以我利用 .gitginore 创造一条记录, 创建完 page 分支后又将该文件删除来创造两个需要的分支.     
    page 分支创造好后就可以将 **hexo-site** 下的内容提交到 master 分支上了.  
3. 安装部署工具
    ```
    cd hexo-site
    npm install hexo-deployer-git --save
    ```
4. 配置部署参数
    打开 Hexo 配置文件寻找 **deploy** 部分.格式如下:  
    ```yml
    deploy:
      type: git
      repo: https://github.com/username/username.github.io 
      branch: page
      message: [message]
    ```
    如果你想使用 ssh 来连接, **repo** 使用 ssh 连接即可.  
    **message** 是自定义提交信息, 如果不设置有默认格式.  
5. 部署至远程库
    ```
    hexo clean && hexo deploy
    ```
    这个动作会产生一个 **.deploy_git** 目录, 并可能提示你去配置一些 git 设置, 最终会将站点的静态文件推送到指定的分支当中.  
6. 设置 Pages  
    登入 Github 进入仓库 -> Setting -> Pages.  
    Build and deployment 下的 Source 选择 Deploy from a branch, Branch 下选择使用的分支 `page`.  

大功告成了.  
