---
title: "go内存泄漏"
tags: ["中文","cgo","内存泄漏"]
categories: ["golang","进阶"]
cover: https://random.imagecdn.app/376/252
top_img: https://random.imagecdn.app/1920/400
date: 2022-10-27T15:47:38+08:00
---

# 内存泄漏  
go自带gc但不意味着不会发生内存泄漏问题, go的坑很多, 内存泄漏问题很容易碰到  

# cgo
在使用cgo时发生的内存泄漏, pprof无法排查  

# References
[go程序内存泄露问题快速定位](https://www.hitzhangjie.pro/blog/2021-04-14-go%E7%A8%8B%E5%BA%8F%E5%86%85%E5%AD%98%E6%B3%84%E9%9C%B2%E9%97%AE%E9%A2%98%E5%BF%AB%E9%80%9F%E5%AE%9A%E4%BD%8D/)