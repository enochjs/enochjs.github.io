---
title: charles总结
date: 2021-02-25 19:05:14
categories:
  - 工具
tags:
---

### charles 是什么？

```
官方解释：
  Charles is an HTTP proxy / HTTP monitor / Reverse Proxy
  that enables a developer to view all of the HTTP and SSL / HTTPS traffic
  between their machine and the Internet.
  This includes requests, responses and the HTTP headers
  (which contain the cookies and caching information).
```

解释：charles 是一个代理、监控、反向代理的服务器，可以抓取 http、https 的网络请求，包含 requests, responses and the HTTP headers

### 什么是代理（proxy）

代理类似于中间商，我将需求告诉代理服务器，代理服务器处理之后，发给真正的服务器，然后将结果返回给我，而 charles 代理的是 http、https 请求（这个时候就可以对请求做任意的改动，如修改入参、修改 header、修改 response 等）

### 手机如何通过 charles 走代理

1. 手机开启代理
   设置 => 无线局域网 => 选择 wifi => 点击 wifi，进入网络设置 => 配置代理 => 选择手动 => 服务器填写：电脑 ip 端口写 8888（charles 默认端口 8888）
2. 如何查看电脑 ip

- mac => ifconfig en0
- windows => ipconfig
- charles => help => local address

3. 代理 https (https 需要开启 ssl，手机需要安装证书，否则无法访问)

- 手机上需要装 charles 的证书
  a. 手机代理到电脑之后访问 chls.pro/ssl (手机代理到 charles 之后，charles 就相当于服务器，证书由当前电脑上的 charles 下发，所以每个电脑都是不一样的，测试机访问不同的电脑，都需要安装对应电脑上下发的证书)
  b. 下载证书
  c. 安装证书
  d. ios10 以上需要信任证书 （设置 => 通用 => 关于本机 => 证书信任设置 => 开启信任）
  e. 描述文件 （设置 => 通用 => 描述文件与设备管理，可查看证书信息）
  f. charles => proxy => ssl proxy settting => host: \*, port: 443
- 开启 ssl proxy
  ![](ssl-proxy.jpg)
  ![](ssl-proxy-setting.jpg)
  ![](ssl-proxy-setting-add.jpg)

### 常见用法 Q&A

1. 我想让我访问的 xxx 网站的 html、js 等资源，访问到我本地（如测试环境地址为 xxx.test.xxx, 我希望手机访问这个网站的时候，访问我本地的资源文件）

- charles => tools => rewrite
- ![](rewrite-assets.jpg)

2. 我想给目标的请求加 header 怎么办呢？

- charles => tools => rewrite
- 添加 header
  ![](header.jpg)
  ![](add-header.jpg)
- 当然也可以添加其他的，如 queryParam、body 等
  ![](other.jpg)

2. 设置 filter
   ![](filter.jpg)

3. focus
   选中链接 => 右键 => focus

### 相关链接

[download](https://www.charlesproxy.com/latest-release/download.do)
[charles-register](https://www.zzzmode.com/mytools/charles/)
