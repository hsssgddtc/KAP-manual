---
layout: docs15-cn
title:  基本运维
categories: 教程
permalink: /cn/docs15/tutorial/odbc.html
version: v1.2
since: v0.7.1
---

## 基本运维
KAP服务器每天会接受不同用户提交的多个Cube构建任务，且往往因为Cube设计不当或集群环境异常等原因，导致Cube构建失败或时间过长，需要运维人员提高警惕；此外，KAP服务运行一段时间之后，一些存在于HBase或HDFS上的数据会成为无用数据，需要定时对存储器进行垃圾清理。


