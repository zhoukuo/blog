---
title: 用 TestNG 和 PowerMock 做单元测试
date: 2016-08-25 17:36:18
tags: [测试]
author: zhoukuo
---
单元测试又称为模块测试，是对软件中最小可测单元进行检查和验证。单元测试需要掌握内部设计和编码的细节知识，往往需要开发测试驱动模块和桩模块来辅助完成，一般由开发人员来执行测试。
<!--more-->
本文假设读者已经了解单元测试的基本概念，但对如何实施不了解，因此本文重点介绍了在Java项目中使用 **TestNG** 和 **PowerMock** 编写单元测试所必须掌握的知识，帮助大家快速了解如何进行单元测试的编写。

## 测试工具
单元测试离不开测试框架和Mock工具，目前Java社区主流的测试框架主要包括JUnit和TestNG，而Mock工具主要是EasyMock、Mockito和PowerMock，通过对比分析，我们最终选择 TestNG + PowerMock 作为我们的测试工具。

## 环境部署
开发环境不同，环境部署的方法也不同，本文以Eclipse为例，介绍如何部署环境。
1. 下载
   [powermock-mockito-testng-1.6.5.zip](http://dl.bintray.com/johanhaleby/generic/powermock-mockito-testng-1.6.5.zip)
   由于工具的作者打包时忘掉了下面这个包，所以这个包也需要下载。
   [powermock-api-mockito-common.jar](http://central.maven.org/maven2/org/powermock/powermock-api-mockito-common/1.6.5/powermock-api-mockito-common-1.6.5.jar)
   以上这些包已经包含mockito和testng，所以不需要单独下载了。

2. 安装
   安装过程非常简单，只要把所有jar包放到classpath路径下就可以了。

　

## TestNG篇

### TestNG 简介
TestNG是一个测试框架，其灵感来自JUnit和NUnit的，但引入了一些新的功能，使其功能更强大，使用更方便，尤其是测试分组、测试参数化等特性使测试更加灵活高效。

### 第一个单元测试
以下是一个最简单的单元测试代码：

```java
import org.testng.annotations.Test;
import static org.testng.Assert.assertEquals;

@PrepareForTest(UpdateGetNewEnvsnImpl.class)
public class TestNGSimpleTest extends PowerMockTestCase {
    @Test
    public void testAdd() {
        String str = "TestNG is working fine";
        assertEquals("TestNG is working fine", str);
    }
}
```

编写完成后，通过Eclipse中的Run As菜单中“TestNG Test”命令来执行。

接下来，我们来看一下一个单元测试代码包含哪些内容：
* 导入TestNG相关的包
  通常，我们不需要关注需要哪些包，使用对应的方法时，eclipse会提示你导入对应的包。
* 创建测试类
  一般一个测试类对应一个产品类，测试类需要添加@PrepareForTest注解，以便与被测类建立关联，而且测试类必须继承自 PowerMockTestCase 类。
* 创建测试方法
  测试方法需要添加@Test注解，并且在方法中调用验证方法(assertEquals)，不包含验证步骤不会影响执行，但这个测试就没什么意义了。

### 基本注解（Annotation）
TestNG通过注解提供各种功能支持，所以了解TestNG的基本注解非常重要，常用的注解包括以下几种：

| 注解　　　　         |       描述      |
| -------------------  | --------------- |
| @BeforeSuite　　　　 | 注解的方法将只运行一次，在所有测试套件运行之前 |
| @AfterSuite　　　　  | 注解的方法将只运行一次，在所有测试套件运行之后 |
| @BeforeClass　　　　 | 注解的方法将只运行一次，在当前类所有方法运行之前 |
| @AfterClass　　　　  | 注解的方法将只运行一次，在当前类所有方法运行之后 |
| @BeforeTest　　　　  | 注解的方法在内部类的每个<test>标签运行前运行 |
| @AfterTest　　　　   | 注解的方法在内部类的每个<test>标签运行后运行 |
| @BeforeMethod　　　　| 注解的方法在当前类的每个@Test注解的方法运行前运行 |
| @AfterMethod　　　　 | 注解的方法在当前类的每个@Test注解的方法运行后运行 |
| @Test　　　　        | 注解的方法作为测试方法执行 |
| @DataProvider　　　　| 注解的方法作为测试数据的提供者，此方法必须返回Object[][]，测试方法可以通过@Test的dataProvider属性使用此数据 |
| @Parameters　　　　  | 将参数传递给@Test方法 |

这里不会详细介绍每一种注解的用法，后面会有专门章节介绍。

### 断言
对预期结果进行验证是单元测试不可或缺的重要环节，而验证主要通过断言，TestNG提供了丰富的断言，以下是常用的几种：
* assertEquals
* assertNotEquals
* assertNull
* assertNotNull
* assertTrue
* assertFalse

从前面的例子，我们可以了解断言的用法。
需要注意的是，assertTrue、assertFalse 仅在验证方法返回Boolean值时使用，其它情况不建议使用。

### 异常测试
在单元测试时，除了正常情况需要测试意外，异常情况也需要测试，与正常情况的验证方式不同，TestNG通过注解对异常预期进行验证，示例如下：

```java
@Test(expectedExceptions = IllegalArgumentException.class,           
      expectedExceptionsMessageRegExp="NullPoint")
public void testException() {
    throw new IllegalArgumentException("NullPoint");
}
```
不仅可以对异常的类型验证，异常的消息内容也可以验证。

### 测试分组
有时候，我们只想执行某一类的测试用例，而不是全部用例，通过分组我们可以很容易实现。
```java
@Test(groups = {"systemtest"})
public void testLogin() {
    System.out.println("this is test login");
}
```
对应的配置文件：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE suite SYSTEM "http://testng.org/testng-1.0.dtd" >
<suite name="Suite1">
    <test name="test1">
        <groups>
            <run>
                <include name="functiontest" />
            </run>
        </groups>
    </test>
</suite>
```
执行分组测试时和以往不同，必须在xml文件上点击右键，选择 “TestNG Test” 来执行测试。

### 生成testng.xml文件
如果想按照分组来执行用例，那就离不开testng.xml文件，手工编写比较繁琐，还容易出错。幸好TestNG提供了自动生成的命令，在测试包上点击右键 ——> TestNG ——> Convert to TestNG。

有一点需要注意的是，你在哪里右键很重要，这决定了你配置文件中包含哪些测试类。

### 参数化测试
软件测试往往需要测试大量的数据集，这样才能保证软件的稳定性和鲁棒性。JUnit没有提供方便传递测试参数的机制，所以，针对每个测试数据集，都需要单独写代码进行测试。这样浪费很多时间和精力重复写测试代码，它们只是参数不一样，测试逻辑完全一样。同时，测试代码和测试数据没有分离，为今后的维护埋下隐患。

TestNG在参数化测试方面，比JUnit有较大的优势。@DataProvider方式使代码和测试数据分离，方便扩展和维护，并能够提供比较复杂的参数，方便产生具有一定规律的测试数据集。

```java
@DataProvider(name = "certStatus")
public Object[][] NewCertStatus() {
    return new Object[][] {
        {CertStatus.INSTALL_SUCCESS, false},
        {CertStatus.INSTALL_ERROR, false},
        {CertStatus.ISSUE_SUCCESS, true},
        {CertStatus.DOWNLOAD_SUCCESS, true},
        {CertStatus.DOWNLOAD_ERROR, true},
    };
}

@Test(dataProvider = "certStatus")
public void TestJudgeStatus(CertStatus certStatus, boolean expected) throws Exception {
    when(bu.getCertStatus()).thenReturn(certStatus);
    boolean actually = (Boolean) Whitebox.invokeMethod(updateGetCertImpl, "judgeStatus", bu);
    assertEquals(actually, expected);
}
```

### 忽略测试
某些测试方法由于某种原因，如：未编写完成，或已经无效等，暂时不希望继续执行这些测试，我们可以通过设置enabled属性来完成。
```java
@Test(enabled = false)
public void testIgnore() {
    System.out.println("This test case will ignore");
}
```

### 测试私有方法
通常在包外访问私有方法只能通过反射(reflect)技术，像下面这样：

```java
@Test
public void testValidate() {
    ......
    Class<UpdateGetNewEnvsnImpl> cls = UpdateGetNewEnvsnImpl.class;
    Method method = cls.getDeclaredMethod("validate", new Class[]{String.class});
    method.setAccessible(true);
    method.invoke(sut, oldEnvsn);
    ......
}
```

但有了PowerMockito，访问变得简单了，PowerMockito内置了WhiteBox类，包含了调用私有方法的接口，下面的例子说明了如何测试私有方法validate。

```java
import org.powermock.reflect.Whitebox;

@Test
public void testValidate() {
    ......
    Whitebox.invokeMethod(sut, “validate”, oldEnvsn);
    ......
}
```
　

## PowerMockito 篇

### PowerMockito 简介
在做单元测试的时候，我们会发现我们要测试的方法会引用很多外部依赖的对象，比如：（访问数据库，发送邮件，网络通讯，远程服务, 文件系统等等）。 而我们没法控制这些外部依赖的对象，为了解决这个问题，我们就需要用到Mock工具来模拟这些外部依赖的对象，来完成单元测试。

![](/img/powermock.png)

PowerMockito是PowerMock框架的一部分，它是在Mockito框架上的扩展，通过提供定制的类加载器以及一些字节码篡改技巧的应用，PowerMock 实现了对静态方法、构造方法、私有方法以及 Final 方法的模拟支持，对静态初始化过程的移除等强大的功能。因为 PowerMock 在扩展功能时完全采用和被扩展的框架相同的 API, 熟悉 PowerMock 所支持的模拟框架的开发者会发现 PowerMock 非常容易上手。PowerMock 的目的就是在当前已经被大家所熟悉的接口上通过添加极少的方法和注释来实现额外的功能。

### 模拟(Mock)有哪些关键点?

在谈到模拟时，你只需关心三样东西: 方法模拟，设定预期，验证结果。

**方法模拟(stub)**
方法模拟就是给特定的方法调用返回固定值，在官方说法中称为stub，当谈到方法模拟方法，通常你有一系列的选择，或许你希望返回一个指定的值，抛出一个异常或者什么都不做。

这咋一听起来工作量很大，但通常并非这样。许多mocking框架的一个重要功能就是你不需要提供 stub 的实体方法，也不用在执行测试期间stub那些未被调用的方法或者未使用的属性。

**设置预期**
模拟测试的一个关键的特性就是你能够告诉它你预期的结果。例如，你可以期望一个特定的函数被准确的调用3次，或不被调用，或调用至少两次但不超过5次，或者需要满足特定类型的参数、特定值和以上任意的组合的调用。可能性是无穷的。

通过设定预期结果，说明你期望发生的事情。因为它是一个模拟测试，所以实际上什么也没发生。但是，对于被测试的类来说，它并无法区分这种情况。所以测试方法能够调用函数并让它做它该做的。

**验证结果**
设置预期和验证预期是同时进行的。设置预期在调用测试类的函数之前完成，验证预期则在它之后。所以，首先你设定好预期结果，然后去验证你的预期结果是否正确。

在一个单元测试中，如果你设定的预期没有得到满足，那么这个单元测试就是失败了。

### 一个完整的带有Mock对象的单元测试

我们先来看一下，一个完整的单元测试用例是什么样子，后面我会详细介绍每个部分。

```java
@PrepareForTest(UpdateGetNewEnvsnImpl.class)
public class UpdateGetNewEnvsnImplTest extends PowerMockTestCase {
    @Mock
    private IEnvsnService mockedEnvsnService;
    @Mock
    private ICertUpdateService mockedCertUpdateService;
    ......
    @InjectMocks
    private UpdateGetNewEnvsnImpl sut;

    @BeforeClass
    public void beforeClass() {
        sut = new UpdateGetNewEnvsnImpl();
        MockitoAnnotations.initMocks(this);
    }

    @Test
    public void TestExcuteWithBUIsNullAndIsOnlineFalse() throws ServiceException {
        // 数据准备
        String newEnvsn = "807000100026080";
        HttpParam expected = new HttpParam();
        expected.set(HttpParam.EXCUTE_RESULT_KEY, HttpParam.EXCUTE_SUCCESS_VALUE);
        expected.set(HttpParam.ENVSN, newEnvsn);
        // Mock对象
        when(mockedCertUpdateService.findByOldId(oldEnvsn)).thenReturn(null);
        when(mockedEnvsnService.getNewEnvsn(oldEnvsn)).thenReturn(newEnvsn);
        // 方法调用
        HttpParam actually = sut.excute(mockedReq);
        // 结果验证
        verify(mockedCertUpdateService).add();
        assertEquals(actually, expected);
    }
}
```

### 对象模拟

* 使用mock函数模拟
```java
public class MockitoExample1 {
    @Test
    public void shorthand(){
        // 模拟LinkedList 的一个对象
        LinkedList mockedList = mock(LinkedList.class);
        mockedList.add(1);
        verify(mockedList).add(1);
    }
```

* 使用注解模拟
```java
public class MockitoExample2 {  
    @Mock
    private List mockedList;  
    
    @BeforeClass
    public beforeClass() {  
        MockitoAnnotations.initMocks(this);  
    }
  
    @Test
    public void shorthand() {  
        mockedList.add(1);  
        verify(mockedList).add(1);  
    }
}
```

### 方法模拟(stub)


* 模拟有返回值的方法
```java
when(bu.getCertStatus()).thenReturn(CertStatus.ISSUE_ERROR);
```

* 模拟有返回值的方法抛出异常
```java
when(mockedList.get(1)).thenThrow(new RuntimeException());
```

* 模拟无返回值的方法
```java
PowerMockito.doNothing().when(cu).update(bu);
```

* 模拟无返回值的方法抛出异常
```java
PowerMockito.doThrow(new ServiceException(SYSTEM_EXCEPTION)).when(cu).add(bu);
```

* 模拟私有方法
假设validate是一个私有方法：
```java
UpdateGetNewEnvsnImpl spySut = PowerMockito.spy(sut);
// 模拟无返回值的私有方法
PowerMockito.doNothing().when(spySut, "validate", anyString());
// 模拟有返回值的私有方法
PowerMockito.doReturn(true).when(spySut, "validate", anyString());
// 模拟私有方法抛出异常
PowerMockito.doThrow(new ServiceException(SYSTEM_EXCEPTION)).when(spySut, "validate", anyString());
HttpParam actually = spySut.excute(mockedReq);
```

* 模拟构造方法
```java
PowerMockito.whenNew(RAServiceImpl.class).withNoArguments().thenReturn(mockedRAServiceImpl);
```

* 调用真实方法
```java
PowerMockito.doCallRealMethod().when(mockedSut).excute(req);
```

默认情况下，对于所有有返回值且没有stub过的方法，mockito会返回相应的默认值。对于内置类型会返回默认值，如int会返回0，布尔值返回false。对于其他type会返回null。这里一个重要概念就是： mock对象会覆盖整个被mock的对象，因此没有stub的方法只能返回默认值。重复stub两次,则以第二次为准。如下将返回"second"：
```java
    when(mockedList.get(0)).thenReturn("first");
    when(mockedList.get(0)).thenReturn("second");
```

如果是下面这种形式，则表示第一次调用时返回“first”，第二次调用时返回“second”，可以写n多个。如果实际调用的次数超过了stub过的次数，则会一直返回最后一次stub的值。
```java
    when(mockedList.get(0)).thenReturn("first").thenReturn("second");
```

### 方法验证

* 是否调用
```java
verify(mockedList).add("added");
```

* 调用次数
```java
verify(mockedList, times(1)).add("once");
verify(mockedList, atLeastOnce()).add("twice");
verify(mockedList, atLeast(1)).add("twice");
verify(mockedList, atMost(5)).add("twice");
verify(mockedList, never()).add("twice");
verify(mockedList, times(0)).add("once");
```

* 执行顺序
```java
@Test  
public void verification_in_order() {
    List list = mock(List.class);  
    List list2 = mock(List.class);  
    list.add(1);  
    list2.add("hello");
    list.add(2);  
    list2.add("world");  
    //将需要排序的mock对象放入InOrder  
    InOrder inOrder = inOrder(list,list2);  
    //下面的代码不能颠倒顺序，验证执行顺序  
    inOrder.verify(list).add(1);  
    inOrder.verify(list2).add("hello");  
    inOrder.verify(list).add(2);  
    inOrder.verify(list2).add("world");  
}
```

* 自定义类对象
  验证实际值是否符合预期值，我们通常用assertEquals(actual, expected)方法，这个方法会调用actual类型(actual和expected的类型相同)的equals()方法进行比较，因此，我们自己定义的类需要重载equals()方法后，才能通过assertEquals()进行验证。否则只能分别验证对象的每个属性。
  需要注意的是，直接使用assertEquals(actual, expected)方法永远返回false。
```java
//未重载equals()前
assertEquals(actually.EXCUTE_RESULT_KEY),expected.EXCUTE_RESULT_KEY);
assertEquals(actually.ERROR_CODE), expectedE.RROR_CODE);
assertEquals(actually.ENVSN), expected.ENVSN));
//重载equals()后
assertEquals(actually, expected);
```

* 验证私有方法
```java
PowerMockito.verifyPrivate(sut).invoke("validate");
```

### 参数匹配器(Argument Matcher)
PowerMockito在模拟方法时，如果参数的值不匹配，那么这个模拟是不会生效的，如果只是模拟值类型的参数，而且这个值是确定的，我们只需要提供对应的值就可以了，但是当我们模拟的是对象，或者是个不确定的值(如当前日期时间)，我们就只能通过匹配器来进行适配了。

PowerMockito为我们内置了一些匹配器，如 anyObject，anyString，anyBoolean，anyInt，anyFloat，anyList，anyMap 等，通常想要匹配我们自己定义的类也很简单，使用anyObject()并转换成我们自己的类，像下面这样：

```java
Mockito.doNothing().when(mockedCertUpdateService).update((CertUpdateBU)anyObject());
```

同时用户可以自定义参数匹配器，例如：
```java
package css.matcher;

