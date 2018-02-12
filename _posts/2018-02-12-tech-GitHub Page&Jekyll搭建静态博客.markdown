---
layout: post
title:  "GitHub Page&Jekyll搭建静态博客（一）"
date:   2018-02-12 00:00:00
categories: tech
author: "hotacool"
description: 如何使用github快速搭建个人博客
---

*很多人都希望有个地方可以把自己的想法、感悟分享给大家。虽然有很多公共网站提供了便利的分享空间，但更希望有个专属的博客。建立网站，通常我们会想到，需要购买域名，网站存储的虚拟空间，网站开发和维护。然而，如果只是需要有个展示自我的地方，也就是说需要一个静态网站（由HTML静态页面构成，不需要动态交互，后台处理），那么完全可以考虑利用github提供的pages，来制作一个静态博客。*
<br/>网上有很多github pages建博客的文章，但大多已过时。这里简单介绍下目前的方法，以及如何通过jekyll这个工具进行管理、定制网站，并通过域名配置，发布到万网，可以在互联网中自由访问。

## GitHub Page 创建一个简单网站
1. 在github新建一个仓库(github->new repository)。之前会在github账户下创建page，但现在需要基于单独的仓库来创建。
![new repository](http://img.hb.aicdn.com/5f3fa948147bdf84afe2e7e9dc039330956e96184126d-cxmNiv_fw658)
2. 设置好你喜欢的名称，创建好一个repository后，进入settings，可以找到'GitHub Pages'对应设置的地方。
![github page setting](http://img.hb.aicdn.com/dd78b01d6dedbd7ad16b7e9ec395b18c2f435f141ff6c-VsmTCP_fw658)
现在无法设置source，因为repository还没有添加文件，没有分支。你可以git clone repository到本地，然后添加文件，git comment后git push。但简单的，你也可以直接选择一个主题，自动生成。
3. 点击'choose a theme'，会有几个供选择的网站主题，选择后，会到提交更改界面，点击Commit changes。就会提交自动生成好网站文件和设置。
4. 再次回到setting，会发现'GitHub Pages'处提示'Your site is published at https://username.github.io/repoName/'。点击该链接，跳转进你新建的网站首页。work done！
![github page setting](http://img.hb.aicdn.com/f5ce9f3a2715341d7d349757362ae5e244e4912f3bb24-aWpKmm_fw658)
 <br/>跳转：
![your page](http://img.hb.aicdn.com/62153991195e71470faa0bc4d0091f3806c06eeef0015-hSyXtU_fw658)

## 自定义设置
![source](http://img.hb.aicdn.com/6c84ee0bfc24c9eb016b460d175fa38658e1590a365b4-w2SKuf_fw658)
<br/>上面是我们自动生成的网站代码。
```
_config.yml # 站点配置文件
index.md # 首页
```
从目的上讲，我们已经完成了静态网站的搭建。然并卵，总不能只在一个index.md文件中修改，只展示一个index界面吧。

github page的本质，实际上提供了代码托管的空间，利用jekyll这个简单的博客静态站点生成工具来生成、管理。网上有很多jekyll模板，github提供的模板都相对简单，我们可以去下载更多模板[jekyll themes](http://jekyllthemes.org/)。

这里我们简单使用[fresh](http://jekyllthemes.org/themes/fresh/)模板。下载zip，解压后，将文件copy到我们的本地repo（删除之前的文件）。现在的文件：

```
.
├── LICENSE
├── README.md
├── _config.yml
├── _includes
├── _layouts
├── _pages
├── _posts
├── _site
├── assets
└── index.html
```
1. 文件夹_layouts：用于存放模板的文件夹
2. 文件夹_posts：用于存放博客文章的文件夹，里面是markdown格式的文章
3. 文件夹css：存放博客所用css的文件夹
4. _coinfig.yml：jekyll的配置文件，里面可以定义相当多的配置参数，具体配置参数可以参照其官网
5. index.html：项目的首页

选定了一个比较完整的模板。我们只需要添加博客文章即可。在_posts文件夹，参照其他.markdown文件。我们按照相同的命名规则，添加文件。编辑后push到github。博客网站就可以得到持续的更新。

目前选定了模板，我们可以实现发布文章，更新博客这些基本功能。但如何更灵活的优化我们的网站，必然需要更深入的了解jekyll这个工具。关于jekyll相关内容，下篇文章再单独介绍。

## 限制
既然是免费的，自然会有一些限制。简单来说：
* 网站即一个仓库的空间小于1G 
* 月流量 100G 
* 每小时更新不超过 10 次 

详见[Usage limits](https://help.github.com/articles/what-is-github-pages/)

但作为大多数个人博客来说，是无关痛痒的。当然，如果你是知名博客，就更不应该贪小便宜用免费了。

另外，作为博客来说，我们希望在互联网上被搜到。然而，github屏蔽了百度爬虫(可能是认为过于牛虻)，所以很遗憾，我们在github上的文章并不能被百度爬到。。关于这点，也有一些方法可以解决。一个简单的思路，是利用国内的coding.net提供的page功能。具体也在后文叙述。

