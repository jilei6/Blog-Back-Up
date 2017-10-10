title: 基于Docker镜像的个人hexo博客 （二）
author: Jaren
date: 2017-09-18 17:15:18
tags:
---
>之前已经讲过通过Docker搭建个人博客的过程了，今天拿了一台新电脑正好来验证一个新电脑上怎么通过docker来部署测试我的个人博客系统   
    OK，首先，验证下电脑型号，之前的电脑，hexo环境是一步步搭建的。
 
 <!-- more -->    
   
   
   ![ico原来的样子](/assets/blogImg/hexo1.png) 
   
   这个是拿来的个人电脑。之前未安装docker，今天拿他来测试下docker环境下的hexo部署（retain屏截出来的图着实吓我一跳）  
    ![ico原来的样子](/assets/blogImg/hexo2.png)   
    
 ######  一、按照我之前的文章安装Docker客户端和环境，此处不懂得还可以上翻文章，不再赘述。

 ######  二、安装好以后，进入Kitematic中，点击左上角new按钮下载hexo-with-hexo-hey镜像

   ![ico原来的样子](/assets/blogImg/hexo3.png) 

######  三、拷贝博客文件夹至新电脑  
  
    ![ico原来的样子](/assets/blogImg/hexo4.png)    
    
    如图三所示，重新指定刚才拷贝的文件路径，重启即可以看到本地测试页面。下图中红框中的是我测试发布的照片    
    
    ![ico原来的样子](/assets/blogImg/hexo5.png)  
    
 ######  四、打开如图三所示的exec命令行，进行发布操作    
   
   ![ico原来的样子](/assets/blogImg/hexo6.png)  
   ![ico原来的样子](/assets/blogImg/hexo7.png)    
   
   当然了，别忘了添加新电脑的ssh验证，最后放一张发布上去的博客截图，  
   
   ![ico原来的样子](/assets/blogImg/hexo8.png)   
   
   OK，发布成功了，有了Docker，是不是更加方便了呢？