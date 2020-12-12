---
layout: wiki
title: 本地启动 Node.js 服务
categories: Node.js, server
description: 本地启动 Node.js 服务
keywords: Node.js, server 
---

在前端开发的过程中，我们经常需要去查看 Vue、React 等前端框架打包后的效果，而当你没有服务器来部署打包后的 dist 文件时，如何能在本地快速浏览以及调试问题呢？

## http-server

这里我们介绍一个 Node.js 的服务工具 —— `http-server`。

`http-server` 是一个简单的、零配置的命令行工具。它的功能足够强大，可以用于生产使用，但是它的简单性和可操作性足以用于测试、本地开发和学习。

具体的使用方式，就以官方仓库文档为主：[http-serve](https://github.com/http-party/http-server#readme)

## serve

[serve](https://github.com/vercel/serve) 也是一个静态站点、单页应用程序的搭建服务，同样推荐。
