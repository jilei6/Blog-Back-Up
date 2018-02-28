---
title: iOS自动化打包(Jenkins + gitlab + fir + shell)
author: Jaren
date: 2018-01-05 17:08:05
tags:
     - Jenkins
     - gitlab
     - 自动化打包
---
>**概述**：iOS 的打包是一件令人心烦的事，经常会因为打包而打断当前做的事情。程序员时间本来就宝贵，能不能把打包这件事完全交给机器去做，自己只需要发一个打包命令给机器就不用管呢？答案是肯定的。
<!-- more -->

安装Jenkins
安装jenkins方法有两种:  
 1,使用homebrew安装.   
 2,直接下载安装包安装;  
 方法一:使用homebrew命令行安装  
  安装homebrew  
  
  
       $ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
安装Jenkins  
    
    $ brew install jenkins   
 启动jenkins  
 
        $ jenkins   
方法二:使用安装包安装   
 下载安装包地址:[Download Jenkins][Download Jenkins]
  选择对应的系统安装包,我用的是mac所以选择Mac OSX
  ![docker工作架构图-_-!](/assets/blogImg/jenkins1.png)     
   安装后会自动打开http://localhost:8080/,如果是brewhome安装,请浏览器中手动打开该网址:     
    
       http://localhost:8080/   
 jenkins默认端口号为:8080,如果端口冲突请修改端口号:    
 
       $ default write /Library/Preferences/org.jenkins-ci httpPort xxxx    
 注意: xxxx为你要修改的端口号.    
 第一次进入http://localhost:8080/  页面会要求输入初始密码:  
 ![docker工作架构图-_-!](/assets/blogImg/jenkins2.png)      
  使用终端按照上面的路径获取密码,这里使用的是cat命令获取(如需权限请使用sudo cat):      
   
       sudo cat /Users/Shared/Jenkins/Home/secrets/initialAdminPassword  
   ![docker工作架构图-_-!](/assets/blogImg/jenkins3.png)    
   进入选择是否安装插件页面,一般情况下选择第一个(安装建议的插件),第二个是自定义安装插件.    
    ![docker工作架构图-_-!](/assets/blogImg/jenkins4.png)     
  
   选择后进入插件安装页面,这时一般都会有些插件安装失败,不过这没有关系,因为进入jenkins后还可以重新把安装失败的安装一遍.     

   注意: 有时候回出现安装插件时怎么都不动的情况,如果很久都不动也不要傻傻的等在那里浪费时间,只需要关机重启一下就好了(会跳到注册页面).    
    ![docker工作架构图-_-!](/assets/blogImg/jenkins5.png)   
    点击continue进入注册第一个管理员账户页面:    
    ![docker工作架构图-_-!](/assets/blogImg/jenkins6.png)   
    填写完信息,保存,会进入完成页面,点击开启进入下面页面:
    ![docker工作架构图-_-!](/assets/blogImg/jenkins7.png)   
   插件安装

   因为我是打从gitlab管理源码并打ios包,然后发布到fir上面,所以需要以下插件:
 
       1,gitLab Plugin
       2,gitLab Hook Plugin
       3,Xcode integration
       4,keychains and provisioning profiles Management
       5,CocoaPods Jenkins Integration
       6,build Timeout
       7,description setter plugin
       8,Email Extension Plugin
       9,SSH Agent Plugin
       10, workSpace Cleanup Plugin
       11,fir-Plugin

 进入Jenkins界面后,点击系统管理:   
    ![docker工作架构图-_-!](/assets/blogImg/jenkins8.png)    
    会进入管理页面,你会看到这个页面有一片红,不要紧张,这里就是提示的上面安装时哪些插件安装失败了.    
    ![docker工作架构图-_-!](/assets/blogImg/jenkins9.png)   
    在该页面向下划选择管理插件:    
    ![docker工作架构图-_-!](/assets/blogImg/jenkins10.png)    
     进入插件选择页面:    
    ![docker工作架构图-_-!](/assets/blogImg/jenkins11.png)     
    选择插件点击直接安装进入安装页面:    
   ![docker工作架构图-_-!](/assets/blogImg/jenkins12.png)   
    有些会安装失败,没关系,按照上面多安装几次就好了.这时可以按照此方法,吧上面的一片红一个一个安装.

