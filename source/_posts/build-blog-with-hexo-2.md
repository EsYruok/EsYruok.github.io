---
title: GitHub Pages + Hexo + NexT 搭建博客(配置篇)
tags:
  - hexo
abbrlink: e3e38b21
date: 2024-01-07 01:26:48
---
在上一篇博客中只记录了搭建过程, 这篇文章会记录一下对 Hexo 与 NexT 的配置. <!--more-->  

> 原本是想通过这篇文章一边学习一边写下每一项配置的意义和配置方法, 不知不觉写了巨长的篇幅, 有些过于冗长, 自己看都看不下去. 文章的本意是帮助快速的完成一个基本的配置, 所以文章会精简一些, 只记录我认为比较常用且重要的配置, 日后再折腾的时候不用再去一层一层的查找文档.  
>  想细致学习的同学不用着急, 在配置文件中官方原本就附带了注释内容, 比较清晰且附带了官方文档的链接地址. 完整读一遍配置文件就可了解到所有的配置内容.    

## Hexo 配置 (_config.yml)
如果想折腾 SEO 的话, 在不涉及隐私的情况下尽量填写, 有些选项对 SEO 有用处.  

### 站点信息  
```yml
title: title
subtitle: subtitle
description: description
keywords: keywords
author: author
language: zh-CN
timezone: 'Asia/Shanghai'
```
配置项的 key 名称即内容简单易懂. **title, subtitle, description, author** 这四项是可以显示在界面上的, 填上看看效果然后根据自己的意愿修改内容即可. 如果你想使用多语言, **language** 将你想使用的语言都写上. 

