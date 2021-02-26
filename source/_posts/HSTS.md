---
title: HSTS
date: 2021-02-26 10:31:58
categories: 
  - 网络
tags:
---

### HSTS是什么
HSTS全称HTTP Strict-Transport-Security
```
The HTTP Strict-Transport-Security response header (often abbreviated as HSTS)
lets a web site tell browsers that it should only be accessed using HTTPS, instead of using HTTP.
```
就是是hsts是由服务端设置response header 为 Strict-Transport-Security，告诉浏览器，只支持https请求，http请求会被307重定向为https

Strict-Transport-Security: max-age= expire-time
Strict-Transport-Security: max-age= expire-time; includeSubDomains
Strict-Transport-Security: max-age=expire-time; preload

### 注意事项
```
Note: The Strict-Transport-Security header is ignored by the browser when your site is accessed using HTTP;
this is because an attacker may intercept HTTP connections and inject the header or remove it.
When your site is accessed over HTTPS with no certificate errors,
the browser knows your site is HTTPS capable and will honor the Strict-Transport-Security header.
```
就是说，http 请求设置Strict-Transport-Security浏览器是不会生效的，因为攻击者可能会拦截请求更改header，如果https的请求通过了认证，浏览器就会设置Strict-Transport-Security 

### 作用
让所有的请求都通过https访问，不允许http请求访问

### 验证
1. 访问 http://www.baidu.com, 查看network, 会发现会被307重定向到https
  ![](baidu307.jpg)
  ![](baidu307detail.jpg)

2. chrome 访问 chrome://net-internals/#hsts, 我们会发现已经有了hsts配置，如下图
  ![](hsts-detail.jpg)

3. 删除 hsts配置
  ![](hsts-delete.jpg)

4. 在访问 http://www.baid.com, 查看network, 会发现有一个302的请求（客户端重定向）
  ![](baidu302.jpg)
  ![](baidu302detail.jpg)

5. 我们再看看 https://www.baid.com 的response header
  ![](baidu-https-detail.jpg)

6. 这个时候再去看看chrome hsts设置，已经又有了

总结：一开始浏览器没有hsts设置的时候，浏览器是不会强制将请求重定向到https的，一般的解决方案是，由302重定向到https，这个时候https返回hsts设置，浏览器校验证书通过，设置hsts信息，再访问的时候，浏览器就会根据配置自动重定向到https，向服务器发送https请求。（这也就是为什么我们第一次访问的时候回307，因为我们之前已经访问过百度，浏览器已经配置过hsts了）