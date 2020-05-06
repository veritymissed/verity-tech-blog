---
title: Nginx 開源版本安裝
date: 2020-04-07 16:28:11
tags:
- Web
- Nginx
- DevOps
- Backend
- Security
- SSL
- Linux
- Server
---

## 預備

- Nginx官方的文件寫的蠻容易理解，我就直接照他的文件 [Installing NGINX Open Source](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/) 來做。

- 我的環境為Ubnutu 18.04("Bionic")

## 安裝流程

### 1.

```sh
sudo wget https://nginx.org/keys/nginx_signing.key
sudo apt-key add nginx_signing.key
```

### 2. 更改
我們要使用`apt-get`下載Nginx，直接在`/etc/apt/sources.list`新增打算安裝Nginx的版本跟位置。格式如下，`<CODENAME>`是Ubuntu版本的名稱。


```sh
deb https://nginx.org/packages/mainline/debian/ <CODENAME> nginx
deb-src https://nginx.org/packages/mainline/debian/ <CODENAME> nginx
```

> 列一下Ubuntu官網查到的版本名稱：
- Debian 9 (“Stretch”)
- Debian 10 (“Buster”)
- Ubuntu 16.04 LTS (“Xenial”) (i386, x86_64, ppc64le, - aarch64)
- Ubuntu 18.04 LTS (“Bionic”)
- Ubuntu 19.04 (“Disco”)
- Ubuntu 19.10 (“Eoan”)


用vim編輯檔案`vim /etc/apt/sources.list`

我們可以看到原先在檔案裡面的其它套件使用的ubuntu版本名稱是`bionic`，按照這個名稱加入Nginx的source到最下面。

```sh
## Uncomment the following two lines to add software from Canonical's
## 'partner' repository.
## This software is not part of Ubuntu, but is offered by Canonical and the
## respective vendors as a service to Ubuntu users.
# deb http://archive.canonical.com/ubuntu bionic partner
# deb-src http://archive.canonical.com/ubuntu bionic partner

deb http://security.ubuntu.com/ubuntu bionic-security main restricted
# deb-src http://security.ubuntu.com/ubuntu bionic-security main restricted
deb http://security.ubuntu.com/ubuntu bionic-security universe
# deb-src http://security.ubuntu.com/ubuntu bionic-security universe
deb http://security.ubuntu.com/ubuntu bionic-security multiverse
deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable
# deb-src [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable
# deb-src http://security.ubuntu.com/ubuntu bionic-security multiverse
deb http://archive.ubuntu.com/ubuntu bionic universe

# 加入 Nginx install的source，照上面使用`bionic`這個版本
deb https://nginx.org/packages/mainline/ubuntu/ bionic nginx
deb-src https://nginx.org/packages/mainline/ubuntu/ bionic nginx
-- INSERT (paste) --                                                            
```

根據指令移除可能存在的舊版本，安裝剛剛在`source.list`中指定的nginx版本
``` sh
$ sudo apt-get remove nginx-common
$ sudo apt-get update
$ sudo apt-get install nginx
```

安裝完後執行
```sh
nginx
```

### 檢查Nginx是否正確跑起來

1. ps - 用`ps` 檢查一下nginx是否執行中

``` sh
ps aux | grep "nginx"
...
root      9463  0.0  0.1 141116  1552 ?        Ss   07:30   0:00 nginx: master process nginx
www-data  9464  0.0  0.6 143788  6396 ?        S    07:30   0:00 nginx: worker process
root      9466  0.0  0.1  14428  1088 pts/0    S+   07:30   0:00 grep --color=auto nginx

```

2. curl - 用`curl`對localhost送出request

```sh
curl -I 127.0.0.1
...
HTTP/1.1 200 OK
Server: nginx/1.14.0 (Ubuntu)
Date: Tue, 07 Apr 2020 07:52:45 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 07 Apr 2020 07:28:05 GMT
Connection: keep-alive
ETag: "5e8c2b85-264"
Accept-Ranges: bytes

```

3. 瀏覽器打開`localhost`或是`127.0.0.1`

![browser](./browser.png)

顯示畫面如上圖就是成功跑起Nginx了，Nginx會監聽 port 80。

## Controlling NGINX

就是 Nginx的指令，可以關閉、重新載入設定檔等的指令。

```sh
nginx -s <SIGNAL>
```
- quit – Shut down gracefully
- reload – Reload the configuration file
- reopen – Reopen log files
- stop – Shut down immediately (fast shutdown)


## References

[Installing NGINX Open Source](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/)
