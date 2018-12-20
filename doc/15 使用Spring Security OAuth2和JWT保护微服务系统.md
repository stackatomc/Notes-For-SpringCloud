# 15 使用Spring Security OAuth2和JWT保护微服务系统

标签：Spring Cloud

---

## 15.1 JWT简介

1. 上一章讲述的是通过Spring Security OAuth2保护SPring Cloud架构的微服务系统，但是缺陷是每次请求都需要经过Uaa服务区验证当前Token的合法性，并只需要查询该Token对应的用户的权限。在高并发场景下，会存在性能瓶颈，概删的方法是将Uaa服务集群部署并加上缓存。
2. 这里针对这种缺点，采用Spring Scurity OAuth2和JWT的方式，避免每次请求都需要远程调度Uaa服务。采用Spring Security OAuth2和JWT的方式，Uaa服务只验证一次，返回JWT。返回的JWT包含了用户的所有信息，包括权限信息。
3. JWT标准意在将各个主体的信息包装为JSON对象。主题信息是通过数字签名进行加密验证的，尝试用HAMC算法或者RSA（公钥私钥的非对称性加密）算法对JWT进行签名，安全性很高
4. JWT的结构：由3个部分组成，Header头.Payload有效载荷.Signature签名。头（令牌类型JWT和使用的算法类型）、Payload（包含了用户的一些信息和Claim保留、公开、私人权利/声明）、Signature（用Base64编码后的Header、Payload和密钥进行签名）
5. JWT的使用场景：最常见的是认证，一旦用户登陆成功获取JWT后，后续的每个请求将携带该JWT，该JWT包含了用户信息、权限点等信息，根据该JWT包含的信息，资源服务可以控制该JWT可以访问的资源范围。因为JWT的开销很小，并且能够在不同域中使用，单点登陆是一个广泛使用JWT的场景。另外，还用于信息交换，由于JWT使用签名加密，安全性很高，另外根据Signature还可以验证内容是否被篡改
6. JWT认证流程：浏览器登陆username+password请求获取JWT，服务器判断无误后经过用私钥加密返回以JWT的形式返回给客户端。以后的每次请求中，获取到该JWT的客户端都需要携带该JWT(在Header加JWT)，而服务器检查JWT用公钥解密获取用户信息给客户端相应。这样做的好处就是以后的请求都不需要通过Uaa服务来判断该请求的用户以及该用户的权限。所以在为服务系统中，可以利用JWT实现单点登陆。