但是有些插件在上面是搜不到的,例如fir-Plugin,那只能自己去下载fir-Plugin然后使用高级方法安装.

[fir插件下载][fir插件下载](插件修复文件)

[新版fir-plugn][新版fir-plugn]

注意:新版本的fir插件默认要上传dysm file文件,旧版本的默认不上传,这里使用旧版本,不然会报错,如果报错换回旧版本就好了   

    Deployment failed : Prefix string too short
    Build step 'Upload to fir.im' marked build as failure    
   下载插件后进入插件管理->高级选项,然后找到上传插件,选择已下载好的fir-plugin.hpi插件.并点击上传等待安装成功,   
    ![docker工作架构图-_-!](/assets/blogImg/jenkins13.png)    
    ps: 安装完成插件后可以重启Jenkins,关于Jenkins服务关闭和重启方法:   
     
     关闭Jenkins服务:  在Jenkins服务器网址后面加上exit    http://localhost:8080/exit

     重启Jenkins服务:  在Jenkins服务器网址后面加上restart    http://localhost:8080/restart  
   创建任务   
   ![docker工作架构图-_-!](/assets/blogImg/jenkins14.png)     
   ![docker工作架构图-_-!](/assets/blogImg/jenkins15.png)   
   ![docker工作架构图-_-!](/assets/blogImg/jenkins16.png)  
   源码管理

我这里是用git管理的源码,所以选择git:   
 ![docker工作架构图-_-!](/assets/blogImg/jenkins17.png)  
  这个时候会报错,没有权限访问git仓库,因为这是私有仓库,所以需要配置SSH获取访问权限.

配置SSH需要在git上和Jenkins都要配置.因为Jenkins系统使用的是Jenkins这个系统账号,所以需要使用jenkins账户这个SSH.

切到jenkins账号生成新的ssh密钥,公钥在上传的git服务器就好了

1,打开终端名为jenkins的用户,并设置一个密码

    sudo dscl . passwd /Users/jenkins  
设置jenkins的密码:

![docker工作架构图-_-!](/assets/blogImg/jenkins18.png)   
2,切换到jenkins用户

    su jenkins
出现以下提示,表示切换成功   

![docker工作架构图-_-!](/assets/blogImg/jenkins19.png)   
3,以jenkins用户身份生成ssh key

    ssh-keygen -t rsa -C "你的邮箱标识"
4,把公共密钥放到git服务器, jenkins 私钥,jenkins.pub 公钥 

获取公钥

    $ cat /Users/Shared/Jenkins/.ssh/id_rsa.pub
5,将私钥放到jenkins中进行创建Credentials

私钥获取

    $cat /Users/Shared/Jenkins/.ssh/id_rsa  
  ![docker工作架构图-_-!](/assets/blogImg/jenkins20.png)   
  ![docker工作架构图-_-!](/assets/blogImg/jenkins21.png)   
  ![docker工作架构图-_-!](/assets/blogImg/jenkins22.png)   



点击保存

回到刚刚创建的项目,点击配置   
  ![docker工作架构图-_-!](/assets/blogImg/jenkins23.png)   

点击保存,进行构建    
  ![docker工作架构图-_-!](/assets/blogImg/jenkins24.png)   

构建完成查看结果    
![docker工作架构图-_-!](/assets/blogImg/jenkins25.png)   
![docker工作架构图-_-!](/assets/blogImg/jenkins26.png)   
![docker工作架构图-_-!](/assets/blogImg/jenkins27.png)   



