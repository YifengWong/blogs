## Spring 涉及结构

<pre lang="shell">
- pom.xml
- src
    - main
        - resources
            - settings-development.properties #配置文件内容
        - webapp
            - WEB-INF
                - web.xml #配置默认启动项
                - xxx-servlet.xml #配置profile文件

</pre>

## properties文件内容

1. 定义key=value
2. key将被xml文件引用，因此应当注意命名统一
3. 不同配置文件应当含有相同key值。


<pre lang="shell">
db.user=root
db.password=123456
db.jdbcUrl=jdbc:mysql://localhost:3306/database?createDatabaseIfNotExist=true&useSSL=false
</pre>

## xxx-servlet.xml 文件配置beans

1. profile=""即为该配置文件
2. classpath指向了resource资源文件夹


<pre lang="xml">
<!-- 开发环境配置文件 -->
<beans profile="development">
    <context:property-placeholder location="classpath:settings-development.properties"/>
</beans>

</pre>

## web.xml指定默认启动项

<pre lang="xml">
<!-- 配置spring的默认profile -->
<context-param>
    <param-name>spring.profiles.default</param-name>
    <param-value>development</param-value>
</context-param>
</pre>

## 使用

#### 1. 相关配置文件使用，如servlet.xml（spring-mvc.xml）中，使用${}占位引用变量

<pre lang="xml">
<!-- About database, JPA and Hibernate config -->
<!-- Set dataSource with JPA -->
<bean id="dataSource" class="org.apache.tomcat.dbcp.dbcp.BasicDataSource">
    <property name="driverClassName" value="com.mysql.jdbc.Driver" />
    <property name="url" value="${db.jdbcUrl}" />
    <property name="username" value="${db.user}" />
    <property name="password" value="${db.password}" />
</bean>
</pre>

#### 2. Junit中调用不同上下文

1. ContextConfiguration 指定上下文配置
2. ActiveProfiles指定了默认的激活profile

<pre lang="java">
@RunWith(SpringJUnit4ClassRunner.class)
@WebAppConfiguration
@ContextConfiguration({"file:src/main/webapp/WEB-INF/xxx-servlet.xml"})
@ActiveProfiles(profiles = "development")
public abstract class TestBase {
}
</pre>

#### 3. mvn命令指定profile

1. 使用-D指定参数key=value
2. 使用-P直接指定profile


<pre lang="shell">
mvn clean package tomcat7:run -Dspring.profiles.active=development
# 或
mvn clean package tomcat7:run -P development
</pre>