---
layout: post
title:  "搭建自己的博客网站"
date:   2019-09-21 00:00:00 -0500
categories: diary 
---

今天是我的生日
<img src="\assets\emoji.jpg" width="3%" height="3%" >

---

&emsp;很多时候我们会写一些博客，将自己所见及所得并加上自己的思考与大家分享。当我们准备开始写博客时，通常会有以下几种选择：
1. 使用国内的博客网站，如csdn、cnblogs等，还有一些应用比如有道云笔记可以在好友之间进行分享，这些网站提供了这样一个发表博客的平台，也提供了很好的功能，但是在个性化方面就差点意思；

2. 如果想满足个性化的需求，那么我们可以自己搭建一个博客网站。可是如果要自己搭一个博客网站，不光得买服务器，而且还有环境部署和后期维护，一套下来整下来是挺麻烦的；

&emsp;以上两种方式或多或少都有些不尽如人意，不过不用慌，今天我就要介绍一种快捷方便的方式来写博客，这就是用github来搭建自己的博客网站。

&emsp;既可以满足个性化需求，让博客页面更生动丰富，而且无需自己买服务器、部署环境，因为服务器完全由github提供。github为每个用户提供一个“Github Pages”功能，它允许用户在Web上实时发布网站内容。现在我们就利用这个功能来建网站。

---

&emsp;

# 概览

