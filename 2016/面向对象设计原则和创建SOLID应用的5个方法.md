# 面向对象设计原则和创建SOLID应用的5个方法

zhoukuo@2016-08-09

最近我听到了很多关于函数式编程(FP)，受之启发我觉得也应该关注面向对象编程(OOP)和面向对象设计(OOD)，因为在设计系统时这些仍然非常重要。

我们将以SOLID原则为起点开始我们的旅程。SOLID原则是类级别的，面向对象的设计理念，它们与测试工具一起帮你改进腐坏的代码。SOLID由程序员们最喜欢的Bob大叔[Robert C. Martin](http://butunclebob.com/ArticleS.UncleBob.PrinciplesOfOod)提出，它其实是五个其他缩略词的组合——SRP， OCP， LSP， ISP， DIP，我会在下面有更深入的介绍。最重要的是，SOLID原则使你的软件变得更有价值。

![](http://incdn1.b0.upaiyun.com/2014/04/ca4429b3b5f254aca3bdb020e5851ffc.jpg)

## 单一职责原则(SRP)

单一职责原则(Single Responsibility Principle，SRP）指出，一个类发生变化的原因不应该超过一个。这意味着代码中每个类，或者类似的结构只有一个功能。

在类中的一切都与该单一目的有关，即内聚性。这并不是说类只应该含有一个方法或属性。

类中可以包括很多成员，只要它们与单一的职责有关。当类改变的一个原因出现时，类的多个成员可能多需要修改。也可能多个类将需要更新。

下面的代码有多少职责？

```java
class Employee {
    public Pay calculatePay() {...}
    public void save() {...}
    public String describeEmployee() {...}
}
```

正确答案是3个。

在一个类中混合了1)支付的计算逻辑，2)数据库逻辑，3)描述逻辑。如果你将多个职责结合在一个类中，可能很难实现修改一部分时不会破坏其他部分。混合职责也使这个类难以理解，测试，降低了内聚性。修改它的最简单方法是将这个类分割为三个不同的相互分离的类，每个类仅仅有一个职责：数据库访问，支付计算和描述。

## 开闭原则(OCP)

