# 15 使用Spring Security OAuth2和JWT保护微服务系统

标签：Spring Cloud

---

## 15.1 JWT简介

1. 上一章讲述的是通过Spring Security OAuth2保护SPring Cloud架构的微服务系统，但是缺陷是每次请求都需要经过Uaa服务区验证当前Token的合法性，并只需要查询该Token对应的用户的权限。在高并发场景下，会存在性能瓶颈，概删的方法是将Uaa服务集群部署并加上缓存。
2. 这里针对这种缺点，采用Spring Scurity OAuth2和JWT的方式，避免每次请求都需要远程调度Uaa服务。采用Spring Security OAuth2和JWT的方式，Uaa服务只验证一次，返回JWT。返回的JWT包含了用户的所有信息，包括权限信息。
