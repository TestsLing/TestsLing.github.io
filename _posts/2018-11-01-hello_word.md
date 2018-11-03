---
layout:     post
title:      "Hello 2018"
subtitle:   " \"Hello World, Hello Blog\""
date:       2018-11-01 12:00:00
author:     "憧憬"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 生活
    - Meta
---

> “Yeah It's on. ”


## 前言

憧憬 的 Blog 就这么开通了。

2018 年，Longing 总算有个地方可以好好写点东西了。


作为一个程序员， Blog 这种轮子要是挂在大众博客程序上就太没意思了。一是觉得大部分 Blog 服务都太丑，二是觉得不能随便定制不好玩。之前因为太懒没有折腾，结果就一直连个写 Blog 的地儿都没有。


<p id = "build"></p>
---

## 正文

接下来说说搭建这个博客的技术细节。  

正好之前就有关注过 [GitHub Pages](https://pages.github.com/) + [Jekyll](http://jekyllrb.com/) 快速 Building Blog 的技术方案，非常轻松时尚。

其优点非常明显：

* **Markdown** 带来的优雅写作体验
* 非常熟悉的 Git workflow ，**Git Commit 即 Blog Post**
* 利用 GitHub Pages 的域名和免费无限空间，不用自己折腾主机
	* 如果需要自定义域名，也只需要简单改改 DNS 加个 CNAME 就好了 
* Jekyll 的自定制非常容易，基本就是个模版引擎


---

配置的过程中也没遇到什么坑，基本就是 Git 的流程，相当顺手

Jekyll 主题直接 fork 了 Hux Blog

本地调试环境需要 `gem install jekyll`，结果 rubygem 的源居然被墙了……后来手动改成了我大淘宝的镜像源才成功

Theme 的 CSS 是基于 Bootstrap 定制的，看得不爽的地方直接在 Less 里改就好了（平时更习惯 SCSS 些）



## 后记

回顾这个博客的诞生，纯粹是出于个人兴趣，干程序员这么久，终于可以找个地方把以前的学习笔记发布出来了。

—— 憧憬 后记于 2018.11.01


