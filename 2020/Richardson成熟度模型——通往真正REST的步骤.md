# 转：【译】Richardson成熟度模型——通往真正REST的步骤

原文地址：
[Richardson Maturity Model - steps toward the glory of REST](http://martinfowler.com/articles/richardsonMaturityModel.html)

最近我在阅读[Rest In Practice](https://www.amazon.com/gp/product/0596805829?ie=UTF8&tag=martinfowlerc-20&linkCode=as2&camp=1789&creative=9325&creativeASIN=0596805829)的草稿，这本书由我的一些同事撰写。他们希望通过这本书解释如何利用Restful Web Service来解决企业面临的很多集成上的问题。该书的核心观点是目前的Web就是一个大规模分布式系统能够很好地工作的证据，而我们则可以借助这个观点来让系统集成更加容易。

![](https://martinfowler.com/articles/images/richardsonMaturityModel/overview.png)

图1： 通往REST的步骤

为了帮助解释Web风格系统的专门属性，作者们使用了由[Leonard Richardson](http://www.crummy.com)发明的RESTful成熟度模型，该模型在一次[QCon Talk](http://www.crummy.com/writing/speaking/2008-QCon/act3.html)中被谈及。通过该模型可以很好地思考如何使用REST，因此我也尝试着对它添加一些我个人的解释。(关于协议的例子只是为了更好的说明，我并不认为去写代码并测试它们是值得做的，因此在细节中也许会存在问题。)

## LEVEL 0

该模型的出发点是使用HTTP作为远程交互的传输系统，但是不会使用Web中的任何机制。本质上这里你是为了使用你的远程交互而利用HTTP作为隧道机制(Tunneling Mechanism)，通常是基于[远程过程调用(Remote Procedure Invocation)](http://www.eaipatterns.com/EncapsulatedSynchronousIntegration.html)的。

![](http://martinfowler.com/articles/images/richardsonMaturityModel/level0.png)

图2： Level 0交互的一个例子

假设我需要和我的医生进行预约。我的预约软件首先需要知道在指定日期上我的医生的空闲时间，因此它会首先向医院预约系统发出一个请求来获取该信息。在Level 0的场景中，医院会通过某个URI来暴露出该服务端点(Service Endpoint)。然后我会向该URL发送一个文档作为请求，文档中包含了请求的所有细节。

```
POST /appointmentService HTTP/1.1
[various other headers]

<openSlotRequest date = "2010-01-04" doctor = "mjones"/>
```

然后服务器会传回一个包含了所需信息的文档：

```
HTTP/1.1 200 OK
[various headers]

<openSlotList>
  <slot start = "1400" end = "1450">
    <doctor id = "mjones"/>
  </slot>
  <slot start = "1600" end = "1650">
    <doctor id = "mjones"/>
  </slot>
</openSlotList>
```

例子中我使用了XML，但是内容实际上可以是任何格式：JSON，YAML，键值对，或者其它自定义的格式。

有了这些信息，下一步就是创建一个预约。这同样可以通过向某个端点(Endpoint)发送一个文档来完成。

```
POST /appointmentService HTTP/1.1
[various other headers]

<appointmentRequest>
  <slot doctor = "mjones" start = "1400" end = "1450"/>
  <patient id = "jsmith"/>
</appointmentRequest>
```

如果一切正常的话，那么我能够收到一个预约成功的响应：

```
HTTP/1.1 200 OK
[various headers]

<appointment>
  <slot doctor = "mjones" start = "1400" end = "1450"/>
  <patient id = "jsmith"/>
</appointment>
```

如果发生了问题，比如有人在我前面预约上了，那么我会在响应体中收到某种错误信息：

```
HTTP/1.1 200 OK
[various headers]

<appointmentRequestFailure>
  <slot doctor = "mjones" start = "1400" end = "1450"/>
  <patient id = "jsmith"/>
  <reason>Slot not available</reason>
</appointmentRequestFailure>
```

到目前为止，这都是非常直观的基于RPC风格的系统。它是简单的，因为只有Plain Old XML(POX)在这个过程中被传输。如果你使用SOAP或者XML-RPC，原理上也是基本相同的，唯一的不同是你将XML消息包含在了某种特定的格式中。

## LEVEL 1 - 资源

在Richardson成熟度模型中，通往真正REST的第一步是引入资源(Resource)这一概念。所以相比将所有的请求发送到单个服务端点(Service Endpoint)，现在我们会和单独的资源进行交互。

![](http://martinfowler.com/articles/images/richardsonMaturityModel/level1.png)

图3：Level 1中加入了资源

因此在我们的首个请求中，对指定医生会有一个对应资源：

```
POST /doctors/mjones HTTP/1.1
[various other headers]

<openSlotRequest date = "2010-01-04"/>
```

响应会包含一些基本信息，但是每个时间窗口则作为一个资源，可以被单独处理：

```
HTTP/1.1 200 OK
[various headers]

<openSlotList>
  <slot id = "1234" doctor = "mjones" start = "1400" end = "1450"/>
  <slot id = "5678" doctor = "mjones" start = "1600" end = "1650"/>
</openSlotList>
```

有了这些资源，创建一个预约就是向某个特定的时间窗口发送请求：

```
POST /slots/1234 HTTP/1.1
[various other headers]

<appointmentRequest>
  <patient id = "jsmith"/>
</appointmentRequest>
```

如果一切顺利，会收到和前面类似的响应：

```
HTTP/1.1 200 OK
[various headers]

<appointment>
  <slot id = "1234" doctor = "mjones" start = "1400" end = "1450"/>
  <patient id = "jsmith"/>
</appointment>
```

目前的区别在于，如果某个人需要针对该预约做一些操作，比如预约一些检查，那么首先需要得到该预约资源，该资源的URI类似这样：http://royalhope.nhs.uk/slots/1234/appointment ，然后向该资源发送请求。

对于像我这样喜欢以面向对象方式思考的人而言，这就好比对象的标识符(Object Identity)。不是通过传入参数而调用一个函数，而是通过调用某个特定对象上的某个方法，同时将其它信息作为参数传入到该方法中。

## LEVEL 2 - HTTP动词

在LEVEL 0和LEVEL 1中一直使用的是HTTP POST来完成所有的交互，但是有些人会使用GET作为替代。在目前的级别上并不会有多大的区别，GET和POST都是作为隧道机制(Tunneling Mechanism)让你能够通过HTTP完成交互。LEVEL 2避免了这一点，它会尽可能根据HTTP协议定义的那样来合理使用HTTP动词。

![](http://martinfowler.com/articles/images/richardsonMaturityModel/level2.png)

图4：Level 2添加了HTTP动词

获取医生的时间窗口信息，意味着需要使用GET。

```
GET /doctors/mjones/slots?date=20100104&status=open HTTP/1.1
Host: royalhope.nhs.uk
```

响应和之前使用POST发送请求时一致：

```
HTTP/1.1 200 OK
[various headers]

<openSlotList>
  <slot id = "1234" doctor = "mjones" start = "1400" end = "1450"/>
  <slot id = "5678" doctor = "mjones" start = "1600" end = "1650"/>
</openSlotList>
```

在Level 2中，向上面那样使用GET来发送一个请求是至关重要的。HTTP将GET定义为一个安全的操作，它并不会对任何事物的状态造成影响。这也就允许我们可以以不同的顺序，若干次调用GET请求而每次还能够获取到相同的结果。一个重要的结论就是它能够允许参与到路由中的参与者使用缓存机制，该机制是让目前的Web运转的如此良好的关键因素之一。HTTP包含了许多方法来支持缓存，这些方法可以在通信过程中被所有的参与者使用。通过遵守HTTP的规则，我们可以很好地利用该能力。

为了创建一个预约，我们需要使用一个能够改变状态的HTTP动词，POST或者PUT。这里我使用和前面相同的一个POST请求：

```
POST /slots/1234 HTTP/1.1
[various other headers]

<appointmentRequest>
  <patient id = "jsmith"/>
</appointmentRequest>
```

在使用POST和PUT的选择中所做的权衡取舍超出了我想在这篇文章阐述的内容，或许我会在将来的某一天专门为这个问题写一篇文章。但是我仍然还是需要指出一些人将POST和PUT的关系和CREATE以及UPDATE的关系做出关联，这是不正确的。在它们之中的选择和以上有很大的不同。

即便我使用了和LEVEL 1中相同的POST请求，在远程服务响应时还是存在一个显著的区别。如果一切顺利，服务会返回一个201响应来表明这个世界上新增了一个资源。

```
HTTP/1.1 201 Created
Location: slots/1234/appointment
[various headers]

<appointment>
  <slot id = "1234" doctor = "mjones" start = "1400" end = "1450"/>
  <patient id = "jsmith"/>
</appointment>
```

在201响应中包含了一个Location属性，它是一个URI。将来客户端可以通过GET请求获取到该资源的状态。以上的响应还包含了该资源的信息从而省去了一个获取该资源的请求。

当出现问题时，还有一个不同之处，比如某人预约了该时段：

```
HTTP/1.1 409 Conflict
[various headers]

<openSlotList>
  <slot id = "5678" doctor = "mjones" start = "1600" end = "1650"/>
</openSlotList>
```

以上响应的重要之处在于它使用了HTTP响应码来表明了问题所在。在上例中，409表明了该资源已经被更新了。相比使用200作为响应码再附带一个错误信息，在LEVEL 2中我们会明确地类似上面的响应方式。具体使用什么响应码是由协议设计者来决定，但是当错误发生的时候，应该会有不为2xx的响应码被返回。LEVEL 2引入了HTTP动词以及HTTP响应码。

这里悄悄混入了一些不一致的地方。REST建议使用所有的HTTP动词，REST的初衷就是学习和借鉴Web。但是World-Wide Web实际上并没有很多地使用PUT以及DELETE。的确是有合理的原因来更多地使用PUT以及DELETE，但是Web并不是证据之一。

Web提供的关键元素在于将安全的(比如GET操作)以及不安全的操作进行了严格的区分，再配合一些响应状态码来帮助交流过程中遇到的种种错误。

## LEVEL 3 - 超媒体控制(Hypermedia Controls)

最后的一个级别引入了你可能已经听说过的一个概念，这个概念的缩写是不那么好看的HATEOAS(Hypertext As The Engine Of Application State)。它解决的问题是，如何从获取到的时间窗口列表知道该如何创建一个预约。

![](http://martinfowler.com/articles/images/richardsonMaturityModel/level3.png)

图5：Level 3添加了超媒体控制

还是使用在Level 2中使用过的GET作为首个请求：

```
GET /doctors/mjones/slots?date=20100104&status=open HTTP/1.1
Host: royalhope.nhs.uk
```

但是响应中添加了一个新元素：

```
HTTP/1.1 200 OK
[various headers]

<openSlotList>
  <slot id = "1234" doctor = "mjones" start = "1400" end = "1450">
     <link rel = "/linkrels/slot/book" 
           uri = "/slots/1234"/>
  </slot>
  <slot id = "5678" doctor = "mjones" start = "1600" end = "1650">
     <link rel = "/linkrels/slot/book" 
           uri = "/slots/5678"/>
  </slot>
</openSlotList>
```

每个时间窗口信息现在都包含了一个URI用来告诉我们如何创建一个预约。

超媒体控制(Hypermedia Control)的关键在于它告诉我们下一步能够做什么，以及相应资源的URI。相比事先就知道了如何去哪个地址发送预约请求，响应中的超媒体控制直接在响应体中告诉了我们如何做。

预约的POST请求和Level 2中类似：

```
POST /slots/1234 HTTP/1.1
[various other headers]

<appointmentRequest>
  <patient id = "jsmith"/>
</appointmentRequest>
```

然后在响应中包含了一系列的超媒体控制，用来告诉我们后面可以进行什么操作：

```
HTTP/1.1 201 Created
Location: http://royalhope.nhs.uk/slots/1234/appointment
[various headers]

<appointment>
  <slot id = "1234" doctor = "mjones" start = "1400" end = "1450"/>
  <patient id = "jsmith"/>
  <link rel = "/linkrels/appointment/cancel"
        uri = "/slots/1234/appointment"/>
  <link rel = "/linkrels/appointment/addTest"
        uri = "/slots/1234/appointment/tests"/>
  <link rel = "self"
        uri = "/slots/1234/appointment"/>
  <link rel = "/linkrels/appointment/changeTime"
        uri = "/doctors/mjones/slots?date=20100104@status=open"/>
  <link rel = "/linkrels/appointment/updateContactInfo"
        uri = "/patients/jsmith/contactInfo"/>
  <link rel = "/linkrels/help"
        uri = "/help/appointment"/>
</appointment>
```

超媒体控制的一个显著优点在于它能够在保证客户端不受影响的条件下，改变服务器返回的URI方案。只要客户端查询”addTest”这一URI，后台开发团队可以根据需要随意修改与之对应的URI(除了最初的入口URI不能被修改)。

另一个优点是它能够帮助客户端开发人员进行探索。其中的链接告诉了客户端开发人员下面可能需要执行的操作。它并不会告诉所有的信息：获取资源最新信息以及取消操作的control都指向了相同的URI-开发人员需要自行分辨哪个使用GET，哪个使用DELETE。但是至少它提供了一个思考的起点，当有需要时让开发人员去在协议文档中查看相应的URI。

同样地，它也让服务器端的团队可以通过向响应中添加新的链接来增加功能。如果客户端开发人员留意到了以前未知的链接，那么就能够激起他们的探索欲望。

目前并没有绝对的标准来规定如何表述超媒体控制(Hypermedia Control)。在这里我的做法是使用REST in Practice团队推荐的做法，它遵循ATOM（[RFC 4287](http://atompub.org/rfc4287.html)）。使用了一个\<link\>元素，其中的uri属性指明了目标URI，rel属性用来描述关系类型。意义明确的关系(比如self用来描述它自身)的URI是不加修饰的，任何特定于该服务器的都是一个完整的URI。ATOM规定了针对众所周知的linkrels的定义是[Registry of Link Relations](http://www.iana.org/assignments/link-relations.html) 。当我写这篇文章时，还被ATOM目前的进展所限定，而ATOM目前被作为Level 3 REST服务的领导者。

## Levels的意义(The Meaning of the Levels)

我应该强调一下，Richardson成熟度模型(RMM)虽然是思考REST中有哪些元素的好方法，但是它并不直接定义REST中的级别。Roy Fielding也阐明了这一点：[Level 3 RMM是REST的前置条件](http://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven)。和软件中的众多专有名词一样，REST也有很多的定义，但是由于Roy Fielding创造了这个名词，他的定义会权威很多。

我认为RMM的用处在于它提供了一个层层递进的思考RESTful背后本质思想的方法。正因为如此，我将它视为一个工具来帮助我们学习概念，而不是作为某种评估机制。我觉得现在还没有足够多的案例来确证RESTful是实现系统集成的正确方式，尽管我十分认为它是一个很有吸引力的方案，因此我也会在很多场合推荐使用它。

将这些和Ian Robinson交流后，他强调了在Leonard Richardson首次发表RMM时，他觉得最有吸引力的一点是RMM和通用设计方法之间关系。

- Level 1 解释了如何通过分治法(Divide and Conquer)来处理复杂问题，将一个大型的服务端点(Service Endpoint)分解成多个资源。
- Level 2 引入了一套标准的动词，用来以相同的方式应对类似的场景，移除不要的变化。
- Level 3 引入了可发现行(Discoverability)，它可以使协议拥有自我描述(Self-documenting)的能力。

结果就是，这一模型帮助我们思考我们想要提供的HTTP服务是何种类型的，同时也勾勒出人们和它进行交互时的期望。

转自：https://blog.csdn.net/dm_vincent/article/details/51341037