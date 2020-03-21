---
title: Express server設定SSL，使用https建立安全連線
date: 2019-6-17 14:37:20
tags: ["SSL", "NodeJS", "Javascript", "Security", "Backend"]
---

近年的網頁都有要求SSL（安全的socket層）協定的要求，也就是希望使用者瀏覽器與伺服器的傳輸間雙方的socket層能夠經過加密步驟，希望中間訊息不被第三方攔並檢視內容。

今天要讓個人網站設定完成SSL並使用https。我先選定免費提供憑證的服務商 _sslforfree.com_

## 使用相關環境：

- 主機：[Linode](https://linode.com)
- 域名服務商： [Godaddy](https://tw.godaddy.com)
- SSL憑證提供者：[sslforfree.com](https://sslforfree.com)
- Backend： ExpressJS
- Process Manager: PM2

## 基本設定

首先在我的Linode主機的主目錄下：

`$HOME/PROJECT/server.js`
```js
var createError = require('http-errors');

var express = require('express');
var path = require('path');
var logger = require('morgan');

var app = express();

app.use(logger('dev'));
app.use(express.json());
app.use(express.urlencoded({ extended: false }));

...

// catch 404 and forward to error handler
app.use(function(req, res, next) {
next(createError(404));
});

// error handler
app.use(function(err, req, res, next) {
// set locals, only providing error in development
res.locals.message = err.message;
res.locals.error = req.app.get('env') === 'development' ? err : {};

// render the error page
res.status(err.status || 500);
res.render('error');
});


var http = require('http');
var port ='80'
app.set('port', port);
var server = http.createServer(app);

server.listen(port);

```


我們多加入一行
```js
...
app.use(express.urlencoded({ extended: false }));
//加入下面這行
app.use(express.static(path.join(__dirname, './public')));
...

```
這段middleware表示可以直接取得靜態資源，`$HOME/PROJECT/public/`內的檔案可以直接透過 `http://my-domain/filename`取得。

因為我們接下來的認證流程，第一步SSL供應商必須要透過我們的domain取得他提供的亂數檔驗證。

## 申請免費的SSL憑證


### Step. 1

接下來到 [sslforfree.com](https://sslforfree.com)，輸入我們在Godaddy上註冊的域名，點選 `Create Free SSL Certificate`

![Step. 1](https://user-images.githubusercontent.com/6461602/77152743-8037b680-6ad3-11ea-9b9d-1f80980fa584.png)

### Step. 2

進入下個頁面後，點選`Manual Veification`，才會跳出下半部內容。
接著點選 `Manually Verify Domain`

![Step. 2](https://user-images.githubusercontent.com/6461602/77153107-24b9f880-6ad4-11ea-85a9-591c889eda28.png)

### Step. 3

接著下載頁面中步驟1.指定的那兩個檔案。
這兩個檔名應該要是步驟2.下面兩個連結：

`http://yourdomain/檔名1或檔名2`
`http://www.yourdomain/檔名1或檔名2`。

我重複嘗試幾次中，有幾次下載的檔案會有附檔名`.dms`，有時候沒有

![Step. 3](https://user-images.githubusercontent.com/6461602/77153109-2683bc00-6ad4-11ea-81d7-8cc02073779a.png)

接著用scp將這兩個亂數檔上傳到Linode主機上`$HOME/PROJECT/public/.well-known/acme-challenge/`內


#### Local
``` sh
scp ~/Desktop/cert/uTYowWbA6VjbDta0lXG_PbLaG1Oz1R7Cn8o6ttkj-gw ~/Desktop/cert/-ZaeIs8TdLgdpyvBrvW8kst4bqnIOlRojTjC6HM3LMk root@172.105.209.163:~/site/public/.well-known/acme-challenge/
```

#### Linode機器上，PM2重啟server.js

```bash
# $HOME/PROJECT/
pm2 restart server.js
```

接著就可以點選步驟2.的兩個網址
`http://yourdomain/檔名1或檔名2`
`http://www.yourdomain/檔名1或檔名2`。

Domain與我們的Express server設定無誤的話，點下去應該可以下載剛剛上傳的兩個亂數檔。確定下載成功後，就可以點下面的`Download SSL Certificate`

### Step. 4

成功驗證後會跳出這樣的頁面，產生Certificate/Private Key/CA Bundle，下面`Download All SSL Certificate Files`可以打包成壓縮檔下載。

![Step. 4](https://user-images.githubusercontent.com/6461602/77152362-c7717780-6ad2-11ea-8a78-e3fc4871bf29.png)

### Step. 5

在`server.js`新增幾行使用`fs`讀取下載下來的私鑰與憑證，我放在`$PROJECT/ssl/`下。

```js
...
// render the error page
res.status(err.status || 500);
res.render('error');
});

var fs = require('fs')
var privateKey  = fs.readFileSync(__dirname + '/ssl/private.key');
var certificate = fs.readFileSync(__dirname + '/ssl/certificate.crt');
var credentials = { key: privateKey, cert: certificate};

var http = require('http');
var port ='80'
...
```

### Step. 6
修改一下我們的server設定，新增一個httpsServer由`https`模組`createServer`

```js
...
var privateKey  = fs.readFileSync(__dirname + '/ssl/private.key');
var certificate = fs.readFileSync(__dirname + '/ssl/certificate.crt');
var credentials = { key: privateKey, cert: certificate};

var http = require('http');
var https = require('https');

var server = http.createServer(app);
var httpsServer = https.createServer(credentials ,app);

server.listen(80, 'verityfolio.site');
httpsServer.listen(443, 'verityfolio.site');
...

```

#### Linode機器上，PM2重啟server.js

```bash
# $HOME/PROJECT/
pm2 restart server.js
```


### Step. 7

接著就可以到瀏覽器查看了。

![Browser Screeshot](https://user-images.githubusercontent.com/6461602/77222797-9dcb5580-6b91-11ea-833b-43a76b934892.png)

點選瀏覽器url那一列的鎖頭，查看裡面憑證。

![Browser Screeshot](https://user-images.githubusercontent.com/6461602/77222790-88562b80-6b91-11ea-85c7-afbe0ef283ca.png)

可以看到我們成功申請的憑證與到期日期。

### Step Final
最後一步，寫一個`middleware`，將`http`的流量重新導向到`https`上。

```js
...
app.use(express.urlencoded({ extended: false }));
app.use(express.static(path.join(__dirname, './public')));
//加入下面這個 middleware
app.use((req, res, next) => {
  if(req.protocol === 'http'){
    res.redirect(301, `https://${req.headers.host}${req.url}`)
  }
  next()
})

...

// catch 404 and forward to error handler
app.use(function(req, res, next) {
next(createError(404));
});
...

```

## 中間碰到問題

### Step. 3 驗證時的問題

原先GoDaddy上DNS的設定是這樣
![Godaddy Screeshot](https://user-images.githubusercontent.com/6461602/77222823-e4b94b00-6b91-11ea-8272-eb5fbec399fa.png)

第一個`A`紀錄`172.*`是我Linode的IP，第二個`A`紀錄是Godaddy預設的404頁面，如果其他IP都回覆404時，最後會跳到`184.*`

我在瀏覽器上點擊連結，測試下載兩個亂碼檔時都是正確下載。但是點下`Download SSL Certificate`讓sslforfree去驗證時都會失敗。

看了一下失敗訊息，發現都是因為他透過我的domain下載結果都直接戳到`180.*`那台機器Orz。

後來索性把那個IP拿掉，等驗證完再加回來。
![Godaddy Screeshot](https://user-images.githubusercontent.com/6461602/77222817-cf442100-6b91-11ea-8681-8edc0af43fb1.png)

這樣就成功讓sslforfree網站驗證下載我們上傳的亂數檔了。



## Refs
- [Installing an SSL certificate on Node.js](https://www.namecheap.com/support/knowledgebase/article.aspx/9705/33/installing-an-ssl-certificate-on-nodejs)
