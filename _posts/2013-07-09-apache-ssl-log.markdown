---
layout: post
title: "apache ssl 配置日志"
date: 2013-07-09
tags: 笔记 Apache ssl
---

1. httpd.conf 中要加上 Listen 443，或者在全局范围内加，或者在&lt;IfModule ssl\_module&gt;&lt;/IfModule&gt; 中加。当然，在后者中加应该会更好一些。
