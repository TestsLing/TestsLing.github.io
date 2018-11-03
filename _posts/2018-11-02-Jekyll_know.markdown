---
layout:     post
title:      "Jekyll快速入门"
subtitle:   "使用jekyll搭建一个博客需要知道的东西"
date:       2018-11-02 14:00:00
author:     "憧憬"
header-img: "img/post-bg-alibaba.jpg"
header-mask: 0.3
catalog:    true
tags:
  - web
  - jekyll
---
## Jekyll

```
jekyll build  						当前文件夹中的内容将会生成到./site文件夹中
jekyll build --文件夹     			  指定编译目录
jekyll build --源文件  --目标文件夹	   将目标源文件生成到目标文件夹
jekyll build --watch				监听文件
jekyll serve --detach				脱离终端运行
jekyll serve --watch  				启动服务并自动编译

```

#### jekyll基本知识

```
    ---
    layout: post
    title: Blogging Like a Hacker
    ---
    头部预定义变量
    _config.yml 全局配置文件
    _posts 放置博客文章的文件夹
    文件访问时 根目录是new的目录里  文件访问时使用绝对路径
    设置文章描述
    在md文件前面自定以变量 excerpt_separator: <!--more-->
    在要断点的描述后加上  <!--more--> 即可
    post.excerpt | excerpt_separator  前台获取短字描述
```

```
    自定义变量：在头信息自定义变量后，在内容里使用{{ page.变量名 }}来使用变量
    注意：github个人博客有缓存多刷新就ok了
    循环遍历列表:
      for post in site.posts  
    //使用变量
    post.title post.url
      endfor  

    文章摘要：
    Jekyll 会自动取每篇文章从开头到第一次出现excerpt_separator的地方作为文章的摘要， 并将此内容保存到变量post（根据循环放的容器定）.excerpt中

    移除标签：{{ post.excerpt | remove: '<p>' | remove: '</p>' }}

    创建html页面:
    1.直接命名html文件在根目录下
    2.创建文件夹然后在每个文件夹里创建index.html代表新页面

    引用:
      include footer.html  该语法引用了_include目录下的footer.html文件

    传递参数:
      include footer.html param="value"  

    使用参数:
    {{ include.param }}

    post URL:(指向地址)
      post_url 2010-07-21-name-of-post  
    post URL:(指向_post目录下内容)
      post_url /subdir/2010-07-21-name-of-post  

    categories:这个变量的值是以空格隔开的词，类似命名空间一样最终显示在url的干净url
        例：在md头信息里加categories: jekyll update

    开启分页功能(注意只有html文件有效，md和textile无效):
    开启分页功能很简单，只需要在 _config.yml里边加一行，并填写每页需要几行：
        paginate: 5
        paginate_path: "blog/page:num"

    引入css等静态文件：
    ’/‘代表着根目录，和laravel以及tp很类似

    分页功能：
    在Gemfile里加入gem "jekyll-paginate"以后在终端输入bundle install 更新插件
    然后在_config.yml里加入
    gems: [jekyll-paginate]
    paginate: 12(一页多少个)
    paginate_path: "page/:num/"
    访问分页  /page/1
    然后在index.html(注意：只能在index.html里使用分页)里输入代码
      for post in paginator.posts  
      endfor  
    注意：插件改配置需要重启服务才能生效
```

