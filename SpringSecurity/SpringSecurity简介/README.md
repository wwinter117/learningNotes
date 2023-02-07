# SpringSecurity简介

## 认证和授权

认证：缺点用户是否能够登陆

授权：判断用户是否有权限去做某些事情


**身份验证的工作原理**

![[Pasted image 20230207141852.png]]



## 相似产品

### shiro

### SpringSecurity和shiro的不同点

# SpringSecurity原理

## 本质

`SpringSecurity` 本质是一个过滤器链，重要的三个如下：

**`FilterSecurityInterceptor`**：方法级的权限过滤器，基本位于过滤器的最底部

**`ExceptionTranslationFilter`**：异常过滤器，用来处理在认证授权过程中抛出的异常

**`UsernamePasswordAuthenticationFilter`**：对 `/login` 的 `post` 请求做拦截，检验表单中用户名和密码

## 过滤器是如何进行加载的？

`SpringBoot` 对 `SpringSecurity` 提供了自动化配置方案，可以用更少的配置来使用 `SpringSecurity` （如果我们不用`SpringBoot`，则需要自己配置过滤器，麻烦）

### 如何使用Springsecurity配置过滤器？

主要通过 `DelegationgFilterProxy`

### UserDetailsService接口

当我们需要自定义登录的判断逻辑时，需要实现该接口

在 `UsernamePasswordAuthenticationFilter` 这个过滤器中，`attemptAuthentication` 方法接收 `post` 请求传过来的用户名和密码，然后进行验证，验证通过则调用 `successfulAuthentication` 方法，失败则调用 `unsuccessfulAuthentication` 方法。

如果现在我们需要用自己的数据库中的用户名和密码去验证，则需要自定义一个类继承 `UsernamePasswordAuthenticationFilter`，然后重写 `attemptAuthentication` 方法，以及    `successfulAuthentication` 方法和方法，而 `attemptAuthentication` 方法只是接收了请求传过来的用户名和密码，查数据库需要写到 `UserDetailsService` 接口中，即**在该接口中去写查数据库中用户名和密码的逻辑代码**

总结分为两步：
1. 创建类继承 `UsernamePasswordAuthenticationFilter`，重写三个方法
2. 创建类实现 `UserDetailsService`，便携查询数据过程，返回 `User` 对象

### PasswordEncoder接口

数据加密接口，用于对返回 `User` 对象里面密码加密

# 入门案例

只需要引入依赖

**为什么在引入 SpringSecurity 之后就无法直接访问接口？**

当程序启动时，`SpringSecurity` 第一件事就是去找安全过滤器链类型的 `Bean`，叫做 `SecurityFilterChain` 

![[Pasted image 20230207142627.png]]

而这个 `Bean` 被注册时：

![[Pasted image 20230207135910.png]]

这也就是为什么我们在引入了 `SpringSecurity` 之后无法直接访问接口，`Spring` 会自动保护所有的后端接口

**那为什么会出现登录页面呢？**

默认会使用 `formLogin` 的样式：

![[Pasted image 20230207144725.png]]


![[Pasted image 20230207140505.png]]

**如何自定义配置类？**

前面说过，当请求过来时，`spring` 第一件事就是去找 `SecurityFilterChain` ，那么我们可以创建一个类，使用 `@EnableWebSecurity` 来标识当前类是 `Security` 的配置类，替代 `Spring` 提供的 `defaultSecurityFilterChain`，然后在自定义的配置类中使用 `@Bean` 来注册一个 `SecurityFilterChain` 

![[Pasted image 20230207145319.png]]

我们使用了 `httpBasic` 的样式之后变成了弹窗：

![[Pasted image 20230207145035.png]]

**使用 postman 向后端发请求**

当直接发送时，我们没有提供任何授权信息，返回401

![[Pasted image 20230207145905.png]]
可以这样提供授权信息，如下图

![[Pasted image 20230207150016.png]]
![[Pasted image 20230207150153.png]]
但是该方式存在许多漏洞，我们可以使用 `JWT` 来替代

# JWT

## 原理

![[Pasted image 20230207151223.png]]

1. 客户端发送请求，想要访问后端 `API` 
2. 请求被 `JwtAuthFilter` 这个过滤器拦截
3. `JwtAuthFilter` 首先会检查请求中是否存在 `JWT`
4. 然后 `JwtAuthFilter` 会向 `UserDetailsService` 请求用户信息，`UserDetailsService` 再去内存或者数据库等地方取用户信息，最终用户详细信息会被传回 `JwtAuthFilter` 
5. `JwtAuthFilter` 会进行信息比对
6. 通过之后，会更新安全上下文信息持有者 `SecurityContexHolder` ，然后调用 `FilterChain` 
7.  `SecurityContexHolder` 会设置用户已通过身份验证，这样当我们传递到下一个过滤器时，系统才知道用户已通过身份验证
8. 最后 `SecurityContexHolder` 会将请求转发给 `DispatcherServlet` ...

## 实现

引入依赖

```xml
<dependency>  
    <groupId>io.jsonwebtoken</groupId>  
    <artifactId>jjwt</artifactId>  
    <version>0.9.1</version>  
</dependency>
```

创建一个 `JwtAuthFilter` ，用来拦截每一个请求，检查请求是否含有 `Jwt`

