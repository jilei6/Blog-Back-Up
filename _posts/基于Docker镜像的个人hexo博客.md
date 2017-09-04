title: 基于Docker镜像的个人hexo博客
toc: true
author: Jaren
date: 2017-08-31 17:03:22
tags:
   - Html
   - Docker
   - Github
---
  ![ico原来的样子](/assets/blogImg/snow.jpg) 
  > 起因：本着程序员以懒为荣的态度，一直想着开一个自己的专属博客，但实在是太懒了，一直未能成行，但是最近看到比我还懒的长老也开始玩博客了，实在是不能忍了，毅然决定放下那份荣耀，毕竟维斯特洛大陆已经凛冬已至了。博客走起~
<!-- more -->
  

       先说下整体思路基于hexo的个人博客，采用docker镜像解决多PC协同问题,采用gitHub和coding双托管博客，采用Travis自动化发布更新博客文章，放一张设计图，图水勿喷
  ![docker工作架构图-_-!](/assets/blogImg/hexo+docker+github+Travis的设计图.png) 


#### 一、搭建Hexo 

正文：

hexo是一款基于Node.js的静态博客框架, [hexo github链接] [hexo github链接]:,这篇教程是针对与Mac的，[参考链接] [参考链接]:，由于原文讲到的hexo是以前的老版本，所以现在的版本配置的时候会有些改动。

其实写博客，一方面是给自己做笔记，可以提升自己的写作、总结能力，一个技术点我们会使用，并不难，但是要做到让让别人也能听懂我们讲得，还是需要一定的技巧和经验的。很多类似于简书、CSDN、博客园也都可以写文章，但是页面的样式不能自定义，而且也不是太喜欢，当然简书还算好点得。所以打算用hexo做一个博客。不罗嗦了，直接上搭建步骤，注意这篇教程是针对Mac系统的。本文亦假设首次安装hexo

配置环境

安装Node（必须）

作用：用来生成静态页面的

到Node.js[官网][官网]下载相应平台的最新版本，一路安装即可。

安装Git（必须）

作用：把本地的hexo内容提交到github上去.

申请GitHub（必须）

作用：是用来做博客的远程创库、域名、服务器之类的，怎么与本地hexo建立连接等下讲。

github账号我也不再啰嗦了,没有的话直接申请就行了，跟一般的注册账号差不多，SSH Keys，看你自己了，可以不配制，不配置的话以后每次对自己的博客有改动提交的时候就要手动输入账号密码，配置了就不需要了，怎么配置我就不多说了，网上有很多教程。（以上如果通过docker下载镜像进行部署，均可省略，其过程可参考我的上一篇[博文][博文]）

#### 1. 创建Github 域名和空间

###### 1.1注册

首先你需要注册一个[Github][github]账号，已有的直接略过该段，注意username，这会影响到你的域名，你的域名将会是 username.github.io ，所以认真的取个名字吧。

 ![docker工作架构图-_-!](/assets/blogImg/GitHub1.png) 


注册过程可能需要验证你的邮箱，其他就不在赘述。

###### 1.2 创建仓库

然后需要创建一个仓库(repository) 来存储我们的网站，点击首页任意位置出现的New repository按钮创建仓库, Respository name 中的username.github.io 的username一定与前面的Owner 一致，记住你的username下面会用到，如果是正确的话是可以绿色的可以直接创建成功的，我的是因为已经创建过该仓库所以报错的，可以直接忽略掉。
 ![docker工作架构图-_-!](/assets/blogImg/github6.png) 



第一步就已经完成了，下面是安装。

#### 2. 安装

安装Git

// 如果已安装HomeBrew 无需执行此行

$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

$ brew install git  // 安装Git
你也可以通过下载[安装程序][安装程序]来安装

安装Nodejs

先安装nvm，这是Nodejs版本管理器，可以轻松切换Nodejs版本。 这里有两种方式安装。如果使用curl的方式安装，安装完成之后一定要重启终端。

###### 2.1. Homebrew 安装方式，此安装方式无需重启

$ brew install nvm

$ mkdir ~/.nvm

$ export NVM_DIR=~/.nvm

$ . $(brew --prefix nvm)/nvm.sh
##### 2.2. curl安装方式

$ curl https://raw.github.com/creationix/nvm/master/install.sh | sh
安装完成后，重启终端并执行下列命令即可安装 Node.js。

$ nvm install 4
安装Hexo

以上所有都安装完成之后再安装Hexo

$ sudo npm install hexo-cli -g
所有必须工具已经安装完成，下面我们就可以生成博客，上传至我们的Github 仓库了。

#### 3. 编写，发布

接下来我们需要用Hexo初始化一个博客，然后更改一些自定义的配置，或者加上自己喜欢的主题，写上第一篇文章，然后发布到自己的个人Github网站(username.github.io)。

##### 3.1 创建博客

将下面的 username 替换成你自己的username(其实也无所谓，作者强迫症)，执行成功后，会创建出一个名为 username.github.io 的文件夹。

$ hexo init username.github.io
##### 3.2 更改配置

##### 3.3 主题安装

为了使博客不太难看，我们需要安装一个主题，切换至刚刚生成的Hexo 目录，安装主题

$ cd username.github.io

$ git clone https://github.com/litten/hexo-theme-yilia.git themes/yilia
##### 3.4 配置

首先我们切换到博客目录下，这个jilei6.github.io是我自己创建的文件夹，主要用来进行博客的管理，ls看下目录下的详情

 ![docker工作架构图-_-!](/assets/blogImg/github3.png) 
