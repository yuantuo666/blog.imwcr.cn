---
title: Ubantu 安装MySQL+PHP+Redis
tags:
  - Linux
  - MySQL
  - 技术
  - 服务器
  - 网站
  - 网路
id: '320'
categories:
  - - VPS
  - - Web
  - - 代码片段
  - - 后端
  - - 网络
date: 2023-06-06 13:24:20
cover: /images/2023/03/lars-kienle-IlxX7xnbRF8-unsplash-scaled.jpg
coverWidth: 1200
coverHeight: 600
---

本文仅用作记录。

```
sudo apt install software-properties-common
sudo add-apt-repository ppa:ondrej/php

sudo apt update
sudo apt install php8.2 php8.2-curl php8.2-mysql

sudo apt install redis

sudo apt install mysql-server

sudo apt install nginx

sudo ufw allow 'Nginx Full'
sudo ufw status
```

如果系统已经安装 Apache / Httpd，需要先卸载

```
sudo apt remove apache2
```

## 配置 MySQL

```
mysql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password by 'mynewpassword';

<Ctrl+C>

sudo mysql_secure_installation
```

添加一个账号

```
mysql -u root -p

CREATE DATABASE TEST;

CREATE USER 'username'@'localhost' IDENTIFIED BY 'strong_password';

GRANT ALL PRIVILEGES ON TEST.* TO 'username'@'localhost' WITH GRANT OPTION;
```

## 配置 supervisor

```
vim /etc/supervisor/conf.d/test.conf

[program:test]
directory=/root/test
command=php start.php start
autostart=true
autorestart=true
startretries=3
redirect_stderr=true
stdout_logfile=/root/test.log
environment=

supervisorctl update
supervisorctl start test
supervisorctl status test
```

## 配置 Nginx

```
vim /etc/nginx/sites-enabled/test.conf

# 示例1
server {

    server_name example.com;
    root /var/www/html/;

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.2-fpm.sock;
    }
}

# 示例2
server {
listen 80;
listen [::]:80;

root /root/test/public;

# Add index.php to the list if you are using PHP
# index index.php index.html;

server_name test;

location / {
proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header REMOTE-HOST $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_pass  http://127.0.0.1:8787;
}


# deny access to .htaccess files, if Apache's document root
# concurs with nginx's one
#
#location ~ /\.ht {
#deny all;
#}
}


sudo nginx -t
sudo systemctl restart nginx
```