import org.mockito.ArgumentMatcher;
import cn.org.bjca.css.domain.sys.cert.business.CertUpdateBU;

public class AnyCertUpdateBU extends ArgumentMatcher<CertUpdateBU> {
    public boolean matches(Object list) {
        return true;
    }
}
```

自定义匹配器其实就是一个包含一个返回boolean值的方法的类，它从ArgumentMatcher<>模板类继承。在使用时我们只需要**修改类名**和**实例化ArgumentMatcher<>**就可以了。

需要注意的是：如果使用参数匹配器，那么**所有的参数都要使用参数匹配器**，不管是stub还是verify的时候都一样。

### 重置Mock
有时，我们想要清除所有的互动和预设，可以通过重置Mock对象来实现。
```java
@Test  
public void reset_mock() {  
    List list = mock(List.class);  
    when(list.size()).thenReturn(10);  
    list.add(1);
    assertEquals(10,list.size());  
    //重置mock，清除所有的互动和预设  
    reset(list);
    Assert.assertEquals(0,list.size());
}
```
　

## 测试用例设计篇
单元测试属于白盒测试，因此用例设计也要遵守白盒测试设计的原则和方法。

### 用例设计原则

* 保证一个模块中的所有独立路径至少被使用一次
* 对所有逻辑值均需测试 true 和 false
* 在上下边界及可操作范围内运行所有循环
* 检查内部数据结构以确保其有效性

### 用例设计方法
白盒测试的方法：总体上分为静态方法和动态方法两大类：
**静态分析**是一种不通过执行程序而进行测试的技术。静态分析的关键功能是检查软件的表示和描述是否一致,没有冲突或者没有歧义。
**动态分析**的主要特点是当软件系统在模拟的或真实的环境中执行之前、之中和之后 , 对软件系统行为的分析。动态分析包含了程序在受控的环境下使用特定的期望结果进行正式的运行。它显示了一个系统在检查状态下是正确还是不正确。在动态分析技术中，最重要的技术是路径和分支测试。下面要介绍的六种覆盖测试方法属于动态分析方法。

1. 语句覆盖
所谓语句覆盖：就是设计若干个测试用例，运行被测程序，使得每一可执行语句至少执行一次。这里的“若干个”，意味着使用测试用例越少越好。语句覆盖率的公式可以表示如下：
语句覆盖率 = 被评价到的语句数量 / 可执行的语句总数 x 100%

2. 判定覆盖
使设计的测试用例保证程序中每个判断的每个取值分支（t or f）至少经历一次。
[优点]：判定覆盖具有比语句覆盖更强的测试能力，而且具有和语句覆盖一样的简单性，无需细分每个判定就可以得到测试用例。
[缺点]：往往大部分的判定语句是由多个逻辑条件组合而成（如，判定语句中包含AND、OR、CASE），若仅仅判断其整个最终结果，而忽略每个条件的取值情况，必然会遗漏部分测试路径。

3. 条件覆盖
条件覆盖是指选择足够的测试用例，使得运行这些测试用例时，判定中每个条件的所有可能结果至少出现一次，但未必能覆盖全部分支
条件覆盖要检查每个符合谓词的子表达式值为真和假两种情况，要独立衡量每个子表达式的结果，以确保每个子表达式的值为真和假两种情况都被测试到。

4. 判定/条件覆盖
判定-条件覆盖就是设计足够的测试用例，使得判断中每个条件的所有可能取值至少执行一次，同时每个判断的所有可能判断结果至少执行，即要求各个判断的所有可能的条件取值组合至少执行一次。

5. 条件组合覆盖
在白盒测试法中，选择足够的测试用例，使所有判定中各条件判断结果的所有组合至少出现一次，满足这种覆盖标准成为条件组合覆盖。

6. 路径覆盖
每条可能执行到的路径至少执行一次。

其中语句覆盖是一种最弱的覆盖，判定覆盖和条件覆盖比语句覆盖强，满足判定/条件覆盖标准的测试用例一定也满足判定覆盖、条件覆盖和语句覆盖，条件组合覆盖是除路径覆盖外最强的，路径覆盖也是一种比较强的覆盖，但未必考虑判定条件结果的组合，并不能代替条件覆盖和条件组合覆盖。

### 举例
```java
if (A && B)
    action1();
