---
title: Gitflow Workflow
date: 2017-09-26 18:01:11
categories: IT
tags: [Git]
---
# Gitflow Workflow
官方完整版见[这里](https://www.atlassian.com/git/tutorials/comparing-workflows)

![](http://ww1.sinaimg.cn/large/e5aac86bgy1fjx4p2gf78j20u80lm40o.jpg)

1) feature 从dev branch出feature分支进行需求开发，完成改动后rebase合并回dev
2) dev dev累积足够的feature或者发布日期快到，从dev分出release_xx分支
3) release release_xx分支经过测试和bugfix之后，合并到master(如果有bug_fix，需要合并到dev)。
4) master release合并到master后，使用tag用来记录历史发布的版本。
5) hotfix 用户需要基于历史的版本进行bugfix，那么基于对应的tag，branch得到hotfix分支，完成fix，然后合并到master和dev。
