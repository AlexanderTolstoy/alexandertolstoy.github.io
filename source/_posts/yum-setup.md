---

title: yum setup 

categories: 

- OS 

tags: 

- yum
- linux  
- EPEL

---

## 各种linux下面的yum设置 ##

### CentOS 设置 alibaba mirror ##

[相关链接](http://mirrors.aliyun.com/help/centos)

1、备份

> mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
2、下载新的CentOS-Base.repo 到/etc/yum.repos.d/

CentOS 5
> wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-5.repo

CentOS 6
> wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo

CentOS 7
> wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo 

3、之后运行yum makecache生成缓存
> yum makecache

<!-- more -->

### 安装EPEL ###

关于EPEL

[EPEL Wiki](https://fedoraproject.org/wiki/EPEL)

利用yu安装EPEL 资源库

> yum -y install epel-release.noarch

配置阿里云 EPEL资源库

> mv /etc/yum.repos.d/epel.repo /etc/yum.repos.d/epel.repo.backup 
> wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-6.repo
> yum clean all && yum update && yum makecache


