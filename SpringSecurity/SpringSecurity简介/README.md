# SpringSecurity简介

![[Pasted image 20230310071700.png]]

### 工作原理

我们的后端使用的是springboot容器运行在内置的tomcat服务上

一个spring应用最先执行的是过滤器filter，因此在上图中最先执行的是我们的JwtAuthFilter，会对每个请求做一个过滤，并且会按照一定的规则去验证检查请求中的token

1. 首先filter会检查我们所发的请求中是否存在JWT token，如果说没有token，后端直接发送一个403响应response给我们的客户端即可；如果请求中有token，那么就进入接下来的验证流程

