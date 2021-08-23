---
title: cookies、sessionStorage和localStorage的区别
date: 2018-04-15 14:27:43
tags: [前端]
categories: [前端]
---

## cookies

大小最多只有4k,每次访问接口http都会自动带上cookies

主要作用是用来保存登陆信息

## localStorage

存储大小有5M，把出局存储在本地，如果浏览器被关闭了，下次打开他还在

应用场景：购物车，表单暂存

## sessionStorage

存储大小也是5M，存储在session对象中，如果浏览器被关闭了sessionStorage会被清空