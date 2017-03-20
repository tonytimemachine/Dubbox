## Dubbox RPC远程服务调用实战

### 一 DubboX介绍

DUBBO是一个分布式服务框架，致力于**提供高性能和透明化的RPC远程服务调用方案**，是阿里巴巴SOA**服务化治理方案**的核心框架，每天为2,000+个服务提供3,000,000,000+次访问量支持，并被广泛应用于阿里巴巴集团的各成员站点。

架构图如下:

![](/assets/dubbo.png)



**节点角色说明：**

* **Provider:**
  暴露服务的服务提供方。
* **Consumer:**
  调用远程服务的服务消费方。
* **Registry:**
  服务注册与发现的注册中心。
* **Monitor:**
  统计服务的调用次调和调用时间的监控中心。
* **Container:**
  服务运行容器。

**调用关系说明：**

* 0. 服务容器负责启动，加载，运行服务提供者。
* 1. 服务提供者在启动时，向注册中心注册自己提供的服务。
* 2. 服务消费者在启动时，向注册中心订阅自己所需的服务。
* 3. 注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。
* 4. 服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。
* 5. 服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心。

**\(1\) 连通性：**

* 注册中心负责服务地址的注册与查找，相当于目录服务，服务提供者和消费者只在启动时与注册中心交互，注册中心不转发请求，压力较小
* 监控中心负责统计各服务调用次数，调用时间等，统计先在内存汇总后每分钟一次发送到监控中心服务器，并以报表展示
* 服务提供者向注册中心注册其提供的服务，并汇报调用时间到监控中心，此时间不包含网络开销
* 服务消费者向注册中心获取服务提供者地址列表，并根据负载算法直接调用提供者，同时汇报调用时间到监控中心，此时间包含网络开销
* 注册中心，服务提供者，服务消费者三者之间均为长连接，监控中心除外
* 注册中心通过长连接感知服务提供者的存在，服务提供者宕机，注册中心将立即推送事件通知消费者
* 注册中心和监控中心全部宕机，不影响已运行的提供者和消费者，消费者在本地缓存了提供者列表
* 注册中心和监控中心都是可选的，服务消费者可以直连服务提供者

**\(2\) 健状性：**

* 监控中心宕掉不影响使用，只是丢失部分采样数据
* 数据库宕掉后，注册中心仍能通过缓存提供服务列表查询，但不能注册新服务
* 注册中心对等集群，任意一台宕掉后，将自动切换到另一台
* 注册中心全部宕掉后，服务提供者和服务消费者仍能通过本地缓存通讯
* 服务提供者无状态，任意一台宕掉后，不影响使用
* 服务提供者全部宕掉后，服务消费者应用将无法使用，并无限次重连等待服务提供者恢复

**\(3\) 伸缩性：**

* 注册中心为对等集群，可动态增加机器部署实例，所有客户端将自动发现新的注册中心
* 服务提供者无状态，可动态增加机器部署实例，注册中心将推送新的服务提供者信息给消费者

**\(4\) 升级性：**

* 当服务集群规模进一步扩大，带动IT治理结构进一步升级，需要实现动态部署，进行流动计算，现有分布式服务架构不会带来阻力：



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

### 二  Zookeeper的安装和配置

将Zookeeper的安装包\(zookeeper-3.4.6.tar.gz\)上传至服务器,解压缩,修改环境变量,即可通过zkServer.sh启动zookeeper服务。

```
[root@tony local]# tar -zxvf zookeeper-3.4.6.tar.gz ##解压缩
[root@tony local]# mv zookeeper-3.4.6 zookeeper ##重命名
[root@tony local]# mv /usr/local/zookeeper/conf/zoo_sample.cfg  /usr/local/zookeeper/conf/zoo.cfg #重命名配置文件
[root@tony local]# vim /etc/profile ##修改系统环境变量配置文件
export ZOOKEEPER_HOME=/usr/local/zookeeper ##增加zookeeper主目录至系统环境变量
export PATH=$PATH:$ZOOKEEPER_HOME/bin 

[root@tony local]# source /etc/profile #立即生效
[root@tony bin]# sh /usr/local/zookeeper/bin/zkServer.sh start ##启动zookeeper服务
[root@tony bin]# ps -ef|grep zookeeper ##查看zookeeper进程
```

### 三 基于Dubbo协议的服务化案例

### 



