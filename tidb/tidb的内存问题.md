---
title: tidb的内存问题
categories:
  - 数据库技术
  - 踩坑记录
tags:
  - 性能优化
  - tidb
halo:
  site: http://156.224.24.61:8090
  name: f6fc7343-94fa-4416-87cf-ef82ba9eb556
  publish: true
---

### 1. 问题描述细化

在最近处理一个 tidb数据库中表数据重复项去除的任务时，我们采用了一种通过 `row number` 开窗函数来筛选出每组唯一数据中时间戳最新的记录，并计划将其重新插入到原表中的方案。然而，在执行实际插入操作的过程中，遇到了 tidb报出内存限制错误的问题，这使得我们的任务执行受阻。将tidb中的某张表数据去重后重新插回，目前的思路是使用`row number`开窗以后选取时间最新的一条，然后插回原表，但是在插入的过程中会报出现内存限制错误

### 2. 错误详细说明

在执行`insert into select... from ...`这样的语法结构后，会抛出以下的错误:  
> Your query has been cancelled due to exceeding the allowed memory limit for a single SQL query. Please try narrowing your query scope or increase the tidb_mem_quota_query limit

也就是说，在执行大量数据插入的操作的时候，tidb返回了内存溢出的相关错误，表明当前SQL操作所需要的内存超过了tidb配置的内存阈值

### 3. 分析原因

**数据量：** 在去重之前，数据量有**70w**条左右，对于分布式数据库来说属于小数据，就算开窗也不至于比mysql的性能还差
**SQL代码：** 为省事方便，直接使用的是`row_number`进行开窗

### 4. 解决方案尝试

### 5. 验证

### 6. 后续改进和总结
