# WebPOS

## 实验报告

### 压力测试
使用docker运行WebPOS服务，分配了1核CPU。  
模拟了500个用户每隔1s添加一件商品到购物车，一共添加五件的情况。  
Mac测试结果
```shell
================================================================================
---- Global Information --------------------------------------------------------
> request count                                       2500 (OK=2500   KO=0     )
> min response time                                   4484 (OK=4484   KO=-     )
> max response time                                  29897 (OK=29897  KO=-     )
> mean response time                                 16929 (OK=16929  KO=-     )
> std deviation                                       3893 (OK=3893   KO=-     )
> response time 50th percentile                      17190 (OK=17190  KO=-     )
> response time 75th percentile                      19452 (OK=19452  KO=-     )
> response time 95th percentile                      23308 (OK=23308  KO=-     )
> response time 99th percentile                      24627 (OK=24627  KO=-     )
> mean requests/sec                                 26.042 (OK=26.042 KO=-     )
---- Response Time Distribution ------------------------------------------------
> t < 800 ms                                             0 (  0%)
> 800 ms < t < 1200 ms                                   0 (  0%)
> t > 1200 ms                                         2500 (100%)
> failed                                                 0 (  0%)
================================================================================
```
Debian 11测试结果
```shell
================================================================================
---- Global Information --------------------------------------------------------
> request count                                       2500 (OK=2490   KO=10    )
> min response time                                     12 (OK=12     KO=60000 )
> max response time                                  60001 (OK=59964  KO=60001 )
> mean response time                                 20150 (OK=19990  KO=60000 )
> std deviation                                      16059 (OK=15891  KO=0     )
> response time 50th percentile                      13722 (OK=13697  KO=60000 )
> response time 75th percentile                      21052 (OK=20849  KO=60001 )
> response time 95th percentile                      53501 (OK=53429  KO=60001 )
> response time 99th percentile                      57829 (OK=57535  KO=60001 )
> mean requests/sec                                 22.523 (OK=22.432 KO=0.09  )
---- Response Time Distribution ------------------------------------------------
> t < 800 ms                                            24 (  1%)
> 800 ms < t < 1200 ms                                   5 (  0%)
> t > 1200 ms                                         2461 ( 98%)
> failed                                                10 (  0%)
---- Errors --------------------------------------------------------------------
> i.g.h.c.i.RequestTimeoutException: Request timeout to localhos     10 (100.0%)
t/127.0.0.1:18080 after 60000 ms
================================================================================
```

第一个结果是Mac上跑出来的，但是后面测试的时候一直返回500，只能换到了工位上的一台性能更弱的机器上做实验。
变相测试了一下垂直扩展的作用:(  
可以看出500个用户并发访问系统压力较大，请求返回时间较长。在性能更弱的机器上甚至直接有几个请求超时了。

### 使用haproxy做水平扩展
同样是在工位上的机器上做的实验，开了四个Docker容器作为服务端，使用haproxy做负载均衡，得到的结果
```shell
================================================================================                                                                              
---- Global Information --------------------------------------------------------                                                                              
> request count                                       2500 (OK=2500   KO=0     )                                                                              
> min response time                                      5 (OK=5      KO=-     )                                                                              
> max response time                                  28593 (OK=28593  KO=-     )                                                                              
> mean response time                                  5606 (OK=5606   KO=-     )
> std deviation                                       9034 (OK=9034   KO=-     )
> response time 50th percentile                       1304 (OK=1304   KO=-     )
> response time 75th percentile                       2994 (OK=2994   KO=-     )
> response time 95th percentile                      24312 (OK=24312  KO=-     )
> response time 99th percentile                      28343 (OK=28343  KO=-     )
> mean requests/sec                                 60.976 (OK=60.976 KO=-     )
---- Response Time Distribution ------------------------------------------------
> t < 800 ms                                           975 ( 39%)
> 800 ms < t < 1200 ms                                 209 (  8%)
> t > 1200 ms                                         1316 ( 53%)
> failed                                                 0 (  0%)
================================================================================
```
平均响应时间大幅度减小，整个系统的性能得到了提高。

### 使用redis cluster做缓存



The demo shows a web POS system , which replaces the in-memory product db in aw03 with a one backed by 京东.


![](jdpos.png)

To run

```shell
mvn clean spring-boot:run
```

Currently, it creates a new session for each user and the session data is stored in an in-memory h2 db. 
And it also fetches a product list from jd.com every time a session begins.

1. Build a docker image for this application and performance a load testing against it.
2. Make this system horizontally scalable by using haproxy and performance a load testing against it.
3. Take care of the **cache missing** problem (you may cache the products from jd.com) and **session sharing** problem (you may use a standalone mysql db or a redis cluster). Performance load testings.

Please **write a report** on the performance differences you notices among the above tasks.

