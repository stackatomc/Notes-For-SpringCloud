# 5 服务注册和发现Eureka

标签：Spring Cloud

---

## 5.1 Eureka简介

1. Eureka是用于服务注册和发现的组件，最开始主要应用于亚马逊公司旗下的云计算服务平台AWS.Eureka分为Eureka服务注册中心，Eureka Client为Eureka客户端。
2. 特点：完全开源，是Netflix公司的开源产品。Eureka和其他组件，比如负载均衡组件Ribbon、熔断器组件Hystrix、熔断器监控组件HystrixDashboard组件、熔断器聚合监控Turbine组件，以及网管Zuul组件相互配合，所以很容易实现服务注册、负载均衡、熔断和只能路由等功能。这些组件都是由Netflix公司开源的，一起被成为NetflixOSS组件。Netflix OSS组件由Spring Cloud整合为Spring Cloud Netflix组件，是Spring Cloud构架微服务的核心组件，也是基础组件
3. eureka的基本架构：register service服务注册中心；provider service服务提供者；consumer service服务消费者。后两个都是Eureka Client

>测试eureka server中抛出异常之一及解决方法：
>Description:
Cannot determine embedded database driver class for database type NONE
Action:
If you want an embedded database please put a supported one on the classpath. If you have database settings to be loaded from a particular profile you may need to active it (no profiles are currently active).
这是因为spring boot默认会加载org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration类，DataSourceAutoConfiguration类使用了@Configuration注解向spring注入了dataSource bean。因为工程中没有关于dataSource相关的配置信息，当spring创建dataSource bean因缺少相关的信息就会报错。
解决方法：注意在启动类加上，@EnableAutoConfiguration(exclude={DataSourceAutoConfiguration.class})

### 5.4 源码解析Eureka

1. Eureka的一些概念：register服务注册、renew服务续约(30s发送心跳)、fetch registeries获取服务注册列表信息(可以使用JSON和XML数据格式进行通信)、Cancel服务下线(DiscoveryManager.getInstance().shutdownComponent())、Eviction服务剔除(当eureka client连续90s没有向eureka server发送服务续约即心跳，服务注册中心将会将该服务实例从服务注册列表删除，即服务剔除)
2. eureka的高可用架构：实现是对eureka server集群，当一个服务实例被剔除时，该服务注册列表信息和服务续约信息会被复制到每个eureka server节点
3. 源码主要分析Eureka Client端，主要是DiscoveryClient类，追溯是InstanceInfoReplicator在run方法中执行D的register方法，而I在D初始化时调用。而eureka-server则主要有BootstrapContext类，在程序启动时具有最先初始化的权限，调用PeerAwareInstanceRegistryImpl和PeerUerkaNodes类均有关于高可用，该方法有一个register提供服务注册，并且将服务注册后的信息同步到其他的eureka server服务中。并且这里eureka client通过http向eureka server注册，所以eureka server也会提供哦那个一个 服务注册的API接口给eureka client，位于Alt+左键查看该register方法，最终被ApplicationResource类的addInstance()方法即服务注册的接口调用。
4. renew()等同理，从DiscoveryClient处开始分析

### 5.5 高可用的Eureka Server集群

1. 可以使用application-{profile}.properties配置不同的开发环境。还可以用yml多profile的格式
2. 尝试了指定profile用maven打包启动.命令为：mvn spring-boot:run Dspring.profiles.active=peer
3. 尝试了第二种执行过程，用yml多profiles时，命令可分布进行编译mvn clean package->进入target->java -jar eureka-server.1.0-SNAPSHOT.jar - -spring.profiles.active=peer1
