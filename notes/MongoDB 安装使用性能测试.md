---
tags: [Linux]
title: MongoDB 安装使用性能测试
created: '2022-08-23T10:29:54.989Z'
modified: '2022-08-23T10:30:09.195Z'
---

# MongoDB 安装使用性能测试

## mongo 命令

## 性能测试

[ycsb](https://github.com/pingcap/go-ycsb) go 语音版本

```
# 编译命令
make

# load 数据
./bin/go-ycsb load mongodb -p "mongodb.url=mongodb://dds-2ze060b34974a9641.mongodb.rds.aliyuncs.com:3717/ycsb?replicaSet=mgset-62137253" -p "mongodb.username=root" -p 'mongodb.password=!@#god123' --threads 100 -P workloads/workloadb

# run 测试
./bin/go-ycsb run mongodb -p 'mongodb.url=mongodb://dds-2ze060b34974a9641.mongodb.rds.aliyuncs.com:3717/admin?replicaSet=mgset-62137253' -p 'mongodb.username=root' -p 'mongodb.password=!@#god123' --threads 100 -P workloads/workloadb
```

## macOS 安装

```
brew tap mongodb/brew
brew install mongodb-community@4.4
```

[Install](https://www.mongodb.com/docs/manual/tutorial/install-mongodb-on-os-x/)
