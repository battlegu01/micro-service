#   <p align=center>micro-service</p>
##  为什么需要微服务？
####    传统架构的痛点
在没有服务层的架构中，后台服务器中运行代码直接通过ORM框架与DB相连，这样带来了如下的问题。
*   代码到处拷贝
*   复杂性扩散
*   SQL质量无法保障，业务相互影响
*   疯狂的DB耦合
#### 服务层的引入
*   调用简单

有服务层之前，业务方访问用户数据，需要通过DAO拼装SQL访问。有服务层之后，业务方通过RPC访问用户数据，就像调用一个本地函数一样，非常之爽：
```
User = UserService::GetUserById(uid);
```
传入一个uid，得到一个User实体，就像调用本地函数一样，不需要关心序列化，网络传输，后端执行，网络传输，范序列化等复杂性。
*   复用性，防止代码拷贝

所有user数据的存取，都通过user-service来进行，代码只此一份，不存在拷贝。升级一处升级，bug修改一处修改。 
* 专注性 屏蔽底层复杂性

避免各个业务模块都能直接操作数据库，所有的SQL由更加专业的团队提供，对其他团队来讲降低了业务的复杂性。
* 数据库解耦
数据库能够更容易得被隔离，避免了大量的join操作

* 附：脱离业务谈架构，纯属耍流氓 

##  微服务架构的两大体系
### REST API
#### 什么是REST
> REST这个词，是Roy Thomas Fielding在他2000年的博士论文中提出的。REST的名称"表现层状态转化"中，省略了主语。"表现层"其实指的是"资源"（Resources）的"表现层"。所谓"资源"，就是网络上的一个实体，或者说是网络上的一个具体信息。它可以是一段文本、一张图片、一首歌曲、一种服务，总之就是一个具体的实在。你可以用一个URI（统一资源定位符）指向它，每种资源对应一个特定的URI。要获取这个资源，访问它的URI就可以，因此URI就成了每一个资源的地址或独一无二的识别符。

说的更通俗一点就是，网络上一切资源都以HTTP协议进行传输，以URL为一切资源的入口，服务与服务之间通过HTTP URL进行传输和通信，以API为通信的规范和准则.

基于HTTP的REST支持不同的格式, 比如JSON或者二进制, JSON是一种比XML内容更加紧凑的文本方式. 需要注意的是: 基于HTTP的通信, 适合大流量的场景.
#### 基于RPC的微服务架构

RPC也叫远程过程调用, 你以为你在调用本地的一个方法,但实际上该方法是远程服务器产生的.依赖借口定义(SOAP, Thrift, Protocol buffers等), 轻松生成客户端和服务端的桩代码. 例如, 我可以用一个Java服务暴露一个SOAP接口, 然后使用WSDL(Web Service Definition Language)定义的接口生成.NET客户端的代码.

这些RPC的实现会帮你生成服务端和客户端的桩代码,从而让你快速开始编码. 基本不用花时间, 我就可以在服务之间进行内容交互了. 这也是RPC的主要卖点之一: 易于使用.

#### 常用框架
1. Thrift
国外用的多，源于Facebook，后捐献给Apache基金。是Apache的顶级项目 Apache Thrift。使用者包括Facebook、Evernote、Uber、Pinterest等大型互联网公司。 而在开源界，Apache Hadoop/HBase也在使用Thrift作为内部通讯协议。 这是目前最为成熟的框架，优点在于稳定、高性能。缺点在于它仅提供RPC服务，其他的功能，包括限流、熔断、服务治理等，都需要自己实现，或者使用第三方软件。
2. Dubbo
国内用的多，阿里公司开源。 性能上略逊于Apache Thrift，但自身集成了大量的微服务治理功能，使用起来相当方便。 Dubbo的问题在于，该系统目前已经很长时间没有维护更新了。 官网显示最近一次的更新也是8个月前。
3. Google Protobuf
和Apache Thrift类似，Google Protobuf也包括数据定义和服务定义两部分。问题是，Google Protobuf一直只有数据模型的实现，没有官方的RPC服务的实现。 直到2015年才推出gRPC，作为RPC服务的官方实现。但缺乏重量级的用户。Thrift 提供多种高性能的传输协议，但在数据定义上，不如Protobuf强大。

