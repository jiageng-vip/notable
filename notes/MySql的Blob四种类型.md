---
tags: [Mysql]
title: MySql的Blob四种类型
created: '2022-05-23T03:08:47.756Z'
modified: '2022-05-28T09:00:15.214Z'
---

# MySql的Blob四种类型
MySQL中，BLOB是一个二进制大型对象，是一个可以存储大量数据的容器，它能容纳不同大小的数据。BLOB类型实际是个类型系列（TinyBlob、Blob、MediumBlob、LongBlob），除了在存储的最大信息量上不同外，他们是等同的。
### MySQL的四种BLOB类型
```
类型                  大小(单位：字节) 
TinyBlob              最大 255 
Blob                  最大 65K 
MediumBlob            最大 16M 
LongBlob              最大 4G 
```
实际使用中根据需要存入的数据大小定义不同的BLOB类型。
需要注意的是：如果你存储的文件过大，数据库的性能会下降很多。
