# SpringCloud：

## 0,SpringCloud升级,部分组件停用:

1,Eureka停用,可以使用zk作为服务注册中心

2,服务调用,Ribbon准备停更,代替为LoadBalance

3,Feign改为OpenFeign

4,Hystrix停更,改为resilence4j

​		或者阿里巴巴的sentienl

5.Zuul改为gateway

6,服务配置Config改为  Nacos

7,服务总线Bus改为Nacos

# 1.环境搭建

## 1.创建父工程Pom文件依赖

![1594901402079](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1594901402079.png)

## 文件

![1594907705720](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1594907705720.png)

## 2.创建子模块

**cloud-provider-payment8001**

1. 改写pom文件

### 改写yml

![1594945863827](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1594945863827.png)

### 启动类

![1594945883025](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1594945883025.png)

### 创建数据库，及创建实体类，还有entity

```sql

CREATE TABLE payment (

 id BIGINT(20) NOT NULL AUTO_INCREMENT COMMENT id,
 SERIAL VARCHAR(200) DEFAULT,
 PRIMARY KEY(id)
) ENGINE=INNODB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8

```

![1594945951561](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1594945951561.png)

​	![1594946357099](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1594946357099.png)

### dao ==Mapper配置文件

![1594949224118](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1594949224118.png)

![1594949235319](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1594949235319.png)

### Service层

![1594949266078](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1594949266078.png)

![1594949273885](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1594949273885.png)

### Contorller

![1594949294327](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1594949294327.png)

### 测试：

 http://localhost:8001/payment/get/1 



![1594954365787](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1594954365787.png)



![1594954394610](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1594954394610.png)

## 3.热部署

### 3.1父工程

![1594954544953](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1594954544953.png)



![1594954584699](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1594954584699.png)

### 3.2setting

![1594954691293](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1594954691293.png)

### 3.3

![1594954722702](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1594954722702.png)

### 3.4重启idea

## 4.order模块微服务消费者，调用

### 4.1建立模块cloud-consumer-order80

### 4.2改pom

### 4.3yml

![1594976724385](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1594976724385.png)

### 4.4复制pay模块的实体类，rntities

### 4.5写contorller

因为这里是消费者类,主要是消费,那么就没有service和dao,需要调用pay模块的方法

​		并且这里还没有微服务的远程调用,那么如果要调用另外一个模块,则需要使用基本的api调用

使用RestTemplate调用pay模块,：**就是调用pay模块的功能**

![1594976873273](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1594976873273.png)

#### 	将restTemplate注入到容器

```java
package com.atguigu.springcloud.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration
public class ApplicationContextConfig {

    @Bean
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
}

```

写contorller

```java
package com.atguigu.springcloud.contorller;

import com.atguigu.springcloud.entities.CommonResult;
import com.atguigu.springcloud.entities.Payment;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.client.RestTemplate;

import javax.annotation.Resource;

@RestController
@Slf4j
public class OrderContorller {

    public static final String PAYMANT_URL="http://localhost:8001";

    @Resource
    private RestTemplate restTemplate;

    @PostMapping("/consumer/payment/create")
    public CommonResult<Payment> create( Payment payment){
        return restTemplate.postForObject("http://localhost:8001/payment/create",payment,CommonResult.class);
    }

    @GetMapping("/consumer/payment/get/{id}")
    public CommonResult<Payment> getPaymentById(@PathVariable("id") Long id){
        return restTemplate.getForObject("http://localhost:8001/payment/get/"+id,CommonResult.class);
    }

}

```

测试：

先启动pay模块，在启动order模块

http://localhost/consumer/payment/get/32

## 5.工程重构

抽取entities的实体类

### 5.1创建模块

cloud-api-commons

### 5.2pom

### 5.3entities实体类的复制

### 5.4maven命令clean和 install

### 5.5pay模块和order模块删除entities在导入pom

![1594977421303](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1594977421303.png)

### 5.6测试

![1594977537296](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1594977537296.png)

# 2.服务注册与发现

## 6.Eureke

前面我们没有服务注册中心,也可以服务间调用,为什么还要服务注册?

当服务很多时,单靠代码手动管理是很麻烦的,需要一个公共组件,统一管理多服务,包括服务是否正常运行,等

Eureka用于**==服务注册==**,目前官网**已经停止更新**

![1594987320372](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1594987320372.png)

![1594987327621](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1594987327621.png)

![1594987356632](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1594987356632.png)

### 1.单机版Eureka

创建工程：

#### 1,创建项目cloud_eureka_server_7001

#### 2.pom文件

#### 3.yml文件

```yml
server:
  port: 7001
eureka:
  instance:
    hostname: localhost #eureka服务端实例的名字
  client:
    register-with-eureka: false #不向eureka注册自己
    fetch-registry: false #表示自己就是注册中心
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/    #设置与eureka server交互的地址查询服务和注册服务都需要依赖这个地址


```

#### 4.在启动类@EnableService注解

#### 5.测试：

 http://localhost:7001/ 

![1594990790572](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1594990790572.png)

#### 6.EurekaClient端cloud-provider-payment8001

##### 6.1添加pom

```java

        <!-- eureka-client -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>

```



##### 6.3修改yml

```yaml
eureka:
  client:
    register-with-eureka: true  #向eureka注册自己  #表示自己就是注册中心
    fetchRegistry: true #是否从eurekaservice抓取已有的信息，默认true
    service-url:
      defaultZone: http://localhost:7001/eureka   #设置与eureka server交互的地址查询服务和注册服务都需要依赖这个地址



```

##### 6.4添加@EnableEurekaClien注解在启动类上

##### 6.6测试

 http://localhost:7001/ 

![1594990989719](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1594990989719.png)

### 2.集群Eureka构建步骤

#### Eureka集群原理说明

![1594991122985](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1594991122985.png)

```
1.就是向eureka注册了的模块，会把自己的一些信息向eureka共享
2.order模块而回把自己的信息注册给eureka，当需要调用pay模块的时候，会在eureka中获取pay的地址
3.通过heepclient调用，会在本地的jvm也会缓冲一份；每30秒更新一次
```

解决办法：搭建Eureka注册中心集群，实现负载均衡+故障容错

互注册，互相守护

![1594991380753](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1594991380753.png)

#### 创建Eureka集群

##### 1.创建cloud-eureka-server7002模块

##### 2.pom文件

和7001模块一样就行

##### 3.修改host文件

![1594993700880](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1594993700880.png)

##### 4.修改application.yml文件

7001：

```yaml
server:
  port: 7001
eureka:
  instance:
    hostname: eureka7001.com #eureka服务端实例的名字
  client:
    register-with-eureka: false #不向eureka注册自己
    fetch-registry: false #表示自己就是注册中心
    service-url:
      defaultZone: http://eureka7002.com:7002/eureka/    #设置与eureka server交互的地址查询服务和注册服务都需要依赖这个地址


```

7002：

```yaml
server:
  port: 7002
eureka:
  instance:
    hostname: eureka7002.com #eureka服务端实例的名字
  client:
    register-with-eureka: false #不向eureka注册自己
    fetch-registry: false #表示自己就是注册中心
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/    #设置与eureka server交互的地址查询服务和注册服务都需要依赖这个地址


```

