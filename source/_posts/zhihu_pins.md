---
title: 爬虫，知乎，用户想法
author: Li Pie
categories: 摸鱼
tags: 
- zhihu
- requests
- spider
date: 2021-04-07
---

继爬知乎用户回答后，又要收集我一个非常喜欢的用户的想法了。

![zhihu_pins_1](https://cdn.jsdelivr.net/gh/Victue/PaperBlog@main/source/images/zhihu_pins/zhihu_pins_1.png)

这两个链接感觉都很像，好像只有offset和limit区别

首先验证一下是否与回答api使用相同的加密方式

![zhihu_pins_2](https://cdn.jsdelivr.net/gh/Victue/PaperBlog@main/source/images/zhihu_pins/zhihu_pins_2.png)

```python
hash_url = '/api/v4/members/people/pins?offset=10&limit=10&includes=data%5B*%5D.upvoted_followees%2Cadmin_closed_comment'
```



![zhihu_pins_3](https://cdn.jsdelivr.net/gh/Victue/PaperBlog@main/source/images/zhihu_pins/zhihu_pins_3.png)

![zhihu_pins_4](https://cdn.jsdelivr.net/gh/Victue/PaperBlog@main/source/images/zhihu_pins/zhihu_pins_4.png)

嗯，offset=20&limit=20 √，那么显然，10 √

![zhihu_pins_5](https://cdn.jsdelivr.net/gh/Victue/PaperBlog@main/source/images/zhihu_pins/zhihu_pins_5.png)

经测试，已经是成了。具体的可以参照用户回答文档。



附：

一件很奇怪的事情，offset=20&limit=20的是第二页所有回答，offset=10&limit=10是第一页后10个回答，第一页前10个回答api应该为offset=0&limit=10，但是在XHR中并没有出现该内容

