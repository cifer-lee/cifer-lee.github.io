---
layout: post
title: "SSL/TLS Strong Encryption: An Introduction"
date: 2013-07-08
category: cryptographic
---

# 加密技术

## 加密算法

假设Alice想给银行发条信息，让银行为她

### 对称型加密算法

### 公钥加密算法

## 信息摘要

## 数字签名

# 证书

## 证书内容
```
 Subject               Distinguished Name, Public Key  
 Issuer                Distinguished Name, Signature  
 Period of Validity    Not Before Date, Not After Date  
 Administrative Information       Version, Serial Number
```

Distinguished Name Information

Common Name                 CN      Name being certified
Organization or Company     O       Name is associated with this organization
Organizational Unit         OU      Name is associated with this organization unit, such as a department    OU=Research Institute
City/Locality               L       Name is located in this City                                            L=Snake City
State/Province              ST      Name is located in this State/Province                                  ST=Desert
Country                     C       Name is located in this Country (ISO code)                              C=XZ

## 证书颁发者


# 安全套接层

有很多版本的SSL协议。SSL 3.0加入了certificate chain loading机制。这个机制使得服务器可以将自己的证书以及颁发者的证书一起发送给浏览器。这个机制也使得浏览器能够在还没有支持这个颁发者的情况下验证服务器的证书，因为颁发者会包含在certificate chain上。

## 建立会话

第一步是加密套件协商（Cipher Suite Negotiation），这一步使得客户端和服务器协商选定一个双方都支持的Cifer Suite。SSL 3.0 协议规范包含31种Cipher Suites。每种Cipher Suite包含下面三个东西：
* Key Exchange Method
* Cipher for Data Transfer
* Message Digest for creating the Message Authentication Code (MAC)
