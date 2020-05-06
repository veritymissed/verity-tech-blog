---
title: Linux link筆記
tags:
  - linux
  - system
  - link
  - ubuntu
date: 2019-05-15 11:37:20
---

# Linux link 相關筆記

## 軟連結、硬連結

### inode
`inode`這東西可以想像成指標（pointer），在Linux系統中，他會指向硬碟的某個物理位置

```sh
ls -li # 可以列出檔案的inode值

256342 lrwxrwxrwx 1 root root  47 Apr 10 08:24 blogverity -> /etc/nginx/sites-available/blogverity.info.conf
256081 lrwxrwxrwx 1 root root  34 Apr  7 07:28 default -> /etc/nginx/sites-available/default
256343 -rw-r--r-- 2 root root 449 Apr 10 08:04 hardlink

# /etc/nginx/sites-available
ls -li

256343 -rw-r--r-- 2 root root  449 Apr 10 08:04 blogverity.info.conf

#root左邊那個2就是硬連結的數量
```

### 硬連結（Hard link）:

同一個inode值，如果有其他link也是這個值，就是這個inode的 __硬連結__，硬連結數量 __可以多個__，刪除其一會減少連結數量。只有連結數量為1時刪除會刪除整個檔案。

### 軟連結（Symbolic link）:

指向一個絕對路徑或相對路徑，刪除軟連結本身，不會影響目標檔案。若目標檔案被刪除或被移動，軟連結本身不會變動，這軟連結可以稱為 __被遺棄__。

從上面可以看到，軟連結只會顯示 `->`，硬連結不會特別顯示這個箭頭，但硬連結的inode值會與指向的目標inode值相同。




## ln — 建立連結指令

預設都是建立硬連結（Hard link）

`ln 目標的路徑 連結名稱`

若要建立軟連結（Symbolic link）: `ln -s 目標的路徑 連結名稱`

```sh
ln -s /etc/nginx/sites-available/blogverity.info.conf blogverity

ll
(等於ls -all)列出所有檔案

blogverity -> /etc/nginx/sites-available/blogverity.info.conf
```

## unlink 刪除連結

`unlink 連結名稱`

```sh
blogverity -> /etc/nginx/sites-available/blogverity.info.conf
unlink blogverity

...
ls -all 檢查連結已刪除

#若連結指向的是一個目錄（資料夾）
unilink target_dir/

```


## ln 常用參數
```sh
-s, –symbolic: 建立軟連結symbolic link。
-f, –force: 若目標檔案已經存在，直接覆蓋檔案。
-i, –interactive: 若目標檔案已經存在，會先詢問，不會直接覆蓋檔案。
-n, –no-clobber： 不會覆蓋任何檔案。
```


## References

- [Wili](https://zh.wikipedia.org/wiki/符号链接)
- [LINUX 技術手札](https://www.opencli.com/linux/ln-create-link-command)
- [Linux的硬連結和軟連線（符號連結）](https://codertw.com/程式語言/561681/)
