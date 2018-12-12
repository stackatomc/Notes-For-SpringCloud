# 5 服务注册和发现Eureka

标签：Spring Cloud

---

## 5.1 Eureka简介

1. Eureka是用于服务注册和发现的组件，最开始主要应用于亚马逊公司旗下的云计算服务平台AWS.Eureka分为Eureka服务注册中心，Eureka Client为Eureka客户端。
2. 特点：完全开源，是Netflix公司的开源产品。Eureka和其他组件，比如负载均衡组件Ribbon、熔断器组件Hystrix、熔断器监控组件HystrixDashboard组件、熔断器聚合监控Turbine组件，以及网管Zuul组件相互配合，所以很容易实现服务注册、负载均衡、熔断和只能路由等功能。这些组件都是由Netflix公司开源的，一起被成为NetflixOSS组件。Netflix OSS组件由Spring Cloud整合为Spring Cloud Netflix组件，是Spring Cloud构架微服务的核心组件，也是基础组件