看到success证明构建成功, 这所名已经可以从gitlab上面正常拉去代码了,第一步成功

Jenkins账户下进行更换ruby源

因为需要更新pod文件,所以要把jenkens账户下的ruby源更换成淘宝的ruby源(首先要切换到jenkins账户下)

###### 删除默认的官方源

    gem sources -r https://rubygems.org/
###### 添加淘宝源

    gem sources -a https://ruby.taobao.org/
###### 查看当前源

    gem sources -l                       
如果不更新,pod会出错
```
Cloning spec repo `master` from `https://github.com/CocoaPods/Specs.git` (branch `master`)

  $ /usr/bin/git clone https://github.com/CocoaPods/Specs.git master --progress

  Cloning into 'master'...

  fatal: unable to access 'https://github.com/CocoaPods/Specs.git/': SSLRead() return error -9845

[!] Unable to add a source with url `https://github.com/CocoaPods/Specs.git` named `master`.

(/usr/bin/git clone https://github.com/CocoaPods/Specs.git master --progress

Cloning into 'master'...

fatal: unable to access 'https://github.com/CocoaPods/Specs.git/': SSLRead() return error -9845

)

You can try adding it manually in `~/.cocoapods/repos` or via `pod repo add`.
```
PATH配置:
如果出现以下问题:1.pod:command not found, 2:wget:command not found 这类问题,可通过配置PATH解决,如果没有这样的问题,可不配置.

配置: 终端输入$PATH,得到path如下    
![docker工作架构图-_-!](/assets/blogImg/jenkins28.png)   

打开Jenkins->系统管理->系统设置->全局属性->Environment variables->键值对列表中设置    
![docker工作架构图-_-!](/assets/blogImg/jenkins29.png)   

构建    

![docker工作架构图-_-!](/assets/blogImg/jenkins30.png)   
![docker工作架构图-_-!](/assets/blogImg/jenkins31.png)   

打包脚本
```

echo 'start build TestDemo'

pwd

whoami

export LANG=en_US.UTF-8

export LANGUAGE=en_US.UTF-8

export LC_ALL=en_US.UTF-8

#工程环境路径

workspace_path=.

#项目名称

project_name=FamilyDoctor

echo"第一步，更新库文件"

/usr/local/bin/pod update --verbose --no-repo-update

echo "第二步，清除缓存文件...................."

xcodebuild clean

rm -rf archive

rm -f $project_name.ipa

echo "第三步，设置打包环境，准备开始打ipa包...................."

sed -i '' 's/\app-store\<\/string\>/\development\<\/string\>/' archieveOpt.plist

sed -i '' 's/ProvisioningStyle = Automatic;/ProvisioningStyle = Manual;/' $project_name.xcodeproj/project.pbxproj

echo "第四步，执行编译生成.app命令"

xcodebuild archive -workspace $project_name.xcworkspace -scheme $project_name -configuration Release -archivePath archive/$project_name.xcarchive CODE_SIGN_IDENTITY="iPhone Developer: Zhengyang Wu (JYGL73395F)" PROVISIONING_PROFILE_SPECIFIER="development_patient_kevin"

echo "第五步，执行编译生成.ipa命令"

xcodebuild -exportArchive -exportOptionsPlist archieveOpt.plist -archivePath archive/$project_name.xcarchive -exportPath ./ -configuration Release

xcodebuild这是apple提供给我们的编译打包命令, 可以看到这里xcodebuild 使用了三次,

 第一次是是清除文件,

第二次是设置编译打包的参数,

第三次是设置输出目录等

解释:

1,$project_name是声明的项目名称变量,例如project_name="A",这样在脚本中所有出现的$project_name都会被替换成A.

2,-workspace 指明的是工作空间的名称,如果指明了这个参数,那就必须指明-scheme参数.

3,-configuration Release这个不是必须的,因为默认打的就是Release包,如果想打Debug包可以单独设置.

