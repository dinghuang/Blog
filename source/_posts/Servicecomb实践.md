---
title: Servicecomb实践
date: 2019-03-20 15:47:00
tags:
    - Java
categories: Java
---

# Servicecomb实践
[Git地址](https://github.com/apache/servicecomb-pack/blob/master)

Apache ServiceComb Pack 是华为开源的一个微服务应用的数据最终一致性解决方案。

## 关键特性
- 高可用：支持高可用的集群模式部署。
- 高可靠：所有的关键事务事件都持久化存储在数据库中。
- 高性能：事务事件是通过高性能gRPC来上报的，且事务的请求和响应消息都是通过Kyro进行序列化和反序列化。
- 低侵入：仅需2-3个注解和编写对应的补偿方法即可引入分布式事务。
- 部署简单：支持通过容器（Docker）进行快速部署和交付。
- 补偿机制灵活：支持前向恢复（重试）及后向恢复（补偿）功能。
- 扩展简单：基于Pack架构很容实现多种协调协议，目前支持TCC、Saga协议，未来还可以添加其他协议支持。

## 架构
ServiceComb Pack 架构是由 alpha 和 omega组成，其中：

- alpha充当协调者的角色，主要负责对事务进行管理和协调。
- omega是微服务中内嵌的一个agent，负责对调用请求进行拦截并向alpha上报事务事件。

下图展示了alpha, omega以及微服务三者的关系： 

![](https://ws1.sinaimg.cn/large/9bc4cb9fly1g16q1uo7a2j20mp07yweq.jpg)

基础上我们除了实现saga协调协议以外，还实现了[TCC协调协议](https://github.com/apache/servicecomb-pack/blob/master/docs/design_zh.md)。 详情可浏览ServiceComb Pack 设计文档。

### Omega内部运行机制
omega是微服务中内嵌的一个agent。当服务收到请求时，omega会将其拦截并从中提取请求信息中的全局事务id作为其自身的全局事务id（即Saga事件id），并提取本地事务id作为其父事务id。在预处理阶段，alpha会记录事务开始的事件；在后处理阶段，alpha会记录事务结束的事件。因此，每个成功的子事务都有一一对应的开始及结束事件。
![](https://ws1.sinaimg.cn/large/9bc4cb9fly1g16q8mdkrvj20wo0haaau.jpg)

### 服务间通信流程
服务间通信的流程与Zipkin的类似。在服务生产方，omega会拦截请求中事务相关的id来提取事务的上下文。在服务消费方，omega会在请求中注入事务相关的id来传递事务的上下文。通过服务提供方和服务消费方的这种协作处理，子事务能连接起来形成一个完整的全局事务。
![](https://ws1.sinaimg.cn/large/9bc4cb9fly1g16q92b65tj20wi0ca74s.jpg)

### Saga 具体处理流程
Saga处理场景是要求相关的子事务提供事务处理函数同时也提供补偿函数。Saga协调器alpha会根据事务的执行情况向omega发送相关的指令，确定是否向前重试或者向后恢复。

#### 成功场景
成功场景下，每个事务都会有开始和有对应的结束事件。
![](https://ws1.sinaimg.cn/large/9bc4cb9fly1g16q9lthpvj21cw0qijsz.jpg)

#### 异常场景
异常场景下，omega会向alpha上报中断事件，然后alpha会向该全局事务的其它已完成的子事务发送补偿指令，确保最终所有的子事务要么都成功，要么都回滚。
![](https://ws1.sinaimg.cn/large/9bc4cb9fly1g16qaa8gjwj21cr0rkwg6.jpg)

#### 超时场景 (需要调整）
超时场景下，已超时的事件会被alpha的定期扫描器检测出来，与此同时，该超时事务对应的全局事务也会被中断。
![](https://ws1.sinaimg.cn/large/9bc4cb9fly1g16qatqfjbj21cu0rgwfx.jpg)

### TCC 具体处理流程
TCC(try-confirm-cancel)与Saga事务处理方式相比多了一个Try方法。事务调用的发起方来根据事务的执行情况协调相关各方进行提交事务或者回滚事务。

#### 成功场景
成功场景下， 每个事务都会有开始和对应的结束事件
![](https://ws1.sinaimg.cn/large/9bc4cb9fly1g16qbrxbazj21lo0wydin.jpg)

#### 异常场景
异常场景下，事务发起方会向alpha上报异常事件，然后alpha会向该全局事务的其它已完成的子事务发送补偿指令，确保最终所有的子事务要么都成功，要么都回滚。
![](https://ws1.sinaimg.cn/large/9bc4cb9fly1g16qcdd807j21ku0vu76t.jpg)

## omega、alpha的TSL双向证书
Saga 现在支持在omega和alpha服务之间采用 TLS 通信.同样客户端方面的认证（双向认证）。

### 准备证书 （Certificates）
你可以用下面的命令去生成一个用于测试的自签名的证书。 如果你想采用双向认证的方式，只需要客户端证书。

```
# Changes these CN's to match your hosts in your environment if needed.
SERVER_CN=localhost
CLIENT_CN=localhost # Used when doing mutual TLS

echo Generate CA key:
openssl genrsa -passout pass:1111 -des3 -out ca.key 4096
echo Generate CA certificate:
# Generates ca.crt which is the trustCertCollectionFile
openssl req -passin pass:1111 -new -x509 -days 365 -key ca.key -out ca.crt -subj "/CN=${SERVER_CN}"
echo Generate server key:
openssl genrsa -passout pass:1111 -des3 -out server.key 4096
echo Generate server signing request:
openssl req -passin pass:1111 -new -key server.key -out server.csr -subj "/CN=${SERVER_CN}"
echo Self-signed server certificate:
# Generates server.crt which is the certChainFile for the server
openssl x509 -req -passin pass:1111 -days 365 -in server.csr -CA ca.crt -CAkey ca.key -set_serial 01 -out server.crt 
echo Remove passphrase from server key:
openssl rsa -passin pass:1111 -in server.key -out server.key
echo Generate client key
openssl genrsa -passout pass:1111 -des3 -out client.key 4096
echo Generate client signing request:
openssl req -passin pass:1111 -new -key client.key -out client.csr -subj "/CN=${CLIENT_CN}"
echo Self-signed client certificate:
# Generates client.crt which is the clientCertChainFile for the client (need for mutual TLS only)
openssl x509 -passin pass:1111 -req -days 365 -in client.csr -CA ca.crt -CAkey ca.key -set_serial 01 -out client.crt
echo Remove passphrase from client key:
openssl rsa -passin pass:1111 -in client.key -out client.key
echo Converting the private keys to X.509:
# Generates client.pem which is the clientPrivateKeyFile for the Client (needed for mutual TLS only)
openssl pkcs8 -topk8 -nocrypt -in client.key -out client.pem
# Generates server.pem which is the privateKeyFile for the Server
openssl pkcs8 -topk8 -nocrypt -in server.key -out server.pem
```

### TLS为Alpha服务开启TLS
1.为alpha-server修改application.yaml文件，在alpha.server部门增加ssl配置。
```
alpha:
  server:
    ssl:
      enable: true
      cert: server.crt
      key: server.pem
      mutualAuth: true
      clientCert: client.crt
```

1. 将server.crt 和 server.pem 文件放到alpha-server的root 2目录。如果你想双向认证，合并所有client证书到一个client.crt文件,并把client.crt文件放到root目录.
2. 重新启动alpha服务器.

### 为Omega启用TLS
1. 获取CA证书串(chain), 如果你是将alpha服务运行在集群中，你可能需要去合并多个CA证书到一个文件中.
2. 为客户端应用修改application.yaml文件, 在alpha.cluster 部分增加ssl配置.

```
alpha:
  cluster:
    address: alpha-server.servicecomb.io:8080
    ssl:
      enable: false
      certChain: ca.crt
      mutualAuth: false
      cert: client.crt
      key: client.pem
```

1. 把ca.crt文件放到客户端应用程序的root目录 file under the client application root directory.如果你想用双向认证，仍需要把client.crt和client.pem放到root目录下.
2. 重新启动客户端应用程序.


## 与Spring结合使用

### Saga中的Event简介
```
public enum EventType {
  SagaStartedEvent,
  TxStartedEvent,
  TxEndedEvent,
  TxAbortedEvent,
  TxCompensatedEvent,
  SagaEndedEvent
}
```
- SagaStartedEvent: 代表Saga事务的开始，Alpha接受到该事件会保存整个saga事务的执行上下文，其中包括多个本地事务/补偿请求
- TxStartedEvent: 本地事务开始事件，其中包含了本地事务执行的上下文（调用方法名，以及相关调用参数）
- TXEndedEvent: 本地事务结束事件
- TxAbortedEvent: 本地事务执行失败事件，包含了事务执行失败的原因
- TxCompensatedEvent: 本地事务补偿事件，Alpha会将本地事务执行的上下文传递给Omega，这样不需要Omega自己维护服务调用的状态。
- SagaEndedEvent: 标志着saga事务请求的结束

![](https://ws1.sinaimg.cn/large/9bc4cb9fly1g16qo3gfn4j20x80ibn0v.jpg)
成功场景下，全局事务事件SagaStartedEvent对应SagaEndedEvent ，每个子事务开始的事件TxStartedEvent都会有对应的结束事件TXEndedEvent。 
![](https://ws1.sinaimg.cn/large/9bc4cb9fly1g16qof1a9kj20zc0j3dk0.jpg)
异常场景下，Omega会向Alpha上报中断事件TxAbortedEvent，然后Alpha会根据全局事务的执行情况， 想其它已成功的子事务(以完成TXEndedEvent)的服务发送补偿指令，以确保最终所有的子事务要么都成功，要么都回滚。
![](https://ws1.sinaimg.cn/large/9bc4cb9fly1g16qos3fkjj20ha099abd.jpg)
超时场景下，已超时的事件会被alpha的定期扫描器检测出来，同时该超时事务对应的全局事务也会被中断。
![](https://ws1.sinaimg.cn/large/9bc4cb9fly1g16qp1vtvlj20v00f0q4k.jpg)
1. 用户发送Request请求调用业务方法(business logic)
2. preIntercept向alpha发送TxStartedEvent
3. 被AOP拦截的方法(business logic)被调用
4. 当执行成功时postIntercept发送TxEndedEvent到alpha
5. 最后业务方法向用户发送response

### 与Spring和Mysql结合使用
[项目地址](https://github.com/dinghuang/servicecomb-test)

通过源码编译，克隆代码
```
git clone https://github.com/apache/servicecomb-pack.git
```
在``alpha/alpha-server/pom.xml``文件中加入mysql依赖
```
<dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
</dependency>
```
构建docker镜像
```
cd ./servicecomb-pack
mvn clean install -DskipTests -Pdocker
```
成功后如图所示
```
[@dinghuangMacPro:~]$ docker images
REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
pack-web                 0.3.0               77dedfe8e865        15 seconds ago      131MB
alpha-server             0.3.0               3a34b8cd4224        38 seconds ago      144MB
```
启动mysql镜像，如果本地有的话
创建库saga，用户saga，密码password，并执行数据库脚本``schema-mysql.sql``

启动alpha-server
```
docker run -d -p 8080:8080 -p 8090:8090 --link mysql:mysql.servicecomb.io -e JAVA_OPTS=-Dspring.profiles.active=mysql -e -Dspring.datasource.url=jdbc:mysql://127.0.0.1:3306/saga?useSSL=false alpha-server:0.3.0
```

启动对应的3个应用，分别说shop，order，hotel

访问api
```
curl -X POST http://127.0.0.1:8081/shop/userName/orderName/hotelName
```

### 事务解析
请求流程示意图：用户发起请求到shop，shop分别调用order和hotel。
![](https://ws1.sinaimg.cn/large/9bc4cb9fgy1g193ofrsu9j20q00umta3.jpg)

使用TCC模式，TCC原理图如图所示：
![](https://ws1.sinaimg.cn/large/9bc4cb9fgy1g193pjsh7tj20nc0blq9i.jpg)

#### 情况一：正常事务结束
![](https://ws1.sinaimg.cn/large/9bc4cb9fly1g199o8nyxcj21660e6128.jpg)
![](https://ws1.sinaimg.cn/large/9bc4cb9fly1g199omza6aj21nq044goa.jpg)
![](https://ws1.sinaimg.cn/large/9bc4cb9fly1g199oxaun5j21my03ctb9.jpg)
事务记录成功，订单酒店表都有数据。

#### 情况二：父事件中调用订单成功后，出现异常
```
//父事务
@TccStart(timeout = 2)
@PostMapping("/shop_tcc/{name}/{order}/{hotel}")
public String shopTcc(@PathVariable String name, @PathVariable String order, @PathVariable String hotel) {
        //调用订单服务的请求
        template.postForEntity("http://127.0.0.1:8082/order_tcc/{name}/{order}",null, String.class, name, order);
         //异常
        postBooking();
        //调用酒店服务的请求
        template.postForEntity("http://127.0.0.1:8083/hotel_tcc/{name}/{hotel}",null, String.class, name, hotel);
        return name + " order " + order + "hotel " + hotel + " cars OK";
}
```
```
//订单(酒店)中的代码逻辑
@Transactional(rollbackFor = Exception.class)
void cancel(OrderDO orderDO) {
        orderRepository.deleteById(orderDO.getId());
}

@Transactional(rollbackFor = Exception.class)
void confirm(OrderDO orderDO) {
        orderRepository.insert(orderDO);
}

@Participate(confirmMethod = "confirm", cancelMethod = "cancel")
@Transactional(rollbackFor = Exception.class)
public void orderTcc(OrderDO orderDO) {
}
```
数据库结果如图所示
![](https://ws1.sinaimg.cn/large/9bc4cb9fly1g199hjb9mdj213u0bowkw.jpg)
订单和商店的表都没有生成数据。

#### 情况三：父事件中调用订单和酒店成功后，出现异常
```
//父事务
@TccStart(timeout = 2)
@PostMapping("/shop_tcc/{name}/{order}/{hotel}")
public String shopTcc(@PathVariable String name, @PathVariable String order, @PathVariable String hotel) {
        //调用订单服务的请求
        template.postForEntity("http://127.0.0.1:8082/order_tcc/{name}/{order}",null, String.class, name, order);
        //调用酒店服务的请求
        template.postForEntity("http://127.0.0.1:8083/hotel_tcc/{name}/{hotel}",null, String.class, name, hotel);
         //异常
        postBooking();
        return name + " order " + order + "hotel " + hotel + " cars OK";
}
```
数据如图所示：
![](https://ws1.sinaimg.cn/large/9bc4cb9fly1g199w48vfaj217e0ciajm.jpg)
订单表与酒店表都没有产生数据

#### 情况四：父事件超时
```
//父事务
@TccStart(timeout = 2)
@PostMapping("/shop_tcc/{name}/{order}/{hotel}")
public String shopTcc(@PathVariable String name, @PathVariable String order, @PathVariable String hotel) throws InterruptedException {
        //调用订单服务的请求
        template.postForEntity("http://127.0.0.1:8082/order_tcc/{name}/{order}",null, String.class, name, order);
        //调用酒店服务的请求
        template.postForEntity("http://127.0.0.1:8083/hotel_tcc/{name}/{hotel}",null, String.class, name, hotel);
         //超时
        Thread.sleep(10000);
        return name + " order " + order + "hotel " + hotel + " cars OK";
}
```
发现TCC的timeout选项好像没有作用。。。。看了下源码，的确没有用到，源码如下
![](https://ws1.sinaimg.cn/large/9bc4cb9fly1g19adzx8ahj2246138drc.jpg)
ServiceComb在0.3.0加入了TCC的支持，所以有些功能还待完善把。

#### 情况五：订单服务启动，酒店服务未启动
关闭hotel服务，执行后数据如下：
![](https://ws1.sinaimg.cn/large/9bc4cb9fly1g19ajirqhpj215s0aiq99.jpg)
订单和酒店数据库都没有数据

#### 情况六: 模拟运行过程中alpha服务挂起，订单、酒店、商店服务正常运行：
订单、酒店、商店服务后来日志显示心跳连接失效
![](https://ws1.sinaimg.cn/large/9bc4cb9fgy1g19aq910axj223k0gstfo.jpg)
请求数据返回错误信息，数据库表均未写入数据。
![](https://ws1.sinaimg.cn/large/9bc4cb9fgy1g19ap4jromj211602kwjk.jpg)

重新启动alpha服务，订单、酒店、商店服务重新连接到alpha，业务正常运行。

#### 情况七: 模拟运行过程中alpha服务的mysql挂起，订单、酒店、商店服务正常运行：
请求未进入业务逻辑之前，alpha服务报错，请求未执行。
mysql重启成功后，alpha服务正常运行，请求数据正常执行。

### 总结
ServiceComb对于数据最终一致性的解决现阶段0.3.0是满足业务逻辑的，但是对于失败重试、超时等功能这一块还不支持，后期应该会扩展。ServiceComb功能比较简单，但是可以通过对omega的事物id结合调用链追踪实现业务流程与事务的追溯。