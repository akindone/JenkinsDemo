# 用Jenkins+Docker搭建Android持续集成平台

## 背景描述

随着公司扩大和测试团队的引入，原有的本地打包已经无法满足需求，所以考虑搭建Android持续集成打包平台。因为我司前后端发版是通过Docker+Jenkins，所以就沿用它。

网上对于搭建Jenkins的教程很多，但是基于Docker的很少，同时要考虑服务器上无法翻墙，所以自己摸索并填了很多坑，因此记录一下。

建议提前了解一下：

[持续集成是什么？](http://www.ruanyifeng.com/blog/2015/09/continuous-integration.html)

[Jenkins是什么？](http://www.cnblogs.com/zz0412/p/jenkins01.html)

[Docker入门gitbook](https://www.gitbook.com/book/yeasy/docker_practice)

[Android Gradle 使用教程](http://www.jianshu.com/p/e83baf81c57a)

## 设计思路

实现目标：可以根据参数打不同flavor／不同运行环境的包；收集打包成功后的成果（Artifact）；把api上传到fir后，在建构列表展示出下载链接。

思路：先在本地配置Docker+Jenkins，本地测试打包成功后，再把dockerfile上传到服务器。以下配置讲的是在本地（mac）搭建一个Docker+Jenkins的持续集成环境。

## 开始配置

1. #### 配置Docker

   1. #### 去官网[下载](https://store.docker.com/editions/community/docker-ce-desktop-mac)最新的Docker，并安装

      因为在docker里下载jenkins速度比较慢，所以建议给docker配置阿里镜像加速，具体可以网搜一下教程，配置完如下图

      ![jenkins——001](/Users/jike/Downloads/jenkins——001.jpg)

   2. #### 修改Dockerfile文件

      Docker作用相当于一个虚拟机，你可以在它里面安装你需要的环境，但开销比虚拟机要小。

      我们这里需要配置java和android。在[Docker hub](https://hub.docker.com/)上可以搜到不少android的配置，可以参考一下。

      Dockerfile是构建一个容器的默认的配置文件，我们编写一个Dockerfile，在这个Docker容器内配置所需要的环境。

      [Dockerfile的常用指令](http://fpliu-blog.chinacloudsites.cn/it/software/Docker/Dockerfile)：

      FROM：指定一个基础镜像，必须写在文件的第一行。基础镜像可以是官方远程仓库或者本地仓库

      USER：设置容器的运行用户，默认是`root`

      RUN：运行任何被基础镜像支持的命令。如果基础镜像选择了`ubuntu`，那么只能执行`ubuntu`支持的命令。比如RUN mkdir temp

      ENV：用于设置环境变量，设置了后，后续的`RUN`命令都可以使用，并在容器启动时、启动后也一直有效。

      ​

   3. #### 运行Docker

      命令行进入Dockerfile目录，输入：`docker build -t <tagname> . ` 将开始执行脚步中的内容

      完成后再在命令行输入`docker run -p 8080:8080 -p 50000:50000 <tagname> `这样就会开始运行了安装好所需环境的镜像

   4. #### 等脚本跑完在浏览器输入http://localhost:8080/，进入jenkins登录界面。

      如果是第一次运行jenkinsA，需要输入密钥，这个在跑第二个命令时会输出到命令行，注意查看一下。

2. #### 改造Android项目

   1. 在项目根目录新建**gradle.properties**文件，在这里指定默认的api运行环境

      ```
      ...
      API_URL_TYPE=test
      ```

      ​

   2. 在一个安全的目录下存放storeFile，并新建keystore.properties文件，注意此文件的私密性

      ```
      storePassword=xxx
      keyPassword=xxx
      keyAlias=xxx
      storeFile=file/director/storefile.jks # 这个是key文件相对于module的路径
      ```

   3. 修改module下的build.gradle文件

      ```
      // Load keystore 增加读取keystore属性文件的代码
      def keystorePropertiesFile = rootProject.file("keystore.properties")
      def keystoreProperties = new Properties()
      keystoreProperties.load(new FileInputStream(keystorePropertiesFile))

      android{
        defaultConfig{
          // 自定义配置变量
          buildConfigField 'String', 'API_URL', API_URL
        }
      }

      // 设置签名配置
      signingConfigs {
        release {
          storeFile file(keystoreProperties['storeFile'])
          storePassword keystoreProperties['storePassword']
          keyAlias keystoreProperties['keyAlias']
          keyPassword keystoreProperties['keyPassword']
        }
      }

      // 指定release包的签名配置
      buildTypes{
        release{
          signingConfig signingConfigs.release
        }
      }
      ```

3. #### 创建Jenkins打包项目

   1. ####  

## 总结

## 参考文档

> [使用Jenkins搭建iOS/Android持续集成打包平台](http://debugtalk.com/post/iOS-Android-Packing-with-Jenkins/)
>
> [在mac osx 下使用 Jenkins对Android 进行持续集成](http://blog.csdn.net/yuan514168845/article/details/51586315)
>
> [基于Docker容器搭建Android SDK环境](http://fpliu-blog.chinacloudsites.cn/it/os/android/sdk/installation/on-docker)
>
> [github添加DeployKey](https://gist.github.com/zhujunsan/a0becf82ade50ed06115)
>
> 

## 下载链接





