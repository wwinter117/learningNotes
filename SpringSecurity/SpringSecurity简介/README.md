# SpringSecurity简介

## 认证和授权

认证：缺点用户是否能够登陆

授权：判断用户是否有权限去做某些事情

## 相似产品

### shiro

### SpringSecurity和shiro的不同点

# SpringSecurity原理

## 本质

SpringSecurity本质是一个过滤器链，重要的三个如下：

**FilterSecurityInterceptor**：方法级的权限过滤器，基本位于过滤器的最底部

**ExceptionTranslationFilter**：异常过滤器，用来处理在认证授权过程中抛出的异常

**UsernamePasswordAuthenticationFilter**：对/login的post请求做拦截，检验表单中用户名和密码

## 过滤器是如何进行加载的？

SpringBoot对SpringSecurity提供了自动化配置方案，可以用更少的配置来使用SpringSecurity（如果我们不用SpringBoot，则需要自己配置过滤器，麻烦）

### 如何使用Springsecurity配置过滤器？

主要通过`DelegationgFilterProxy`

### UserDetailsService接口

用于自定义登录的判断逻辑，需要实现该接口

在UsernamePasswordAuthenticationFilter这个过滤器中，attemptAuthentication方法接受post请求传过来的用户名和密码，然后进行验证，验证通过则调用successfulAuthentication方法，失败则调用方法successfulAuthentication。

如果现在我们需要用自己的数据库中的用户名和密码去验证，则需要自定义一个类继承UsernamePasswordAuthenticationFilter，然后重写attemptAuthentication方法，以及successfulAuthentication方法和unsuccessfulAuthentication方法，而attemptAuthentication方法只是接收了请求传过来的用户名和密码，查数据库需要写到UserDetailsService接口中，即**在该接口中去写查数据库中用户名和密码的逻辑代码**

总结分为两步：
1. 创建累继承UsernamePasswordAuthenticationFilter，重写三个方法
2. 创建类实现UserDetailsService，便携查询数据过程，返回User对象

### PasswordEncoder接口

数据加密接口，用于对返回User对象里面密码加密

# 微服务权限方案