##### 5.配置7002的启动类

##### 6.测试：看看有咩有相互注册

![](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\Snipaste_2020-07-17_21-41-53.png)

7001：

![](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\Snipaste_2020-07-17_21-36-47.png)

##### 将支付服务pay,order微服务发布到上面2台Eureka集群配置中

7001，7002都是一样修改yml文件中的eureka的路径：

```java
  defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/  #集群版
```

测试：

1. 先启动Eurekaservice服务 7001/7002

   ![1594994332611](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1594994332611.png)

2. 在启动服务者提供模块 provifer 8001服务

3. 在启动消费 80

4. 在测试 http://localhost/consumer/payment/get/1 

   ![1594994358462](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1594994358462.png)

### 3.将支付服务提供者pay模块也配置为集群

#### 0.创建模块8002，pom和8001一样

#### 1.application.yml也复制8001的，修改端口为8002

#### 2.配置文件也复制8001端口--mapper

#### 3.主启动类也复制，修改为自己的名字

#### 4.8001模块的contorller service，dao等等都复制

#### 5.修改8001/8002的contorller

![1595040115572](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595040115572.png)

现在的地址会改变，所以要变成动态的了，现在的订单服务地址不能写死

![1595040219570](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595040219570.png)

所以改为微服务的名字：![1595040246921](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595040246921.png)

**使用@LoadBalanced注解赋予RestTemplate负载均衡的能力**

![1595040287370](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595040287370.png)

#### 6.测试

先启动Eurekaserver7001、7002服务

![1595040425408](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595040425408.png)

![1595040436264](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595040436264.png)

在启动服务提供者provider，8001、8002

 http://localhost/consumer/payment/get/1 

![1595040453707](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595040453707.png)

![1595040475051](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595040475051.png)

结果：负载均衡出现，8001、8002端口交替出现

### 4.actuator微服务信息完善，

服务名修改

#### 修改8001yml文件

![1595041592443](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595041592443.png)

修改之后：

![1595041629808](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595041629808.png)

### 5.eureka服务发现Discovery

**对于注册进eureka里面的微服务，可以通过这个服务来获取服务信息。相对而言就是可以拿取这个微服务的信息，暴露一样**

#### 5.1修改8001contorller

添加注解

![1595042881697](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595042881697.png)

添加功能

![1595042993412](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595042993412.png)

启动类加上@EnableDiscveryClient注解

#### 5.2测试

先要启动EurekaServer，7001/7002服务

再启动8001主启动类，需要稍等一会

http://localhost:8001/payment/discovery

![6](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\Snipaste_2020-07-17_21-41-59.png)

控制台：![](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\Snipaste_2020-07-18_11-24-26.png)

### 6.Eureka的自我保护

当出现一下图片：说明进入保护模式

![1595053231909](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595053231909.png)

概述：保护模式主要用于一组客户端和EurekaServer之间的网络分区场景下的保护，一旦进入保护模式，EurekaServer将会尝试保护其服务注册的信息，不在删除服务注册表中的数据，也就是不会注销任何微服务。

#### 6.1为什么会产生Eureka自我保护机制？

为了防止EurekaClient可以正常运行，但是EurekaServe网络不通的情况下，EurekaServer不会立即将EurekaClient服务注销。

#### 6.2什么是自我保护机制？

每个EurekaClient会隔一段时间向EurekaClient发送心跳，来证明自己还活着，还可以用，不让EurekaServer注销。但是，EurekaServer在一定时间（默认90s）没有收到EurekaClnen（但是EurekaClient真真真的存在，是因为其他原因，才没发送这个心跳）的心跳，EurekaServer会进入自己保护机制，不会立即注销EurekaClient。

![1595053767475](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595053767475.png)

![1595053775358](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595053775358.png)

![1595053786873](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595053786873.png)

#### 6.3怎么禁止自我保护（一般生产环境中不会禁止自我保护）

##### 6.3.1注册中心eureakeServer端7001设置

出厂默认是开启自我保护机制的

> eureka.server.enable-self-preservation = true

关闭：

![1595055049376](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595055049376.png)

效果：

![](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\Snipaste_2020-07-18_14-39-38.png)

##### 6.3.2配置eurekaClient端8001

配置yml文件

![1595055243284](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595055243284.png)

##### 6.3.3测试

先启动7001，在启动8001，再关闭8001里面就会在那里剔除8001

## 7.Zookeeper

## 8.Consul

### 8.1官网：https://www.consul.io/intro/index.html

#### 8.1.1能干嘛

![1595059328945](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595059328945.png)

https://www.consul.io/downloads.html 官网下载

中文下载文档https://www.springcloud.cc/spring-cloud-consul.html

#### 8.1.2下载，安装，解压后：

![1595059407936](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595059407936.png)

#### 8.1.3查看版本

**consul --version**

使用开发者模式：**consul agent -dev**

通过以下地址可以访问Consul的首页：http;//localhost:8500：

![1595059527595](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595059527595.png)

### 8.2服务提供者创建模块将放入consul

创建服务提供者：cloud-providerconsul-payment8006

##### 8.2.1pom

##### 8.2.3YML

```yaml
server:
  port: 8006


spring:
  application:
    name: consul-provider-order
  cloud: ##注册地址consul
    consul:
      host: localhost
      port: 8500
      discovery:
        service-name: ${spring.application.name}
```



##### 8.2.4启动类

![1595059793592](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595059793592.png)

##### 8.2.5业务类Contorller

![1595059807500](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595059807500.png)

##### 8.2.6测试

![Snipaste_consulceshi3](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\Snipaste_consulceshi3.png)

http://localhost:8006/payment/consul：

![1595059864146](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595059864146.png)

### 8.3消费服务放入consul

#### 8.3.1consumerconsul-order80

#### 8.3.2pom

#### 8.3.3yml

```
server:
  port: 80


spring:
  application:
    name: consul-cousmer-order
  cloud: ##注册地址consul
    consul:
      host: localhost
      port: 8500
      discovery:
        service-name: ${spring.application.name}
```

#### 8.3.4启动类

#### 8.3.5配置类

![1595064312075](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595064312075.png)

#### 8.3.6contorller

![1595064339669](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595064339669.png)

#### 8.3.7测试：

![1595064396670](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595064396670.png)

8.3.8访问测试地址我失败了

# 3.服务调用

## 9.Ribbon负载均衡：

Spring Cloud Ribbon是基于Netflix Ribbon实现的一套客户端负载均衡的工具

Ribbon式Netflix发布开源项目，主要功能式提供客户端的软件负载均衡算法和服务调用。Ribbon客户端组件提供了一系列完善的配置项如连接超时，重试等！在配置文件中出现Load Blancer后面的所有机器，Ribbon会自动帮助基于你某种规则取链接这些机器，我们式容易实现Ribbon自定义的负载均衡算法.

**目前Ribbon也进入了维护模式。**

### 9.1Ribbon能干嘛

**LB负载均衡（Load Balancer)是什么？**

就是将用户的请求平摊分配到多个服务上，从而达到系统的HA（高可用）。

常见的负载均衡有软件Nginx，LVS,硬件F5等！

**Ribbon本地负载均衡客户端VSNginx服务端负载均衡的区别？**

