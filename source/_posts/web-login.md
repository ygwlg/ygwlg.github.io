---
title: web-login
date: 2025-04-13 15:59:13
tags:
---

保存用户的状态

1. Cookie-session机制

流程：服务器生成session：用户登录后，服务端创建唯一sessionid并将数据存储在内存、数据库或者缓存中；将session id通过set-cookie返回给浏览器；后序请求浏览器自动附加Cookie，服务器端通过Cookie中的session id查找用户数据

2. Token机制

流程：用户登录成功后，服务器端生成签名的JSON WEB TOKEN（JWT），包括（userid，role，有效期）；返回Token给客户端；前端将Token存储到localStorage，sessionStorage或者cookie；后序请求携带Token


Q1：比对密码的哈希值时是否会有碰撞的问题？

A：根据鸽巢原理，足够多的值放到有限的hash空间里就一定会发生碰撞，但是成熟的哈希算法（例如SHA-256）的碰撞概率极小。

