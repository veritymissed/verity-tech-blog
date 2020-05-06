---
title: Oauth 2.0 筆記（2.1），Authorization code grant 授權碼方式
date: 2020-02-17 14:37:20
tags: ["Auth", "Oauth", "Web", "Login", "SignIn"]
---

### 1\. Authorization Code Grant Type Flow

- Authorization code <-> Access token
- 應用程式從Oauth server(authorization server)得到一段 authorization code
- Redirect URL

#### 相關參數

##### client

- client_id
- client_secret
- Registered Redirect URIs
- Supported Grant Types
- 上述參數油Oauth server提供
- `state` => 不是由Oauth提供

##### user

- login
- password

Ex.


| Client Registration      |                                                                                                                             | User Account |                                      |
|--------------------------|-----------------------------------------------------------------------------------------------------------------------------|--------------|--------------------------------------|
| client_id                | 0oappxseeykp1peAl0h7                                                                                                        | login        | determined-mandrill@example.com      |
| client_secret            | NpUnJg78q1qXq88jjYC84PMxpuGlHUoKTSkQTd2q                                                                                    | password     | Unsightly-Toucan-Average-Butterfly-8 |
| Registered Redirect URIs | https://www.oauth.com/playground/authorization-code.html<br>https://www.oauth.com/playground/authorization-code-with-pkce.html |              |                                      |
| Supported Grant Types    | authorization_code <br> refresh_token <br> implicit                                                                                     |              |                                      |




#### 流程

儲存 `state` 參數在Cookie或Session中，表示目前狀態


1. Register a Client - 如果要使用Oauth API，要先在Oauth提供者上，為自己的應用程式註冊一個客戶帳戶或是開發者帳戶。比方說開發中的App要使用Facebook的Oauth，就必須註冊開發者帳戶，並回答打算使用Oauth的App的相關資訊。

```
https://dev-396343.oktapreview.com/oauth2/default/v1/authorize?
  response_type=code
  &client_id=0oappxseeykp1peAl0h7
  &redirect_uri=https://www.oauth.com/playground/authorization-code.html
  &scope=photo+offline_access
  &state=BszOMcMVkHwPMXzk
```

2. Redirect URL

進入redirect URL 後，根據這頁裡面提供的code更改標頭的 `state`，表示這次流程的
```
?state=BszOMcMVkHwPMXzk&code=qFfxoJoLIFGQ8AtvBKED
```

攻擊者如何攻擊？
攻擊者可以偽造成client，向Oauth server送出GET請求，並得到code，所以才需要`state`
這個機制。儲存在client端並由client自身送出，的`state`參數。Client端驗證Oauth server回傳回來的`state`值，相同才能確認這個response是屬於這次流程。

3. Exchange the Authorization Code 交換授權碼

```
POST https://dev-396343.oktapreview.com/oauth2/default/v1/token

grant_type=authorization_code
&client_id=0oappxseeykp1peAl0h7
&client_secret=NpUnJg78q1qXq88jjYC84PMxpuGlHUoKTSkQTd2q
&redirect_uri=https://www.oauth.com/playground/authorization-code.html
&code=wCurajyb_OYlqzsYGVba
```

```
{
  "token_type": "Bearer",
  "expires_in": 3600,
  "access_token": "eyJraWQiOiIxZ3hlY0JSWGhrSUV6U3J1LTVVZjQ2WHhuVE5HVWFOeVVqeDlXUjF4MXFFIiwiYWxnIjoiUlMyNTYifQ.eyJ2ZXIiOjEsImp0aSI6IkFULkFSSm9Eb0hJbmkwYktkUlpSSGx4TFJ5LUp2LUlNb2hJUnY1RWpXblhnVTAuVkE2ajdtYitWaG9oNUY1bEUyd2c4MGtvUlg0UVphdFo2Q3BwU2NwWUZBZz0iLCJpc3MiOiJodHRwczovL2Rldi0zOTYzNDMub2t0YXByZXZpZXcuY29tL29hdXRoMi9kZWZhdWx0IiwiYXVkIjoiYXBpOi8vZGVmYXVsdCIsImlhdCI6MTU4MTU4NjQyNSwiZXhwIjoxNTgxNTkwMDI1LCJjaWQiOiIwb2FwcHhzZWV5a3AxcGVBbDBoNyIsInVpZCI6IjAwdXBxMTdmZWtUSUxPWGtzMGg3Iiwic2NwIjpbIm9mZmxpbmVfYWNjZXNzIiwicGhvdG8iXSwic3ViIjoiZGV0ZXJtaW5lZC1tYW5kcmlsbEBleGFtcGxlLmNvbSJ9.fHRiGRAdf5UFf3UpIq7BoVUBHMZwhJjHcFbHI87xya2yi3t_G6C32gnpCafp-qGVJDNIx8yfWqmatey8HvojDAlqm5bdUWBF8j7J2fUwh7dFF885BI3I_GcV6XZ_VUxtes5pHlDg_Fxrb-QqkL9iWDC4DiYJEgXYkF_Zg0uH_ZcLJMV589NGHgatOiD1uz87xdYmGcWADZNFxhT_UNFRMje2RzWsZwlysXizWLQL6Pdm3Q60EO6xl0a8KbPXmpZ-4ykPFMpTrrwIw8epw9vaWbq7ZYYqsyf9qHR7q3INMO2LPBlsVLiT2p3SrfLO220TzXWBqJn-I6M8LVod0IeS-g",
  "scope": "offline_access photo",
  "refresh_token": "Wr4c66TVXmEjDBIrxy2mi-z1MFVeay3M2_Gao0QGFBc"
}
```

## Bearer token

[A Guide To OAuth 2.0 Grants - Alex Bilbie](https://alexbilbie.com/guide-to-oauth-2-grants/)
