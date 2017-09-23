#### 0.WEB-INF结构:

WEB-INF
-   web.xml(web应用配置,servlet等)
-   tickets-servlet.xml(servlet主要配置)
-   business-config.xml(数据库配置)

<hr />

#### 1.新建business-config.xml文件

<pre lang="xml">
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
       xmlns:mvc="http://www.springframework.org/schema/mvc"  
       xmlns:context="http://www.springframework.org/schema/context"  
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
       xmlns:jpa="http://www.springframework.org/schema/data/jpa"  
       xsi:schemaLocation="  
        http://www.springframework.org/schema/beans  
        http://www.springframework.org/schema/beans/spring-beans.xsd  
        http://www.springframework.org/schema/mvc  
        http://www.springframework.org/schema/mvc/spring-mvc.xsd  
        http://www.springframework.org/schema/context  
        http://www.springframework.org/schema/context/spring-context.xsd  
        http://www.springframework.org/schema/data/jpa  
        http://www.springframework.org/schema/data/jpa/spring-jpa.xsd">  
    <!-- Notice the URL about jpa, it is easy to be ignored. -->  
  
    <mvc:annotation-driven/>  
  
    <context:component-scan base-package="com.tickets"/>  
  
    <!-- About database, JPA and Hibernate config -->  
    <!-- Set dataSource with JPA -->  
    <bean id="dataSource" class="org.apache.tomcat.dbcp.dbcp.BasicDataSource">  
        <property name="driverClassName" value="com.mysql.jdbc.Driver" />  
        <property name="url" value="jdbc:mysql://localhost:3306/tickets?createDatabaseIfNotExist=true&useSSL=false" />  
        <property name="username" value="root" />  
        <property name="password" value="123456" />  
    </bean>  
  
    <!-- Set entityManagerFactory-->  
    <bean id="entityManagerFactory"  
          class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">  
        <property name="dataSource" ref="dataSource" />  
        <!-- Set the entities package to scan -->  
        <property name="packagesToScan" value="com.tickets.business.entities" />  
        <property name="jpaVendorAdapter">  
            <bean class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter">  
            </bean>  
        </property>  
        <!-- Set Hibernate config into jpaProperties-->  
        <property name="jpaProperties">  
            <props>  
                <!-- Hibernate config -->  
                <!-- Dialet could be DIY -->  
                <prop key="hibernate.dialect">org.hibernate.dialect.MySQLDialect</prop>  
                <prop key="hibernate.hbm2ddl.auto">update</prop>  
            </props>  
        </property>  
    </bean>  
  
    <!-- Enable the repositories scan -->  
    <jpa:repositories base-package="com.tickets.business.entities"  transaction-manager-ref="transactionManager" entity-manager-factory-ref="entityManagerFactory"/>  
  
    <bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">  
        <property name="entityManagerFactory" ref="entityManagerFactory" />  
    </bean>  
  
    <bean id="exceptionTranslation"  
          class="org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor">  
    </bean>  
    <!-- End of database config -->  
  
</beans>  
</pre>

#### 2.引入到servlet.xml中

<pre lang="xml">
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
       xmlns:mvc="http://www.springframework.org/schema/mvc"  
       xmlns:context="http://www.springframework.org/schema/context"  
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
       xsi:schemaLocation="  
        http://www.springframework.org/schema/beans  
        http://www.springframework.org/schema/beans/spring-beans.xsd  
        http://www.springframework.org/schema/mvc  
        http://www.springframework.org/schema/mvc/spring-mvc.xsd  
        http://www.springframework.org/schema/context  
        http://www.springframework.org/schema/context/spring-context.xsd">  
  
    <mvc:annotation-driven/>  
  
    <context:component-scan base-package="com.tickets"/>  
  
    <mvc:resources mapping="/imges/**" location="/images/" />  
    <mvc:resources mapping="/css/**" location="/css/" />  
    <mvc:resources mapping="/js/**" location="/js/" />  
  
    <bean id="messageSource" class="org.springframework.context.support.ResourceBundleMessageSource">  
        <property name="basename" value="Messages" />  
    </bean>  
  
    <!-- SpringResourceTemplateResolver automatically integrates with Spring's own -->  
    <!-- resource resolution infrastructure, which is highly recommended.          -->  
    <bean id="templateResolver"  
          class="org.thymeleaf.spring4.templateresolver.SpringResourceTemplateResolver">  
        <property name="prefix" value="/WEB-INF/templates/" />  
        <property name="suffix" value=".html" />  
        <!-- HTML is the default value, added here for the sake of clarity.          -->  
        <property name="templateMode" value="HTML" />  
        <!-- Template cache is true by default. Set to false if you want             -->  
        <!-- templates to be automatically updated when modified.                    -->  
        <property name="cacheable" value="true" />  
    </bean>  
  
    <!-- SpringTemplateEngine automatically applies SpringStandardDialect and      -->  
    <!-- enables Spring's own MessageSource message resolution mechanisms.         -->  
    <bean id="templateEngine"  
          class="org.thymeleaf.spring4.SpringTemplateEngine">  
        <property name="templateResolver" ref="templateResolver" />  
        <!-- Enabling the SpringEL compiler with Spring 4.2.4 or newer can speed up  -->  
        <!-- execution in most scenarios, but might be incompatible with specific    -->  
        <!-- cases when expressions in one template are reused across different data -->  
        <!-- ypes, so this flag is "false" by default for safer backwards            -->  
        <!-- compatibility.                                                          -->  
        <property name="enableSpringELCompiler" value="true" />  
    </bean>  
  
    <bean id="viewResolver" class="org.thymeleaf.spring4.view.ThymeleafViewResolver">  
        <property name="templateEngine" ref="templateEngine" />  
    </bean>  
  
    <!-- Import the config of jpa and hibernate -->  
    <import resource="business-config.xml"/>  
  
</beans>
</pre>