### URL  
##### 永久链接
```yml
url: https://username.github.io/
permalink: posts/:abbrlink/ # 我这里使用了 hexo-abbrlink 插件
permalink_defaults:
```
permalinks 是指文章路径的形式, 例如:  
```yml
# 一篇文章在 source/_post/hello-world.md
permalinks:year/:month/:day/:title/
# 生成静态文件后文章的路径为 2013/07/14/hello-world/
```
可使用的变量查看 [Permalinks](https://hexo.io/zh-cn/docs/permalinks).  
我所使用的 **hexo-abbrlink** 变量是插件提供的, 使用文章的时间与文件名计算 hash 值来作为变量值.
**pretty_urls** 中的两选项要你配置了 permalinks 中使用了 index.html 才有意义. 修改看不出效果的不要疑惑哈.   
```yml
# permalink: posts/:abbrlink/index.html 这样链接中才会有 index.html 让你来美化
pretty_urls:
    trailing_index: true 
    trailing_html: true 
```

##### hexo-abbrlink
安装插件模块:  
```sh
npm install hexo-abbrlink --save
```
添加配置:  
```yml
abbrlink:
 alg: crc32      #support crc16(default) and crc32
 rep: hex        #support dec(default) and hex
```

### 文章配置  
##### 默认布局
```yml
default_layout: post
```
这影响你使用 `hexo new title` 时使用的默认布局.  

##### 文章资源文件夹
```yml
post_asset_folder: true 
```
开启文章资源文件夹, 开启 post_asset_folder 后, 在你进行 hexo new 创建文章时会自动帮你创建文章资源文件夹.    

> hexo 中资源文件夹有两种形式, 全局资源文件夹和文章资源文件夹.  
>- 全局资源文件夹  
    比如说你需要一个存放图片资源的地方那么可以建立一个目录存放 `source/images`, 在文章中通过 `![](/images/image.jpg)` 引用, 所有文章都可以以这个路径引用.  
>- 文章资源文件夹  
    如果博客中的图片非常多全放在一个目录中就非常乱了. 文章资源文件夹就是创建一个与你文章同名的目录, 比如 `source/hello-world`, 在文章中通过相对路径 `![](image.jpg)` 引用. 只有这个这个文章可以使用这个相对路径.  
    
##### 代码高亮引擎
```
syntax_highlighter: highlight.js 
```
自带两种高亮引擎, 选自你使用的写上名称即可. 

##### 分页目录
```yml
pagination_dir: pages
# 翻页后的URL : http://example.com/pages/2
```
**pagination_dir** 是当你的文章列表翻页后 URL 的形式.  

##### 类别与标签
```yml
default_category: uncategorized
```
默认分类, 当你没有给文章分类信息时, 那么它就属于这个默认分类.  
```yml
category_map: 
  tech: Technology
  jishu: Technology
tag_map:
```
分类和标签别名, 可以将多个分类/标签映射成一个分类/标签.  

## NexT 配置
### 主题
##### 布局
```yml
scheme: Gemini
```
NexT 提供了四种布局样式, 官方提仓库提供了一些例子可以对每种样式进行一个大概的预览再选择. [awesome-next](https://github.com/next-theme/awesome-next#live-preview).  

##### 深色模式
```yml
darkmode: true
```
深色模式, 开启就可以跟据浏览器的亮暗模式自动进行切换.  

##### 图标
```yml
favicon:
  small: /images/favicon-16x16-next.png
```
这个是标签页上的那个小图标. 可以将自定义图标放在 source/images 目录当中并且修改配置上文件名.  

##### 主题颜色
```yml
theme_color: 
  light: "#222"
  dark: "#222"
```
主题颜色, 就是 title 后面很小一部分.  

##### 代码块高亮主题
```yml
codeblock:
  # Code Highlight theme
  # All available themes: https://theme-next.js.org/highlight/
  theme:
    light: base16/silk-light
    dark: vs2015
```
代码高亮主题, 可以通过给的网址去预览各种主题, 将选好的名称添上.  

##### 背景丝带
```yml
canvas_ribbon:
  enable: true
  size: 300 # The width of the ribbon
  alpha: 0.6 # The transparency of the ribbon
  zIndex: -1 # The display level of the ribbon
```
在背景画布上装饰一个可变化的丝带.  

### 菜单
```yml
menu:
  home: / || fa fa-home
```
默认支持的菜单只有 home 和 archives. 不过 NexT 支持我们自定义创建菜单项目.  
> - 创建自定义页面  
    1. 使用 hexo new page custom-name 创建一个自定义的页面  
    2. 修改新建页面的 index.md 添加 Front-matter 信息与内容  
    3. 编辑 menu 配置. (例如: `custom: /custom-name/ || fa fa-custom`)  
> - 创建 tags 页面  
    一种特殊的自定义页面, tags 页面只需在 index.md 的 Front-matter 信息中将 type 设置为 tags, 就可以将所有文章的 tag 显示在页面中.  
> - 创建 categories 页面  
    与 tags 相同, 只是 type = categories.  

NexT 还支持动态子菜单, 例如:  
```yml
menu:
  home: / || fa fa-home
  archives: /archives/ || fa fa-archive
  Docs:
    default: /docs/ || fa fa-book
    Getting Started:
      default: /getting-started/ || fa fa-flag
      Installation: /installation.html || fa fa-download
      Configuration: /configuration.html || fa fa-wrench
```
上面的配置方式会在拥有子菜单的页面 (例如 docs ) 上方产生相应的子菜单.  

### 侧边栏
##### 头像
```yml
avatar:
  url: /images/avatar.gif
  rounded: true 
  rotated: true
```
头像, 更改方式和 favicon 相同.  

##### 站点状态
```yml
site_state: true 
```
将文章,标签, 类别数量显示在侧边栏当中.  

##### 社交链接
```yml
social:
  #GitHub: https://github.com/yourname || fab fa-github
  #E-Mail: mailto:yourname@gmail.com || fa fa-envelope
```
侧边栏中添加一些社交账号的图标, 点击可以快速导航.    

##### 友情链接
```yml
links:
  Title: https://example.com # 友情链接的title和链接, 如果没有则不会显示友情链接块
```
友情链接, 出现在侧边栏下方.  

##### 公共版权许可
```yml
creative_commons:
  license: by-nc-sa
  size: small
  sidebar: true
  post: true
  language:
```
版权许可信息, sidebar 设置让你在侧边栏中有个图标, post 设置让你的许可信息出现在每篇文章的末尾.  

### 页脚  
```yml
footer:
  ...
```
页面最底部显示的一系列信息.  

### 文章
##### 目录
```yml
toc:
  enable: true
```
文章目录, 浏览文章时, 会自动根据 Markdown 格式在侧边栏相同的位置产生一个目录   

##### 摘要
```yml
excerpt_description: false
```
控制文章摘要的方式, 如果使用 true 则要在文章中使用 description 变量来储存摘要, false 则在文章内容中添加 `<!--more-->` 来表示标签之前的内容为摘要. 建议使用 false.  

##### 阅读全文
```yml
read_more_btn: true 
```
文章列表中显示"阅读全文"按钮.  

##### 文章属性显示
```yml
post_meta:
```
这部分是选择展示出来的文件属性. 可以在文章列表中看到, 也可以在文章详情看到, 都是在标题下方.  
```yml
symbols_count_time: # 字数统计,阅读时间
  separated_meta: true
  item_text_total: false
```
字数统计与大概阅读时间. 显示位置与 post_meta 相同. 需要额外安装插件模块:  
```sh 
npm install hexo-word-counter
hexo clean
```
```yml
tag_icon: true
```
tag 会显示在文章底部, true 表示在 tag 前使用图标, false 是使用一个 #

##### 打赏按钮
```yml
reward_settings:
reward:
```
文章末尾添加一个打赏按钮.  

##### Follow Me
文章内版本的 social.
```yml
follow_me:
  #Twitter: https://twitter.com/username || fab fa-twitter
```
这个也是显示在文章末尾.  

##### 返回顶部按钮
```yml
back2top:
  enable: true
```
左下角一个返回顶部按钮.  

##### 阅读进入条
```yml
reading_progress:
  enable: true
```
阅读文章时, 顶部有一个带颜色的阅读进度条.  

### 插件
##### 数学公式渲染
math 配置提供渲染 Markdown 中数学公式的能力.  
```yml
math:
  every_page: false # `mathjax: true`
  mathjax:
    enable: true
    # Available values: none | ams | all
    tags: none
```
every_page 代表是否要每个页面都加载渲染引擎, 如果设置成 false 需要在想要进行公式渲染页面的 Front-matter 中手动添加 mathjax: true 开启渲染.  
使用 mathjax 需要卸载 Hexo 原有的渲染器改用 hexo-renderer-pandoc.  
```sh 
# sudo pacman -S pandoc # 渲染器依赖 pandoc
# npm un hexo-renderer-marked
# npm i hexo-renderer-pandoc --save
# hexo clean && hexo s
```

##### Loacal Search
搜索功能, 它会出现在菜单上.    
```sh
npm install hexo-generator-searchdb
```
Hexo 配置.  
```yml
search:
  path: search.xml
  field: post
  content: true
  format: html
```
NexT 配置.  
```yml
local_search:
  enable: true
  top_n_per_article: 1
  unescape: false
  preload: false
```

##### RSS
```sh
npm install hexo-generator-feed --save
```
配置文件, 配置在 Hexo 或 NexT 哪个配置里都行.  
```yml
feed:
  enable: true 
  type: atom 
  path: atom.xml 
  limit: 20 
  content_limit: 140
  order_by: -date 
```
页面链接可以添加在 menu , follow 或 social 中都可以. 
```yml 
RSS: /atom.xml || fa fa-rss
```
  
Ok, 一个包括所有基本 DIY 配置的博客就搭建完成了. 当然还有很多关于性能, SEO, CDN 等配置感兴趣的同学继续加油!  