1. [创建github仓库](#anchor1)

2. [搭建ruby环境](#anchor2)

   + [安装ruby](#anchor2_1)

   + [安装DevKit](#anchor2_2)

3. [使用jekyll快速搭建静态博客网站](#anchor3)

   + [安装jekyll](#anchor3_1)

   + [创建jekyll项目并运行](#anchor3_2)

&emsp;

---

<span id = "anchor1">&emsp;</span>

# 创建github仓库

当然创建github仓库前，首先你得先有一个github账号，如果没有请前往[github](http://github.com)注册一个。

有了github账号后，然后创建github仓库，在这个仓库目录中编写静态html内容就行了，github平台则会提供web服务器将这些静态内容向外展示，这就是Github Page功能。Github Page可以理解成一个github为我们提供的免费的云服务器。

>*“您可以将任何您喜欢的代码存储在Github资源库中，但要使用GitHub Pages功能实现全面效果，您的代码应该被构造为典型的网站，例如主入口点是一个名为index.html的HTML文件。”*
<p align="right">——引用自博客“<a href="https://developer.mozilla.org/zh-CN/docs/Learn/Common_questions/Using_Github_pages" target="_blank">应该如何使用Github Pages?</a>”</p>

Github Pages有两类：

+ 一种是以“username.github.io”命名的repository，那么他的master分支上的文件就能通过“username.github.io”访问

+ 另一种是建立其他名称的repositories，比如这个repository名字叫node，那么建一个gh-pages分支，该分支下的文件就能在“username.github.io/node”下访问到

我们现在选择第一种命名方式来创建仓库。比如我的用户名是“xi-jiawei”，在我的github账户下创建了一个“xi-jiawei.github.io”的仓库，便可以访问[xi-jiawei.github.io](https://xi-jiawei.github.io/)来浏览我的博客内容。

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="\assets\create_repo2.png" width="70%" height="70%" >
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">新建‘xi-jiawei.github.io’仓库</div>
</center>

也可以选择第二种方式来创建仓库。比如创建了一个“articles”的仓库，然后进入仓库页面，新建一个gh-pages分支，如下图所示。

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="\assets\create_repo_branch2.png" width="70%" height="70%" >
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">“articles”仓库新建gh-pages分支</div>
</center>

然后放一些静态文件到仓库里，比如在Git Bash上简单地输入以下命令轻松地创建一个html文件。

```
git clone https://github.com/xi-jiawei/articles
cd articles
echo "Hello World" > index.html
git add --all
git commit -m "Initial commit"
git push -u origin master
```
最后访问[xi-jiawei.github.io/articles](https://xi-jiawei.github.io/articles/)就可以看到仓库“articles”中存放的内容。

如果只是人工编写静态页面远远不够，实际上，gitHub搭建个人网站可基于jekyll或者hexo或者其它工具，这些工具都提供了各式的模板快速生成博客网站。github推荐的是jekyll这个静态网站构建工具，jekyll常用在Github中写静态页面的博客，样式也更好看，而且能直接链接到源码主页（注：获取jekyll资源请访问“[Jekyll Themes](http://jekyllthemes.org/)”）。

&emsp;

---

<span id = "anchor2">&emsp;</span>

下面将主要介绍如何使用jekyll快速搭建自己的静态博客网站。

# 搭建ruby环境

由于jekyll是用ruby语言写的一个静态网页生成工具，所以要先配置ruby环境。ruby环境分两步完成，首先是安装ruby，然后是安装DevKit开发工具。Devkit是ruby的开发工具，安装了DevKit后就可以使用gem命令安装各类gem包。

本文展示的是在Windows x64平台上搭建ruby环境的步骤。关于Linux上如何搭建ruby环境请移步“[Linux（CentOS 7）安装ruby](https://blog.csdn.net/u012120103/article/details/103939545)”。

<span id = "anchor2_1"></span>

## 安装ruby

首先，前往[ruby官网](https://rubyinstaller.org/downloads/)下载安装包。

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="\assets\download_ruby2.png" width="70%" height="70%" >
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">下载ruby和DevKit</div>
</center>

如果嫌麻烦，可以直接下载带DevKit的exe安装文件，如下图。

![rubyinstaller-devkit-2.5.6-1-x64.exe](\assets\download_ruby5.png "rubyinstaller-devkit-2.5.6-1-x64.exe")

这里我下载的是ruby压缩包文件和DevKit工具包：
+ [ruby-2.3.3-x64-mingw32.7z](https://dl.bintray.com/oneclick/rubyinstaller/ruby-2.3.3-x64-mingw32.7z)
+ [DevKit-mingw64-64-4.7.2-20130224-1432-sfx.exe](https://dl.bintray.com/oneclick/rubyinstaller/DevKit-mingw64-64-4.7.2-20130224-1432-sfx.exe)

然后解压“ruby-2.3.3-x64-mingw32.7z”，添加环境变量。

<center>
    <img src="\assets\add_ruby_path2.png" width="50%" height="50%" >
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">添加ruby路径到环境变量</div>
</center>

最后在cmd中输入“ruby -v”检验Ruby解释器是否安装成功。

```
ruby -v
```

<span id = "anchor2_2"></span>

## 安装DevKit

首先解压前面下载的DevKit工具包文件“DevKit-mingw64-64-4.7.2-20130224-1432-sfx.exe”，比如我解压到“D:\Users\veev\Web\”路径下，然后进入解压后的文件根目录下：
```
cd D:\Users\veev\Web\DevKit
```
进入文件夹后，执行以下命令，对DevKit进行初始化
```
ruby dk.rb init
```
执行完“ruby dk.rb init”后，目录“D:\Users\veev\Web\DevKit”下会生成“config.yml”文件。

找到文件“config.yml”并打开编辑，在末尾添加一行：
```
- D:\Users\veev\Web\ruby-2.3.3-x64-mingw32
```
保存文件“config.yml”的修改。输入“ruby dk.rb review”命令检验配置是否成功。
```
ruby dk.rb review
```
如果看到以下内容，表示配置成功：
```
Based upon the settings in the 'config.yml' file generated
from running 'ruby dk.rb init' and any of your customizations,
DevKit functionality will be injected into the following Rubies
when you run 'ruby dk.rb install'.
```
最后，执行“ruby dk.rb install”命令完成DevKit的安装
```
ruby dk.rb install
```
添加DevKit路径“D:\Users\veev\Web\DevKit\bin”到环境变量。在cmd中输入“gem -v”检验DevKit工具包是否安装成功。
```
gem -v
```
到此，已完成ruby的环境搭建。

&emsp;

---

<span id = "anchor3">&emsp;</span>

下面开始使用jekyll搭建自己的静态博客网站。jekyll提供各种博客模板，在模板基础上修改即可，而不必从零开始创建一个新的博客项目。

# 使用jekyll快速搭建静态博客网站

>*Jekyll 是一个简单、可扩展的静态站点生成器。用你喜欢的 标记语言书写内容并交给 Jekyll，它利用模板创建一个 静态网站。在整个处理过程中，你可以调整你想要的网址样式、 在模板中显示哪些数据等等。*
<p align="right">——引用自jekyll官网“<a href="https://www.jekyll.com.cn/docs/">快速入门 | Jekyll • 简单的博客、静态网站工具</a>”</p>

<span id = "anchor3_1"></span>

## 安装jekyll

jekyll包托管在ruby的gem服务器上，所以可以直接输入命令“gem install jekyll”即可安装jekyll，类似于python的pip、centos的yum。
```
gem install jekyll
```
在cmd中输入“jekyll -v”检验jekyll是否安装成功
jekyll -v

<span id = "anchor3_2"></span>

## 创建jekyll项目并运行

输入命令“jekyll new articles”，即可创建一个名为“articles”的jekyll项目，这是一个比较简易的静态博客网站程序。
```
jekyll new articles
```
进入刚新创建的jekyll项目根目录，然后输入命令“jekyll serve”便可运行此项目程序。
```
cd article
jekyll serve
```
<div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">注：旧版的jekyll项目可能需要输入“bundle exec jekyll serve”才能运行项目。
</div>

然后在浏览器中打开 http://localhost:4000 网址，检验博客是否建立成功。

<center>
    <img src="\assets\jekyll_articles.PNG" width="50%" height="50%" >
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">jekyll模板生成的博客网站</div>
</center>

当然，这只是一个简易的jekyll模板。如果想获取更多更好的模板，请访问“[Jekyll Themes](http://jekyllthemes.org/)”。

&emsp;

---

&emsp;

至此，搭建自己的静态博客网站完成。