if (C || D)
    action2();
```

1. 语句覆盖最弱，只需要让程序中的语句都执行一遍即可。上例中只需设计测试用例使得A=true，B=true，C=true即可。
2. 分支覆盖又称判定覆盖：使得程序中每个判断的true分支和false分支至少经历一次，即判断的真假均曾被满足。上例需要设计测试用例使其分别满足下列条件即可
2.1 A=true，B=true，C=true，D=false
2.2 A=true，B=false，C=false，D=false

3. 条件覆盖
要使得每个判断中的每个条件的可能取值至少满足一次。上例中第一个判断应考虑到A=true，A=false，B=true，B=false；第二个判断应考虑到C=true，C=false，D=true，D=false，所以上例中可以设计测试用例满足下列条件
3.1 A=true，B=true，C=true，D=true
3.2 A=false，B=false，C=false，D=false

4. 路径覆盖：要求覆盖程序中所有可能的路径。所以可以设计测试用例满足下列条件
4.1 A=true，B=true，C=true，D=true
4.2 A=false，B=false，C=false，D=false
4.3 A=true，B=true，C=false，D=false
4.4 A=false，B=false，C=true，D=true

## 参考资料

[PowerMock使用手册](http://www.javadoc.io/doc/org.powermock/powermock-api-mockito/1.6.5)