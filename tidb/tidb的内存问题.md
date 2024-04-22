---
title: tidb的内存问题
categories:
  - tidb
tags:
  - 内存调优
halo:
  site: http://156.224.24.61:8090
  name: f6fc7343-94fa-4416-87cf-ef82ba9eb556
  publish: false
---

## 1. 问题背景

将tidb中的某张表数据去重后重新插回，目前的思路是使用row number开窗以后选取时间最新的一条，然后插回原表，但是在插入的过程中会报出现内存限制错误
