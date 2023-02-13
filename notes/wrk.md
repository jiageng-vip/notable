---
tags: [pressureTest]
title: wrk
created: '2022-09-05T09:07:58.343Z'
modified: '2022-09-05T09:25:49.755Z'
---

# wrk

> 简介： wrk是一个开源的、热门的、现代的单机HTTP基准测试工具，目前在github开源平台累计了26.9k的star数目，足以可见wrk在Http基准测试领域的热门程度。它结合了多线程设计和可扩展的事件通知系统，如epoll和kqueue，可以在有限的资源下并发出极致的的负载请求。并且内置了一个可选的LuaJIT脚本执行引擎，可以处理复杂的HTTP请求生成、响应处理以及自定义压测报告。

## 前言

wrk是一个开源的、热门的、现代的单机HTTP基准测试工具，目前在github开源平台累计了26.9k的star数目，足以可见wrk在Http基准测试领域的热门程度。它结合了多线程设计和可扩展的事件通知系统，如epoll和kqueue，可以在有限的资源下并发出极致的的负载请求。并且内置了一个可选的LuaJIT脚本执行引擎，可以处理复杂的HTTP请求生成、响应处理以及自定义压测报告。

wrk项目地址：https://github.com/wg/wrk

### 安装wrk

mac下安装：
```
brew install wrk
```
其他平台参考：https://github.com/wg/wrk/wiki

### 基础使用

```
wrk -t12 -c100 -d30s --latency http://localhost:8010/healthz
```

如上指令描述了采用12个线程，100个链接，针对http://localhost:8010/healthz 接口服务，持续压测30s。wrk本身不是依赖线程数来模拟并发数的所以线程数量设置在核心数左右最好，线程数多了测试系统消耗大，可能带来反效果。亲测核心数一致的线程数和两倍核心数的线程数，前者压出的QPS更高。结果如下：、、
```
Running 30s test @ http://localhost:8010/healthz （运行30s测试）
  12 threads and 100 connections（12个线程100个连接）
    Thread Stats        Avg(均值)      Stdev(标准差值)     Max(最大值)   +/- Stdev(正负标准差值)
    Latency(延迟)         1.39ms        668.10us           23.95ms       90.34%
    Req/Sec(每秒请求数)    5.44k          545.23             10.27k        76.47%
  Latency Distribution(延迟直方图)
     50%    1.32ms (50%请求延迟在1.32ms内)
     75%    1.49ms (75%请求延迟在1.49ms内)
     90%    1.72ms (90%请求延迟在1.72ms内)
     99%    4.77ms (99%请求延迟在4.77ms内)
  1952790 requests in 30.08s, 271.90MB read (共1952790次请求，用时30s，传输了271.9M数据)
Requests/sec(每秒请求数):  64930.12
Transfer/sec(每秒传输数据):      9.04MB
```

wrk的结果相比ab测试结果来说，多了一个延时直方图，有了这个直方图，我们可以更清晰的看到延迟的分布情况。这也是博主选择wrk最重要的原因

### 常用指令说明
```
    -c, --connections: 要保持打开的HTTP连接的总数，每个线程处理数N =连接/线程
    -d, --duration:    测试持续时间, 如 2s, 2m, 2h
    -t, --threads:     测试线程总数
    -s, --script:      指定加载lua测试扩展脚本
    -H, --header:      添加请求头信息, 如"User-Agent: wrk"
        --latency:     打印延迟直方图信息
        --timeout:     如果在此时间内没有收到响应，则记录超时.
```

-开头的指令为简写的，后面两个打印延迟直方图和超时设置没有简写的，只能--开头指定

### 高阶用法，lua测试脚本
wrk内置了全局变量，全局方法，以及五个测试请求发起流程的方法，还有一个模拟延迟发送的方法，wrk是内置对象，在lua测试脚本的每个方法内都可以直接使用

* 全局变量
```
-- 全局的变量
wrk = {
  scheme  = "http",
  host    = "localhost",
  port    = nil,
  method  = "GET",
  path    = "/",
  headers = {},
  body    = nil,
  thread  = userdata,
}
```

- 全局方法
```
-- 返回请求字符串值，其中包含所传递的参数和来自wrk表的值。例如：返回 http://www.kailing.pub
function wrk.format(method, path, headers, body);

-- 获取域名的IP和端口，返回table，例如：返回 `{127.0.0.1:80}`
function wrk.lookup(host, service)

-- 判断addr是否能连接，例如：`127.0.0.1:80`，返回 true 或 false
function wrk.connect(addr)
```

- 请求过程方法
```
-- 请求前，对每个线程调用一次，并接收表示该线程的userdata对象。
function setup(thread)
    thread.addr = "http://www.kailing.pub"        -- 设置请求的地址
    thread:get("name")        -- 获取全局变量的值
    thread:set("name", "kl") -- 在线程的环境中设置全局变量的值
    thread:stop()           -- 停止线程
end
--初始化，每个线程执行一次
function init(args) --args为从命令行传过来的额外参数
    print(args)
end
--发起请求，每次请求执行一次，返回包含HTTP请求的字符串。每次构建新请求的开销都很大，在测试高性能服务器时，
--一种解决方案是在init()中预先生成所有请求，并在request()中进行快速查找。
function request()
    requests = requests + 1
    return wrk.request()
end
--响应处理，每次请求执行一次
function response(status, headers, body)
    responses = responses + 1
end
--请求完成，每次测试执行一次。done()函数接收一个包含结果数据的表和两个统计数据对象，分别表示每个请求延迟和每个线程请求速率。
--持续时间和延迟是微秒值，速率是以每秒请求数来度量的。
function done(summary, latency, requests)
    for index, thread in ipairs(threads) do
        local id = thread:get("id")
        local requests = thread:get("requests")
        local responses = thread:get("responses")
        local msg = "thread %d made %d requests and got %d responses"
        print(msg:format(id, requests, responses))
    end
end
```

整个脚本处理过程被分为准备阶段、运行阶段、完成阶段。准备阶段在目标IP地址被解析并且所有线程都已经初始化但还没有启动之后开始。运行阶段从对init()的单个调用开始，然后对每个请求周期调用request()和response()。init()函数接收脚本的任何额外命令行参数，这些参数必须用“——”与wrk参数分隔开。

### lua测试脚本案例分析
案例：我们线上有一个带缓存场景的接口服务，根据appId的值的查询结果缓存，所以，如果单纯对指定的appId压测，就变成了测试缓存系统的负载了，测试不出实际的服务性能，这个场景就需要测试工具发起每次请求的测试参数都是动态的。根据这个场景我们定制了如下的lua测试脚本：
```
-- 测试指令：wrk -t16 -c100 -d5s -sreview_digress_list.lua --latency htt://127.0.0.1:8081
wrk.method ="GET"
wrk.path = "/app/{appId}/review_digress_list"

function request()
    -- 动态生成每个请求的url
    local requestPath = string.gsub(wrk.path,"{appId}",math.random(1,10))
    -- 返回请求的完整字符串：http://127.0.0.1//app/666/review_digress_list
    return wrk.format(nil, requestPath)
end
```
