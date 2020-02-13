---
title: HTTP服务器的缺省banner
date: 2020-01-14 21:30:00
tags:
---

前一段时间在搭建服务器，进行安全扫描的时候，出现 `缺省banner` 的漏洞,下面我介绍下常见的缺省banner的设置方式。

<!-- more -->

什么是缺省banner呢？

Apache服务器
```
HTTP/1.1 200 OK
Date: Tue, 14 Jan 2020 13:51:18 GMT
Server: Apache/2.4.38 (Win64) PHP/7.3.2 # 这里显示服务器的版本号和PHP的版本号
X-Powered-By: PHP/7.3.2 # 这里显示PHP的版本号
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Transfer-Encoding: chunked
Content-Type: text/html; charset=UTF-8
```
nginx服务器
```
HTTP/1.1 200 OK
Server: nginx/1.14.2
Date: Tue, 14 Jan 2020 13:58:09 GMT
Last-Modified: Fri, 29 Nov 2019 09:01:02 GMT
Content-Type: text/html
Content-Length: 2410
ETag: "5de0de4e-96a"
Accept-Ranges: bytes
```

如果暴露远端http服务器的版本信息，这可能使得攻击者了解远程系统类型以便进行下一步的攻击。软件建议就是通过修改缺省banner。

Apache修改：
在httpd.conf中添加下面的语句：
```conf
    ServerTokens Prod
    ServerSignature Off
```
隐藏PHP版本信息：
修改php.ini中的配置：
```conf
 将 expose_php On

 修改成 expose_php Off
```
nginx修改：
修改nginx.conf，在其中添加：
```conf
server_tokens off;
```