yilia主题就在themes目录下了,_config.yml主要是主目录下的一些配置，很清晰，每个地方应该如何填写也很简单



     title: Jaren的移动城堡
     Subtitle: 凡所有相,皆是虚妄
     description: 对佛学有些许造诣的互联网小兵一枚。
     author: Jaren
     language: zh-CN
     timezone: Asia/Shanghai #时区
     email: 434450279@qq.com
     keywords: “iOS,Java,python,Linux,Hadoop,开发者,程序猿,程序媛,极客,编程,代码,开源,IT网站,Developer,Programmer,Coder"
    url: http://www.jilei.site
    root: /
    permalink: :year/:month/:day/:title/
    permalink_defaults:
    source_dir: source
    public_dir: public
    tag_dir: tags
    archive_dir: archives
    category_dir: categories
    code_dir: downloads/code
    i18n_dir: :lang
    skip_render:
    new_post_name: :title.md # File name of new posts
    default_layout: post
    titlecase: false # Transform title into titlecase
    external_link: true # Open external links in new tab
    filename_case: 0
    render_drafts: false
    post_asset_folder: false
    relative_link: false
    future: true
    highlight:
    enable: true
    line_number: true
    auto_detect: false
    tab_replace:
    index_generator:
 
    per_page: 10
    order_by: -date
    default_category: uncategorized
    category_map:
    tag_map:
    archive: 1
    category: 1
    tag: 1
    port: 4000
    server_ip: localhost
    logger: false
    logger_format: dev
    date_format: YYYY-MM-DD
    time_format: HH:mm:ss
    per_page: 10
    pagination_dir: page
    theme: yilia
    deploy:
    type: git
    repo:
     coding:   https://git.coding.net/JarenBig/JarenBig.coding.me.git
     github: https://github.com/jilei6/jilei6.github.io.git
     sitemap:
     path: sitemap.xml
    baidusitemap:
    path: baidusitemap.xml
    feed:
    type: atom
    path: atom.xml
    limit: 100
    jsonContent:
    meta: false
    pages: false
    posts:
    title: true
    date: true
    path: true
    text: false
    raw: false
    content: false
    slug: false
    updated: false
    comments: false
    link: false
    permalink: false
    excerpt: false
    categories: false
    tags: true

里面的配置项很简单，不过还是有一些需要提一下

 ![docker工作架构图-_-!](/assets/blogImg/github2.png) 


##### 3.5 主题配置：

主题配置文件在username.github.io/themes/yilia/_config.yml中修改，这里略过。[配置详情][配置详情]



### 4 写文章

所有基础框架都已经创建完成，接下来可以开始写你的第一篇博客了

在username.github.io/source/_posts下创建你的第一个博客吧，例如，创建一个名为测试.md的文件，用Markdown大肆发挥吧，注意保存。

如：

---

title: 测试

---

> 鲁班七号，智商二百五，刮风了，下大了。

##### 4.1 测试

$ hexo s
测试服务启动，你可以在浏览器中输入https://localhost:4000访问了。

##### 4.2 安装hexo-deployer-git自动部署发布工具

$ npm install hexo-deployer-git --save
##### 4.3 发布

测试没问题后，我们就生成静态网页文件发布至我们的Github pages 中。

$ hexo clean && hexo g && hexo d
如果这是你的第一次，终端会让你输入Github 的邮箱和密码，正确输入后，就会把你的博客上传至Github 了。以后在每次把博客写完后，执行一下这个命令就可以直接发布了。

至此，一个hexo的个人博客基本上就Ok了,但是，还有一些配置可能需要我们处理一下，比较百度收录啊，配置评论啊，微信分享打赏，部署到[coding][coding]的服务啊，这些我们下面简单讲一下

#### 5 配置coding
##### 5.1

  其实大家都知道，github毕竟是在国外，国内百度的爬虫是爬不到了，而且国内访问其实是有点慢的，所以，我将博客也部署在了coding上,创建过程和github是一样的，就不在赘述

![docker工作架构图-_-!](/assets/blogImg/coding1.png) 

##### 5.2 接下来开启pages服务

进入项目之后，点击侧边栏的代码按钮，再选择page服务，前提是开启page服务是需要完善一些个人信息的，但是coding是免费提供该服务，所以还是值得的

![docker工作架构图-_-!](/assets/blogImg/coding2.png) 

开启之后如下所示，如果你没有绑定自己的的域名的话，下面的coding page 应该是已经运行在 https://Youname.coding.me

![docker工作架构图-_-!](/assets/blogImg/coding3.png) 

然后就在刚才的hexo 配置文件中配置该部署路径就好了，OK你的coding 部署也完成了。

##### 5.3 修改微信分享呗屏蔽的问题

修改themes/yilia/layout/_partial/post/share.ejs该文件如下即可

![docker工作架构图-_-!](/assets/blogImg/coding4.png) 
##### 总结：
 
当然了，如果你还像我的博客一样，自助一个相册功能的话，还是需要做一些优化的，这一点下一篇我们再给大家讲
![docker工作架构图-_-!](/assets/blogImg/coding5.png)

[hexo github链接]: https://github.com/hexojs/hexo
[参考链接]: http://ibruce.info/2013/11/22/hexo-your-blog/?utm_source=tuicool
[官网]:https://nodejs.org/en/
[github]: https://github.com
[博文]: http://www.jilei.site/2017/08/23/Dockers-%E6%98%AF%E4%B8%AA%E4%BB%80%E4%B9%88%E9%AC%BC%EF%BC%9F/
[安装程序]: https://sourceforge.net/projects/git-osx-installer/
[配置详情]: https://github.com/litten/hexo-theme-yilia
[coding]:https://coding.net/user
