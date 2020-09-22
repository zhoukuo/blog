# 用wrk进行性能测试

zhoukuo@2020-07-03

## 一、WRK简介

wrk 是一款针对 Http 协议的基准测试工具，它能够在单机多核 CPU 的条件下，使用系统自带的高性能 I/O 机制，如 epoll，kqueue 等，通过多线程和事件模式，对目标机器产生大量的负载。

wrk是开源的, 代码在 github 上：https://github.com/wg/wrk

安装：https://www.cnblogs.com/savorboard/p/wrk.html

**优势**：

```
轻量级性能测试工具
```
```
安装简单
```
```
学习曲线基本为0，几分钟就学会使用了
```
```
基于系统自带的高性能I/O机制，如epoll，kqueue，利用异步的事件驱动框架，通过很少的线程就可以压出很大的并发量，
例如几万、几十万，这是很多性能测试工具无法做到的
```

**劣势**：
```
wrk 目前仅支持单机压测，后续也不太可能支持多机器对目标机压测，因为它本身的定位，并不是用来取代 JMeter, LoadRunner 
等专业的测试工具。
```

## 二、格式与用法

```bash
使用方法: wrk <选项> <被测HTTP服务的URL>                           

Options:                                           
  -c, --connections <N>  跟服务器建立并保持的TCP连接数量 
  -d, --duration    <T>  压测时间          
  -t, --threads     <N>  使用多少个线程进行压测，压测时，是有一个主线程来控制我们设置的n个子线程间调度  
                                                  
  -s, --script      <S>  指定Lua脚本路径      
  -H, --header      <H>  为每一个HTTP请求添加HTTP头     
      --latency          在压测结束后，打印延迟统计信息  
      --timeout     <T>  超时时间    
  -v, --version          打印正在使用的wrk的详细版本信息                                              

<N>代表数字参数，支持国际单位 (1k, 1M, 1G)
<T>代表时间参数，支持时间单位 (2s, 2m, 2h)
```

## 三、简单压测及结果分析

做一个简单的压测，分析下结果：

```bash
wrk -t8 -c200 -d30s --latency  http://www.bing.com
```

输出：

```bash
Running 30s test @ http://www.bing.com

  8 threads and 200 connections

  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    46.67ms  215.38ms   1.67s    95.59%
    Req/Sec     7.91k     1.15k   10.26k    70.77%
  Latency Distribution
     50%    2.93ms
     75%    3.78ms
     90%    4.73ms
     99%    1.35s
  1790465 requests in 30.01s, 684.08MB read
Requests/sec:  59658.29
Transfer/sec:     22.79MB
```

以上是使用8个线程200个连接，对bing首页进行了30秒的压测，并要求在压测结果中输出响应延迟信息。

以下是解释压测结果：

```bash
Running 30s test @ http://www.bing.com （压测时间30s）

8 threads and 200 connections （共8个测试线程，200个连接）

Thread Stats   Avg      Stdev     Max      +/- Stdev
             （平均值） （标准差）  （最大值） （正负一个标准差所占比例）
Latency        46.67ms  215.38ms  1.67s    95.59%
（延迟）
Req/Sec        7.91k    1.15k     10.26k   70.77%
（请求数）

Latency Distribution （延迟分布）
50%    2.93ms
75%    3.78ms
90%    4.73ms
99%    1.35s （99分位的延迟：%99的请求在1.35s以内）
1790465 requests in 30.01s, 684.08MB read （30.01秒内共处理完成了1790465个请求，读取了684.08MB数据）
Requests/sec:  59658.29 （平均每秒处理完成59658.29个请求）
Transfer/sec:   22.79MB （平均每秒读取数据22.79MB）
```

## 四、使用lua脚本进行压测

lua脚本是一种轻量小巧的脚本语言，用标准c语言编写，并以源代码形式开放，其设计目的是为了嵌入应用程序中，从而为程序提供灵活的扩展和定制功能。wrk工具嵌入了lua脚本语言，因此，在自定义压测场景时，可在wrk目录下使用lua定制压测场景。

### 1、lua声明周期

共有三个阶段，启动阶段，运行阶段，结束阶段。wrk支持在这三个阶段对压测进行个性化。

