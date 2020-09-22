# Golang面向对象思想和实现

zhoukuo@2016-08-03

golang中并没有明确的面向对象的说法，实在要扯上的话，可以将struct比作其它语言中的class。

## 类声明
```golang
type Poem struct {
    Title  string
    Author string
    intro  string
}
```

这样就声明了一个类，其中没有public、protected、private的的声明。golang用另外一种做法来实现属性的访问权限：属性的开头字母是大写的则在其它包中可以被访问，否则只能在本包中访问。类的声明和方法亦是如此。


## 类方法声明

```golang
func (poem *Poem) publish() {
    fmt.Println("poem publish")
}
```
或者
```golang
func (poem Poem) publish() {
    fmt.Println("poem publish")
}
```
和其它语言不一样，golang声明方法和普通方法一致，只是在func后增加了poem \*Poem这样的声明。加\*和没有加\*的区别在于一个是传递指针对象，一个是传递值对象。


## 实例化对象
实例化对象有好几种方式

```golang
poem := &Poem{}
    poem.Author = "Heine"
    poem2 := &Poem{Author: "Heine"}
    poem3 := new(Poem)
    poem3.Author = "Heine"
    poem4 := Poem{}
    poem4.Author = "Heine"
    poem5 := Poem{Author: "Heine"}
```
实例化的时候可以初始化属性值，如果没有指明则默认为系统默认值。加&符号和new的是指针对象，没有的则是值对象，这点和php、java不一致，在传递对象的时候要根据实际情况来决定是要传递指针还是值。

**tips**：当对象比较小的时候传递指针并不划算。


## 构造函数

查看官方文档，golang并没有构造函数一说。如果一定要在初始化对象的时候进行一些工作的话，可以自行封装产生实例的方法。

```golang
func NewPoem(param string, p ...interface{}) *Poem
```
示例：

```golang
func NewPoem(author string) (poem *Poem) {
    poem = &Poem{}
    poem.Author = author
    return
}
 
poem6 := NewPoem("Heine")
```

## 继承
确切的说golang中叫做组合（composition）

```golang
type Poem struct {
    Title  string
    Author string
    intro  string
}
 
type ProsePoem struct {
    Poem
    Author string
}
```

ProsePoem属性中声明了Poem，表示组合了Poem的属性和方法。可以像如下方式调用：

```golang
prosePoem := &ProsePoem{}
prosePoem.author = "Heine"
```
如果其中属性有冲突，则以外围的为主。

```golang
type ProsePoem struct {
    Poem
    Author string
}
```

当访问Author的时候默认为ProsePoem的Author，如果需要访问Poem的Author属性可以使用prosePoem.Poem.Author来访问。

```golang
prosePoem := &ProsePoem{}
prosePoem.Author = "Shelley"
prosePoem.Poem.Author = "Heine"
fmt.Println(prosePoem)
```

从输出中可以很直观看到这一点。

```golang
&{{ Heine } Shelley}
```

方法的继承和属性一致，这里不再罗列，通过组合的话可以很好的实现多继承。


## 方法重载
方法重载就是一个类中可以有相同的函数名称，但是它们的参数是不一致的，在java、C++中这种做法普遍存在。golang中如果尝试这么做会报重新声明（redeclared）错误，但是golang的函数可以声明不定参数，这个非常强大。

```golang
func (poem *Poem) recite(v ...interface{}) {
    fmt.Println(v)
}
```

其中v …interface{}表示参数不定的意思，其中v是slice类型，fmt.Println方法也是这样定义的。如果要根据不同的参数实现不同的功能，要在方法内检测传递的参数。


## 接口

关于面向对象中还一个重要的东西就是接口了，golang中的接口和其它语言都不太一样，是golang值得称道设计之一。详细了解接口还需要一段时间，下次再分享吧。
<完>

原文地址：http://www.01happy.com/golang-oop/