4,-archivePath archive/$project_name.xcarchive 根据需要指定打包路径,其中扩展名记得要是.xcarchive

archive命令用于告诉编译器,这里是要执行打包命令,archivePath指定打包路径

5,CODE_SIGN_IDENTITY,它指明的是证书的标识,这个可以从钥匙串中拷贝,有一点需要注意,我们需要将自己的开发证书先拷贝到系统证书中,这是因为钥匙串中的证书,拥有者是管理者,不是jenkins用户,jenkins用户是没有权限操作证书的,拷贝到系统下面,jenkins用户就有权限了.

6,PROVISIONING_PROFILE_SPECIFIER看名字就知道是profile文件,同上我们需要将管理员目录中的profile拷贝到jenkins的profile目录中,其中管理员的profile目录在:

/users/$用户名/library/MobileDevice/Provisioning\ Profiles($用户名一般是你电脑的名字,如我的是lei).

jenkins的profile目录位于(如果没有相应的目录自己创建):

/users/shared/library/MobileDevice/Provisioning\ Profiles(如果没有访问权限,记得使用sudo获取权限)

ps如果不确定对应的profile是哪个,就删掉所有的profile,安装对应的.

7,-exportOptionsPlist archieveOpt.plist这里的archieveOpt.plist是我们在工程目录下新建的.plist文件,该文件指明打包的类型和teamid.

```
位置如图:   
![docker工作架构图-_-!](/assets/blogImg/jenkins32.png)   

.plist内容:   
![docker工作架构图-_-!](/assets/blogImg/jenkins33.png)   

provisioningProfile是xcode9必须的,不然会打包失败,xcode9以前不用加.

teamID就是打包使用的证书的teamID

method对应的是打包了类型,一般有development,app-store,ad-hoc,enterprise等选项可供选择.

8,-exportPath ./指明ipa的输出目录就在当前目录.(可根据需要调整)

进行构建可成功打出ipa包,可去当前目录下查看

上传fir

1,点击构建后操作.添加upload to fir   
![docker工作架构图-_-!](/assets/blogImg/jenkins34.png)   
![docker工作架构图-_-!](/assets/blogImg/jenkins35.png)   


2,fir.im Token是必填的,查看方法：请登录fir.im后，点击头像选择API token进行查看   
![docker工作架构图-_-!](/assets/blogImg/jenkins36.jpeg)   

3,2.IPA/APK Files（可选）

接下来，选择生成 ipa/apk 文件路径

注意：如果没有填写该选项，插会件自动默认查找 Jenkins 创建的项目目录下的 apk/ipa 文件

4,Build Notes(可选)

作用：上传 fir.im 后，可显示出更新日志

添加描述和链接

上传fir后肯定想显示一个链接,能直接跳到fir的二维码页面

添加set build description    

![docker工作架构图-_-!](/assets/blogImg/jenkins37.png)   
![docker工作架构图-_-!](/assets/blogImg/jenkins38.png)   

构建后会出现如下效果:    
![docker工作架构图-_-!](/assets/blogImg/jenkins39.png)   

可以看出显示的是html格式,并不是可以点击的链接,这是因为jenkins没有设置支持html语法,需要在

jenkins->系统管理->configure global security->Markup Formatter中选择"safe HTML"就可以了.   
![docker工作架构图-_-!](/assets/blogImg/jenkins40.png)   

效果如下:   
![docker工作架构图-_-!](/assets/blogImg/jenkins41.png)   

至此打包完成. 


    
    
    
    
[Download Jenkins]:  https://jenkins.io/download/  
[fir插件下载]:https://link.jianshu.com/?t=http://7qn9ic.com1.z0.glb.clouddn.com/fir-plugin-fixed.hpi
[新版fir-plugn]:https://link.jianshu.com/?t=http://7xju1s.com1.z0.glb.clouddn.com/fir-plugin-1.9.5.hpi
