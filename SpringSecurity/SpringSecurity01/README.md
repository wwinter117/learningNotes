SpringBoot+Mybatis+SpringSecurity实现用户权限数据库管理，角色和用户的关系通过数据库配置控制

## 验证流程

大致逻辑：拦截登录请求，去数据库查询信息，然后验证

## 表的设计

总共有三张表：用户表、角色表以及用户角色关系表

```SQL
# 插入测试数据
insert into SYS_USER (id,username, password) values (1,'admin', 'admin');
insert into SYS_USER (id,username, password) values (2,'abel', 'abel');

insert into SYS_ROLE(id,name) values(1,'ROLE_ADMIN');
insert into SYS_ROLE(id,name) values(2,'ROLE_USER');

insert into SYS_ROLE_USER(SYS_USER_ID,ROLES_ID) values(1,1);
insert into SYS_ROLE_USER(SYS_USER_ID,ROLES_ID) values(2,2);
```

## SpringBoot引入依赖

父级起步依赖（提供相关的maven默认依赖，以及版本信息）：

```xml
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-parent</artifactId>
<version>1.3.0.RELEASE</version>
```

其他的版本信息：

```xml
<mybatis.version>3.2.7</mybatis.version> 
<mybatis-spring.version>1.2.2</mybatis-spring.version>
```

引入依赖：

```xml
<dependencies>
        <!--springboot-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <dependency>
            <groupId>org.thymeleaf.extras</groupId>
            <artifactId>thymeleaf-extras-springsecurity4</artifactId>
        </dependency>
        
        <!--db-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>6.0.5</version>
        </dependency>
        <dependency>
            <groupId>com.mchange</groupId>
            <artifactId>c3p0</artifactId>
            <version>0.9.5.2</version>
            <exclusions>
                <exclusion>
                    <groupId>commons-logging</groupId>
                    <artifactId>commons-logging</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <!--mybatis-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>${mybatis.version}</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>${mybatis-spring.version}</version>
        </dependency> 
</dependencies>
```

写一个启动类：

```java
package cn.wwinter.SpringSecurityDemo01;  
  
import org.mybatis.spring.annotation.MapperScan;  
import org.springframework.boot.SpringApplication;  
import org.springframework.boot.autoconfigure.SpringBootApplication;  
  
/**  
 *  启动类  
 *  
 * @Author zhangdongdong  
 * @Date 2023/2/6  
 */
@SpringBootApplication  
@MapperScan("cn.wwinter.SpringSecurityDemo01.mapper")  
public class SpringSecurityDemo01 {  
    public static void main(String[] args) {  
        SpringApplication.run(SpringSecurityDemo01.class, args);  
    }
}
```
	
配置文件信息：

```
ms.db.driverClassName=com.mysql.jdbc.Driver
ms.db.url=jdbc:mysql://localhost:3306/cache?characterEncoding=utf-8&useSSL=false
ms.db.username=root
ms.db.password=admin
ms.db.maxActive=500

logging.level.org.springframework.security= INFO
spring.thymeleaf.cache=false

```

## Mybatis配置


通过配置类配置数据源

扫描mapper.xml文件

dao扫描器

事务管理

## 代码实现

### controller层

### service层

### mapper层

### java bean

