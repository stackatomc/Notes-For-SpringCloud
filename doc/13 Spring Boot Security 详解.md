# 13 Spring Boot Security 详解

标签：Spring Cloud

---

## 13.1.1 Spring Security简介

1. Spring Security为JavaEE企业级开发提供了全面的安全防护
2. 提供了细粒度呃但权限控制，可以惊喜到每一个API接口，每一个业务的方法，或者每一个操作数据库的DAO层的方法
3. 特点是对环境的无依赖性、低代码耦合性。Spring Security提供了数十个安全模块，模块与模块间的耦合性低，模块之间可以自由组合案例实现特定需求得安全功能，具有较高得可定制性
4. 安全，“认证”“授权”，采用注解的方式控制权限

---

## 13.3 Spring Boot Security案例详解

1. 引入spring boot security起步依赖
2. 配置Spring Security
2.1 配置WebSecurityConfigurerAdapter，配置如何验证用户信息.用类SecurityConfig继承作为配置，开启功能。简单配置后就实现了用uesrname和password来进行认证，自动生成一个登陆表单，应用得每一个请求都需要认证等
2.2 配置HttpSecurity.上一配置了如何验证用户信息，该步骤用于配置是否所有得用户都需要身份验证？工程得哪些资源要哪些不需要验证，如何知道要支持基于表单得身份验证
2.3 通过controller、Thymeleaf等

## 13.3.4 Spring Security方法级别上的保护

1. 在SecurityConfig上加上@EnableGlobalMethodSecurity注解，开启方法级别的保护，带参数如开启注解prePostEnabled=true等。即可在controller中的转发方法上用@PreAuthorize("hasRole('ADMIN')")或@PreAuthorize("hasAuthority('ROLE_ADMIN')"),@PreAuthorize("hasAnyRole('ADMIN','USER')")或@PreAuthorize("hasAnyAuthority('ADMIN','USER')")