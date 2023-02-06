SpringBoot+Mybatis+SpringSecurity实现用户权限数据库管理，角色和用户的关系通过数据库配置控制

具体代码演示

Gitee：[springboot-in-action](https://gitee.com/wwinter117/springboot-in-action)
Github：[springboot-in-action](https://github.com/wwinter117/springboot-in-action)

## 验证流程

大致逻辑：拦截登录请求，去数据库查询信息，然后验证

## 数据库MySQL

建立一个数据库`security_demo01`

![[Pasted image 20230206192432.png]]

总共有三张表：用户表、角色表以及用户角色关系表

```SQL
# 建立三个表：t_user， t_role， t_role_user
CREATE TABLE t_user(id INT PRIMARY KEY auto_increment, username VARCHAR(20) NOT NULL, password VARCHAR(200));
CREATE TABLE t_role(id INT PRIMARY KEY  auto_increment, name VARCHAR(20) NOT NULL);
CREATE TABLE t_role_user(id INT PRIMARY KEY auto_increment, user_id INT, role_id INT);

# 插入数据
insert into t_user (id,username, password) values (1,'admin', 'admin');
insert into t_user (id,username, password) values (2,'alice', 'alice');

insert into t_role(id,name) values(1,'role_admin');
insert into t_role(id,name) values(2,'role_user');

insert into t_role_user(user_id,role_id) values(1,1);
insert into t_role_user(user_id,role_id) values(2,2);
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

```java
package cn.wwinter.SpringSecurityDemo01.conf;  
  
import com.mchange.v2.c3p0.ComboPooledDataSource;  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.context.annotation.Bean;  
import org.springframework.core.env.Environment;  
  
import java.beans.PropertyVetoException;  
  
public class DBconfig {  
    @Autowired  
    private Environment env;  
  
    @Bean(name="dataSource")  
    public ComboPooledDataSource dataSource() throws PropertyVetoException {  
        ComboPooledDataSource dataSource = new ComboPooledDataSource();  
        dataSource.setDriverClass(env.getProperty("ms.db.driverClassName"));  
        dataSource.setJdbcUrl(env.getProperty("ms.db.url"));  
        dataSource.setUser(env.getProperty("ms.db.username"));  
        dataSource.setPassword(env.getProperty("ms.db.password"));  
        dataSource.setMaxPoolSize(20);  
        dataSource.setMinPoolSize(5);  
        dataSource.setInitialPoolSize(10);  
        dataSource.setMaxIdleTime(300);  
        dataSource.setAcquireIncrement(5);  
        dataSource.setIdleConnectionTestPeriod(60);  
        return dataSource;  
    }  
}
```

扫描mapper.xml文件

```java
package cn.wwinter.SpringSecurityDemo01.conf;  
  
import org.mybatis.spring.SqlSessionFactoryBean;  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.context.ApplicationContext;  
import org.springframework.context.annotation.Bean;  
  
import javax.sql.DataSource;  
  
public class MyBatisConfig {  
  
    @Autowired  
    private DataSource dataSource;  
    
    @Bean(name = "sqlSessionFactory")  
    public SqlSessionFactoryBean sqlSessionFactory(ApplicationContext applicationContext) throws Exception {  
        SqlSessionFactoryBean sessionFactory = new SqlSessionFactoryBean();  
        sessionFactory.setDataSource(dataSource);  
        // sessionFactory.setPlugins(new Interceptor[]{new PageInterceptor()});  
        sessionFactory.setMapperLocations(applicationContext.getResources("classpath*:mapper/*.xml"));  
        return sessionFactory;  
    }
}
```

dao扫描器

```java
package cn.wwinter.SpringSecurityDemo01.conf;  
  
import org.mybatis.spring.mapper.MapperScannerConfigurer;  
import org.springframework.context.annotation.Bean;  
import org.springframework.context.annotation.Configuration;  
  
@Configuration  
public class MyBatisScannerConfig {  
    @Bean  
    public MapperScannerConfigurer MapperScannerConfigurer() {  
        MapperScannerConfigurer mapperScannerConfigurer = new MapperScannerConfigurer();  
        mapperScannerConfigurer.setBasePackage("cn.wwinter.SpringSecurityDemo01.conf.dao");  
        mapperScannerConfigurer.setSqlSessionFactoryBeanName("sqlSessionFactory");  
        return mapperScannerConfigurer;  
    }  
}
```

开启事务

```java
package cn.wwinter.SpringSecurityDemo01.conf;  
  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.context.annotation.ComponentScan;  
import org.springframework.context.annotation.Configuration;  
import org.springframework.jdbc.datasource.DataSourceTransactionManager;  
import org.springframework.transaction.PlatformTransactionManager;  
import org.springframework.transaction.annotation.TransactionManagementConfigurer;  
  
import javax.sql.DataSource;  
  
@Configuration  
@ComponentScan  
public class TransactionConfig implements TransactionManagementConfigurer {  
    @Autowired  
    private DataSource dataSource;  
  
    @Override  
    public PlatformTransactionManager annotationDrivenTransactionManager() {  
        return new DataSourceTransactionManager(dataSource);  
    }  
}
```


## 代码实现

### controller层

### service层

### mapper层

### java bean

