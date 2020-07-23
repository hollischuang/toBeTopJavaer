H2是一个开源的嵌入式（非嵌入式设备）数据库引擎，它是一个用Java开发的类库，可直接嵌入到应用程序中，与应用程序一起打包发布，不受平台限制。

H2与Derby、HSQLDB、MySQL、PostgreSQL等开源数据库相比，H2的优势为：
* Java开发，不受平台限制；
* H2只有一个jar包，占用空间小，适合嵌入式数据库；
* 有web控制台，用于管管理数据库。

接下来介绍Spring+Mybatis+H2的数据库访问实践，参考：https://blog.csdn.net/xktxoo/article/details/78014739

添加H2数据库依赖：

```
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>1.4.190</version>
</dependency>
```


      
H2数据库属性文件配置如下，本文采用内存模式访问H2数据库：
``` 
driver=org.h2.Driver
# 内存模式
url=jdbc:h2:mem:testdb;MODE=MYSQL;DB_CLOSE_DELAY=-1
# 持久化模式
#url= jdbc:h2:tcp://localhost/~/test1;MODE=MYSQL;DB_CLOSE_DELAY=-1
```

H2数据库访问的Spring配置文件为：

``` 
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns="http://www.springframework.org/schema/beans"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:jdbc="http://www.springframework.org/schema/jdbc"
       xsi:schemaLocation="
            http://www.springframework.org/schema/beans
                http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
            http://www.springframework.org/schema/tx
                http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
            http://www.springframework.org/schema/jdbc
                http://www.springframework.org/schema/jdbc/spring-jdbc-3.0.xsd">

    <!-- 引入属性文件 -->
    <bean id="propertyConfigurer" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="locations">
            <list>
                <value>classpath:config.properties</value>
            </list>
        </property>
    </bean>

    <!-- 自动扫描DAO -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.xiaofan.test" />
    </bean>

    <!-- 配置Mybatis sqlSessionFactory -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="configLocation" value="classpath:mybatis_config.xml"/>
        <property name="mapperLocations" value="classpath:user_mapper.xml"/>
    </bean>

    <!-- 配置数据源 -->
    <bean id="dataSource"
          class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="${driver}" />
        <property name="url" value="${url}" />
        <!--<property name="username" value="sa" />-->
        <!--<property name="password" value="123" />-->
    </bean>

    <!-- 初始化数据库 -->
    <jdbc:initialize-database data-source="dataSource" ignore-failures="DROPS">
        <jdbc:script location="classpath:sql/ddl.sql" />
        <jdbc:script location="classpath:sql/dml.sql" />
    </jdbc:initialize-database>

    <!-- 配置事务管理 -->
    <tx:annotation-driven transaction-manager="transactionManager" proxy-target-class="true"/>
    <bean id="transactionManager"
          class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

</beans>
```

初始化数据库的DDL语句文件为：
```
CREATE TABLE `user` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(100) NOT NULL,
  `age` int(11) NOT NULL,
  PRIMARY KEY (`id`)
);
```

初始化数据库的DML语句文件为：
```
insert into `user` (`id`,`name`,`age`) values (1, 'Jerry', 27);
insert into `user` (`id`,`name`,`age`) values (2, 'Angel', 25);
```

编写测试文件，如下：

```java
/**
 * Created by Jerry on 17/7/30.
 */
@ContextConfiguration(locations = {"classpath:config.xml"})
@RunWith(SpringJUnit4ClassRunner.class)
public class Test extends AbstractJUnit4SpringContextTests{

    @Resource
    UserDAO userDAO;

    @org.junit.Test
    public void testInsert() {

        int result = userDAO.insert(new User(null, "LiLei", 27));

        Assert.assertTrue(result > 0);
    }

    @org.junit.Test
    public void testUpdate() {
        int result = userDAO.update(new User(2L, "Jerry update", 28));

        Assert.assertTrue(result > 0);
    }

    @org.junit.Test
    public void testSelect() {
        User result = userDAO.findByName(new User(null, "Jerry", null));

        Assert.assertTrue(result.getAge() != null);
    }

    @org.junit.Test
    public void testDelete() {
        int result = userDAO.delete("Jerry");

        Assert.assertTrue(result > 0);
    }

}
```