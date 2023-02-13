---
tags: [pressureTest]
title: Apache ab性能测试结果分析
created: '2022-05-02T07:52:00.495Z'
modified: '2022-09-14T02:37:31.845Z'
---

# Apache ab性能测试结果分析
### ab常用参数
- -n ：总共的请求执行数，缺省是1；
- -c： 并发数，缺省是1；
- -t：测试所进行的总时间，秒为单位，缺省50000s
- -p：POST时的数据文件
- -w: 以HTML表的格式输出结果
```
ab -c 100 -n 10000 http://106.13.49.215:6868/
This is ApacheBench, Version 2.3 <$Revision: 1430300 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking 106.13.49.215 (be patient)
Completed 1000 requests
^C

Server Software:        
Server Hostname:        106.13.49.215   #请求的URL主机名
Server Port:            6868     #请求端口

Document Path:          /  #请求路径
Document Length:        5039 bytes    #HTTP响应数据的正文长度

Concurrency Level:      100    #并发用户数，这是我们设置的参数之一
Time taken for tests:   11.317 seconds    #所有这些请求被处理完成所花费的总时间 单位秒
Complete requests:      1097     #总请求数量，这是我们设置的参数之一
Failed requests:        159     #表示失败的请求数量
   (Connect: 0, Receive: 0, Length: 159, Exceptions: 0)
Write errors:           0
Total transferred:      5674622 bytes     #所有请求的响应数据长度总和。包括每个HTTP响应数据的头信息和正文数据的长度
HTML transferred:       5527624 bytes   #所有请求的响应数据中正文数据的总和，也就是减去了Total transferred中HTTP响应数据中的头信息的长度
Requests per second:    96.94 [#/sec] (mean)   #吞吐量，计算公式：Complete requests/Time taken for tests  总请求数/处理完成这些请求数所花费的时间
Time per request:       1031.611 [ms] (mean)    #用户平均请求等待时间，计算公式：Time token for tests/（Complete requests/Concurrency Level）。处理完成所有请求数所花费的时间/（总请求数/并发用户数）     
Time per request:       10.316 [ms] (mean, across all concurrent requests)  #服务器平均请求等待时间，计算公式：Time taken for tests/Complete requests，正好是吞吐率的倒数。也可以这么统计：Time per request/Concurrency Level
Transfer rate:          489.68 [Kbytes/sec] received    表示这些请求在单位时间内从服务器获取的数据长度，计算公式：Total trnasferred/ Time taken for tests，这个统计很好的说明服务器的处理能力达到极限时，其出口宽带的需求量。

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        1  516 1148.7      3    7077
Processing:     4  212 415.3     27    2193
Waiting:        3  178 339.1     26    1833
Total:          6  728 1158.6     89    7445

Percentage of the requests served within a certain time (ms)
  50%     88    #50%的请求在88ms内返回
  66%    909
  75%   1043
  80%   1151
  90%   2294
  95%   3019
  98%   4093  #98%的请求在4093ms内返回
  99%   7024
 100%   7445 (longest request)
```