#### REST与PRC之间的对比
1. 以Apache Thrift为代表的二进制RPC，支持多种语言（但不是所有语言），四层通讯协议，性能高，节省带宽。相对Restful协议，使用Thrift RPC，在同等硬件条件下，带宽使用率仅为前者的20%，性能却提升一个数量级。但是这种协议最大的问题在于，无法穿透防火墙。
2. 以Spring Cloud为代表所支持的Restful 协议，优势在于能够穿透防火墙，使用方便，语言无关，基本上可以使用各种开发语言实现的系统，都可以接受Restful 的请求。 但性能和带宽占用上有劣势。

## 服务发现
将容器应用部署到集群时，其服务地址，即IP和端口, 是由集群系统动态分配的。那么，当我们需要访问这个服务时，如何确定它的地址呢？这时，就需要服务发现(Service Discovery)了.

如果使用预定义的端口，服务越多，发生冲突的可能性越大，毕竟，不可能有两个服务监听同一个端口。管理一个拥挤的比方说被几百个服务所使用的所有端口的列表，本身就是一个挑战，添加到该列表后，这些服务需要的数据库和数量会日益增多。因此我们应该部署无需指定端口的服务，并且让Docker为我们分配一个随机的端口。唯一的问题是我们需要发现端口号，并且让别人知道。

#### zookeeper
>Zookeeper是这种类型的项目中历史最悠久的之一，它起源于Hadoop，帮助在Hadoop集群中维护各种组件。它非常成熟、可靠，被许多大公司（YouTube、eBay、雅虎等）使用。其数据存储的格式类似于文件系统，如果运行在一个服务器集群中，Zookeper将跨所有节点共享配置状态，每个集群选举一个领袖，客户端可以连接到任何一台服务器获取数据。

> Zookeeper的主要优势是其成熟、健壮以及丰富的特性，然而，它也有自己的缺点，其中采用Java开发以及复杂性是罪魁祸首。尽管Java在许多方面非常伟大，然后对于这种类型的工作还是太沉重了，Zookeeper使用Java以及相当数量的依赖使其对于资源竞争非常饥渴。因为上述的这些问题，Zookeeper变得非常复杂，维护它需要比我们期望从这种类型的应用程序中获得的收益更多的知识。这部分地是由于丰富的特性反而将其从优势转变为累赘。应用程序的特性功能越多，就会有越大的可能性不需要这些特性，因此，我们最终将会为这些不需要的特性付出复杂度方面的代价。

> Zookeeper为其他项目相当大的改进铺平了道路，“大数据玩家“在使用它，因为没有更好的选择。今天，Zookeeper已经老态龙钟了，我们有了更好的选择。

#### consul
> Consul是强一致性的数据存储，使用gossip形成动态集群。它提供分级键/值存储方式，不仅可以存储数据，而且可以用于注册器件事各种任务，从发送数据改变通知到运行健康检查和自定义命令，具体如何取决于它们的输出。

> 与Zookeeper和etcd不一样，Consul内嵌实现了服务发现系统，所以这样就不需要构建自己的系统或使用第三方系统。这一发现系统除了上述提到的特性之外，还包括节点健康检查和运行在其上的服务。

> Zookeeper和etcd只提供原始的键/值队存储，要求应用程序开发人员构建他们自己的系统提供服务发现功能。而Consul提供了一个内置的服务发现的框架。客户只需要注册服务并通过DNS或HTTP接口执行服务发现。其他两个工具需要一个亲手制作的解决方案或借助于第三方工具。

> Consul为多种数据中心提供了开箱即用的原生支持，其中的gossip系统不仅可以工作在同一集群内部的各个节点，而且还可以跨数据中心工作。
