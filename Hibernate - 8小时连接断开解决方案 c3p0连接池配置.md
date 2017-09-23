#### 1.pom.xml引入依赖

<pre lang="xml">
<dependency>  
    <groupId>org.hibernate</groupId>  
    <artifactId>hibernate-c3p0</artifactId>  
    <version>/*hibernate版本一致*/</version>  
</dependency>
</pre>

#### 直接配置entityManagerFactory中的属性

<pre lang="xml">
<bean id="entityManagerFactory" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
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
            <!-- Dialect could be DIY -->
            <prop key="hibernate.dialect">com.tickets.business.MySQL5Dialect</prop>
            <prop key="hibernate.hbm2ddl.auto">update</prop>

            <!-- Use c3p0 as connection pool -->
            <prop key="hibernate.connection.driver_class">com.mysql.jdbc.Driver</prop>
            <prop key="hibernate.connection.url">jdbc:mysql://localhost:3306/tickets?createDatabaseIfNotExist=true&amp;useSSL=false</prop>
            <prop key="hibernate.connection.username">root</prop>
            <prop key="hibernate.connection.password">123456</prop>

            <prop key="hibernate.connection.provider_class">org.hibernate.c3p0.internal.C3P0ConnectionProvider</prop>
            <prop key="hibernate.c3p0.min_size">5</prop>
            <prop key="hibernate.c3p0.max_size">20</prop>
            <prop key="hibernate.c3p0.timeout">120</prop>
            <prop key="hibernate.c3p0.idle_test_period">3000</prop>
        </props>
    </property>
</bean>

</pre>