Nginx式服务器负载均衡，客户端所有请求都会交给nginx，然后nginx实现转发请求。==负载均衡是由服务端实现的。

Ribbon是本地负载均衡，在调用微服务接口时，会在注册中心上获取注册信息服务列表之后缓存到JVM本地，从而在本地实现RPC远程服务调用技术。

**集中式LB（服务端负载均衡）：**在服务的消费放方之间使用独立的LB设施（可以式硬件，如F5还可以是软件：Nginx），由该设备负责把访问请求通过某种策略发至服务提供方。

**进程内LB（本地负载均衡）：**将LB逻辑集成到消费方，消费方从注册中心获知哪些地址可用，然后再从这些地址中选出一个合适的服务器。Ribbon就是进程内LB，集成消费方进程，消费通过它来获取服务提供方的地址。

**负载均衡+RestTemplate调用**

![1595080007404](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595080007404.png)

Ribbon在工作时分成两步：

第一步选择EurekaServer，它优先选择在同一个区域内负载较小的server。

第二步再根据用户指定的策略，再从server取到服务注册列表中选择一个地址。

其中Ribbon有很多种策略：轮询，随机，根据响应时间加权

### 9.2使用Ribbon

我们使用的中已经集成了Ribbon

```
 <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>

```

![1595080454464](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595080454464.png)

我们也可以手动引入

![1595080506694](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595080506694.png)

### 9.3RestTemplate类

![1595080879174](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595080879174.png)

getForObject方法/getForEntity方法

![1595080943130](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595080943130.png)

> getForEntity：返回的是ResponseEntity对象，包含了更详细的信息
>
> getForObject：返回的json字符串

![1595081084201](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595081084201.png)

### 9.4Ribbon常用负载均衡算法

**IRule接口,Riboon使用该接口,根据特定算法从所有服务中,选择一个服务,**

**Rule接口有7个实现类,每个实现类代表一个负载均衡算法**

![1595122097288](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595122097288.png)

### 9.5使用Ribbon，替换算法

**还是Eureka服务注册**

![1595122187095](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595122187095.png)

#### 9.5.1修改模块cloud-consumer-oeder80

#### 9.5.2新建packge

![1595122312179](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595122312179.png)

不能放在ComponentScan扫描的包下，以及子包

#### 9.5.3配置规则类

![1595122388120](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595122388120.png)

#### 9.5.4启动类添加@RibbonClient注解

![1595122471762](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595122471762.png)

#### 9.5.5测试

 http://localhost/consumer/payment/get/1 

### 9.6Ribbon负载均衡算法

#### **默认系统的Ribbon负载均衡轮询的算法的原理**

![1595123107491](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595123107491.png)

### 9.7自定义负载均衡算法

1. 给Pay模块（8001，8002）的contorller添加一个方法

![1595133816892](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595133816892.png)

2. 修改80order模块

   ![1595133893804](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595133893804.png)

3. 自定义一个接口，以及实现类

   ![1595133937013](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595133937013.png)

   实现类

![1595134172389](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595134172389.png)

4. Contorller 找1. 的8001，8002的contorller

   

![1595134358532](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595134358532.png)

5. 测试：是轮询的。

![1595134403451](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595134403451.png)

![1595134415496](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595134415496.png)

## 10.OpenFegin

Fegin是一个声明式的Web服务客户端，还集成了Eureka和Ribbon实现负载均衡的HTTP客户端

**Feign是一个声明式的web服务客户端，让编写web服务客户端变得非常容易，只需创建一个接口并在接口上添加注解即可**

### 10.1Fegin能干吗

![1595142619410](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595142619410.png)

Fegin集成了Ribbon

利用Ribbon维护了pay模块服务列表信息，并且通过轮询实现客户端负载均衡。而Fegin只需要定义服务绑定接口且声明式的方法，就可以实现了，简单优雅。

### 10.2Fegin和OpenFegin的区别

![1595142793476](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595142793476.png)

### 10.3使用openfegin

```
之前我们使用的是RestTemplate+ribbon实现请求的封装处理，现在是fegin
```

#### 10.3.1创建模块：cloud_order_feign-80

#### 10.3.2pom

#### 10.3.3yml

```yaml
server:
  port: 80
eureka:
  client:
    register-with-eureka: false
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka, http://eureka7002.com:7002/eureka

```

#### 10.3.4启动类

![1595143054189](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595143054189.png)

#### 10.3.5业务类

**业务逻辑接口+@FeignClient配置调用provider服务**

**新建PaymentFeignService接口并新增注解@FeignClient**

![1595143399968](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595143399968.png)

Contorller

![1595143437897](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595143437897.png)

#### 10.3.6测试

http://localhost/consumer/payment/get/1

![1595143489496](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595143489496.png)

### 10.4openFegin的超时机制：

OpenFegin默认等待时间是1秒，超过1秒直接报错

演示：

服务提供方8001故意写错一下：

![1595149288176](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595149288176.png)

添加在消费方80的端口也:

![1595149367201](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595149367201.png)

消费方80的contorller

![1595149389915](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595149389915.png)

测试：http://localhost/consumer/payment/feign/timeout

![1595149468059](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595149468059.png)

**OpenFeign默认等待一秒钟，超过后报错**

Yml开启配置：

![1595149525743](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595149525743.png)

测试：

![1595149570113](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595149570113.png)

### 10.5OpenFegin日志打印功能

1. Fegin开启了日志打印功能，可以配置Fegin的日志级别，congFegin中了解Http请求的细节。

2. Fegin的日志级别

   ![1595206721827](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595206721827.png)

3. 配置Bean

   ![1595206760051](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595206760051.png)

4. 配置yml

   ![1595206783508](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595206783508.png)

5. 测试：

 http://localhost/consumer/payment/get/1 

![1595206855886](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595206855886.png)

# 4.服务降级

## 11.Hystrix

### 1.Hystrix的概述

#### 1.Hystrix是什么

用于处理在某个分布式调用微服务出现异常，错误等的一个机制。避免一个微服务出错之后，全部出错，把某一个出错的微服务进行包装，返回。不让程序出现很严重的一连串出错。

![1595209849477](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595209849477.png)

#### 2.Hystrix能干嘛

##### 服务熔断

（请求）类比保险丝达到最大服务访问后，直接拒绝访问，拉闸限电，然后调用服务降级的方法并返回友好提示.

服务的降级->进而熔断->恢复调用链路

**当某个服务出现问题,卡死了,不能让用户一直等待,需要关闭所有对此服务的访问**

​			**然后调用服务降级**

##### 服务降级

（再出现错误的时候，不会让整个服务失败）服务器忙，请稍候再试，不让客户端等待并立刻返回一个友好提示，fallback

哪些情况会触发熔断：

程序运行异常，超时，服务熔断触发服务降级，线程池/信号量打满也会导致服务降级

**比如当某个服务繁忙,不能让客户端的请求一直等待,应该立刻返回给客户端一个备选方案**

##### 服务限流

秒杀高并发等操作，严禁一窝蜂的过来拥挤，大家排队，一秒钟N个，有序进行

**限流,比如秒杀场景,不能访问用户瞬间都访问服务器,限制一次只可以有多少请求**

### 2.使用Hystrix

#### 2.1,创建带降级机制的pay模块 :

##### 1.创建模块

