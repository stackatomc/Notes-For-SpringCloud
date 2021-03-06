# 14 使用Spring Cloud OAuth2保护微服务系统

标签：Spring Cloud

---

## 14.1 什么是OAuth2

1. 在认证和授权的过程中，主要包括以下3中角色。服务提供方Authorization Server、资源持有者Resource Server、客户端Client。
2. 认证流程是用户（资源持有者）打开客户端，客户端询问用户授权。用户统一授权。客户端向授权服务申请授权。授权服务器对客户端进行人获赠们也包括用户信息的认证，认证成功后授权基于令牌。客户端获取令牌后，携带令牌向资源服务器请求资源。资源服务器确认令牌正确无误，向客户端释放资源。

---

## 14.2 如何使用Spring OAuth2

1. Spring OAuth2分为两部分，分别是OAuth2 Provider和OAuth2 Client
2. OAuth2 Provider用于配置代表用户的OAuth2客户端信息，被用户允许的客户端就可以访问被OAuth2保护的资源。OAuth2 Provider通过管理和验证OAuth2令牌来控制客户端是否有权限访问被其保护起来的资源。OAuth2 Provider的角色分为Authorization Service授权服务和Resource Service资源服务，通常他们不再同一个服务中，可能一个A对应多个R，Spring OAuth2需要配合Spring Security一起使用，所有的请求由SpringMVC控制器处理，并经过一系列的Spring Security过滤器
3. Spring Security过滤器链中两个节点，这两个节点是向Authorization Service获取验证和授权的。授权节点:默认为/oauth/authorize，获取Token节点：默认为/oauth/token
4. Authorization Server：（1）ClientDetailsServiceConfigurer配置客户端信息(clientId、secret、scope、authorizedGrantTypes、authorities，客户端信息可以存储在数据库中，这样就可以通过更改数据库来实时更新客户端信息的数据。Spring OAuth2已经设计好了数据库的表，且不可变。创建数据库的脚本书上已提供)、（2）AuthorizationServerEndpointsConfigurer配置授权Token的节点和Token服务，默认开启所有验证类型除了密码验证(authenticationManger配置才能开启密码认证、userDetailsService获取用户认证信息的接口、authorizationCodeServices配置验证码服务、implicitGrantService配置管理implict验证的状态、tokenGranter配置Token Granter)、Token的管理状态（支持3中InMemoryTokenStore、JdbcTokenStore即Token存储在数据库中需要引入spring-jdbc加上上一届的数据库脚本、JwtTokenStore采用JWT形式，这种形式没有做任何的存储，因为JWT本身包含了用户验证的所有信息，不需要存储，但需要引入spring-jwt的依赖）、（3）AuthorizationServerSecurityConfigurer（当资源服务和授权服务在同一个服务中，用默认配置即可。如果采用remoteTokenServices资源服务器的每次请求所携带的Token都需要从授权服务做校验，这时还需要配置/oauth/check_token校验节点的校验策略）
5. Resource Server是提供了受OAuth2保护的资源，这些资源为API接口、Html页面、Js文件等。Spring OAuth2提供了实现次保护功能的SPring Security认证过滤器，使用@Configuration、@EnableResourceServer注解，开启Resource Server功能，也可以使用ResrouceServerConfigurer配置tokenServices设置Token如何编码和节码的。当Resource Server和Authorization Server在同一个工程上，则不需要配置此选项。若不在同一个，也可以用RemoteTokenService类
3. OAuth2 Client客户端，用户访问被OAuth2保护起来的资源，客户端需要提供用于存储用户的授权码和访问令牌的机制，需要配置如下两个选项Protected Resource Configuration、Client Configuration客户端配置
4. Protected Resource Configuration(Id、clientId、clientSecret、accessTokenUri、scope、clientAuthenticationScheme、userAuthorizationUri)、Client Configuration（可以使用@EnableOAuth2Client注解来简化配置，另外还需要配置1 创建一个过滤器Bean，Id为oauth2ClientContextFilter用来存储当前请求和上下文的请求，2还需要在Request域内创建AccessTokenRequest类型的Bean）

---

## 14.3 案例分析

1. 为auth-serivce、service-hi。serviec-hi调用auth-service获取Token返回给浏览器，浏览器以后所有的请求都需要携带该Token。但缺点是每次请求都需要资源服务内部远程调度auth-service服务来验证Token的正确性，以及该Token对应得以哦那过户所具有的权限，额外多了一次内部请求。如果在高并发的情况下，auth-service需要集群部署，必能且需要做缓存处理。