**启动阶段**

```lua
function setup(thread)
```

在脚本文件中实现setup方法，wrk就会在测试线程已经初始化但还没有启动的时候调用该方法。wrk会为每一个测试线程调用一次setup方法，并传入代表测试线程的对象thread作为参数。setup方法中可操作该thread对象，获取信息、存储信息、甚至关闭该线程。

```lua
thread.addr - get or set the thread's server address
thread:get(name) - get the value of a global in the thread's env
thread:set(name, value) - set the value of a global in the thread's env
thread:stop() - stop the thread
```

**运行阶段**

```lua
function init(args)  
--由测试线程调用，只会在进入运行阶段时，调用一次。支持从启动wrk的命令中，获取命令行参数；

function delay()  
--在每次发送request之前调用，如果需要delay，那么delay相应时间；

function request()  
--用来生成请求；每一次请求都会调用该方法，所以注意不要在该方法中做耗时的操作；

function response(status, headers, body)  
--在每次收到一个响应时调用；为提升性能，如果没有定义该方法，那么wrk不会解析headers和body；
```

**结束阶段**

```lua
function done(summary, latency, requests)  
--在整个测试过程中只会调用一次，可从参数给定的对象中，获取压测结果，生成定制化的测试报告。
```

### 2、自定义脚本中可访问的变量和方法：

**变量：wrk**

```lua
wrk = {
    scheme  = "http",
    host    = "localhost",
    port    = nil,
    method  = "GET",
    path    = "/",
    headers = {},
    body    = nil,
    thread  = <userdata>,
  }
```

**方法：wrk.fomat wrk.lookup wrk.connect**

```lua
function wrk.format(method, path, headers, body)  
--根据参数和全局变量wrk，生成一个HTTP rquest string。

function wrk.lookup(host, service)  
--给定host和service（port/well known service name），返回所有可用的服务器地址信息。

function wrk.connect(addr)  
--测试与给定的服务器地址信息是否可以成功创建连接
```

### 3、lua脚本压测实例

**压测命令：**

```bash
wrk -t8 -c200 -d30s --latency  -s test.lua http://www.bing.com
```

test.lua是用lua写的压测脚本，如下是压测脚本的实例：

**使用post方法压测**

```lua
wrk.method = "POST"
wrk.headers["S-COOKIE2"]="a=2&b=Input&c=10.0&d=20191114***"
wrk.body = "recent_seven=20191127_32;20191128_111"
wrk.headers["Host"]="api.shouji.**.com"

function response(status,headers,body)
        if status ~= 200 then --将服务器返回状态码不是200的请求结果打印出来
                print(body)
        --      wrk.thread:stop()
        end
end
```

**发送json**

```lua
request = function()
    local headers = { }
    headers['Content-Type'] = "application/json"
    body = {
        mobile={"1533899828"},
        params={code=math.random(1000,9999)}
    }
    local cjson = require("cjson")
    body_str = cjson.encode(body)
    return wrk.format('POST', nil, headers, body_str)
end
```

**wrk读取文件，实现随机header-cookie**

```lua
idArr = {}
falg = 0
wrk.method = "POST"
wrk.body = "a=1"
function init(args)
        for line in io.lines("integral/cookies.txt") do
                print(line)
                idArr[falg] = line
                falg = falg+1
        end
        falg = 0
end

--wrk.method = "POST"
--wrk.body = "a=1"
--wrk.path = "/v1/points/reading"

request = function()
        parms = idArr[math.random(0,4)] --随机传递文件中的参数
        --parms = idArr[falg%(table.getn(idArr)+1)] --循环传递文件中的参数
        wrk.headers["S-COOKIE2"] = parms
        falg = falg+1
        return wrk.format()
end
```

**wrk创建数组并初始化，拼接随机参数**

```lua
idArr = {};
function init(args)
        idArr[1] = "1";
        idArr[2] = "2";
        idArr[3] = "3";
        idArr[4] = "4";
end

request = function()
        parms = idArr[math.random(1,4)]
        path = "/v1/points/reading?id="..parms
        return wrk.format("GET",path)
end
```
