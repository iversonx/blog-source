---
title: HTTP请求处理时的11个阶段
date: 2018-12-14
categories: nginx
tags: 学习笔记
description: 
---

POST_READ 读取到完整的请求头部之后处理阶段

SERVER_REWRITE  URI与location匹配前，修改URI的阶段。用于重定向

FIND_CONFIG 根据URI寻找匹配的location块

REWRITE 找到location块后再修改URI

POST_REWRITE 在重写URL之后的处理阶段

PREACCESS

ACCESS

POST_ACCESS

PRECONTENT

CONTENT

LOG