cloud-hystrix-pay-8001

##### 2.pom

##### 3.yml

```yaml
server:
  port: 8001


spring:
  application:
    name: cloud-provider-hystrix-payment
eureka:
  client:
    register-with-eureka: true   #表识不向注册中心注册自己
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/
    fetch-registry: true    #表示自己就是注册中心，职责是维护服务实例，并不需要去检索服务
```

##### 4.启动类

![1595211432524](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595211432524.png)

##### 5.Service和Contorller

![1595211499971](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595211499971.png)

![1595211520855](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595211520855.png)

##### 6.测试：

 http://localhost:8001/payment/hystrix/TimeOut/1 （每次调用都会花费几秒时间），

 http://localhost:8001/payment/hystrix/ok/1 

![1595225819248](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595225819248.png)

![1595225800686](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595225800686.png)

都可以访问成功。

##### 7.高并发测试

上面的情况在高并发下就会出现延时，

出现2w个访问，那么两个都会出现转圈等待的状态，

高并发测试结论：8001这个端口会慢，加入在加一个端口那么另外一个端口也会出现等待或者8001这个端口直接死亡。

#### 2.2,创建带降级机制的pay模块 :

##### 1.创建模块

cloud-consumer-fegin-hytrix-order80

##### 2.pom文件和8001pom文件一样

##### 3.yml

```
server:
  port: 80

eureka:
  client:
    register-with-eureka: false   #表识不向注册中心注册自己
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/
    #fetch-registry: true    #表示自己就是注册中心，职责是维护服务实例，并不需要去检索服务

#fegin超时一秒就会出现报错，而程序中有延迟，所以来保证程序正常
ribbon:
  ReadTimeout: 5000
  ConnectTimeout: 5000
```

##### 4.启动类

![1595225687014](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595225687014.png)

##### 5.Service和Contorller

![1595225714349](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595225714349.png)

Cotorller

![1595225734467](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595225734467.png)

##### 6.测试：

 http://localhost/consumer/payment/hystrix/TimeOut/11 （延时几秒）

![1595225835279](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595225835279.png)

 http://localhost/consumer/payment/hystrix/ok/31 

![1595225788567](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595225788567.png)

##### 7.高并发测试

2w个线程访问8001，消费端80微服务再去访问正常的OK微服务8001地址，http://localhost/consumer/payment/hystrix/timeout/31

结果：要么转圈圈等待，要么消费端报超时错误

**高并发的原因：8001同一层次的其他接口服务被困死，因为tomcat线程里面的工作线程已经被挤占完毕，80此时调用8001，客户端访问响应缓慢，转圈圈**

##### 8.解决：

![1595226059907](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595226059907.png)

### 3.使用服务降级

#### 1.8001服务降级

服务降级：设置自身调用超时时间的峰值，峰值内可以正常运行，超过了需要有兜底的方法处理，作服务降级fallback。

##### 1.先开始激活的注解

![1595227422301](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595227422301.png)

##### 2.在可能出现错误的方法上面添加注解@HystrixCommand

@HystrixCommand报异常后如何处理，一旦调用服务方法失败并抛出了错误信息后，会自动调用@**HystrixCommand**标注好的fallbackMethod调用类中的指定方法

![1595227635192](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595227635192.png)

##### 3.测试，设置超时为5秒

![](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\Snipaste_2020-07-20_14-37-58.png)

直接走的是下面的fallback的方法。

#### 2.80消费服务降级

**我们自己配置过的热部署方式对java代码的改动明显，但对@HystrixCommand内属性的修改建议重启微服务**

##### 1.yml配置添加

```java
feign:
  hystrix:
    enabled: true #如果处理自身的容错就开启。开启方式与生产端不一样。
```

##### 2.配置开启注解

![1595230488740](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595230488740.png)

##### 3.Contorller配置服务降级

![1595230544215](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595230544215.png)

测试：

#### 3.重构：

#### 解决重复代码问题

**上面8001和80的代码太冗余了**

1. 降级方法与业务类方法都写在一块，耦合度高
2. 每个业务方法都写一样降级方法，重复代码多

**解决方法：**

**配置一个全局的降级方法，所有方法多可以使用，去走这个降级，至于某些特殊创建**

##### 1.创建一个全局fallback方法

##### 2.使用注解将其定义为全局降级方法（默认降级方法）

![1595233884272](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595233884272.png)

##### 3.测试：

![1595233904345](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595233904345.png)

#### 解决代码耦合的问题

![1595234203308](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595234203308.png)

##### 1 修改Order80的模块

**只需要为Feign客户端定义的接口添加一个服务降级处理的实现类即可实现解耦**

##### 2.一个类实现PaymentHystrixService接口，统一为接口里面的方法进行异常处理

![1595234285975](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595234285975.png)

接口还需要指定一下哪个fallback的类：

![1595234497891](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595234497891.png)

##### 3.yml配置：

```yaml
server:
  port: 80

eureka:
  client:
    register-with-eureka: false   #表识不向注册中心注册自己
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/
    #fetch-registry: true    #表示自己就是注册中心，职责是维护服务实例，并不需要去检索服务

ribbon:
  ReadTimeout: 5000
  ConnectTimeout: 5000

feign:
  hystrix:
    enabled: true #如果处理自身的容错就开启。开启方式与生产端不一样。
```

##### 4测试：

先启动Eureka7001

在启动8001，在启动80

http://localhost/consumer/payment/hystrix/ok/31

在关闭8001

测试： http://localhost/consumer/payment/hystrix/ok/7 

结果：

![1595234407045](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595234407045.png)

走的是实现类的那个方法。

### 4.使用服务熔断：

![1595253254052](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595253254052.png)

#### 1.修改模块

cloud-provider-hystrix-payment8001

#### 2.修改PaymentService

![1595253459794](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595253459794.png)

超过十次请求之内，如果六次时报了，就会开启熔断器

![1595253694087](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595253694087.png)

#### 3.Contorller

![1595253558157](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595253558157.png)

#### 4.测试

正确：http://localhost:8001/payment/circuit/31

![1595253774381](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595253774381.png)

错误：http://localhost:8001/payment/circuit/-31

多次点击错误的请求，

![1595253796919](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595253796919.png)

连正确的都会出现问题。

在10秒中窗口期内,它自己会尝试接收部分请求,发现服务可以正常调用,慢慢的当错误率低于60%,取消熔断，链路回复。

![1595253848378](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595253848378.png)

#### 5.熔断的总结：

##### 熔断的类型：

熔断打开：请求将不再当前服务，走的是fallback，内部设置始终一般为MTTP（平均故障处理时间），当打开时长达到锁设定时钟则熔断打开

熔断关闭：熔断关闭不会对服务进行熔断

熔断半开：部分请求根据规则调用当前服务，如果请求成功且符合规则，则会慢慢打开服务，恢复链路。

##### 熔断的步骤：

![1595290839051](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595290839051.png)

![1595290935317](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595290935317.png)

##### 断路器什么时候开始

当请求次数超过了请求总数阈值，错误百分比阈值达到了一定的比例就hi打开断路器。

![1595291199034](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595291199034.png)

断路器的开启或者关闭的条件：

**当满足一定阈值的时候，在一定时间内请求次数超过一定的阈值。**

