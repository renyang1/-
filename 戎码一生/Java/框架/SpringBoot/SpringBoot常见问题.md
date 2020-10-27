## SpringBoot常见问题

### 项目启动相关

#### 聚合工程中项目正常启动，访问接口时404

​	启动类所在模块为公共基础模块，未依赖接口及业务模块，导致在项目启动时未加载业务模块，相关类未注册到Bean中。

![image-20200902205820036](C:\Users\renyang\AppData\Roaming\Typora\typora-user-images\image-20200902205820036.png)

### 注解相关

#### @MapperScan注解

​	使用该注解可以指定扫描mapper的路径，非必须使用，使用的好处是该路径中的xxxmapper.java类上无需使用@Mapper注解该mapper也能被加载到Bean容器中。