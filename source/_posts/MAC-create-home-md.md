---
title: MAC 创建 /home 文件夹
date: 2018-03-23 19:27:50
categories: IT
tags: [mac]
---
直接操作home 文件夹报错:Operation not supported
解决办法：
执行：
```
sudo vim /etc/auto_master
```

注释掉  #/home
之后执行：
```
sudo automount
```
