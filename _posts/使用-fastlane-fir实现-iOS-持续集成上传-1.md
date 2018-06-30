title: 使用 fastlane +fir实现 iOS 持续集成上传
author: Jaren
date: 2018-03-24 17:23:23
tags:
     - Fastlane
     - Fir
---
   >距离上次写iOS自动化打包(Jenkins + gitlab + fir + shell)有一段时间了。毋庸置疑，Jenkins对我们打包的帮助还是很大的——被测试的同学追着要IPA包的日子终于一去不复返了。那为什么又要用fastlane+fir的方式呢?fastlane又是什么呢？和Jenkins的打包方式又有什么区别呢？没关系，接下来我们仔细的研究一下，看看fastlane到底是什么，以及两者之间究竟有什么差异
<!-- more -->

一、Fastlane是什么？

1.作为公司的客户端程序员，少不了在发布应用的时候各种等待。标准的手动发布流程是：编译->打包上传->填写应用更新数据->等待iTunesConnect编译->选择版本发布，整个过程大概需要30分钟左右。关键是这个过程就像windows装系统一样，虽然手工参与的不多，但是要一直守在电脑前等着。程序员都很懒，不喜欢等着电脑傻傻等待。

2.简单来说，Fastlane是一整套的工具组合套装。它里面的很多命令执行的底层并不是自己实现的，而是调用其他的插件或者工具执行的。比如说打包，Fastlane中的gym工具只是xcodebuild工具的一个封装，调用的其实还是xcodebuild中的打包命令而已。所以其实归根到底和xcode中的Archive师出同门而已。Fastlane中大多数命令都是这样类似的原理

3.Fastlane本身没有一套特殊语法，使用的Ruby语言，相信使用过cocoapods的同学应该很容易理解。

4.Fastlane可以非常快速简单的搭建一个自动化发布服务，并且支持Android，iOS，MacOS。他可以实现一条命令从编译到选版发布全程不用干预。作为程序员的你只要一条命令，看集美剧，发布就完成了。截止刚刚Fastlane官网上宣称已经为程序员节省了4百万小时+

二、Fastlane组件
```
deliver: 上传截图, 元数据, app应用程序到App Store

supply: 上传Android app应用程序和元数据到Google Play

snapshot: 自动捕获iOS app应用程序本地截图

screengrab: 自动捕获Android app应用程序本地截图

frameit: 快速截屏并将截屏放入设备中

pem: 自动生成和更新推送通知配置文件

sigh: 开发证书和描述文件下载

produce: 使用命令行在iTunes Connect上创建新的app和开发入口

cert: 自动创建和配置iOS代码签名证书

spaceship: Ruby 库访问 Apple开发者中心和 iTunes Connect

pilot: 最好的方式管理你的TestFlight 测试人员和从终端构建

boarding: 最简单的方式邀请你的TestFlight beta测试人员

gym: iOS app打包签名自动化工具

match: 使用Git同步你的团队证书和配置文件

scan: 最简单方式测试你的 iOS 和 Mac apps
```


等等....我们今天用不了这么多，仅仅需要用到2-3个而已，但是从这些组件可以看到，Fastlane还是很强大的，基本上能满足客户端打包发布的所有需求了。

三、Fastlane和Jenkins自动化的差异和选择

单就笔者对这两者的使用来看，两者之间各有利弊，关键看你的需求

1.Jenkins作为一个服务，需要搭建环境，可以部署在自家的任何一台电脑上。但是Jenkins在配置过程比较多且较为冗杂，容易出错，对于很多初级工程师来说有一定的门槛。同时Jenkins可以结合GIT插件，实现每次代码提交自动打包的自动化，同样这样其实对电脑（Mac）性能有一定的要求。

2.Fastlane则比较简便，通过命令安装好环境之和插件后，就可以直接调用他的相关命令了。但是Fastlane不支持象Jenkins结合GIT插件那样的高度自定义化服务。

