---
title: docker desktop, WSL2 error
author: Li Pie
categories: go
tags: error
date: 2021-03-07
---

安装docker desktop for windows时遇到的错误

Failed to set version to docker-desktop: exit code: -1

WSL2报错

解决方法如下

```
# 管理员运行
netsh winsock reset
```

参考链接：https://www.cnblogs.com/MysticBoy/p/13066611.html