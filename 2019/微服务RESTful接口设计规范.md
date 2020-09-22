# 微服务RESTful接口设计规范
zhoukuo@2019-11-31

网络应用程序，分为前端和后端两个部分。当前的发展趋势，就是前端设备层出不穷（手机、平板、桌面电脑、其他专用设备......）。因此，必须有一种统一的机制，方便不同的前端设备与后端进行通信。这导致API构架的流行，甚至出现 API First 的设计思想。RESTful API是目前比较成熟的一套互联网应用程序的API设计理论。

REST（Representational State Transfer）表述性状态转换，REST指的是一组架构约束条件和原则。 如果一个架构符合REST的约束条件和原则，我们就称它为RESTful架构。REST本身并没有创造新的技术、组件或服务，而隐藏在RESTful背后的理念就是使用Web的现有特征和能力， 更好地使用现有Web标准中的一些准则和约束。虽然REST本身受Web技术的影响很深， 但是理论上REST架构风格并不是绑定在HTTP上，只不过目前HTTP是唯一与REST相关的实例。

## RESTful设计风格

### 推荐格式

#### 1. url格式

```
http(s)://server.com/api-name/{version}/{domain}/{rest-convention}
```

这里，{version}代表api的版本信息。{domain}是一个你可以用来定义任何技术的区域（例如：安全-允许指定的用户可以访问这个区域）或者业务上的区域（例如：同样的功能在同一个前缀之下）。{rest-convention} 代表这个域（domain）下，约定的rest接口集合。

#### 2. 参数格式

##### GET采用两种常见格式
- URL参数（更推荐），如：
```
https://www.example.com/v1.1？name=‘lk-abc%’&age=’lt-10’
```
- 路径参数，如：
```
https://www.example.com/v1.1/employees/{id}
```

##### POST采用两种常见格式

- JSON格式包装参数提交

```
POST  https://www.example.com/v1.1
Content-Type: application/json;charset=utf-8
{"title":"test","sub":[1,2,3]}
```

- FORM表单参数提交

```
POST   https://www.example.com/v1.1
Content-Type: application/x-www-form-urlencoded;charset=utf-8
title=test&sub%5B%5D=1&sub%5B%5D=2&sub%5B%5D=3
```

- 返回体格式

```
{"status”: 200,
"message”:"用户查询返回成功”,
“document”:”https://www.example.com/doc#userinfo”,
   "data”: {
       "className”: "com.fiberhome.smartas.pricecloud.User”,
        "id”:“1b434wtert564564sdffey32”,
        "name”: "lilei",
        "age”: 18,
        "job”: {
             "className”:"com.fiberhome.smartas.pricecloud.Job”,
            "id”: “1b434wtert564564sdffeyey”,
            "name”: “微服务架构师”
        }
    }
}
```



### 协议
考虑到服务的安全性，建议使用https作为API的通信协议，当然http也是可以的。

### 域名
建议将API部署在专有域名下，以此屏蔽消费者对服务提供方的部署细节（可借助于平台的反向代理+路由网关），在服务地图丰富起来之后可以考虑多级域名。

```
https://api.example.com
```

```
https://example.org/api/
```

### 版本

考虑到微服务的平滑升级，可以将API的版本号放入URL，也可以将版本号放在HTTP头信息中，但不如放入URL方便和直观。Github采用这种做法。

```
https://api.example.com/v1/
```

### 复数名词路径

在RESTful架构中，每个网址代表一种资源（resource），所以网址中不能有动词，只能有名词，而且所用的名词往往与数据库的表格名对应。一般来说，数据库中的表都是同种记录的"集合"（collection），所以API中的名词也应该使用复数。

```
https://api.example.com/v1/employees
```

### http协议类型表达资源操作

HTTP协议里的8种方法，及其他衍生方法，常用的Get、post可以间接的实现其余所有的操作，根据框架和浏览器的兼容性选择性使用。

|type|message|
|-|-|
|GET（SELECT）|从服务器取出资源（一项或多项）|
|POST（CREATE）|在服务器新建一个资源|
|PUT（UPDATE）|在服务器更新资源（客户端提供改变后的完整资源）|
|PATCH（UPDATE）|在服务器更新资源（客户端提供改变的属性）|
|DELETE（DELETE）|从服务器删除资源|
|HEAD|获取资源的元数据|
|OPTIONS|获取信息，关于资源的哪些属性是客户端可以改变的|
|TRACE|回显服务器收到的请求，主要用于测试或诊断|
|CONNECT|HTTP/1.1协议中预留给能够将连接改为管道方式的代理服务器|
|MOVE|请求服务器将指定的页面移至另一个网络地址|
|COPY|请求服务器将指定的页面拷贝至另一个网络地址|
|LINK|请求服务器建立链接关系|
|UNLINK|断开链接关系|
|WRAPPED|允许客户端发送经过封装的请求|
|Extension-mothed|在不改动协议的前提下，可增加另外的方法|

