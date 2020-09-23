# 我所偏爱的 C 语言面向对象编程范式

zhoukuo@2016-07-24

面向对象编程不是银弹。大部分场合，我对面向对象的使用非常谨慎，能不用则不用。相关的讨论就不展开了。

但是，某些场合下，采用面向对象的确是比较好的方案。比如 UI 框架，又比如 3d 渲染引擎中的场景管理。C 语言对面向对象编程并没有原生支持，但没有原生支持并不等于不适合用 C 写面向对象程序。反而，我们对具体实现方式有更多的选择。

大部分用 C 写面向对象程序的程序员受 C++ 影响颇深。企图用宏模拟出一个常见 C++ 编译器已经实现的对象模型。于我愚见，这并不是一个好的方向。C++ 的对象模型，本质上是为了追求实现层的性能，并直接体现出来。就有如在 C++ 中被滥用的 inline ，的确有效，却破坏了分离原则。C++ 的继承是过紧的耦合。

我所理解的面向对象，是让不同的数据元有共同的操作方式，适合成组的处理。根据操作方式的不同，我们会对数据元做不同的分组。一个数据可能出现在这个组里，也可以出现在那个组里。这取决于你从不同的方面提取的共性。这些可供统一操作的共性称之为接口（Interface），接口在 C 语言中，表现为一组函数指针的集合。放在 C++ 中，即为虚表。

我所偏爱的面向对象实现方式（使用 C 语言）是这样的：

若有一组数据，我们需要让他们看起来都有一种叫作 foo 的共性。把符合这样的数据都称为 foo_object 。通常，我们会有如下 api 去操控 foo_object 。

```c
struct foo_object;

struct foo_object * foo_create();
void foo_release(struct foo_object *);
void foo_dosomething(struct foo_object *);
```

在具体实现时，会在一个叫 foo.c 的实现文件中，定义出 foo_object 结构，里面有一些 foo_dosomething 所需的数据成员。

但是，以上还不能满足要求。因为，我们会有不同的数据，他们只是表现出 foo_object 某些方面的特性。对于不同的数据，它们在 dosomething 时，实际所做的操作也有所区别。这时，我们需要定义出一个接口，供 foo.c 内部使用。那么，以上的头文件就需要做一些修改，把接口 i_foo 的定义加进去，并修改 create 函数。

```c
struct i_foo {
    void (*foobar)(void *);
};

struct foo_object * foo_create(struct i_foo *iface, void *data);
```

这里稍做解释。i_foo 是供 foo_dosomething 内部使用的一组接口。构造 foo_object 时，我们把一个外部数据 data 和为 foo_object 相关特性定义出的 i_foo 接口捆绑在一起，传入构造函数 foo_create 。一般，我还会会每个符合 foo_object 特性的对象实现一个方法来得到对应的 i_foo ，如：

```c
struct foobar;

struct i_foo * foobar_foo(void);
struct foobar * foobar_create(void);
void foobar_release(struct foobar *);
```

创建一个 foo_object 对象的代码看起来是这样：

```c
struct foobar *foobar = foobar_create();
struct foo_object * fobj = foo_create(foobar_foo() , foobar);
```

struct foo_object 的定义中，必然要记录 i_foo 的接口指针和 data 数据指针。从 C++ 的观点看，foo_object 是基类，它也会有一些基类成员和非虚的成员函数。具体的派生类在实现时，改写了虚表 i_foo 的内容（重载了虚函数）。data 数据是在对基类 foo_object 继承时扩展的数据成员。但，在这里，我们使用了组合的方式来扩展成员。这增加了一层间接性，但提供了更低的耦合。其中的优劣暂且不讨论了。

通常看起来会是这样：

```c
struct foo_object {
    struct i_foo * vtbl;
    void * data;
    void * others;
};

void
foo_dosomething(struct foo_object *fobj)
{
    fobj->vtbl->foobar(fobj->data);
    // do something else
}
```

此处还有另一个问题：data 的生命期该由谁来负责？

生命期管理是个很大的课题。也是大多数使用 C/C++ 开发的软件的复杂度重要来源。我个人倾向于把生命期管理独立出来解决。所以 foo_object 模块一般并不负责 data 的生命期管理。它只负责 struct foo_object 的资源释放。

自己经营自己，是我的 C 语言软件开发的观点之一。我倾向于采用混合语言编程来更好的解决这个问题。比如 C 和 Lua ，或者 C 和 C++ 。如果不采用混合语言编程，那么也可以在之后，增加一个同样用 C 语言编写的层次来管理。这个话题，留到下次来讲。

剥离出生命期管理，代码量可以减少很多，也不容易犯错误。

ps. C 语言是一个弱类型的语言。至少比 C++ 要弱一些。这表现在：

void * 在 C 语言中可以指代任意数据指针。你可以把任意数据指针赋值给一个 void * 变量，也可以把一个 void * 变量赋给特定的指针类型变量。（这在 C++ 中不推荐，并会被编译器警告）

C 语言中的函数指针也比较有趣。通常，不同类型的函数指针相互赋值是会引起编译器警告的（类型不同）。当然，我们可以用一个 void * 来解决问题。但有时候，我们期望让类型检查严格一些，至少我们不希望把一个数据指针赋值给一个函数指针。但希望编译器不要理会函数参数的差异。

在 C 语言中，void (*foo)() 可以被赋予任意返回 void 的函数指针。即，你可以把 void foobar(int) 的地址赋予前面的 foo 变量（这是由 C 标准的参数传递规则保证的）。

所以，在 C 语言编程中需要注意。如果你想定义一个不接受参数的函数，并让编译器帮你检查出那些错误的多传递了参数的语句。你必须在 .h 文件中严格定义 void foo(void) 以示 foo 函数不接受参数。

在传统的 C 语言中，对结构初始化需要非常小心。这里，我们的 i_foo 接口定义就使用了 C 里的结构。这需要非常谨慎小心。（没有 C++ 编译器帮你做这件事）

C99 新增加的语法增强了这点（在初始化结构时，可以不依赖次序，而写出成员的名字）。值得采用。
<完>

原文地址：http://blog.codingnow.com/2010/03/object_oriented_programming_in_c.html