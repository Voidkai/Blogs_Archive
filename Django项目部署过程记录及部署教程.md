# Django项目部署过程记录及部署教程

> 使用Nginx+Gunicorn+Virtualenv+Supervisor在CentOS7(Python/2.7)上部署Django项目<br>
> 部署工具版本信息：<br>
> <b>Nginx/1.12.2   Gunicorn:19.9.0 Virtualenv/15.1.0 Supervisor/3.1.4 </b><br>
> Django项目版本消息：<br>
> <b>Django/2.23 Python/3.5.4</b><br>

项目开发调试一般采用的是Pycharm+mysql(wamp)，因此没有遇到过多配置问题。当要把项目迁移到web服务器主机上时，就会遇到很多的困难，其中最棘手的就是配置问题。<br>
接下来将介绍如何使用Nginx(Web服务器)+Gunicorn+Virtualenv+Supervisor来部署Django项目

##第一步：安装Python3解释器
在我们的Django项目中，采用的是Python3，而大多数的服务器提供商，提供的linux版本中默认的python环境还是python2，同时，linux中的许多服务也是基于python2（此处是部署的第一坑个坑，可能让你不断的重置系统）。因此，我们需要在服务器中安装Python3解释器。

### 1.下载源码压缩文件
安装解释器的第一步就是去[Python官网](https://www.python.org/downloads/)下载Python解释器的源码，这里下载的是[Python3.6.8版本源码压缩包](https://www.python.org/ftp/python/3.6.8/Python-3.6.8.tgz)。

接下来把压缩包上传到Centos7系统的/usr/local/src 目录下，然后切换到该目录下，执行解压缩命令：
`tar --zxvf Python-3.6.8.tgz`
解压完成后进入Python-3.6.8目录下

### 2.解决编译环境问题
因为编译时需要gcc等环境，所以先安装相关环境:`yum -y install gcc* glibc*`

### 3.编译安装
运行脚本文件，对安装进行配置:
`./configure --prefix=/usr/local/python3.6.8 --with-ssl`  
接着开始编译安装`make && male install`  
安装完成后进入python3.6.8目录下`cd /usr/local/python3.6.8/`  
可以看到目录中的四个文件夹bin、include、lib、share

### 4.运行Python解释器
安装完成后`cd bin && ./python3` 以检查是否安装成功

### 5.软链接(选做部分)
软链接实现在任意路径下输入python3进入python3的环境  
`ln -s /usr/local/python3.6.5/bin/python3 /usr/local/bin/python3`  
pip 命令也需要做一下软链接，就可以在任意目录下，用 pip3 命令安装各种模块和包了  
`ln -s /usr/local/python3.6.5/bin/pip3 /usr/local/bin/pip3`

##第二步：安装Virtualenv


##第三步：配置Gunicorn


##第四步：安装Supervisor

##第五步：配置Django