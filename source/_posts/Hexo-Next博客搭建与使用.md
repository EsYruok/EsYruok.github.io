---
title: Hexo-Next博客搭建与使用
date: 2023-02-01 10:33:45
---

对重新翻新博客过程的一个记录。使用`GitHub Pages + Hexo + Next`来搭建博客的过程与Hexo写作的基本使用方法。
<!-- more -->

## 准备工作  
1. Github
先提醒一下Github Pages官方文档里有几点[建议和限制](https://docs.github.com/en/pages/getting-started-with-github-pages/about-github-pages#limits-on-use-of-github-pages) ，大家看是否能满足个人需求再决定是否使用。使用username.github.io为仓库名创建一个**Public**仓库，username为GitHub用户名。
2. Git
前往[Git官网](https://git-scm.com/) 寻找合适自己的版本安装。
3. NodeJs
前往[NodeJs官网](https://nodejs.org/zh-cn/download/) 下载适合自己的版本安装。
NodeJs默认的全局包路径在C盘，想要修改的可以修改一下。先执行命令  
```
npm config set prefix "D:\Mysoftware\nodejs\node_global"
npm config set cache "D:\Mysoftware\nodejs\node_cache"
```
再修改环境变量。
```
用户变量 Path 添加node_global的路径
添加系统变量 NODE_PATH 为node_global的路径
```

## 建立Hexo项目
建议使用管理员权限执行命令
1. 安装Hexo 
```
npm install -g hexo-cli
```
2. 创建项目并初始化
```
hexo init hexo-blog
cd hexo-blog
npm install
```
3. 生成静态文件并启动本地服务器
```
hexo g
hexo s
```
这时通过提示访问 http://localhost:4000 就可以在本地看到一个初始化的博客。hexo -h 也可以看到命令各种选项的说明。  
4. 使用Hexo部署到Github
本地测试成功后，进行部署。
- 安装部署工具
```
npm install hexo-deployer-git --save
```
- 先修改博客根目录配置文件/_config.yml 根据自己的git情况配置
```yml
# Deployment
## Docs: https://hexo.io/docs/one-command-deployment
deploy:
  type: git
  repo: https://github.com/username/username.github.io
  branch: master
 
```
- 部署
```
hexo d
```
通过账户验证后即可在Github的仓库中看到刚刚初始化的网站静态文件。
- 设置Github Pages
前往 **Settings** -> **Pages**，将 **Source** 选择为 **Deploy form a branch**，在 **Branch** 选择分支，耐心等待一会刷新该页面会在上方看到 **Your site is live at https://username.github.io/** 即表示部署成功，访问域名即可。
> 等待过程很慢，可以稍微多等一会，大概20分钟如果还没部署成功可以尝试依次折腾一下
> 1. 看看仓库是否为Public
> 2. 看看仓库名称是否打错，一定要是username.github.io
> 3. 看看仓库中根目录是否有index.html文件，如果没有则去hexo重新生成部署
> 3. 如果找不出任何异常就试试在Pages设置中将Branch选择None，点一下Save，再选回来，点一下Save。

## 安装主题
1. 安装NexT
```
cd hexo-blog
npm install hexo-theme-next
```
2. 修改配置文件
修改博客项目根目录`/_config.yml`
```
theme: next
```
3. 主题更新  
```
cd hexo-blog
npm install hexo-theme-next@latest
```

## 自定义配置
当前状态博客已经可以正常使用了，但为了能够再体现一些作者标志信息或适配访客，可以根据个人需求再进行一些配置微调。  

### 基础配置

配置文件为`/_config.yml`，有些选项不懂的情况下可以先不动。  
#### 个人信息
```yml
title: Hexo                 #博客标题
subtitle: ''                #博客子标题
description: ''             #描述信息 官方文档说是主要用于SEO
author: John Doe            #作者名称
language:                   #使用的语言，需要那种到这里找 https://theme-next.js.org/docs/theme-settings/internationalization.html#Choosing-Language
    - zh-CN
    - en
timezone: 'Asia/Chongqing'  #时区
```
#### 写作相关
```yml
# Writing
default_layout: post  #这里是设置new时候的默认布局，可以选draft page post 对应了scaffolds文件夹中的三种布局
render_drafts: false  #是否显示草稿
future: true          #是否显示未来的文章
```
#### 代码高亮
```yml
highlight:            #两种代码高亮，开启一个就行
prismjs:
```
#### 主题以及部署设置
这部分再前面安装过程中已经介绍过
```yml
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: next

# Deployment
## Docs: https://hexo.io/docs/one-command-deployment
deploy:
  type: git
  repo: https://github.com/username/username.github.io
  branch: master
```
#### 安装搜索插件[hexo-generator-searchdb](https://github.com/next-theme/hexo-generator-searchdb)
- 安装插件
```
npm install hexo-generator-searchdb
```
- 添加配置
```yml
search:
  path: search.xml
  field: post          	#搜索范围 post/page/all
  content: false
  format: html
```
### 主题配置
官方建议为了防止升级时将配置文件覆盖，建议使用Alternate Theme Config方式[配置主题](https://theme-next.js.org/docs/getting-started/configuration.html) ，即在博客跟目录下创建一个_config.next.yml配置文件用于配置主题。因为我们是使用npm安装的主题，所以主题的根目录在`/node_modules/hexo-theme-next/`。  

```shell
cp node_modules/hexo-theme-next/_config.yml _config.next.yml
```
同样这里只列几个我用到的，其他详细信息可以翻翻[官方文档](https://theme-next.js.org/docs/) 。
#### 主题方案
给了四种选项，都设置一下试试看吧
```yml
# Schemes
#scheme: Muse
#scheme: Mist
#scheme: Pisces
scheme: Gemini
```
#### 图标  
网站图标在主题根目录`/source/images`文件夹中，可以同名替换成自己的图片，或者更改成自己图片的路径
```yml
favicon:	
  small: /images/favicon-16x16-next.png
  medium: /images/favicon-32x32-next.png
  apple_touch_icon: /images/apple-touch-icon-next.png
  safari_pinned_tab: /images/logo.svg
  #android_manifest: /manifest.json
```
#### 版权信息  
版权信息会显示在文章最后
```yml
creative_commons: 
  # Available values: by | by-nc | by-nc-nd | by-nc-sa | by-nd | by-sa | cc-zero
  license: by-nc-sa                       #选择协议种类
  # Available values: big | small
  size: small
  sidebar: true                           #是否显示在侧边（头像下方位置）
  post: true                              #是否显示在post文章当中
  # You can set a language value if you prefer a translated version of CC license, e.g. deed.zh
  # CC licenses are available in 39 languages, you can find the specific and correct abbreviation you need on https://creativecommons.org
  language: deed.zh
```
#### 首页菜单
首页菜单默认只开启了home和archives其他需要手动开启。配置的格式为 **名称: 路径 || 图标** 。  
```yml
menu: 
  home: / || fa fa-home
  archives: /archives/ || fa fa-archive
```
- 如果想要添加自定义页面则按照下列方法
a. 添加新页面
```
hexo new page custom-name
```
此命令会在`sorce/custom-name/`文件夹中生成`index.md`文件，该文件则是自定义页面的内容。
b. 编辑index.md
可以在index.md中添加任何想在该页面展示的内容。
c. 编辑配置文件menu项
```yml
menu: 
  custom-name: /custom-name || fa fa-custom
```
- 添加tags或categories页面方法（以tags为例）：
a. 添加tags页面
```yml
hexo new page tags
```
b. 编辑index.md
前往tag页面`blog_root/source/tags/index.md`设置页面类型,添加下面的front-matter
```yml
type: tags
```
c. 取消配置文件中tags的注释

写文章时只要在文章的[Front-matter](https://hexo.io/docs/front-matter#Categories-amp-Tags) 处添加Categories或Tags标签即可生成相应的内容

#### TagCloud
tags页面一些元素的设置  
```yml
# TagCloud settings for tags page.
tagcloud:
  min: 12 # Minimum font size in px
  max: 30 # Maximum font size in px
  amount: 200 # Total amount of tags
  orderby: name # Order of tags
  order: 1 # Sort order
```
#### 头像信息
可以修改页面当中作者的头像，以及样式。  
```yml
avatar:
  # Replace the default image and set the url here.
  url: /images/avatar.gif
  # If true, the avatar will be displayed in circle.
  rounded: true
  # If true, the avatar will be rotated with the cursor.
  rotated: true
```
#### footer信息
```yml
footer:
  # Specify the year when the site was setup. If not defined, current year will be used.
  since: 2020                                      #建站时间

  # Icon between year and copyright info.
  icon:
    # Icon name in Font Awesome. See: https://fontawesome.com/icons
    name: fa fa-heart                              #页脚图案样式
    # If you want to animate the icon, set it to true.
    animated: false                                #动态效果
    # Change the color of icon, using Hex Code.
    color: "#ff0000"                               #图标颜色

  # If not defined, `author` from Hexo `_config.yml` will be used.
  copyright:

  # Powered by Hexo & NexT
  powered: true

  # Beian ICP and gongan information for Chinese users. See: https://beian.miit.gov.cn, http://www.beian.gov.cn
  beian:                                          #备案信息
    enable: false
    icp:
    # The digit in the num of gongan beian.
    gongan_id:
    # The full num of gongan beian.
    gongan_num:
    # The icon for gongan beian. See: http://www.beian.gov.cn/portal/download
    gongan_icon_url:
```
#### 社交信息
`名称: 链接 || 图标名称`，这个图标会显示在头像下方。  
```yml
social:
  GitHub: https://github.com/yourname || fab fa-github
  #E-Mail: mailto:yourname@gmail.com || fa fa-envelope
  #Weibo: https://weibo.com/yourname || fab fa-weibo
  #Twitter: https://twitter.com/yourname || fab fa-twitter
  #FB Page: https://www.facebook.com/yourname || fab fa-facebook
  #StackOverflow: https://stackoverflow.com/yourname || fab fa-stack-overflow
  #YouTube: https://youtube.com/yourname || fab fa-youtube
  #Instagram: https://instagram.com/yourname || fab fa-instagram
  #Skype: skype:yourname?call|chat || fab fa-skype
social_icons:
  enable: true
  icons_only: false
  transition: false
```
#### 文章目录
开启后会为文章自动生成目录。
```yml
toc:
  enable: true
  # Automatically add list number to toc.
  number: true
  # If true, all words will placed on next lines if header width longer then sidebar width.
  wrap: false
  # If true, all level of TOC in a post will be displayed, rather than the activated part of it.
  expand_all: false
  # Maximum heading depth of generated toc.
  max_depth: 6
```
#### 代码高亮主题
主题效果可以去https://theme-next.js.org/highlight/预览。  
```yml
codeblock:
  # Code Highlight theme
  # All available themes: https://theme-next.js.org/highlight/
  theme:
    light: vs2015
    dark: xcode
  prism:
    light: prism
    dark: prism-dark
  # Add copy button on codeblock
  copy_button:
    enable: true
    # Available values: default | flat | mac
    style:
```
#### 显示当前浏览进度
back2top功能可以提供侧面的一个阅读百分比以及一件到顶的按钮。  
```yml
back2top:
  enable: true
  # Back to top in sidebar.
  sidebar: false
  # Scroll percent label in b2t button.
  scrollpercent: false
```
reading_progress是顶部一个动态的进入显示条
```yml
reading_progress:
  enable: true
  # Available values: left | right
  start_at: left
  # Available values: top | bottom
  position: top
  reversed: false
  color: "#37c6c0"
  height: 3px
```
#### 浏览统计
NexT有三种统计插件分别对应设置名称leancloud_visitors，firestore，busuanzi_count。前两种需要进行注册相应的账号，第三种直接打开配置即可。busuanzi在博客本地预览时数据可能不正确，不要担心部署后就正常了。  
```yml
busuanzi_count:
  enable: true
  total_visitors: true
  total_visitors_icon: fa fa-user
  total_views: true
  total_views_icon: fa fa-eye
  post_views: true
  post_views_icon: far fa-eye
```
### 添加背景图片
1. 在主题根目录的source文件夹下添加一个_data文件夹
2. 在其中创建一个styles.styl文件
3. 在文件中写入一下内容
```styl
body {
    background:url(/images/background.jpg);//这里是图片路径
    background-repeat: no-repeat;
    background-attachment:fixed;
    background-position:50% 50%;
    // background-size: 100% 100%;
    background-size: cover;
}
```
4. 取消主题配置文件_config.next.yml中style: source/_data/styles.styl的注释
5. 在source/css/main.styl文件中修改最后的Custom Layer部分
```styl
// Custom Layer
// --------------------------------------------------
for $inject_style in hexo-config('injects.style')
  @import '../_data/styles.styl';
```
6. 重新生成网站

### 修改主题透明度
在添加背景图片的styles.styl文件中添加下列代码即可
```styl
:root {
  --content-bg-color:#ffffffdf;
}
```

## 文章写作  
- 创建一篇新的文章
`layout`代表想要使用的布局，不加时使用默认布局，在`_config.yml`当中`default_layout` 项设置，默认是post布局。  
```
hexo new [layout] <title>
```
使用post布局时文件被储存在`blog_root/source/_post`。而后使用Markdown语法书写自己的内容即可。 
- 使用草稿
创建文章时使用draft布局时代表创建一个草稿，草稿文件被储存在`blog_root/source/_drafts`。这类文章被称为草稿。草稿默认不被显示在页面种，除非在预览时使用`--draft`选项，或在`_config.yml`当中将`render_drafts`设置为`true`。
当你想要发布草稿时使用
```
hexo publish [layout] <title>
```
- Front-matter
[Front-matter](https://hexo.io/zh-cn/docs/front-matter) 是文件最上方以 --- 分隔的区域，用于指定个别文件的变量。前面提过的tags和categories就是在这里指定。  
```
---
title: Hello World
date: 2013/7/13 20:46:25
categories:
- Diary
tags:
- PS3
- Games
---
```
- 主页文章预览
在文章中加入`<!-- more -->`标签，以指定首页显示文章预览的内容。

## 多终端同步
有些情况下，可能会使用多台设备对博客进行维护，但是github page的项目中发布的仅仅是我们`hexo g`时在public当中的内容，就需要对hexo项目本身的内容进行同步。我们可以使用U盘，云盘，git等工具。在这里介绍一下git的思路。（git的操作可以参照github给出的提示使用）使用git同步我们可以新建一个分支来存放博客源码。  
- 首先在github的项目中新建一个分支hexo
**branch** -> **New branch** -> 输入hexo -> **Create branch**  
- 将默认分支设置为hexo
**Settings** -> **Branches** -> **Default branch**  
- 使用git重新clone仓库
```git
git clone git@github.com:username/username.github.io.git
```
- 切换到hexo分支，将除`.git`文件夹以外的所有文件删除  
- 将hexo项目当中除`.git`文件夹以外的所有文件拷贝到git仓库中，并提交
```git
cd username.github.io.git
git add .
git commit -m "add branch"
git push
```
可以去github验证一下两个分支是否是预想的内容。`master`存放这网站的静态文件，`hexo`存放的是博客源码文件
- 在编辑的时候要保证分支处在hexo分支上,重新安装插件
```
npm install
npm install hexo-deployer-git --save
```
- 编辑过后要同步两个分支
```git
hexo d
git push
```

其他终端操作过程大致相同，直接git拉取，安装插件即可使用。  
## 官方文档
本篇当中只提及到了一些我用到的并且我认为比较重要的部分，只是冰山一脚，Hexo和NexT还有很多其他功能，可以去他们的官网查看一下。
Hexo：https://hexo.io/zh-cn/  
NexT：https://theme-next.js.org/

---