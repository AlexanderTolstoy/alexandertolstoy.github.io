---
title: 以网络爬虫实例学习python3
categories: 
- Language
- Python
tags: 
- python3
- network spider
---

## 参考 ##

[The Python Tutorial](https://docs.python.org/3/tutorial/)

[Python3.x爬虫教程：爬网页、爬图片、自动登录 ](http://blog.csdn.net/evankaka/article/details/46849095)

[jecvay's blog](https://jecvay.com/archive)



<!-- more -->
    

## 学习路线图 ##

``` mermaid

graph TB
S0((0)) --> P10[基本原理与流程] ;
P10 --> P20[单页面下载] ;
P20 --> P30[过滤下载<br>某种类型文件下载 如gif 如flv等] ;
P30 --> P40[递归或者关联下载<br>整个网站 或某个树枝 <br>或包括其他网站外部连接] ;
P40 --> P50[模拟登录<br>密码用户名登录] ;
P50 --> P60[数据过滤与结构化] ;
P60 --> P70[数据统计与分析] ;
P70 --> P80[集群化规模化];
P80 --> P90[性能架构优化] ;
P90 --> E0((0)) ;
```



