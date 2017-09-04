title: Dockers 是个什么鬼？
toc: true
author: Jaren
date: 2017-08-23 18:18:07
tags:  
    - Docker
    - Hexo
---
![docker工作架构图-_-!](/assets/blogImg/dockerwork.jpg) 
> 最近因为通过hexo搭建了一个简易的个人静态博客，因为嫌弃hexo部署冗杂的步骤，所以研究了一些自动化部署的机制，其中就了解到了Docker,粗略的研究了下后，想跟大家一起分享下个人的一些心得理解
<!-- more -->

###### **一、官方简介**
- Docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖 > 包到一个可-移植的容器中，然后发布到任何流行的Linux机器上，也可以实现> 虚拟化，容器是完全使用沙-箱机制，相互之间不会有任何接口。

看完这个介绍，我反而是有了很多的疑惑和不解，比如说，什么是应用容器，什么是虚拟化，相互之间不会有任何接口是什么意思？下面一起来仔细研究下Dockers到底是个什么鬼

###### **二、应用容器**
   其实从字面上理解的话，应用容器应该就是装载一些Application（应用）的一个容器吧，那这个容器应该长什么样子呢？其实大家可以先自行想象一下手机里面的APP是不是就是运行在各样的系统之中呢，所以，先暂时可以把容器理解成手机的样子。举个简单的例子，客户端开发一般调试离不开模拟器，iOS的模拟器可以说相当的强大，除了不能打电话，其他的基本上都和真机差不多了，那么在电脑中要跑一个类似于手机的东东，就需要配置和手机类似的的运行环境，而模拟器就相当于这个容器，你运行的时候，可以不必要自己去开一个虚拟机，部署一个iOS的运行环境了，可以然你专注于测试你的代码，这就是容器的最大的作用无需硬件，操作系统，运行环境等，只需要运行起来这个容器就好了，当然，容器的好处远远不止这些，比如更仿真的模拟线上环境进行测试，自动化部署机制，有着较高的隔离性和安全性，极大的节省资源等等好处，所以由此来看，容器的作用和好处还是蛮大的，其实说白了，Dockers就是一个轻量级的虚拟机而已。
   
###### **三、Docker的基本架构**

Docker 的核心组件包括:  
    - docker 客户端 - Client  
    - Docker 服务器 - Docker daemon  
    - Docker 镜像 - Image  
    - Registry 仓库  
    - Docker 容器 - Container  
    
 Docker 架构如下图所示：
  ![docker工作架构图-_-!](/assets/blogImg/dockerwork.jpg) 
  
  Docker 采用的是 Client/Server 架构。客户端向服务器发送请求，服务器负责构建、运行和分发容器。客户端和服务器可以运行在同一个 Host 上，客户端也可以通过 socket 或 REST API 与远程的服务器通信。

**Docker 客户端** 

最常用的 Docker 客户端是 docker 命令。通过 docker 我们可以方便地在 Host 上构建和运行容器。docker 支持很多操作（子命令）。除了 docker 命令行工具，用户也可以通过 REST API 与服务器通信。
   
**Docker 服务器**
  
Docker daemon 是服务器组件，以 Linux 后台服务的方式运行。 Docker daemon 运行在 Docker host 上，负责创建、运行、监控容器，构建、存储镜像。 
  
**Docker 镜像**  
  
可将 Docker 镜像看着只读模板，通过它可以创建 Docker 容器。
例如某个镜像可能包含一个 Ubuntu 操作系统、一个 Apache HTTP Server 以及用户开发的 Web 应用。镜像有多种生成方法：

1.可以从无到有开始创建镜像  
2.也可以下载并使用别人创建好的现成的镜像  
3.还可以在现有镜像上创建新的镜像  
我们可以将镜像的内容和创建步骤描述在一个文本文件中，这个文件被称作 Dockerfile，通过执行 docker build <docker-file> 命令可以构建出 Docker 镜像  
**Docker 容器**  
Docker 容器就是 Docker 镜像的运行实例。

用户可以通过 CLI（docker）或是 API 启动、停止、移动或删除容器。可以这么认为，对于应用软件，镜像是软件生命周期的构建和打包阶段，而容器则是启动和运行阶段。 
**Docker Registry**  

Registry 是存放 Docker 镜像的仓库，Registry 分私有和公有两种。 
DockerHub（https://hub.docker.com/） 是默认的 Registry，由 Docker 公司维护，上面有数以万计的镜像，用户可以自由下载和使用。 
出于对速度或安全的考虑，用户也可以创建自己的私有 Registry。后面我们会学习如何搭建私有 Registry。 
docker pull 命令可以从 Registry 下载镜像。
docker run 命令则是先下载镜像（如果本地没有），然后再启动容器。      
看完上面的内容基本还是对Docker有了一个比较基础的认识，关于docker的安装，docker命令的使用,docker的设计原理等。
###### **四、Docker for Mac的安装使用**
上面讲了docker的一些基本组件，似乎还是不能让你豁然开朗，那好吧，谁让现在已经步入了工程化编码的时代呢，接下来我们用一系列的工具和简单的命令来操作我们的docker这样会让你甚至不用了解它就可以让他为你所用了  

