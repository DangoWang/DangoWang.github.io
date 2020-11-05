---
layout:     post
title:      "selenium+chrome抢餐"
subtitle:   " \"手快有，手慢无\""
date:       2020-11-5 16:40:00
author:     "DD"
header-img: "img/post-bg-alitrip.jpg"
catalog: true
tags:
    - 美餐
    - selenium
---
> 我中毒了
<iframe src="//player.bilibili.com/player.html?aid=584724198&bvid=BV1rz4y1Z7iU&cid=238677930&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>

这篇记录一下美餐爬虫+抢餐的脚本开发过程。最后没有写点餐的动作，怕被公司it搞:tw-1f480:

#### meican
之前用github上的开源项目[meican](https://github.com/LKI/meican)来点餐，但是发现美餐有反爬虫限制，每次刷个二三十次就被反爬虫了。遂放弃。
### selenium
selenium 是一个 web 的自动化测试工具，具体介绍[在这里](https://www.jianshu.com/p/1531e12f8852).它可以前台（或后台）打开浏览器，爬取页面内容。
### 代码
[gist地址](https://gist.github.com/DangoWang/84814da407cf6aacc83e6645f9fb40d6)