**当失败率达到了错误百分比的时候**

**一段时间之后（默认是5秒），这个时候断路器是半开状态，会让其中一个请求进行转发。如果成功，断路器会关闭，若失败，继续开启。**

##### 当断路器打开之后：

![1595291618805](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595291618805.png)

熔断器的一些配置

![1595291655278](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595291655278.png)

![1595291715489](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595291715489.png)

![1595291724797](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595291724797.png)

![1595291733335](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595291733335.png)

##### **熔断整体流程:**

1. 创建HystrixCommand

2. 命令执行

3. 查看缓冲中有没有，有的话就返回

4. 查看断路器有没有打开，

   打开：执行fallback降级，

   没有打开：

   1. 判断线程池资源是否已经满了 ，如果满了就走降级服务fallback

      没有满：Hystrix会根据我们编写的方法采取什么样的方法请求以来服务，是run方法还是构造器方法

      Hystrix会将本次的方法返回值汇报给断路器，熔断器会维护一组数据来判断路器是否开启，来对某个服务进行（请求失败的话）服务降级如果成功的话 那么进入判断是否超时，如果超时出现异常，那么也进入降级方法，最后奖处理的结果返回。

![1595292804408](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595292804408.png)

### 5.Hystrix的监控服务

![1595294823606](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595294823606.png)

#### 1.新建模块仪表盘

cloud-consumer-hystrix-dashboard9001

#### 2.pom

#### 3.yml

#### 4.启动类加上注解@EnableHystrixDashboard

![1595295005163](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595295005163.png)

#### 5.修改pay模块8001都添加一个jar，还需要再启动类加上一个配置

![1595295024941](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595295024941.png)

![1595295039063](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595295039063.png)

#### 6.测试

http://localhost:9001/hystrix，加上要测试的端口

![1595295091533](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595295091533.png)

![1595295144405](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595295144405.png)

#### 7.如何看这个网址

![1595295310446](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595295310446.png)

![1595295287248](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595295287248.png)

![1595295298347](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595295298347.png)

![1595295237142](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595295237142.png)

# 5.服务网关

## 12.Gateway

zuul停更了，目前zuul2还在卡法阶段

### 1.Gateway是什么

![1595301367395](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595301367395.png)

![1595312792901](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595312792901.png)

**gateway之所以性能号,因为底层使用WebFlux,而webFlux底层使用netty通信(NIO)**

### 2.Gateway能干吗

![1595312870440](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595312870440.png)

反向代理，流量控制，熔断，日志监控

### 3.Gateway的特性

![1595313025033](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595313025033.png)

### 4.Gateway与Zuul的区别

![1595313162992](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595313162992.png)

### 5.Zuul的模型

![gateway的6](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\gateway的6.png)

![1595313543627](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595313543627.png)

**WebFlux是什么？**

![1595313620510](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595313620510.png)

### 6.Gateway的三大概念

#### 1.Route(路由)

路由是构建网关的基本模块，它由ID，目标URI，一系列的断言和过滤器组成，如果断言为true则匹配该路由；就好比转发一样，符合某些规则就执行。

#### 2.Predicate（断言）

参考的是java8的java.util.function.Predicate开发人员可以匹配HTTP请求中的所有内容（例如请求头或请求参数），如果请求与断言相匹配则进行路由；就好比路由的条件

#### 3.Filter(过滤)

指的是Spring框架中GatewayFilter的实例，使用过滤器，可以在请求被路由前或者之后对请求进行修改。

![1595313865158](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595313865158.png)

### 7.Gateway的流程

![1595313919885](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595313919885.png)

![1595313995526](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595313995526.png)

**Gateway的核心：路由转发+执行过滤器链**

### 8.搭建工程使用Gateway

#### 1.搭建模块

cloud-gateway-gateway9527

#### 2.pom

![1595314225033](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595314225033.png)

#### 3.yml，添加Gateway的配置

```yaml
server:
  port: 9527
spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      routes:
        - id: payment_routh #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: http://localhost:8001   #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/**   #断言,路径相匹配的进行路由

        - id: payment_routh2
          uri: http://localhost:8001
          predicates:
            - Path=/payment/lb/**   #断言,路径相匹配的进行路由

eureka:
  instance:
    hostname: cloud-gateway-service
  client:
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka

```

#### 4.无业务类，写启动类

![1595314253764](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595314253764.png)

#### 5.测试：

先启动7001，

在启动8001. http://localhost:8001/payment/get/1 ：这个时候可以启动成功

最后开启9527

![1595314323408](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595314323408.png)

#### 6.Gateway配置的另外一种写法

![1595315523073](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595315523073.png)

测试：

![1595315543777](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595315543777.png)

### 9.动态路由：

修改配置yml

![1595317942273](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595317942273.png)

测试：

![1595317991410](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595317991410.png)

### 10.Pridicate断言：

**断言**：外部指定的路径符合断言里面的规则，那么就会跳转到指定路径。

![1595319122814](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595319122814.png)

每次启动都会加载，

![1595319269716](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595319269716.png)

![1595319218824](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595319218824.png)

**After**：可以指定,只有在指定时间后,才可以路由到指定微服务

![1595319585020](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595319585020.png)

测试：

![1595319601264](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595319601264.png)

**Befor**：在什么时间之后

**Between**：在什么时间之间可以访问

**Cookie Route Predicate**：携带Cookie才可以

![1595319871769](D:\SangguiguCloud2020\SpringCloudPhone\1595319871769.png)

没带cookie：**curl和posman一样从相当于是poman一样，postman相当于curl的可视化工具**

![1595319918914](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595319918914.png)

带上cookie

![1595320060160](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595320060160.png)

**Header**：配置

![1595320310609](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595320310609.png)

![1595320253750](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595320253750.png)

**Host**：域名

![1595320476002](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595320476002.png)

![1595320459033](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595320459033.png)

**Method** :指定GET或者Post的请求方式

![1595320635003](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595320635003.png)

### 11.Fitter

过滤器

#### **生命周期：**

![1595322389958](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595322389958.png)

#### **过滤器种类**

全局：GlobalFilter

单一：GatrwayFilter

**与断言类似,比如闲置,请求头,只有特定的请求头才放行,反之就过滤**

![1595322505648](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595322505648.png)

#### 自定义过滤器

![1595322659405](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595322659405.png)

**测试**

： http://localhost:9527/payment/lb?xukai=z3 

![1595322677770](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595322677770.png)

 http://localhost:9527/payment/lb （已经被过滤）

![1595322692104](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595322692104.png)

# 6.服务配置

## 12.Springcloud Config

分布式的问题：**配置的得不到统一**：比如：数据库的这些

![1595472666168](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595472666168.png)

![1595472765557](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595472765557.png)

能干吗？

![1595472876451](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595472876451.png)

https://cloud.spring.io/spring-cloud-static/spring-cloud-config/2.2.1.RELEASE/reference/html/官网地址。

### 1.Config服务端配置与测试

#### 1.在github里面创建一个创库

然后在本地也搭建一个仓库，放入github里面

![1595473491058](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595473491058.png)

![1595473504826](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595473504826.png)

#### 2.创建模块

cloud-config-center-3344

#### 3.pom

#### 4.yml

![1595473624388](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595473624388.png)

