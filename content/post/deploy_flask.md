+++
date = "2016-05-14T22:40:20+08:00"
description = "python web部署"
draft = false
tags = ["flask","nginx","gunicorn","supervisor","centos7"]
title = "centos7上nginx,gunicorn,supervisor部署flask网站"
topics = ["web"]

+++

digital ocean 和 vultr 是两个处于5$价位很好的vps供应商，购买可以用我的链接哦
[购买vultr](http://www.vultr.com/?ref=6889905)

我们目标实现一个支持多个独立域名网站的线上Python环境，这会用到Virtualenv， Flask， Gunicorn， Supervisor， Nginx。
<!--more-->

### 配置用户环境

一个vps可以跑多个站，最好将它们完全隔离，每个站对应一个用户。
```
User        Site
bob         website     ##bob用户有一个dylan的站
```
配置好的目录树应该为这样
```
1. /home/bob/
   1. logs
       1. access.log
       2. error.log
       3. gunicorn_supervisor.logs
   2. website
      1. gunicorn.conf
      2. hello.py
      3. venv/
```

创建用户并且赋予sudo权限
```
useradd bob -d /home/bob -m -s /bin/bash
passwd bob

visudo 或 sudo vim /etc/sudoers
```
全局安装 pip, nginx, vritualenv, supervisor
```
sudo yum -y install nginx, python-pip, supervisor
pip install virtualenv
```
安装flask环境，同时在虚环境里安装gunicorn
```
cd /home/bob
mkdir website/
cd website/

#安装虚幻境，安装flask，gunicorn
virtualenv venv
source venv/bin/activate

pip install flask
pip install gunicorn
```

### deploy flask并且设置好gunicorn
```
vi hello.py

##写入Flask的Hello World

from flask import Flask
app = Flask(__name__)
@app.route('/')
def hello_world():
    return 'Hello World!'
    if __name__ == '__main__':
        app.run()

cd /home/bob/website
#写入gunicorn配置
vi gunicorn.conf

workers = 4
bind = '127.0.0.1:8001'
```

### 配置supervisor

```bash
#配置文件在 /etc/supervisord.d/website.ini

[program:website]
command=/home/bob/website/venv/bin/gunicorn hello:app -c /home/bob/website/gunicorn.conf
directory=/home/bob/website
user=bob
autostart=true
autorestart=true

stdout_logfile=/home/bob/logs/gunicorn_supervisor.log

#启动supervisor
sudo supervisord -c /etc/supervisord.conf

#控制台
sudo supervisorctl
    status 查看当前运行的实例状态，running or failed
    restart start or stop
    reload 载入最新的配置文件，停止原有进程并按新的配置启动、管理所有进程
    update 根据最新的配置文件，启动新配置或有改动的进程，配置没有改动的进程不会受影响而重启

#注意：显示用stop停止掉的进程，用reload或者update都不会自动重启。
```
### 配置nginx

```
$sudo vi /etc/nginx/conf.d/website.conf 

server{
        listen 80;
        server_name  xxx你的域名

        root /home/bob/website/;
        access_log /home/bob/logs/access.log;
        error_log /home/bob/logs/error.log;

        location / {
                proxy_set_header X-Forward-For $proxy_add_x_forwarded_for;
                proxy_set_header Host $http_host;
                proxy_redirect off;

                if (!-f $request_filename) {
                        proxy_pass http://127.0.0.1:8001; //gunicorn的端口
                        break;
                }
        }
}

sudo systemctl start nginx

#当然，如果有静态文件，直接可以在nginx配置访问

#配置静态文件转发
location ~.*(js|css|png|gif|jpg|mp3|ogg)$ {
        root /home/bob/website/static/xxxx;
}

#配置静态页面转发
location ~.*(html)$ {
        root /home/bob/website/app/app_static_pages/;
}
```

如果你完成以上，并配置以下host文件，相信访问域名就可以看到一个运转良好的网站了。
当然，如果是部署到大集群上，可能会用到fabic。

now, enjoy it.






### 参考文章

1. [beiyuu's blog](http://beiyuu.com/vps-config-python-vitrualenv-flask-gunicorn-supervisor-nginx/)
2. [realpython](https://realpython.com/blog/python/kickstarting-flask-on-ubuntu-setup-and-deployment/)
3. [simpleapples](http://www.simpleapples.com/2015/06/configure-nginx-supervisor-gunicorn-flask/)