```
GET  https://api.example.com/v1/employees/ 获取所有雇员
```

### 过滤信息
请求信息应该为集合提供过滤、排序、选择和分页等功能

#### 1.Filtering过滤

使用唯一的查询参数进行过滤：

GET /cars?color=red 返回红色的cars

#### 2.Sorting排序

允许针对多个字段排序

GET /cars?sort=-manufactorer,+model

这是返回根据生产者降序和模型升序排列的car集合

#### 3.Field Selection

移动端能够显示其中一些字段，它们其实不需要一个资源的所有字段，给API消费者一个选择字段的能力，这会降低网络流量，提高API可用性。

GET /cars?fields=manufacturer,model,id,color

#### 4.Paging分页

使用limit和offset。实现分页，缺省limit=20 和offset=0；

GET /cars?offset=10&limit=5

为了将总数发给客户端，使用订制的HTTP头：X-Total-Count

链接到下一页或上一页可以在HTTP头的link规定，遵循Link规定:

```
Link:<https://blog.mwaysolutions.com/sample/api/v1/cars?offset=15&limit=5>;rel="next",
<https://blog.mwaysolutions.com/sample/api/v1/cars?offset=50&limit=3>;rel="last",
<https://blog.mwaysolutions.com/sample/api/v1/cars?offset=0&limit=5>;rel="first",
<https://blog.mwaysolutions.com/sample/api/v1/cars?offset=5&limit=5>;rel="prev",
```

### 返回结果为统一的JSON格式
一方面，出于平台标准化的API管理，另一方面，遵循微服务的宽进严出设计理念，建议RESTful采用标准的JSON格式。

返回结构体

```
   {   "className”: "com.fiberhome.smartas.pricecloud.User”,
       "id”:“1b434wtert564564sdffey32”,
       "name”: "lilei",
       "age”: 18,
       "job”: {
             "className”:"com.fiberhome.smartas.pricecloud.Job”,
            "id”: “1b434wtert564564sdffeyey”,
            "name”: “微服务架构师”
        }
    }
```

注:

```
Object implements Serializable;
JSONObject.fromObject(json);
JSONObject.parseObject(text)
```

### 返回结果应该包含状态码

常见状态码，Http1.1协议完整状态码定义参考地址：

https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html

|code|message|type|description|
|-|-|-|-|
|200|OK|[GET]|服务器成功返回用户请求的数据，该操作是幂等的（Idempotent）|
|201|CREATED|[POST/PUT/PATCH]|用户新建或修改数据成功|
|202|Accepted|[*]|表示一个请求已经进入后台排队（异步任务）|
|204|NO CONTENT|[DELETE]|用户删除数据成功|
|400|INVALID REQUEST|[POST/PUT/PATCH]|用户发出的请求有错误，服务器没有进行新建或修改数据的操作，该操作是幂等的|
|401|Unauthorized|[*]|表示用户没有权限（令牌、用户名、密码错误）|
|403|Forbidden|[*]|表示用户得到授权（与401错误相对），但是访问是被禁止的|
|404|NOT FOUND|[*]|用户发出的请求针对的是不存在的记录，服务器没有进行操作，该操作是幂等的|
|406|Not Acceptable|[GET]|用户请求的格式不可得（比如用户请求JSON格式，但是只有XML格式）|
|410|Gone|[GET]|用户请求的资源被永久删除，且不会再得到的|
|422|Unprocesable entity|[POST/PUT/PATCH]|当创建一个对象时，发生一个验证错误|
|500|INTERNAL SERVER ERROR|[*]|服务器发生错误，用户将无法判断发出的请求是否成功|


### 返回结果中提供帮助链接
RESTful API最好做到Hypermedia，即返回结果中提供链接，连向其他API方法，使得用户不查文档，也知道下一步应该做什么。注：Github就是这么做的

返回体结构

```
{"link": 
    {
	 "document":" https://www.example.com/docs#zoos",
	 "href":"https://api.example.com/zoos",
	 "title":"List of zoos",
	 "type":"application/vnd.yourformat+json"
    }
}
```

### API扩展事项

1. RESTful API依托PaaS平台治理，需要对API进行认证、授权、参数加密等操作，可考虑在HTTP头部加认证token、调用链指令、状态信息等系列信息。
2. 如PPT所言，API分层，会有内部API和外部API之分，两种API的设计或有不同（甚至是不同协议）。
3. 对于API设计的状态选择，建议为无状态、N次幂等，但后续或存在性能优化问题，http2.0待评测。