配置的一些注意：



#### 5.主启动类

**加上@EnableConfigServer注解**

![1595473663886](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595473663886.png)

#### 6.测试

访问的路径格式：、

![1595474064820](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595474064820.png)

label：分支（branch）

name：服务名

profile：环境

测试可不可以从github里面拉去信息

![1595473744050](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595473744050.png)

### 2.Config客户端配置与测试

1.创建模块

cloud-config-client-3355

2.pom

3.创建bootstarp.yml

![1595474346264](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595474346264.png)

4.业务类，contorller，

![1595474358618](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595474358618.png)

5.启动类

![1595474630909](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595474630909.png)

6.测试

 http://localhost:3355/configInfo 

![1595474648912](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595474648912.png)

成功实现了客户端3355访问SpringCloud Config3344通过GitHub获取配置信息

**问题：每次跟新配置就要重新启动微服务，所以动态**

### 3.Config客户端之动态刷新

#### 1.加入pom

![1595477927206](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595477927206.png)

#### 2.修改YML，暴露监控端口

![1595477939675](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595477939675.png)

#### 3.@RefreshScope业务类Controller修改

![1595477951058](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595477951058.png)

4.测试

先启动3344，在启动3355，在修改配置文件，3344刷新会更改配置，而3355不会，**需要运维人员发送Post请求刷新3355**然后在刷新就可以了

post请求：

![1595478075343](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595478075343.png)

在访问： http://localhost:3355/configInfo 

测试成功

# 7.消息路线

## SpringCloud Bus

 ==Spring Cloud Bus配合Spring Cloud Config使用可以实现配置的动态刷新==

**Bus支持两种消息代理：RabbitMQ和Kafka**

![1595819348552](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595819348552.png)

**为什么叫消息总路线**

![1595819419408](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595819419408.png)

**RabbitMQ环境配置**（另见RabbitMq笔记）

![1595819486973](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595819486973.png)

### SpringCloud Bus动态刷新全局广播

**必须先具备良好的RabbitMQ环境先**

#### 1.新建MODEL

cloud-config-client-3366

#### 2.pom

#### 3.yml

![1595819595702](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595819595702.png)

#### 4.启动类

![1595819606077](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595819606077.png)

#### 5。业务

![1595819617698](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595819617698.png)

#### 6.设计思想

![1595819708752](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595819708752.png)

#### 7.准备

给cloud-config-center-3344配置中心服务端添加消息总线支持

添加依赖

```
<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```

修改yml

![1595819771840](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595819771840.png)

给cloud-config-center-3355配置中心服务端添加消息总线支持

添加依赖

```
<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```

修改yml

![1595819800056](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595819800056.png)

**给cloud-config-center-3366一样配置中心服务端添加消息总线支持**

8.测试：

先全部启动，

1. 修改github上面的版本
2. ![1595819934486](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595819934486.png)
3. 一次发送，处处生效
4. http://config-3344.com/config-dev.yml，http://localhost:3355/configInfo，http://localhost:3366/configInfo 获取配置都刷新

**一次修改，广播通知，处处生效**

![1595820019830](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595820019830.png)

# 8.消息驱动

## Spring Cloud Stream：

```markdown
 # 消息中间件不同，可能对操作起来是不同的，为了避免再去学习另外一种东西，而去浪费时间，所以Stream出现了。可以统一操作消息中间件，（类似于JDBC可以操作不一样的数据库）
```

==屏蔽底层消息中间件的差异，降低切换版本，统一消息的编程模型==

Spring Cloud Stream 目前只支持RabbitMQ,kafka

![1595833649569](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595833649569.png)

https://cloud.spring.io/spring-cloud-static/spring-cloud-stream/3.0.1.RELEASE/reference/html/ ：官网

https://m.wang1314.com/doc/webapp/topic/20971999.html：中文指导手册

==设计思想==：

应用程序与消息中间件的隔离，使微服务高度解耦，也可以动态切换消息中间件。

![1595834443043](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595834443043.png)

==Stream为什么可以统一底层差异==：

![1595834585698](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595834585698.png)

Channel：通道，是队列Queue的一种抽象，在消息通讯系统中就是实现存储和转发的媒介，通过对Channel对队列进行配置

==Binder==	：**绑定器，方便连接中间件，屏蔽差异**

![1595834663116](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595834663116.png)

`Spring Cloud Stream编码和使用注解`

![1595835010752](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595835010752.png)

### 使用Spring Cloud Stream

#### 1.创建生产者

需要三个工程：8801生产者，8802，8803消费者

1.pom

2.yml

![1595857062891](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595857062891.png)

3.业务类

![1595857076671](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595857076671.png)

![1595857085721](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595857085721.png)

4.启动类

5.contorller

![1595857111388](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595857111388.png)

测试： http://localhost:8801/sendMessage ：控制台：

![1595857138807](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595857138807.png)、

#### 2.搭建消费者

1.新建model

cloud-stream-rabbitmq-consumer8802

2.pom

3。yml

![1595859268252](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595859268252.png)

4.启动类

5.业务类

![1595859296222](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595859296222.png)

测试： http://localhost:8801/sendMessage 

8801：![1595859329885](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595859329885.png)

8802：![1595859338654](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595859338654.png)

#### 3.搭建消费者-2

新键modelcloud-stream-rabbitmq-consumer8803

pom

==yml都一样和8802==

#### 重复消费问题

连个消费者出现重复消费问题，就是每个消费者拿到一样的数据，而且数量是一样的。

就好比是Rabbitmq的广播模型：![1595861149126](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595861149126.png)

==一条消息多个消费者消费==

![1595861215835](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595861215835.png)

解决：

1.自定义分组。

修改8802，8802yml文件中新建分组

8802：

![1595861283836](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595861283836.png)

8803：![1595861309431](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595861309431.png)

分组一样，就会拿取生产者总数的消息。

测试：启动8801，8802，8803

8801：![1595861437153](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595861437153.png)

8802：![1595861446627](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595861446627.png)

8803：![1595861458179](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595861458179.png)

选在就平衡了，不会出现重复消费。

#### 数据持久化

当数据持久化了，但是没有被消费的化，那么会一直保持再队列里面，等待有人来消费就会被拿走。

测试，先把8802的的分组移除，再关闭重启，8803不移除分组，重启，

重启发现：8802没有得到消息，而8803拿到了消息。

也就是说：没有被消费的消息会被保留在队列里面。

也就是,当我们没有配置分组时,会出现消息漏消费的问题

​		而配置分组后,我们可以自动获取未消费的数据

# 9.Spring Cloud Alibaba:

==因为SpringCloud Netflix项目进入项目维护，所以就出现了Spring Cloud Alibaba，而Spring Cloud Netflix进入维护也不再更新了。==

支持的哪些功能：

![1595923271627](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595923271627.png)

https://github.com/alibaba/spring-cloud-alibaba/blob/master/README-zh.md下载地址

Spring Cloud Alibaba的一些功能：

![1595923334725](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595923334725.png)

https://spring.io/projects/spring-cloud-alibaba#overview ：Spring Cloud Alibaba官网

https://github.com/alibaba/spring-cloud-alibaba/blob/master/README-zh.md ：中文文档

https://spring-cloud-alibaba-group.github.io/github-pages/greenwich/spring-cloud-alibaba.html：英文

