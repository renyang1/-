## SpringBoot中的包扫描问题

`@SpringBootApplication`注解，该注解引入了`@ComponentScan`注解，所以默认的包扫描规则是：程序会自动扫描主启动类所在包及其子包

使用`@ComponentScan`额外指定待扫描的包，但是不能用在主启动类上，因为这样会覆盖掉默认的包扫描规则

#### 如何修改包扫描的位置？

1. 在启动类的SpringBootApplication注解中配置scanBasePackages即可，如下

   ```java
   @SpringBootApplication(scanBasePackages = "org.sang.service")
   ```

   可以配置多个路径

   ```java
   @SpringBootApplication(scanBasePackages = {"org.sang.bean","org.sang.service"})
   ```

2. 在启动类里添加@ComponentScan注解配置basePackages

   ```java
   @ComponentScan(basePackages = {"org.sang.bean","org.sang.service"})
   ```

   **@ComponentScan会覆盖默认的包扫描规则，以上两种选择其一即可。**



