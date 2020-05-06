---
title: Oauth 2.0 筆記，四種常見流程
date: 2020-02-25 14:37:20
tags: ["Auth", "Oauth", "Web", "Login", "SignIn"]
---

## 前言

最近要來整理Oauth的筆記，直接先從Oauth2.0開始。

根據[Oauth2.0的官方文件](tools.ietf.org/html/rfc6749)，先列出2.0的重點：

- 名詞解釋
- Oauth2.0 協定的流程
- Oauth2.0 授權的授與方式（四種方式）
- 存取的token
- 刷新的token


## 名詞定義解釋

我們些列出Oauth2.0裡面常見的角色與名詞，他們在官方文件裡有特定的名稱與定義。

- **受保護的資源（Protected resources）**：與個人隱私等相關的資料。例如Email、姓名、性別、生日、居住地、喜好等

四個角色：

1. **資源擁有者（Resource owner）**：上述受保護資源的擁有者，就是使用者。例如我在Facebook註冊的姓名、性別與點讚的頁面等，這些就是受保護的資源，我就是資源擁有者。

1.  **資源伺服器（Resource server）**：存放這些受保護資源的伺服器。例如Facebook/Google內部專門存放使用者相關個人資料與喜好的伺服器。

1.  **客戶/用戶（Client）**：這邊容易與使用者 - 資源擁有者搞混；這裡的客戶指的是想使用驗證服務，並存取受保護資源的應用程式。例如Momo / Pchome等購物網、希望能讓消費者直接透過Facebook/Google的帳號登入，不用額外在購物網站上完成繁雜的註冊流程。這邊使用驗證身份服務的網站就是客戶（Client），各種需要身份驗證服務的Mobile App也都是客戶。

1.  **授權伺服器（Authorization server）**：這個伺服器執行驗證並授權的流程：驗證資源擁有者的身份成功後，提供客戶存取受保護資源的token。

除此之外還有：

- **存取token（Access token）**：用來存取受保護資源的憑證。

- **刷新token（Refresh token）**：舊的存取token無效或過期後，用來獲得新的存取token。

## Oauth2.0 協定的流程

Oauth 2.0授權的授予方式可以細分位置四種

1. 授權碼（Authorization code）
1. 隱性（Implicit）
1. 資源擁有者的密碼憑證（Resource owner password credentials）
1. 客戶憑證（Client credentials）


這四種的選擇可以根據下列三點來判斷

1. 存取token（Access token）由誰持有
1. 客戶的型態為何？
1. 客戶與資源伺服器是第一方還第三方關係

![Determine the grant type](./Oauth.001.jpeg)



## References
[Oauth2.0的官方文件](tools.ietf.org/html/rfc6749)
[A Guide To OAuth 2.0 Grants - Alex Bilbie](https://alexbilbie.com/guide-to-oauth-2-grants/)