## 1.SpringCloud Alibaba Nacos服务注册和配置中心

**Nacos：Naming Configuration Service**

Nacos：==配置管理和服务管理中心（相当于集成了Eureka和Config和Bus）==

下载地址：https://github.com/alibaba/Nacos

学习文档：https://nacos.io/zh-cn/index.html

 GitHub：https://spring-cloud-alibaba-group.github.io/github-pages/greenwich/spring-cloud-alibaba.html#_spring_cloud_alibaba_nacos_discovery

### 1.安装并且运行：

https://github.com/alibaba/nacos/releases/tag/1.1.4 下载地址

解压安装包，直接运行bin目录下的startup.cmd

测试：http://localhost:8848/nacos  密码：账号都是nacos

![1595923742368](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595923742368.png)

### 2.Nacos作为服务注册中心演示

#### 1.基于Nacos的服务提供者

##### 1.新建Model

##### 2.pom

父pom

![1595923842535](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595923842535.png)

子pom：![1595923881257](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595923881257.png)

##### 3.yml

![1595923900479](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595923900479.png)

##### 4.启动类

![1595923920133](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595923920133.png)

##### 5.业务类

![1595923941527](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595923941527.png)

##### 6.测试：

Nacos页面：![1595923971726](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595923971726.png)

访问： http://localhost:9001/consumer/payment/nacos/13 

![1595924057538](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595924057538.png)

7.新建Model和9001

==配置，pom，yml一样，就是地址不一样，为了节演示nacos的负载均衡==

#### 2.基于Nacos的服务消费者

##### 1.新建Model

cloudalibaba-consumer-nacos-order83

##### 2.pom

`和上面一样`

##### 3.yml

![1595924251874](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595924251874.png)

##### 4.启动类

##### 5.业务类

![1595924262926](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595924262926.png)

测试负载均衡准备

![1595924279862](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595924279862.png)

##### 6.测试

查看naocs：

![1595924317978](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595924317978.png)

测试：http://localhost:83/consumer/payment/nacos/13

![1595924337469](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595924337469.png)

![1595924344601](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595924344601.png)

结论：83访问9001/9002，轮询负载OK，==实现了Rabbin的负载均衡==

#### 3.服务注册中心对比

![1595924452264](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595924452264.png)

![1595924461232](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595924461232.png)

**Nacos支持AP和CP模式的切换**

![1595924482699](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595924482699.png)

### 3.Nacos作为配置中心

##### 1.新建model

cloudalibaba-config-nacos-client3377

##### 2.pom

![1595941058563](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595941058563.png)

##### 3.yml

![1595941101480](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595941101480.png)

为什么要两个yml文件：

![1595941152063](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595941152063.png)

##### 4.启动类

##### 5.contorller业务类

![1595941124736](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595941124736.png)

##### 6.在Nacos中添加配置信息

Nacos中的dataid的组成格式与SpringBoot配置文件中的匹配规则

![1595941199169](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595941199169.png)

操作：

设置DataId的公式：**${spring.application.name}-${spring.profile.active}.${spring.cloud.nacos.config.file-extension}**

![1595941343371](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595941343371.png)

![1595941245548](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595941245548.png)

![1595941303413](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595941303413.png)

##### 7.测试：

启动3377：访问： http://localhost:3377/config/info ：

![1595941404649](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595941404649.png)

而且在nacos中修改文件，自动刷新就可以了，不需要向config那么麻烦

### 4.Nacos配置中心的分类

![1595944772769](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595944772769.png)

Nacos的图形化管理界面：

![1595944814841](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595944814841.png)

==Namespace+Group+Data ID三者关系：==

![1595944835472](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595944835472.png)

![1595944875660](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595944875660.png)

实现配置文件个隔离。

#### DataID方案

指定spring.profile.active和配置文件的DataID来使不同环境下读取不同的配置：就是环境不同访问不同环境

![1595944960169](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595944960169.png)

修改yml文件

![1595944978903](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595944978903.png)

测试：访问  http://localhost:3377/config/info ：

![1595945003040](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595945003040.png)

#### Group方案

新建Group

![1595945110038](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595945110038.png)

![1595945217984](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595945217984.png)

![1595945294687](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595945294687.png)

修改两个yml

![1595945740450](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595945740450.png)

测试：启动：访问

 http://localhost:3377/config/info 

### 5.Linux下安装Nacos

下载Linux版本Nacos： https://github.com/alibaba/nacos/releases/tag/1.1.4 

然后传上虚拟机：解压：` tar -zxvf nacos-server-1.1.4.tar.gz `，

既可以了。

![1595992731798](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1595992731798.png)

## 2.SpringCloud Alibaba Sentinel

### 1.Sentinel概述：

https://github.com/alibaba/Sentinel：官网

Sentinel：轻量级的流量控制，熔断降级java库）

https://github.com/alibaba/Sentinel/releases：下载地址

#### 1.Sentinel能干嘛

![1596024597221](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596024597221.png)

 https://spring-cloud-alibaba-group.github.io/github-pages/greenwich/spring-cloud-alibaba.html#_spring_cloud_alibaba_sentinel ：学习文档

服务使用中的各种问题：服务雪崩，服务降级，服务熔断，服务限流。

![1596024765884](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596024765884.png)

### 2.下载，安装使用Sentinel

1.下载地址：https://github.com/alibaba/Sentinel/releases，下载到本地sentinel-dashboard-1.7.0.jar：

2.8080端口不能被占用，java8环境ok

3.cmd窗口：java -jar sentinel-dashboard-1.7.0.jar 

![1596024884319](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596024884319.png)

4.测试：

登录：http://localhost:8080：

账户密码都是sentinel：

![1596024920537](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596024920537.png)

### 3.使用Sentinel

#### 1.新建工程

cloudalibaba-sentinel-service8401

#### 2，pom

![1596025007644](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596025007644.png)

#### 3.yml

![1596025022632](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596025022632.png)

4.启动类

#### 5.contorller

![1596025036111](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596025036111.png)

#### 6.测试：

==Sentinel采用的懒加载说明==：只用访问的时候才会加载到sentinel

![1596025108456](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596025108456.png)

测试成功。

### 4.Sentinel流控规则

![1596026037170](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596026037170.png)

![1596026088261](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596026088261.png)

#### 1.配置：

==1.流控模式==

![1596026124462](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596026124462.png)

系统默认就是快速失败。

==2.线程模式==

![1596026681094](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596026681094.png)

==3.关联==

：当关联的资源达到阈值时，就限流自己。就是我病了你吃药。

![1596027390904](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596027390904.png)

#### 2.测试：

==1.流控模式测试：==

快速点击访问http://localhost:8401/testA（一秒超过一次）如下：

![1596026200092](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596026200092.png)

==2.线程数测试：==：

访问两个网址： http://localhost:8401/testA  和 http://localhost:8401/testA  ：线程数量为二：

就会出现：![1596026200092](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596026200092.png)

搞清楚==流控模式和线程数模式的区别==

==3.关联：测试==

先测试：

1.先测试 http://localhost:8401/testB看看能不能自己跑通![1596027536084](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596027536084.png)

2.用postman用并发访问http://localhost:8401/testB，然后再去访问http://localhost:8401/testA

