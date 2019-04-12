---
title: Git强制拉取覆盖本地
tags:
  - git
categories: []
date: 2019-04-12 22:17:00
---
第一份copy中git删除本地代码，完全重新初始化了代码，提交后，第二个copy使用正常的git pull 无法更新代码，需要进行强制删除操作。
``` shell
git fetch --all  
git reset --hard origin/master 
git pull
```