```java
package cn.wwinter.springsecurity02_jwt.config;  
  
import cn.wwinter.springsecurity02_jwt.service.JwtService;  
import lombok.RequiredArgsConstructor;  
import org.springframework.stereotype.Component;  
import org.springframework.web.filter.OncePerRequestFilter;  
  
import javax.servlet.FilterChain;  
import javax.servlet.ServletException;  
import javax.servlet.http.HttpServletRequest;  
import javax.servlet.http.HttpServletResponse;  
import java.io.IOException;  
  
/**  
 * 过滤器，用于检查请求中是否包含JWT token  
 * 
 * @Author: zhangdongdong  
 * @CreateTime: 2023-02-07  
 */
@Component  
@RequiredArgsConstructor // 用于生成包含 final 和 @NonNull 注解的成员变量的构造方法  
public class JwtAuthFilter extends OncePerRequestFilter {  
  
    private JwtService jwtService;  
  
    /**  
     *     
     * @param: [request, response, filterChain]  
     * @return: void  
     **/    @Override  
    protected void doFilterInternal(  
            HttpServletRequest request,  
            HttpServletResponse response,  
            FilterChain filterChain) throws ServletException, IOException {  
  
        // 当我们进行一次调用时，需要在标头中传递JWT，该变量用于提取这个标头  
        final String authHeader = request.getHeader("Authorization");  
        final String jwt;  
        final String userEmail;  
  
        // 开始检查JWT是否存在  
        if (authHeader == null || !authHeader.startsWith("Bearer ")) {  
            // 不存在，则继续执行剩下的过滤器  
            filterChain.doFilter(request, response);  
            return;  
        }  
  
        // 标头中有Jwt，提取出来（从"Bearer"之后开始）  
        jwt = authHeader.substring(7);  
  
        // TODO: 从JWT中提取出userEmail  
        userEmail = jwtService.extractUsername(jwt);  
  
          
    }  
      
      
      
}
```

`JwtService`

```java


package cn.wwinter.springsecurity02_jwt.service;  
  
import io.jsonwebtoken.Claims;  
import io.jsonwebtoken.Jwts;  
import io.jsonwebtoken.io.Decoders;  
import io.jsonwebtoken.security.Keys;  
import org.springframework.stereotype.Service;  
  
import java.security.Key;  
import java.util.function.Function;  
  
/**  
 * @Author: zhangdongdong  
 * @CreateTime: 2023-02-07  
 */
@Service  
public class JwtService {  
  
    private static final String SECRET_KEY = "423F4528482B4D6251655368566D597133743677397A24432646294A404E6352";  
  
    public String extractUsername(String token) {  
        return extractClaim(token, Claims::getSubject);  
    }  
  
    /**  
     *  取出token中的单个声明字段  
     */  
    public <T> T extractClaim(String token, Function<Claims, T> claimsResolver) {  
  
        Claims allClaims = extractAllClaims(token);  
  
        return claimsResolver.apply(allClaims);  
    }  
  
    /**  
     *  取出token中所有的声明字段  
     */  
    private Claims extractAllClaims(String token) {  
        return Jwts  
                .parser()  
                .setSigningKey(getSignInKey()) // SigningKey密钥用于创建Jwt的签名部分，用于验证Jwt的发送者信息，并确保没有被更改过  
                .parseClaimsJws(token)  
                .getBody();  
    }  
  
    /**  
     * 生成token  
     */    
    public String generateToken() {  
  
        return null;  
    }  
  
    /**  
     *  获得SigningKey  
     */    
     private Key getSignInKey() {  
        // 对我们的密钥进行解码  
        byte[] keyBytes = Decoders.BASE64.decode(SECRET_KEY);  
        // 使用hmacShaKeyFor登录算法  
        return Keys.hmacShaKeyFor(keyBytes);  
    }
}

```
 ``
引入新的依赖，从Jwt token中提取用户信息

```xml
<dependency>  
    <groupId>io.jsonwebtoken</groupId>  
    <artifactId>jjwt-api</artifactId>  
    <version>0.11.5</version>  
</dependency>  
<dependency>  
    <groupId>io.jsonwebtoken</groupId>  
    <artifactId>jjwt-impl</artifactId>  
    <version>0.11.5</version>  
</dependency>
<dependency>  
    <groupId>io.jsonwebtoken</groupId>  
    <artifactId>jjwt-jackson</artifactId>  
    <version>0.11.5</version>  
</dependency>
```



`SigningKey` 密钥用于创建 Jwt 的签名部分，用于验证 Jwt 的发送者信息，并确保没有被更改过，与 `Jwt` 标头中指定的登录算法结合使用


# web权限方案

**如何设置登陆的用户名和密码？**：

有三种方式可以设置用户名和密码

第一种方式可以通过写配置文件

```properties
spring.security.user.name=wwinter
spring.security.user.password=123456
```

第二种方式可以通过配置类

```java
package cn.wwinter.springsecurity_demo.config;  
  
import org.springframework.context.annotation.Bean;  
import org.springframework.context.annotation.Configuration;  
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;  
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;  
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;  
import org.springframework.security.crypto.password.PasswordEncoder;  
  
/**  
 * @Author zhangdongdong  
 * @Date 2023/2/7  
 */
@Configuration  
public class SecurityConfig extends WebSecurityConfigurerAdapter {  
    @Bean  
    PasswordEncoder passwordEncoder() {  
        return new BCryptPasswordEncoder();  
    }  
    
    @Override  
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {  
        // 对密码进行加密  
        BCryptPasswordEncoder passwordEncoder = new BCryptPasswordEncoder();  
        String password = passwordEncoder.encode("123456");  
        
        // 配置用户名和加密后的密码  
        auth.inMemoryAuthentication().withUser("wwinter").password(password).roles("admin");  
    }  
}
```

还可以通过自定义编写实现类

创建配置类，设置使用 `userDetailsService` 实现类
编写实现类，返回 `User` 对象，`User` 对象有用户名、密码和操作权限


# 微服务权限方案




