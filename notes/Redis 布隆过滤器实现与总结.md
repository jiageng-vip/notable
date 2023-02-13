---
attachments: [Clipboard_2022-05-24-21-19-53.png, Clipboard_2022-05-24-21-20-35.png, Clipboard_2022-05-24-21-20-50.png, Clipboard_2022-05-24-21-21-50.png, Clipboard_2022-05-24-21-22-02.png, Clipboard_2022-05-24-21-23-33.png, Clipboard_2022-05-24-21-26-31.png]
tags: [Redis]
title: Redis 布隆过滤器实现与总结
created: '2022-05-03T11:55:10.019Z'
modified: '2022-05-24T13:30:23.241Z'
---

# Redis 布隆过滤器实现与总结
### 问题引入
1. 原本有10亿个号码，现在又来了10万个号码，要快速准确判断这10万个号码是否在10亿个号码库中？
2. 接触过爬虫的，应该有这么一个需求，需要爬虫的网站千千万万，对于一个新的网站url，我们如何判断这个url我们是否已经爬过了？
3. 一个邮件系统，有上亿的邮件数量，我们要检测某一个邮箱是否正确发送了邮件信息？
4. 提到Redis做缓存查询，我们需要考虑几个问题，缓存穿透、缓存击穿和缓存雪崩。我们该如何解决缓存这种缓存问题呢？
### 布隆过滤
布隆过滤器其实就是，一种数据结构，是由一串很长的二进制向量组成，可以将其看成一个二进制数组。既然是二进制，那么里面存放的不是0，就是1，但是初始默认值都是0。
大致的数据结构如下图:
![](@attachment/Clipboard_2022-05-24-21-19-53.png)
### 添加数据
向布隆过滤器中添加 key 时，会使用多个 hash 函数对 key 进行 hash 算得一个整数索引值然后对位数组长度进行取模运算得到一个位置，每个 hash 函数都会算得一个不同的位置。再把位数组的这几个位置都置为 1 就完成了 add 操作。
![](@attachment/Clipboard_2022-05-24-21-20-35.png)
### 获取数据时
只需要将这个新的数据通过上面自定义的几个哈希函数，分别算出各个值，然后看其对应的地方是否都是1，如果存在一个不是1的情况，那么我们可以说，该新数据一定不存在于这个布隆过滤器中
![](@attachment/Clipboard_2022-05-24-21-21-50.png)
### Redis配置
在Redis中要使用布隆过滤器，可以直接参照该文档,<a herf="https://oss.redislabs.com/redisbloom/">文档地址</a>
> 推荐使用docker使用方式，如果要编译成so动态库，则需要运行在Linux环境中。// 安装docker run -p 6377:6379 --name redis-redisbloom redislabs/rebloom:latest安装完之后，查看docker容器。
![](@attachment/Clipboard_2022-05-24-21-23-33.png)
进入Redis容器，并查看容器模块状态。
```
# 进入容器
docker exec -it 4a695ead6577 /bin/sh
# 登录到Redis
redis-cli
# 查看Redis模块
127.0.0.1:6379> info Modules
# Modules
module:name=bf,ver=20205,api=1,filters=0,usedby=[],using=[],options=[]
```
### 操作演示
操作演示
```// 单个添加
127.0.0.1:6379> bf.add blkey 1
(integer) 1
127.0.0.1:6379> bf.add blkey 2
(integer) 1
127.0.0.1:6379> bf.add blkey 2
(integer) 0
127.0.0.1:6379> bf.add blkey 3
(integer) 1
127.0.0.1:6379> bf.add blkey 4
(integer) 1
```
```
// 批量添加
127.0.0.1:6379> bf.madd blkey 5 6 7 8 4
1) (integer) 1
2) (integer) 1
3) (integer) 1
4) (integer) 1
5) (integer) 0
```
> 通过添加会发现，如果元素已经存在，则返回的是0值。
检测数据
```
// 检测单个值
127.0.0.1:6379> bf.exists blkey 1
(integer) 1
127.0.0.1:6379> bf.exists blkey 2
(integer) 1
127.0.0.1:6379> bf.exists blkey 3
(integer) 1
```
```
// 批量检测
127.0.0.1:6379> bf.mexists blkey 1 2 3 4 5 10
1) (integer) 1
2) (integer) 1
3) (integer) 1
4) (integer) 1
5) (integer) 1
6) (integer) 0
```
> 通过演示会发现，如果元素不存在，则返回的是0值。
### 代码演示
这里用composer来对Redis布隆过滤器进行操作。官方也罗列了几种编程语言的客户端。
![](@attachment/Clipboard_2022-05-24-21-26-31.png)
```
composer require palicao/php-rebloom
```
```
<?php
declare(strict_types=1);

namespace App\Http\Controllers\Redis;

use Illuminate\Http\Request;
use Palicao\PhpRebloom\BloomFilter;
use Palicao\PhpRebloom\RedisClient;
use Palicao\PhpRebloom\RedisConnectionParams;
use Redis;

/**
 * Redis布隆过滤器
 * Class BloomFilterController
 * @package App\Http\Controllers\Redis
 */
class BloomFilterController
{
    private $request;

    private $host = '192.168.0.112';

    private $port = 6377;

    private $bloomFilter;

    public function __construct(Request $request)
    {
        $this->request     = $request->all();
        $this->bloomFilter = new BloomFilter(
            new RedisClient(
                new Redis(),
                new RedisConnectionParams($this->host, $this->port)
            )
        );
    }

    /**
     * 添加删除数据
     * @throws \RedisException
     * @author kert
     */
    public function index()
    {
        // 文章:https://www.cnblogs.com/ysocean/p/12594982.html

        /** @var string $cacheKey 缓存key */
        $cacheKey = 'bloom';
        /** @var int $cacheValue 缓存value */
        $cacheValue = mt_rand(0, 100);
        // 单个添加缓存
        var_dump('插入缓存', $this->bloomFilter->insert((string)$cacheKey, (string)$cacheValue));
        // 单个查询缓存
        var_dump('验证缓存', $this->bloomFilter->exists((string)$cacheKey, (string)$cacheValue));

        /** @var array $batchCacheValue 批量缓存value */
        $batchCacheValue = [mt_rand(0, 100), mt_rand(0, 100), mt_rand(0, 100), mt_rand(0, 100), mt_rand(0, 100)];
        // 批量添加缓存
        var_dump('批量插入缓存', $this->bloomFilter->insertMany((string)$cacheKey, $batchCacheValue));
        // 批量获取缓存
        var_dump('批量验证缓存', $this->bloomFilter->manyExist((string)$cacheKey, $batchCacheValue));
    }
}
```
### 内存对比
这里我们通过模拟邮件发送来对比布隆过滤器和集合各自占用的内存对比。
布隆过滤器
```
public function email()
{
    /** @var string $cacheKey 缓存key */
    $cacheKey = 'bloom:email';
    /** @var array $email 缓存邮箱数据 */
    $emailArray = [];
    for ($i = 0; $i < 1000; $i++) {
        array_push($emailArray, $i . 'wangyi@163.com');
    }
    /** @var array $insertResult 插入结果 */
    $insertResult = $this->bloomFilter->insertMany((string)$cacheKey, $emailArray);
    foreach ($insertResult as $value) {
        if ($value === false) {
            echo '插入失败' . PHP_EOL;
        }
    }
    /** @var array $queryResult 查询结果 */
    $queryResult = $this->bloomFilter->manyExist((string)$cacheKey, $emailArray);
    foreach ($queryResult as $value) {
        if ($value === false) {
            echo '查询失败' . PHP_EOL;
        }
    }
}
```
集合
```
public function emailSet()
{
    /** @var string $cacheKey 缓存key */
    $cacheKey = 'set:email';
    /** @var array $email 缓存邮箱数据 */
    $emailArray = [];
    for ($i = 0; $i < 1000; $i++) {
        array_push($emailArray, $i . 'wangyi@163.com');
    }
    $redis = new Redis();
    $redis->connect($this->host, $this->port);
    var_dump($redis->sAddArray($cacheKey, $emailArray));
}
```
内存对比
```
/**
 *  初始内存:854.40K
 *  布隆过滤器:857.50K ~3k
 *  集合:912.52K ~55k
 */
 ```
 > 通过对比发现，同样的邮箱数量，使用set的方式比使用过滤器的方式，内存至少多使用18倍多。