综上来看，如果想要高度自定义服务，选择Jenkins的自动化集成是比较合适的，但是如果只是要实现拿到一个包或者上传，更加便捷轻量级的过程，对于初级工程师来说应该是个不错的选择。

四、Fastlane安装

系统要求:macOS或 Linux 使用 Ruby 2.0.0及以上版本

1、首先要安装正确的 Ruby 版本。在终端窗口中用下列命令来确认:

    1  ruby -v
2、然后检查 Xcode 命令行工具是否安装。在终端窗口中输入命令：

    xcode-select --install
如果未安装，终端会开始安装，如果报错误：command line tools are already installed, use "Software Update" to install updates.代表已经安装。

3、以上依赖配置好之后就可以通过 rubygem 进行安装了：

    sudo gem install fastlane
安心等待一会，fastlane就安装完成了。



打开终端，cd到你的工程目录，然后执行fastlane init后会看到如下选项

    What would you like to use fastlane for?

    1. Automate screenshots

    2. Automate beta distribution to TestFlight

    3. Automate App Store distribution

    4. Manual setup - manually setup your project to automate your tasks
这四个选项的意思是

自动截屏。这个功能能帮我们自动截取APP中的截图，并添加手机边框（如果需要的话），我们这里不选择这个选项，因为我们的项目已经有图片了，不需要这里截屏。

自动发布beta版本用于TestFlight

自动的App Store发布包。

手动设置。

选择第四个后一路回车即可，我们会看到生成了我们熟悉的fastlane目录，该目录下包含了Appfile和Fastfile。我们打开这两个文件。

Appfile

    # app_identifier("[[APP_IDENTIFIER]]") # The bundle identifier of your app

    # apple_id("[[APPLE_ID]]") # Your Apple email address

    # For more information about the Appfile, see:

    #    https://docs.fastlane.tools/advanced/#appfile
    
发现里面没有任何信息（“#”在ruby里是注释，所以里面没有任何信息）注释的部分中，app_identifier用于指定APP的bundle id，apple_id指的是你的AppleID。

目录结构大概如下所示：Fastfile下面的文件现在在你的目录下面应该是没有的，因为你并没有安装插件和MATCH等所以并不会生成相关文件。
![docker工作架构图-_-!](/assets/blogImg/ff1.png)  

Gemfile 告诉我们fastlane 依赖的gem以及版本等其他信息。这个跟本文主题不大，笔者就不详细描述了

fastlane文件夹里的Appfile看文件也知道，里面是关于本App的信息的；Fastfile则是fastlane的最主要的文件，在这个文件中可以编写我们需要使用的各个工具的顺序、方式等。

这里我们先看一个已经写好的fastlane的内容:

![docker工作架构图-_-!](/assets/blogImg/ff2.png)  


大家可以看到，这段脚本中，有两段lane ，分别是Firim和createIpa 其中Firim中又调用了createIpa和firim“方法”。那么，createIpa中的gym比较特殊的配置项就是：export_method 笔者这里设置的ad-hoc，（# 指定打包所使用的输出方式，目前支持app-store, package, ad-hoc, enterprise, development）

export_options就是设置证书的选项，这个是fastlane为了支持XCODE9，解决了在XCODE9上的BUG，所以XCODE9的小伙伴一定不要忘了加这个选项。

还有就是Firim方法其实是调用的Fir插件的函数。所以这里首先要安装，剩下就

是输入你的Firim的TOKEN就可以了。

fastlane add_plugin firim
到这里，你就可以cd 到项目根目录下面执行

fastlane Firim
耐心等待一会儿之后，  
![docker工作架构图-_-!](/assets/blogImg/ff3.png)  

可以看到输出的APP信息和fastlane打包信息 其中打包大概花费了339秒，上传用了14秒。  
![docker工作架构图-_-!](/assets/blogImg/ff4.png)  


打开Firim个人网页看到，的确也已经上传到了这里。


![docker工作架构图-_-!](/assets/blogImg/ff5.png) 

ipa文件Archive压缩包都已经出现在了工程目录下面，至此，我们的一键打包审核流程完美收工。