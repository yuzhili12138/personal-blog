## ①.yum安装
### 1.1配置nginx源
```
rpm -ivh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm

yum install -y nginx

#启动nginx
nginx

以下是Nginx的默认路径：
(1) Nginx配置路径：/etc/nginx/
(2) PID目录：/var/run/nginx.pid
(3) 错误日志：/var/log/nginx/error.log
(4) 访问日志：/var/log/nginx/access.log
(5) 默认站点目录：/usr/share/nginx/html
```

## ②.[nginx](https://nginx.org/en/download.html)安装包安装
### 2.1 配置nginx源
```
mkdir -p /usr/local/nginx
将下载好的压缩包上传到刚刚新建好的目录下并解压
# 解压
tar -zxvf nginx-1.22.1.tar.gz

# 安装gcc，源码编译依赖 gcc 环境
yum -y install gcc-c++

# 安装pcre，pcre是一个perl库，包括perl兼容的正则表达式库，nginx的http模块使用pcre来解析正则表达式，所以需要安装pcre库
yum install -y pcre pcre-devel

# 安装zlib，zlib 库提供了很多种压缩和解压缩的方式，nginx 使用 zlib 对 http 包的内容进行 gzip
yum install -y zlib zlib-devel

# 安装OpenSSL库
yum install -y openssl openssl-devel

#进入文件夹
cd /usr/local/nginx/nginx-1.22.1

# 执行安装
./configure
make
make install

# 执行完后 输入 whereis nginx 检查是否安装成功，如果出现路径则安装成功
[root@localhost nginx-1.22.1]# whereis nginx
nginx: /usr/local/nginx
# 启动nginx，需进入安装目录
cd /usr/local/nginx/
ls
conf  html  logs  nginx-1.22.1  nginx-1.22.1.tar.gz  sbin
./sbin/nginx

# 查看进程
ps -ef | grep nginx
```



## nginx常用命令
```
# 进入nginx的执行目录
cd /usr/local/nginx/sbin

# 启动nginx
./nginx

# 停止nginx（强制停止）
./nginx -s stop

# 退出nginx（安全退出）
./nginx -s quit

# 重新加载配置文件（修改过配置文件后使用）
./nginx -s reload
# 查看进程
ps -ef | grep nginx
```