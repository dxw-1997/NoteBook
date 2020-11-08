

# Spring

##### 一：注解的上下文配置

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

</beans>
~~~

##### 二：spring的注解

```java
@Autowired //使用类型进行自动装配
@Scope//限制作用域，默认是单列模式
@Configuration//这是一个配置类，相当于我们原来的xml配置文件
@Bean //相当于我们原来的一个bean【id】属性
@Aspect //标明这是一个切面aop
@Componet//组件，表示把这个组件加入到容器中
	mvc三层架构
        dao 【@Repository】
        service【@service】
        controller【@Controller】
```

1.后面三个是compenet的衍生，功能完全一样，不同的层用不同的注解更规范，都是把组件加入到spring容器中去

2.使用注解必须导入aop包，开启注解

##### 三：xml与注解的比较

1.xml管理Bean

2.注解完成属性注入（实战推荐）

3.xml适用于任何场合，结构清晰，维护方便

4.注解维护不方便

##### 四：动态代理

1.动态代理分为两大类

​			基于接口--jdk

~~~java
package com.dai.demo;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class ProxyInvocationHandler implements InvocationHandler {
    private Object target;

    public void setTarget(Object target) {
        this.target = target;
    }

    public Object getProxy(){
        return Proxy.newProxyInstance(this.getClass().getClassLoader(),target.getClass().getInterfaces(),this);
    }
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        return method.invoke(target, args);
    }
}
//生成动态代理的工具类
~~~

【动态代理代理的是接口】

使用AOP要导入的包

~~~xml
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.9.6</version>
        </dependency>
~~~

AOP的xml配置

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd
         http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">
    <context:component-scan base-package="com.dai"/>
    <context:annotation-config/>
</beans>
~~~

```xml
<aop:aspectj-autoproxy/> //开启aop注解，使用aop注解必须开启aop注解属性
```

**执行（\* com.sample.service.impl .. \*。\*（..））**

**解释如下：**

| **符号**                    | **含义**                                                 |
| --------------------------- | -------------------------------------------------------- |
| **执行（）**                | **表达式的主体;**                                        |
| **第一个” \*“符号**         | **表示返回值的类型任意;**                                |
| **com.sample.service.impl** | **AOP所切的服务的包名，即，我们的业务部分**              |
| **包名后面的” ..“**         | **表示当前包及子包**                                     |
| **第二个” \*“**             | **表示类名，\*即所有类。此处可以自定义，下文有举例**     |
| **。\*（..）**              | **表示任何方法名，括号表示参数，两个点表示任何参数类型** |

2.mybatis的工具类

~~~java
package com.dai.util;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.InputStream;

public class MybatisUtil {
    private static SqlSessionFactory sqlSessionFactory =null;
    static {

        try {
            String resource = "mybatis-config.xml";//自己的配置文件信息
            InputStream inputStream = null;
            inputStream = Resources.getResourceAsStream(resource);
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    public static SqlSession getSession(){
        return sqlSessionFactory.openSession();
    }
}
//mybatis的工具类
~~~

mybatis的配置文件

~~~xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <properties resource="db.properties"/>
    <typeAliases>
        <typeAlias type="com.dai.pojo.Person" alias="Person"/>
        <typeAlias type="com.dai.pojo.Teacher" alias="Teacher"/>
        <typeAlias type="com.dai.pojo.Student" alias="Student"/>
    </typeAliases>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="com/dai/mapper/PersonMapper.xml"/>
        <mapper class="com.dai.mapper.TeacherMapper"/>
        <mapper class="com.dai.mapper.StudentMapper"/>
    </mappers>
</configuration>
~~~

##### 五：spring整合mybatis

导入的maven依赖

~~~xml
    <dependencies>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.2</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.49</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.2.9.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>5.2.9.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.9.6</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>2.0.2</version>
        </dependency>
    </dependencies>
~~~

整合mybatis配置文件

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=true&amp;useUnicode=true&amp;characterEncoding=utf-8"/>
        <property name="username" value="root"/>
        <property name="password" value="123456"/>
    </bean>
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="configLocation" value="classpath:mybatis-config.xml"/>
        <property name="mapperLocations" value="classpath:com/dai/mapper/*.xml"/>
    </bean>
    <bean id="sessionTemplate" class="org.mybatis.spring.SqlSessionTemplate">
        <constructor-arg index="0" ref="sqlSessionFactory"/>
    </bean>
</beans>
~~~

##### 六：aop整合事务

配置事务的xml文件

~~~xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <tx:method name="*"/>
        </tx:attributes>
    </tx:advice>
    <aop:config>
        <aop:pointcut id="txPointcut" expression="execution(* com.dai.mapper.*.*(..))"/>
        <aop:advisor advice-ref="txAdvice" pointcut-ref="txPointcut"/>
    </aop:config>
~~~

