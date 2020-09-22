# Golang单元测试

zhoukuo@2016-08-04

一般为了保证整个系统的稳定性，通常都需要编写大量的单元测试，诸如像java的junit，php的phpunit等都提供了类似的功能。golang中的testing包提供了这个测试的功能，结合go test工具搞起来就很方便了。

golang中的单元测试不单有功能测试，也还提供了性能测试，非常给力。

## 功能测试
在golang的src目录下新建目录math，测试目录结构如下：

![单元测试目录](http://cdn.01happy.com/wp-content/uploads/2015/06/golang%E5%8D%95%E5%85%83%E6%B5%8B%E8%AF%95%E7%9B%AE%E5%BD%95.png)

fibonacci.go代码如下，主要有一个Fibonacci函数

```golang
package lib
 
//斐波那契数列
//求出第n个数的值
func Fibonacci(n int64) int64 {
    if n < 2 {
        return n
    }
    return Fibonacci(n-1) + Fibonacci(n-2)
```
fibonacci_test.go就是测试的文件了，golang需要测试文件一律用”_test”结尾，测试的函数都用Test开头，代码如下： 

```golang
package lib
 
import (
    "testing"
)
 
func TestFibonacci(t *testing.T) {
    r := Fibonacci(10)
    if r != 55 {
        t.Errorf("Fibonacci(10) failed. Got %d, expected 55.", r)
    }
}
```

使用go test测试这个程序

```golang
$ go test lib
 ok lib 0.008s
```

如果提示找不到包，则将该代码路径加入环境变量GOPATH就可以了。

```golang
can't load package: package lib: cannot find package "lib" in any of:
```

## 性能测试
结合上面的方法，这里测试一下函数的性能，如果需要进行性能测试，则函数开头使用Benchmark就可以了。

```golang
//性能测试
func BenchmarkFibonacci(b *testing.B) {
    for i := 0; i < b.N; i++ {
        Fibonacci(10)
    }
}
```

接下来执行这个性能测试：

```golang
$ go test -bench=. lib
 PASS
 BenchmarkFibonacci 5000000 436 ns/op
 ok lib 2.608s
```

其中第二行输出表示这个函数运行了5000000次，平均运行一次的时间是436ns。
这个性能测试只测试参数为10的情况。如果有需要可以测试多个参数：

```golang
//测试参数为5的性能
func BenchmarkFibonacci5(b *testing.B) {
    for i := 0; i < b.N; i++ {
        Fibonacci(5)
    }
}
 
//测试参数为20的性能
func BenchmarkFibonacci20(b *testing.B) {
    for i := 0; i < b.N; i++ {
        Fibonacci(20)
    }
}
```

运行一下： 

```golang
$ go test -bench=. lib
 PASS
 BenchmarkFibonacci 5000000 357 ns/op
 BenchmarkFibonacci5 100000000 29.5 ns/op
 BenchmarkFibonacci20 50000 44688 ns/op
 ok lib 7.824s
```

如果性能测试的方法非常多，那需要的时间就会比较久。可以通过-bench=参数设置需要运行的性能测试行数： 

```golang
$ go test -bench=Fibonacci20 lib
 PASS
 BenchmarkFibonacci20 50000 44367 ns/op
 ok lib 2.677s
```
<完>

原文地址：http://www.01happy.com/golang-unit-testing/