### 案例解决
在文章开头，我们引入了几个问题？首先我们想到的第一个技术方案就是通过数据库查询。这样数据更加准确。但是我们需要考虑一个问题，如果数据量很大，没查询一次都走数据库，无疑是给数据库增加了负担。
如果我们通过布隆过滤器来实现，既能解决我们实际的需求，也能解决数据库压力过重的情况。
下面演示代码实现逻辑。
```
/**
* 检测某一个手机号是否已经发送短信内容
* @author kert
*/
public function filterMobile()
{
    /** @var string $cacheKey 缓存key */
    $cacheKey = 'bloom:mobile';
    /** @var array $email 缓存手机号数据(模拟发送过的手机号) */
    $mobileArray = [];
    for ($i = 0; $i < 1000; $i++) {
        array_push($mobileArray, substr(md5((string)$i), 0, 11));
    }
    // 插入布隆过滤器
    $this->bloomFilter->insertMany((string)$cacheKey, $mobileArray);
    // 检测某一个值是否存在
    var_dump($this->bloomFilter->exists((string)$cacheKey, (string)substr(md5((string)100), 0, 11)));
    // output bool(true)
}
```
> 通过上面的演示，我们不难看出，布隆过滤在对数据检测是否存在的情况，要比走数据库好很多。
### 优缺点分析
1. 通过上面内存对比的内容，以及对布隆过滤器实现原理、存储数据格式的了解，我们可以得出布隆过滤器可以节省内存，尤其是数据大的情况下。
2. 布隆过滤器是不支持删除数据的，如果需要删除数据则需要重建缓存信息。
3. 布隆过滤器使用多次hash计算，也会存在hash冲突情况。这几会导致一个问题，当检测过滤器是否存在数据时，检测到存在，实际不一定存在。同时检测到不存在，则缓存中一定不存在。
### 总结
布隆过滤器节省内存，但是也存在一种误差。对于开篇提到的几个案例场景是一种非常不错的选择。
