---
title: pip mirro change
categories: 
- Language
tags: 
- Python
- pip 
---

## pip 镜像设置 ##

由于官方网站 无法正常访问(访问超时)
遂改为 阿里云的 镜像

```bash
 touch ~/.pip/pip.conf
 echo '[global]' >> ~/.pip/pip.conf
 echo 'index-url = http://mirrors.aliyun.com/pypi/simple/'  >> ~/.pip/pip.conf 
 echo ''  >> ~/.pip/pip.conf
 echo '[install]' >> ~/.pip/pip.conf
 echo 'trusted-host=mirrors.aliyun.com'   >> ~/.pip/pip.conf
```
