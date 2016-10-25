---
title: Install and configure the Python ENV on RHEL6.5
categories:   
- Language  
- OS  

tags: 
- Python  
- Eclipse
- RHEL  
- Installation
---

## 说明 ##
在linux中安装python3,并且配置python3开发环境

<!-- more -->

## 参考文献 ##
1. [Python官方网站](https://www.python.org/)
2. [Python Documents 3.0](https://docs.python.org/3/)
3. [廖雪峰Python 教程](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000)
4. [Linux6.4 安装Python3.5,pip,setuptools](http://www.tuicool.com/articles/j2ueiir)
5. [setuptools 28 Installation](https://pypi.python.org/pypi/setuptools#unix-including-mac-os-x-curl)
6. [pip 8.1.2 Installation](https://pypi.python.org/pypi/pip/8.1.2)


## 安装包及下载链接 ##  
1. [eclipse IDE for C/C++ Developers(linux_x64)](http://www.eclipse.org/downloads/download.php?file=/technology/epp/downloads/release/neon/R/eclipse-cpp-neon-R-linux-gtk-x86_64.tar.gz&mirror_id=448)
2. [python3.5.2_tgz](https://www.python.org/ftp/python/3.5.2/Python-3.5.2.tgz)
3. [setuptools-28.0.0.tar.gz](https://pypi.python.org/packages/f7/94/eee867605a99ac113c4108534ad7c292ed48bf1d06dfe7b63daa51e49987/setuptools-28.0.0.tar.gz#md5=9b23df90e1510c7353a5cf07873dcd22)
4. [Python Source Download](https://www.python.org/downloads/source/)
5. [pip-8.1.2.tar.gz](https://pypi.python.org/packages/e7/a8/7556133689add8d1a54c0b14aeff0acb03c64707ce100ecd53934da1aa13/pip-8.1.2.tar.gz#md5=87083c0b9867963b29f7aba3613e8f4a)
6. 

## 安装与配置 ##
### 下载安装包 ###
```bash
cd /download
wget https://www.python.org/ftp/python/3.5.2/Python-3.5.2.tgz
wget https://pypi.python.org/packages/f7/94/eee867605a99ac113c4108534ad7c292ed48bf1d06dfe7b63daa51e49987/setuptools-28.0.0.tar.gz#md5=9b23df90e1510c7353a5cf07873dcd22
wget https://pypi.python.org/packages/e7/a8/7556133689add8d1a54c0b14aeff0acb03c64707ce100ecd53934da1aa13/pip-8.1.2.tar.gz#md5=87083c0b9867963b29f7aba3613e8f4a
```
### prerequire ###
> yum -y groupinstall "Development Tools"
> yum -y install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel
 
### 安装python3.5 ###
> cd /download/Python-3.5.2
> ./configure --prefix=/usr
> make & make install


### 安装pip以及setuptools ###  
> cd setuptools-28.0.0
> python3 setup.py build
> python3 setup.py install
> cd pip-8.1.2
> python3 setup.py build
> python3 setup.py install
> 
4.配置eclipse
5.测试简单的py程序，在eclipse中，以及在terminal中