![1596027842911](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596027842911.png)

结果：

![1596027809059](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596027809059.png)

#### 3.问题：

==这个报错的方法可不可以自定义，自己写，自己处理后续==

### 5.Sentinel流控效果

#### 1.预热

说明：

公式：阈值除以coldFactor（默认值为3），经过预热时长后才会达到阈值

Warm Up：

限流的一种方式：==冷启动==（类似于秒杀这种活动的限流）

==默认coldFactor为3，即请求QPS从threshold/3开始，经预热时长逐渐升至设定的QPS阈值。==

![1596028834365](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596028834365.png)

测试：

先设置Warm Up

![1596028699891](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596028699891.png)

==5s==内猛点击会出现如下情况：

![1596028721793](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596028721793.png)

==10s==以后就不会直接失败

![1596028741749](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596028741749.png)

#### 2.排队等待

![1596029414432](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596029414432.png)

![1596029374380](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596029374380.png)

测试：http://localhost:8401/testA：

### 6.Sentine服务降级

![1596030821291](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596030821291.png)

==Sentinel的断路器是没有半开状态的==



#### 降级策略实战👇

#### 1.RT

概述：![1596030970590](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596030970590.png)

##### 2.测试

新增代码：

![1596031058178](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596031058178.png)

设置：![1596031085574](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596031085574.png)

##### 3.结论：

每秒大于五个还有在规定时间内处理这些，

![1596109803151](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596109803151.png)

#### 2.异常比例

![1596109850311](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596109850311.png)

代码：

```
@GetMapping("/testD")
    public String testD()
    {

        log.info("testD 测试RT");
        int age = 10/0;
        return "------testD";
    }
 
```

设置：

![1596109904499](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596109904499.png)

结论：

![1596109967980](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596109967980.png)

#### 3.异常数

![1596110034105](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596110034105.png)

代码：

```
@GetMapping("/testE")
public String testE()
{
    log.info("testE 测试异常数");
    int age = 10/0;
    return "------testE 测试异常数";
}
 
 
```

配置：

![1596110053273](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596110053273.png)

结论：

![1596110073830](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596110073830.png)

### 7.Sentinel热点key限流

#### 1.基本介绍

![1596112035863](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596112035863.png)

@SentineResources

#### 2.测试：

配置：

![1596112194244](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596112194244.png)

代码：

![1596112131573](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596112131573.png)

测试连接：

 http://localhost:8401/testHotKey?p1=a ：点击多次的话出现：

![1596112220143](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596112220143.png)

出现服务降级方法。

#### 3.参数例外项

==特殊情况：==我们期望p1参数当它是某个特殊值时，它的限流值和平时不一样

配置：

前提：热点参数的注意点，参数必须是基本类型或者String

![1596112996915](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596112996915.png)前提

注意点添加。

测试：

http://localhost:8401/testHotKey?p1=5：==当p1等于5的时候，阈值变为200==

![1596113060013](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596113060013.png)

http://localhost:8401/testHotKey?p1=1 ==当p1不等于5的时候，阈值就是平常的1==

![1596113043801](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596113043801.png)

总结：==程序出现错误的话那么不会走降级方法。SentinelRescous指挥官配置有没有错误，程序出错直接走RubtimeException==


### 9.@SentinelResource注解

用于配置降解服务。

#### 1.按资源名称限流+后续处理

修改Contorller

![1596114838762](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596114838762.png)

##### 2.修改Sentinel：

![1596114877120](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596114877120.png)

##### 3.测试：

 http://localhost:8401/byRescource：降级方法走：下面的那个方法。

![1596114987848](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596114987848.png)

==注意：关闭8401端口那个流控控制会消失==

#### 2.按照Url地址限流+后续处理 

1.添加contorller

![1596115110148](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596115110148.png)

2.修改Sebtinel

![1596115126203](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596115126203.png)

测试：

 http://localhost:8401/rateLimit/byUrl ：

![1596115151841](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596115151841.png)

#### 3.客户自定义限流处理逻辑

##### 1.新建一个处理限流的类

![1596116416840](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596116416840.png)

##### 2.修改Contorller

![1596116548559](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596116548559.png)

##### 3.修改Sentinel配置：

![1596116567707](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596116567707.png)

##### 4.测试：

 http://localhost:8401/rateLimit/customerBlockHandler ：

![1596116586680](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596116586680.png)

测试成功

### 10.服务熔断功能

==sentinel整合ribbon+openFeign+fallback==

#### 1.Ribbon系列

##### 1.生产者9003、9004

cloudalibaba-provider-payment9003/9004

###### 2.pom

![1596164649729](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596164649729.png)

###### 3.启动类

4yml

###### ![1596164770121](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596164770121.png)务类：

![1596164662500](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596164662500.png)

###### 4.测试：

http://localhost:9003/paymentSQL/1：

![1596164684077](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596164684077.png)

##### 2.消费者

###### 1.新建model

cloudalibaba-consumer-nacos-order84

2.pom

![1596164799630](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596164799630.png)

3.yml

![1596164813663](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596164813663.png)

4.contorller

![1596164828637](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596164828637.png)

![1596165053275](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596165053275.png)

5.启动类

###### 6.测试：

目的：fallback管运行异常，blockHandler管配置违规。

测试地址：http://localhost:84/consumer/fallback/1

当没有@SentinelResource是会出现错误页面。

1.只配置fallback：

![1596165385989](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596165385989.png)

测试：http://localhost:84/consumer/fallback/4

![1596165424135](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596165424135.png)

==走的是fallback==

2.只配置blockHandler

![1596165568849](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596165568849.png)



测试：http://localhost:84/consumer/fallback/4：![1596165590206](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596165590206.png)

设置sebtinel降级：

![1596165613261](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596165613261.png)

每秒点击五次：

![1596165630408](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596165630408.png)

3.fallback和blockHandler都配置：

![1596165798345](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596165798345.png)

![1596165710595](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596165710595.png)

只会走blockHandel。

4.忽略属性...：

![1596165873807](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596165873807.png)

使用了那个的话就没有服务降级。直接报error：

![1596165933475](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596165933475.png)

#### 2.Openfegin

##### 1.pom

![1596181112555](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596181112555.png)

在84模块

##### 2.yml开启fegin

![1596181168175](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596181168175.png)

##### 3.接口和皆苦实现类

![1596181209707](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596181209707.png)

实现类

![1596181220980](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596181220980.png)

##### 4.启动类加上开启fegin注解

##### 5.测试：

84调用9003，此时故意关闭9003微服务提供者，看84消费侧自动降级，不会被耗死

### 11.规则持久化

就是每次关闭服务，sentinel的规则都会消失，那么避免这个问题。

修改cloudalibaba-sentinel-service8401

1.pom

![1596181554106](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596181554106.png)

2.yml

![1596181579812](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596181579812.png)

3.添加Nacos的配置

![1596181601249](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596181601249.png)

![1596181622314](E:\Markdown笔记\SpringCloud尚硅谷\SpringCloud.assets\1596181622314.png)

4.启动8401

5.测试：

http://localhost:8401/rateLimit/byUrl 快速访问接口：出现sentinel的默认降级服务

在停止微服务发现sentinel的规则已经没有了，在启动8401

