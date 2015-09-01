---
layout: post
title:  "jekyll主题修改小结"
date:   2015-09-01 22:21:01
categories: jekyll
---
>jekyll是一个简单的免费的Blog生成工具，类似WordPress。但是和WordPress又有很大的不同，原因是jekyll只是一个生成静态网页的工具，不需要数据库支持。但是可以配合第三方服务,例如Disqus。最关键的是jekyll可以免费部署在Github上，而且可以绑定自己的域名。 --百度百科  

当前博客主题原型是[Lagom theme](https://github.com/swanson/lagom)，集合[@onevcat](http://weibo.com/onevcat)博客的一些帅气样式改写而成，在此记录下修改中遇见的问题；

##配置文件
####_config.yml
_config.yml是jekyll默认的配置文本，因为需要增加分页功能，在里面增加了：

```
paginate: 5
paginate_path: "page:num"
```
 具体配置看[jekyll pagination](http://jekyll.bootcss.com/docs/pagination/)

####theme.yml
theme.yml内主要是用户名称设定，一些主题放在_config.yml内；  
gravatar是对应Gravatar图像，配置过Github头像的应该都知道，不清楚的查看下[Gravatar](http://zh-tw.gravatar.com/)或[github自定义头像](http://blog.chinaunix.net/uid-25267728-id-3665771.html)；  
highlight_color代表整体风格颜色；

####_includes
_includes内的html都是模板
#####analytics.html
Google页面数据统计，因为还未使用，所以在`theme.yml`里把`google_analytics_key`注释掉；

#####footer.html
页面footer，包含一些声明等，我这里写了  
```
built with Jekyll, Theme by Lagom theme Powered by maxfong
```
#####sidebar.html
左边栏的图片修改成了120，对应的`screen.css`内需要修改`#gravatar`的`width: 120px;`
一段描述修改了样式，指向`<p class="excerpt">`

#####social.html
社交icon、链接配置的地方  
增加了`class="socialitem"`  
增加了`target="_blank"`,新建标签打开，增点留存  
`<i class="fa fa-github"></i>`这样的icon参考[Cheatsheet](http://fortawesome.github.io/Font-Awesome/cheatsheet/)

#####base.css、screen.css
基础样式，这个里面没有修改

#####layout.css
使用CSS @media，实现布局适配，这里也没有修改

#####mfs.css
增加mfs.css，提取出自己修改的一些CSS样式

#####skeleton.css
同layout.css，这个里面根据不同页面适配修改`.container`以及`.mfsauto`，主要解决屏幕宽度不停的改变所做的适配；

####_layouts
#####default.html
#####post.html

####_posts


####index.html




