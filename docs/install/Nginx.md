## Linux安装

> 安装所需环境
>

```bash
yum -y install gcc gcc-c++ automake pcre pcre-devel zlib zlib-devel openssl openssl-devel
```

> 下载源码（可自选版本）
>

```bash
wget http://nginx.org/download/nginx-1.19.1.tar.gz
```

> 解压自选目录（个人习惯安装在/data目录下）
>

```bash
tar -zxvf  nginx-1.19.1.tar.gz
```

```bash
mkdir /data/nginx/

cd /data/nginx-1.19.1/

#指定安装目录并生成Makefile文件
./configure --prefix=/data/nginx/
#编译源码
make
#安装Nginx
make install
```

> 开机自启动设置
>

vi /lib/systemd/system/nginx.service

加入下面内容

```bash
[Unit]
Description=nginx service
After=network.target

[Service]
Type=forking
ExecStart=/data/nginx/sbin/nginx
ExecReload=/data/nginx/sbin/nginx -s reload
ExecStop=/data/nginx/sbin/nginx -s quit
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

> 如果想卸载怎么玩，下面二步即可
>

- 查找删除即可

```bash
find / -name nginx*
```

- 卸载


```bash
yum remove nginx
```

## Docker安装

> 搜索nginx镜像

```shell
docker search nginx
```

> 拉取nginx镜像

```shell
docker pull nginx
```

> 创建容器，设置端口映射、目录映射


```shell
# 在/root目录下创建nginx目录用于存储nginx数据信息
mkdir ~/nginx
cd ~/nginx
mkdir conf
cd conf
# 在~/nginx/conf/下创建nginx.conf文件,粘贴下面内容
vim nginx.conf
```
```shell

user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```




```shell
docker run -id --name=a_nginx \
-p 80:80 \
-v $PWD/conf/nginx.conf:/etc/nginx/nginx.conf \
-v $PWD/logs:/var/log/nginx \
-v $PWD/html:/usr/share/nginx/html \
nginx
```

- 参数说明：
  - **-p 80:80**：将容器的 80端口映射到宿主机的 80 端口。
  - **-v $PWD/conf/nginx.conf:/etc/nginx/nginx.conf**：将主机当前目录下的 /conf/nginx.conf 挂载到容器的 :/etc/nginx/nginx.conf。配置目录
  - **-v $PWD/logs:/var/log/nginx**：将主机当前目录下的 logs 目录挂载到容器的/var/log/nginx。日志目录
