---
title: hexo学习记录
tags:
---

# 前言
本文实现的是使用hexo编写Markdown文章，把站点的源文件存储在一个GitHub仓库，
而另外把静态文件发布到GitHub上的另外一个仓库，该仓库配置GitHub page主页

- 下面我先简单介绍一下Windows7+github平台+hexo框架的next主题博客开发步骤：
- Github上新建username.github.io仓库并初始化
- PC端安装hexo及其依赖项并熟悉开发流程
- 下载我的主题文件或者下载next主题文件
- 按照next官方教程验证主题
- 按照next官方教程配置站点文件和主题文件
- 按照next官方教程集成第三方服务
- 生成静态文件
- 开启本地服务查看站点效果
- 部署至Github

[参考博客](https://blog.csdn.net/u011475210/article/details/79023429)
[项目源码地址](https://github.com/lishuai2016/lishuai2016.github.io)
[github page主页地址](https://lishuai2016.github.io/)

# 1、安装Hexo
安装 Hexo 相当简单。然而在安装前，您必须检查电脑中是否已安装下列应用程序：
Node.js
Git
如果您的电脑中已经安装上述必备程序，那么恭喜您！接下来只需要使用 npm 即可完成 Hexo 的安装。
$ npm install -g hexo-cli

[参考](https://hexo.io/zh-cn/docs/index.html)

# 2、使用Hexo创建站点、本地预览、发布到GitHub
[hexo源码地址](https://github.com/hexojs/hexo)

(1)hexo init blog  创建站点文件，这个文件不能已经存在，否则创建失败
(2)cd blog         进入站点文件
(3)hexo server     启动本地站点预览
在cmd窗口输出：
D:\hexo\articles>hexo server
INFO  Start processing
INFO  Hexo is running at http://localhost:4000 . Press Ctrl+C to stop.

本地预览访问： http://localhost:4000

(4)hexo new "Hello Hexo"    创建一个名称为Hello Hexo的空文章，重启服务即可在本地预览到

(5)发布
npm install hexo-deployer-git --save   必须，在站点文件执行，否则会报错

在站点的配置文件配置：
deploy:
  type: git
  repo: https://github.com/lishuai2016/lishuai2016.github.io
  branch: master
  message: article
  
执行命令：
hexo clean && hexo deploy
即可在GitHub主页访问到

# 3、遇到的问题总结
## 3.1、引入next主题
没有使用git clone方式：git clone https://github.com/iissnan/hexo-theme-next themes/next
因为需要把源码发布到GitHub上的一个仓库，如果使用git clone的方式，会在commit代码到指定的仓库时遇到问题，采用下载zip源码包，
解压所下载的压缩包至站点的 themes 目录下， 并将 解压后的文件夹名称（hexo-theme-next-0.4.0）更改为 next。
[参考](http://theme-next.iissnan.com/getting-started.html)

## 3.2、文章的图片引入问题
要是通过相对路径引入会因为在发布的时候生成的静态文件按照日期构建层级文件夹，造成在文章中访问不到，因此建议使用绝对路径
如：![图片](/images/Hydrangeas.jpg)


## 3.3、next主题按照官网步骤hexo 语言设置不起作用
[文档](http://theme-next.iissnan.com/getting-started.html)
问题描述：
在主配置文件设置language: zh-Hans不起作用。
解决：
请查看theme/next/languages/目录下是否有zh-Hans.yml 文件.一般是有zh-CN.yml 但是我们按照 hexo 文档language是这样的:
language: zh-Hans 
所以要把zh-CN.yml文件改成名字为zh-Hans.yml就可以了









