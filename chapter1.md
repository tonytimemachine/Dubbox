## Dubbox RPC远程服务调用实战

### 一 DubboX介绍

DUBBO是一个分布式服务框架，致力于**提供高性能和透明化的RPC远程服务调用方案**，是阿里巴巴SOA**服务化治理方案**的核心框架，每天为2,000+个服务提供3,000,000,000+次访问量支持，并被广泛应用于阿里巴巴集团的各成员站点。

Dubbox是当当网根据自身的需求，为Dubbo实现了一些新功能，并将其命名为Dubbox,主要新功能包括主要的新功能包括：

* **支持REST风格远程调用（HTTP + JSON/XML\)**
  ：基于非常成熟的JBoss RestEasy框架，在dubbo中实现了REST风格（HTTP + JSON/XML）的远程调用，以显著简化企业内部的跨语言交互，同时显著简化企业对外的Open API、无线API甚至AJAX服务端等等的开发。事实上，这个REST调用也使得Dubbo可以对当今特别流行的“微服务”架构提供基础性支持。 另外，REST调用也达到了比较高的性能，在基准测试下，HTTP + JSON与Dubbo 2.x默认的RPC协议（即TCP + Hessian2二进制序列化）之间只有1.5倍左右的差距，详见下文的基准测试报告。

![](http://cdn1.infoqstatic.com/statics_s2_20170314-0434/resource/news/2014/10/dubbox-open-source/zh/resources/rest.jpg)

* **支持基于Kryo和FST的Java高效序列化实现**
  ：基于当今比较知名的
  [Kryo](https://github.com/EsotericSoftware/kryo)
  和
  [FST](https://github.com/RuedigerMoeller/fast-serialization)
  高性能序列化库，为Dubbo 默认的RPC协议添加新的序列化实现，并优化调整了其序列化体系，比较显著的提高了Dubbo RPC的性能，详见下图和文档中的基准测试报告。

![](http://cdn1.infoqstatic.com/statics_s2_20170314-0434/resource/news/2014/10/dubbox-open-source/zh/resources/rt.png)

![](http://cdn1.infoqstatic.com/statics_s2_20170314-0434/resource/news/2014/10/dubbox-open-source/zh/resources/tps.png)

* **支持基于嵌入式Tomcat的HTTP remoting体系**：基于嵌入式tomcat实现dubbo的HTTP remoting体系（即dubbo-remoting-http），用以逐步取代Dubbo中旧版本的嵌入式Jetty，可以显著的提高REST等的远程调用性能，并将Servlet API的支持从2.5升级到3.1。（注：除了REST，dubbo中的WebServices、Hessian、HTTP Invoker等协议都基于这个HTTP remoting体系）。

* **升级Spring**：将dubbo中Spring由2.x升级到目前最常用的3.x版本，减少项目中版本冲突带来的麻烦。

* **升级ZooKeeper客户端**：将dubbo中的zookeeper客户端升级到最新的版本，以修正老版本中包含的bug。

上面很多功能已在当当网内部稳定的使用，现在开源出来，供大家参考和指正。也希望感兴趣的朋友也来为Dubbo贡献更多的改进。

注：dubbox和dubbo 2.x是兼容的，没有改变dubbo的任何已有的功能和配置方式（除了升级了Spring之类的版本）。另外，dubbox也严格遵循了Apache 2.0许可证的要求。