![](http://incdn1.b0.upaiyun.com/2014/04/8e8d75be97ad3791d9e070d678557214.jpg)

开闭原则(Open-Closed Principle，OCP)指出：类应该对扩展开放但对修改关闭。“对扩展开放”指的是设计类时要考虑到新需求提出时类可以增加新的功能。“对修改关闭”指的是一旦一个类开发完成，除了改正bug就不再修改它。

这个原则的两个部分似乎是对立的。但是，如果正确地设计类和他们的依赖关系，就可以增加功能而不修改已有的源代码。

通常来说可以通过依赖关系的抽象实现开闭原则，比如接口或抽象类而不是具体类。通过创建新的类实现接口来增加功能。

在项目中应用OCP原则可以限制代码的更改，一旦代码完成，测试和调试之后就很少再去更改。这减少了给现有代码引入新bug的风险，增强软件的灵活性。

为依赖关系使用接口的另一个作用是减少耦合和增加灵活性。

```java
void checkOut(Receipt receipt) {
    Money total = Money.zero;
    for(item : items) {
        total += item.getPrice();
        receipt.addItem(item);
    }
    Payment p = acceptCash(total);
    receipt.addPayment(p);
}
```

那么增加信用卡支持该怎么做？你可能像下面的增加if语句，但这违反OCP原则。

```java
Payment p;
if(credit)
    p = acceptCredit(total);
else
    p = acceptCash(total);
receipt.addPayment(p);
```

更好的解决方案是：

```java
public interface PaymentMethod {
    void acceptPayment(Money total);
}

void checkOut(Receipt receipt, PaymentMethod pm) {
    Money total = Money.zero;
    for(item : items) {
        total += item.getPrice();
        receipt.addItem(item);
    }
    Payment p = pm.acceptPayment(total);
    receipt.addPayment(p);
}
```
这儿有一个小秘密：OCP仅仅用于即将到来的变化可预见的情况，那么只有类似的变化已经发生时应用它。所以，首先做最简单的事情，然后判断会有什么变化，就能更加准确地预见将来的变化。

这意味着等待用户做出改变，然后使用抽象应对将来的类似变化。

## 里氏替换原则(LSP)

![](http://incdn1.b0.upaiyun.com/2014/04/f45d77829a5c1ae1116352f0894e9c71.jpg)

里氏替换原则(Liskov Substitution Principle,LSP)适用于继承层次结构，指出设计类时客户端依赖的父类可以被子类替代，而客户端无须了解这个变化。

因此，所有的子类必须按照和他们父类相同方式操作。子类的特定功能可能不同，但是必须符合父类的预期行为。要成为真正的行为子类型，子类必须不仅要实现父类的方法和属性，也要符合其隐含行为。

一般来说，如果父类型的一个子类型做了一些父类型的客户没有预期的事情，那这就违反LSP。比如一个派生类抛出了父类没有抛出的异常，或者派生类有些不能预期的副作用。基本上派生类永远不应该比父类做更少的事情。

一个违反LSP的典型例子是Square类派生于Rectangle类。Square类总是假定宽度与高度相等。如果一个正方形对象用于期望一个长方形的上下文中，可能会出现意外行为，因为一个正方形的宽高不能(或者说不应该)被独立修改。

解决这个问题并不容易：如果修改Square类的setter方法，使它们保持正方形不变(即保持宽高相等)，那么这些方法将弱化(违反)Rectangle类setter方法，在长方形中宽高可以单独修改。

```java
public class Rectangle {
    private double height;
    private double width;
  
    public double area();
  
    public void setHeight(double height);
    public void setWidth(double width);
}
```
以上代码违反了LSP。

```java
public class Square extends Rectangle {  
    public void setHeight(double height) {
        super.setHeight(height);
        super.setWidth(height);
    }
  
    public void setWidth(double width) {
        setHeight(width);
    }
}
```

违反LSP导致不明确的行为。不明确的行为意味着它在开发过程中运行良好但在产品中出现问题，或者要花费几个星期调试每天只出现一次的bug，或者不得不查阅数百兆日志找出什么地方发生错误。

## 接口隔离原则(ISP)

![](http://incdn1.b0.upaiyun.com/2014/04/7b1eb663f85f8d03b1b6a23fefdb6988.jpeg)

接口隔离原则(Interface Segregation Principle)指出客户不应该被强迫依赖于他们不使用的接口。当我们使用非内聚的接口时，ISP指导我们创建多个较小的内聚度高的接口。

当你应用ISP时，类和他们的依赖使用紧密集中的接口通信，最大限度地减少了对未使用成员的依赖，并相应地降低耦合度。小接口更容易实现，提升了灵活性和重用的可能性。由于很少的类共享这些接口，为响应接口的变化而需要变化的类数量降低，增加了鲁棒性。

基本上，这里的教训是“不要依赖你不需要的东西”。下面是例子：

想象一个ATM取款机，通过一个屏幕显示我们想要的不同信息。你会如何解决显示不同信息的问题？我们使用SRP,OCP和LSP想出一个方案，但是这个系统仍然很难维护。这是为什么？

想象ATM的所有者想要添加仅在取款功能出现的一条信息，“ATM机将在您取款时收取一些费用，您同意吗”。你会如何解决？

可能你会给Messenger接口增加一个方法并使用这个方法完成。但是这会导致重新编译这个接口的所有使用者，几乎所有的系统需要重新部署，这直接违反了OCP。让代码腐坏开始了！

这里出现了这样的情形：对于取款功能的改变导致其他全部非相关功能也变化，我们现在知道这并不是我们想要的。这是怎么回事？

其实，这里是向后依赖在作怪，使用了该Messenger接口每个功能依赖了它不需要，但是被其他功能需要的方法，这正是我们想要避免的。

```java
public interface Messenger {
    askForCard();
    tellInvalidCard();
    askForPin();
    tellInvalidPin();
    tellCardWasSiezed();
    askForAccount();
    tellNotEnoughMoneyInAccount();
    tellAmountDeposited();
    tellBalance();
}
```

相反，将Messenger接口分割，不同的ATM功能依赖于分离的Messenger。

```java
public interface LoginMessenger {
    askForCard();
    tellInvalidCard();
    askForPin();
    tellInvalidPin();
}
  
public interface WithdrawalMessenger {
    tellNotEnoughMoneyInAccount();
    askForFeeConfirmation();
}
  
publc class EnglishMessenger implements LoginMessenger, WithdrawalMessenger {
    ...
}
```

## 依赖反转原则(DIP)

![](http://incdn1.b0.upaiyun.com/2014/04/1e21950fdb68bf1098e085fb07b44f5a.jpg)

依赖反转原则(Dependency Inversion Principle,DIP)指出高层次模块不应该依赖于低层次模块；他们应该依赖于抽象。第二，抽象不应该依赖于细节；细节依赖于抽象。方法是将类孤立在依赖于抽象形成的边界后面。如果在那些抽象后面所有的细节发生变化，那我们的类仍然安全。这有助于保持低耦合，使设计更容易改变。DIP也允许我们做单独测试，比如作为系统插件的数据库等细节。

例子：一个程序依赖于Reader和Writer接口，Keyboard和Printer作为依赖于这些抽象的细节实现了这些接口。CharCopier是依赖于Reader和Writer实现类的低层细节，可以传入任何实现了Reader和Writer接口的设备正确地工作。

```java
public interface Reader { 
    char getchar(); 
}

public interface Writer { 
    void putchar(char c);
}

class CharCopier {
    void copy(Reader reader, Writer writer) {
        int c;
        while ((c = reader.getchar()) != EOF) {
            writer.putchar();
        }
    }
}
  
public Keyboard implements Reader {...}
public Printer implements Writer {...}
```

## 总结

我想SOLID原则是你的工具箱里很有价值的工具。在设计下一个功能或者应用时他们就应该在你的脑海中。正如Bob大叔在他那不朽的帖子中总结的：

```
* 单一职责原则(SRP)　　　一个类有且只有一个更改的原因
```
```
* 开闭原则(OCP)　　　　　能够不更改类而扩展类的行为
```
```
* 里氏替换原则(LSP)　　　派生类可以替换基类被使用
```
```
* 接口隔离原则(ISP)　　　使用客户端特定的细粒度接口
```
```
* 依赖反转原则(DIP)　　　依赖抽象而不是具体实现
```

而且，将这些原则应用在项目中。

原文链接： [zeroturnaround](http://zeroturnaround.com/rebellabs/object-oriented-design-principles-and-the-5-ways-of-creating-solid-applications)
译文链接： [importnew](http://www.importnew.com/10656.html)