因为我用的是mac系统，所以我就机遇MAC来进行一些操作，其他的环境小伙伴们可以自行度娘，原理都一样，一通百通。首先给大家分享2个工具，官网下载是在太慢，我把它放到百度云里免除大家被墙的痛苦和煎熬 分别是：Docker for mac 和DockerToolbox    

**为什么使用Docker for Mac**  
- 启动时不需要再使用dokcer-machine设定启动的默认的环境，省去了使用virtualbox的过程；  
- 享受和在linux下使用docker一样的体验. 总之，新工具更方便！  

**mac下启动docker的工具发展**  
- 最开始使用boot2docker  
- 再到Docker Toolbox  
- 最近新出的 Docker for Mac   

**使用Docker for Mac的一些要求** 

    1. Mac must be a 2010 or newer model, with Intel’s hardware support for memory management unit (MMU) virtualization; i.e., Extended Page Tables (EPT)
    2. OS X 10.10.3 Yosemite or newer
    3. At least 4GB of RAM
    4. VirtualBox prior to version 4.3.30 must NOT be installed (it is incompatible with Docker for Mac)
    主要就是看看你的MAC版本要高于10.10.3 内存要大于4G 而且如果安装过VirtualBox的话，他的版本不能高于4.3.30 （这个其实是有点坑的，影响了我本地虚拟的python环境）没办法
链接: https://pan.baidu.com/s/1i57oY4d 密码: nds2
好了，下载完成以后安装Docker for Mac  
打开下载后的镜像文件：
 ![docker工作架构图-_-!](/assets/blogImg/docker3.png)   
 将Docker拖入Applications即可。  
 在Applications中打开装好的Docker，看到Docker的欢迎页面，说明安装成功了。   ![docker工作架构图-_-!](/assets/blogImg/docker6.png) 
 按照提示，一路往下走，最终会看到Docker已经运行的页面。  
 
 ![docker工作架构图-_-!](/assets/blogImg/docker2.png) 

最终 在你的Launchpad中显示如下3个ICON就是已经完成安装成功了

  ![docker工作架构图-_-!](/assets/blogImg/docker4.png)
  
接下来检查下版本信息
 ```shell
$ docker --version  
Docker version 17.03.1-ce-rc1, build 3476dbf  
  
$ docker-compose --version  
docker-compose version 1.11.2, build dfed245  
  
$ docker-machine --version  
docker-machine version 0.10.0, build 76ed2a6 
```
  至此说明已经安装成功  
  总结：  
   新发布的docker for mac工具简化了启动docker的配置，如果之前使用了boot2docker或者docker toolbox，由于两者使用的虚拟机不同，docker-for-mac工具不兼容之前的虚拟机，所以在更新工具时需要清除之前的配置包括卸载虚拟机和修改环境变量等等。
具体的两者的工具的比较详见. [这里][yahoo].

**创建容器并运行它**  
我们下载的可视化工具终于要大显身手了，哈哈
![docker工作架构图-_-!](/assets/blogImg/docker7.png)
点击左上角的 new按钮创建新的容器，因为在dockerHUB上面已经有很多的镜像容器供我们使用了，所以，我们可以直接下载使用，我在这里下载了三个，因为我主要是为了制作hexo镜像所以可以再搜索框中搜索hexo下载量最高的一个使用
再下载之前还需要解决一个问题，因为众所周知的原因，dockerhub的镜像元下载会非常之慢，我们需要首先切换镜像元，还好国内有家叫DaoCloud的公司帮我们免费解决了这个问题，首先要登录其[官网][dao].进行注册流程，该流程不在赘述，注册完成之后进入该页面
![docker工作架构图-_-!](/assets/blogImg/docker8.png)
点击加速器，进入之后选择MAC系统，给直接给你奋发一个镜像链接然后将其如下图添加     

![docker工作架构图-_-!](/assets/blogImg/docker9.png)

![docker工作架构图-_-!](/assets/blogImg/docker10.png)  

添加完成之后点击apply重新运行
然后再下载容器，你会发现嗖嗖的~~  
下载完成之后，要重新设置镜像的挂载，可视化工具真的是方便啊
![docker工作架构图-_-!](/assets/blogImg/docker11.png)
在这里切换成你的文件夹路径即可，返回Kitematic页面进行重启容器然后测试下
![docker工作架构图-_-!](/assets/blogImg/docker12.png)
OK了

接下来我会实现一个docker+hexo+github+coding+Travis来实现我的个人博客

[dao]: https://dashboard.daocloud.io
[yahoo]: https://docs.docker.com/docker-for-mac/docker-toolbox/?spm=5176.100239.blogcont57215.11.fEXQsz
