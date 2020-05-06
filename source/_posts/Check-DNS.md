---
title: 查詢某個domain的DNS設定
date: 2019-07-03 17:06:30
tags:
- Web
- DevOps
- DNS
- Network
- Linux
---

整理一下幾個查閱某個domain的DNS的方式

## dig

```sh
# 列出全部TYPE
dig google.com ANY +noall +answer

```

## nslookup

```sh
# 列出全部TYPE
nslookup -type=any google.com

```

## host

```sh
host -a google.com
```


## References

- [3 Simple Ways to Check DNS (Domain Name Server) Records in the Linux Terminal](https://www.2daygeek.com/check-find-dns-records-of-domain-in-linux-terminal/)
