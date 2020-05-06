---
title: 查詢某個Port狀態的的方式
date: 2019-07-05 17:06:30
tags:
- Web
- DevOps
- DNS
- Network
- Linux
---

整理一下幾個查詢某個Port狀態的的方式


- 用`nl`開啟socket server監聽port
```sh
nc -l 127.0.0.1 9000
```


測試傳輸層PORT狀態有幾種方式，我們可以選用`nc`或`telnet`，我們打開新的`cmd`頁面


- 使用`telnet`，Telnet可以測試TCP或UDP層的port狀態
```sh
telnet 127.0.0.1 9000
#應會回傳
Trying 127.0.0.1...
Connected to localhost.
```

- 使用 `nc`
```sh
nc -v 127.0.0.1 9000
# 應會回傳
Connection to 127.0.0.1 port 9000 [tcp/cslistener] succeeded!
# 輸入input text，會在socket server那一頁顯示
```

- 使用 `lsof`可以查看使用port的process
```sh
sudo lsof -i
sudo lsof -i | TCP
# 掃描localhost的TCP port由哪些process使用
sudo lsof | grep listen
# 掃描localhost的port被哪些process監聽
sudo lsof -i :8080
# 查詢localhost:8080正由哪個process使用
```

- 使用`netstat`
```sh
netstat
```


## 主機port的掃描（Port scanning）

這行指令會掃描指定機器 2900 ~ 3000與 5000 ~ 5100 這兩個範圍的 TCP 連接埠，看看哪些埠號有開啟。

```sh
nc -vnz -w 1 172.105.209.163 2900-3000 5000-5100
```
這行則是掃描 UDP 的連接埠：
```sh
nc -vnzu 172.105.209.163 1-65535
```

## REFS
[Netcat（Linux nc 指令）網路管理者工具實用範例](https://blog.gtwang.org/linux/linux-utility-netcat-examples/)
[3 種檢查遠端埠號是否開啟的方法](https://www.opencli.com/linux/3-ways-check-remote-server-open-port)
[Linunx 指令 nc 用法](https://myapollo.com.tw/zh-tw/linux-command-nc/)
