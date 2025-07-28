---
title: websocket
date: 2025-02-26 08:40:56
tags: websocket
categories: Web
---

[HTTP参考](https://awesome-programming-books.github.io/http/%E5%9B%BE%E8%A7%A3HTTP.pdf)


## URI和URL

+ URI: 统一资源标识符，用于表示某个资源的字符串标准，包括三个部分：协议标识符，访问的名称或路径，选项（片段标识符）。是一个抽象的概念，不一定只用于互联网

+ URL: 统一资源定位符，是一种特殊的URI，包括四个部分：协议标识符，访问资源的名称或者路径，服务器名称，端口号



## HTTP的事务处理过程

1. 客户端与服务器创建连接

2. 客户端发出请求

3. 服务端接收请求，处理请求并返回应答

4. 关闭连接


## HTTP报文头

### 请求头

+ 请求方法: 包括

|  方法   | 说明  |
|  ----  | ----  |
| GET  | 获取资源 |
| POST  | 传输实体主体 |
| PUT  | 传输文件 |
| HEAD  | 获取报文首部 |
| DELETE  | 删除文件 |
| OPTION  | 询问支持的方法 |
| TRACE | 追踪路径 |

+ Host：请求的目标服务器

+ User-Agent: 发送请求的应用程序或用户代理的信息。一般包括浏览器的名称和版本、操作系统等信息。

+ Accept: 客户端接受哪些类型的数据。

+ Content-Type: 此头部字段指定在POST和PUT请求中发送的数据类型。例如是JSON数据，此头部字段应设置为application/json

+ Content-Length: POST、PUT提交的数据长度

+ Cookie：由服务器发送的cookie信息

+ Authorization： 向服务器提供身份验证信息

+ Referer：原始URL，即从哪个URL跳转到当前页面

+ **协议版本**

### 响应头

+ Server：服务器信息（软件和版本）

+ Connection：连接类型：keep-alive、close、upgrade。keep-alive：持久连接，客户端对服务器的后序请求不必每次都申请建立连接

+ Content-Type: ...

+ Content-length: ...

+ Location: 重定向时，重定向的网址

+ Refresh：多少秒后定向到某个网站

+ Set-Cookies：用于在响应中设置Cookie

+ Date：消息发送的时间

+ Expires：响应题的过期日期和时间

+ Last-Modified：资源最后被修改的日期和时间

+ Cache-Control：控制响应的缓存行为

+ Content-Disposition： 可以让客户端下载文件并建议文件名

+ Upgrade：协议升级
