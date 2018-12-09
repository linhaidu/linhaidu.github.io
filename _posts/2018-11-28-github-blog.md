---
title: "个人博客制作记录"
excerpt: 文章简单介绍利用github-page制作个人博客的过程

categories:
  - Blog
tags:
  - Original


---

### 一、申请github账号

去[github](https://github.com/)主页，申请账号，此处略过。

### 二、申请github-page

新建Repository，名字要跟账号名字保持一致，比如：账号.github.io。这一步设置完之后，实际上已经是一个简单的个人博客了。

### 三、博客主题风格

选择一个自己喜欢的主题，我选的是Jekyll主题，[minimal-mistakes](https://github.com/mmistakes/minimal-mistakes)，直接git clone文件到一个文件夹，然后上传到自己个人主页的Repository就好了。调试的时候，可以采用bundle exec jekyll serve 命令来调试。当然，要先安装bundle。

git相关的命令，可以参考[git-doc](https://git-scm.com/docs)。其中常用的主要有git status，git add，git commit，git push，git checkout，git pull等。根据个人喜好，配置_config.yml文件。

### 四、简历

在github上面找了一个兼容Jekyll主题的在线简历模板，[Orbit](https://github.com/xriley/Orbit-Theme)，直接git clone整个文件夹到个人博客的根目录下面，比如叫cv。然后根据修改其中的index.html文件就可以了。



