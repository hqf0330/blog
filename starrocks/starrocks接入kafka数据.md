---
title: starrocks接入kafka数据
slug: StarRocks02
categories: []
tags: []
halo:
  site: http://156.224.24.61:8090/
  name: e50fbea1-96bc-447f-b5a0-8e2c85206a5a
  publish: true
---
## 故事背景

现在需要用sr来进行全链路替换，直接用**runtime load**将数据从kafka接入到sr中，其中会有一些注意事项和问题细节进行探讨

