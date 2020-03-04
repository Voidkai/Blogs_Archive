# 博客网站SSL配置教程

## 基于OpenSSL + Apache的SSL配置

### 步骤一
前提：保证服务器已经安装有Apache
通过<code>sudo a2enmod ssl</code>开启SSL模块后，重新启动Apache服务器使得Apache服务器可以检测到该模块，重启命令为：<code>sudo service apache2 restart</code>

### 步骤二（申请签名证书）
免费SSL证书申请参考博文：[SSL证书设置](https://kaixuan.site/article/2019/7/24/1.html) 
申请完成后下载Apache服务器对应的证书文件和key文件。
在Apache的安装目录下通过<code>sudo mkdir /etc/apache2/ssl</code>命令创建一个子目录存放证书文件，即申请完成后得到的证书文件和Key文件。

### 步骤三
配置Apache使用SSL
主要通过修改Apache的配置文件来使SSL生效，所需要修改的配置文件为**/etc/apache2/sites-available/default-ssl.conf**
通过命令<code>sudo vim 文件位置</code>编辑ssl配置文件
1. 将ServerName修改为your_domain (IP Address)
2. 将DocumentRoot修改为/var/www/html
3. 设置SSLCertificateFile **crt文件所在位置如(/etc/apache2/ssl/apache.crt)**
4. 设置SSLCertificateKeyFile **Key文件所在位置**
保存并退出
使用以下两个命令重载Apache服务
<code>sudo a2ensite default-ssl.conf</code>
<code>sudo service apache2 restart</code>

到此，配置完成。