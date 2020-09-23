# CURL接口测试入门

zhoukuo@2020-07-15

curl 是常用的命令行工具，用来请求 Web 服务器。它的名字就是客户端（client）的 URL 工具的意思。

它的功能非常强大，命令行参数多达几十种。如果熟练的话，完全可以取代 Postman 这一类的图形界面工具。

![](https://curl.haxx.se/logo/curl-logo.svg)

---

本文介绍它的主要命令行参数，作为日常的参考，方便查阅。内容主要翻译自《curl cookbook》。

**不带有任何参数时，curl 就是发出 GET 请求**

```
$ curl https://www.example.com
```

上面命令向www.example.com发出 GET 请求，服务器返回的内容会在命令行输出。

**-X 参数指定 HTTP 请求的方法**

```
$ curl -X POST https://www.example.com
```

**-d 参数用于发送 POST 请求的数据体**

```
$ curl -d'login=emma＆password=123'-X POST https://google.com/login
# 或者
$ curl -d 'login=emma' -d 'password=123' -X POST  https://google.com/login
```

使用-d参数以后，HTTP 请求会自动加上标头Content-Type : application/x-www-form-urlencoded。并且会自动将请求转为 POST 方法，因此可以省略-X POST。

**-d 数可以读取本地文本文件的数据，向服务器发送**

```
$ curl -d '@data.txt' https://google.com/login
```

**-H 参数添加 HTTP 请求的标头**

```
$ curl -d '{"login": "emma", "pass": "123"}' -H 'Content-Type:application/json' https://google.com/login
```
上面命令添加 HTTP 请求的标头是Content-Type: application/json，然后用-d参数发送 JSON 数据。

**-k 参数指定跳过 SSL 检测**

```
$ curl -k https://www.example.com
```

**--limit-rate 用来限制 HTTP 请求和回应的带宽，模拟慢网速的环境**

```
$ curl --limit-rate 200k https://google.com
```
上面命令将带宽限制在每秒 200K 字节。

**-u 参数用来设置服务器认证的用户名和密码**

```
$ curl -u 'bob:12345' https://google.com/login
```
上面命令设置用户名为bob，密码为12345，然后将其转为 HTTP 标头Authorization: Basic Ym9iOjEyMzQ1。


**完整的post json示例**

```
curl -X POST http://domain.com -H 'Content-Type: application/json' \
     -d '{"authentication": {"dspId": "1111","token": "xxx"},"advertiserIds":["22"]}'
```

