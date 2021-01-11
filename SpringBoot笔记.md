# SpringBoot笔记

> Author: Ying
>
> Version: 9.0.0

## 一、SpringBoot介绍

### 1.1、引言

> * 为了使用SSM框架去开发，准备SSM框架的模板配置
> * 为了Spring整合第三方框架，单独地去编写xml文件
> * 导致SSM项目后期xml文件特别多，维护xml文件的成本是很高的
> * SSM工程部署很麻烦，要依赖第三方的容器
> * SSM开发方式很笨重

### 1.2、SpringBoot介绍

> SpringBoot由Pivota团队研发，本身不是新技术，而是将之前常用的Spring、SpringMVC、data-jpa等常用的框架封装到了一起，帮助你隐藏这些框架的整合细节，实现敏捷开发。
>
> SpringBoot就是一个工具集。

---

> SpringBoot特点：
>
> * SpringBoot项目不需要模板化的配置
> * SpringBoot中整合第三方框架时，只需要导入相应的starter依赖包，就自动整合了
> * SpringBoot默认只有一个.properties的配置文件，不推荐使用xml，后期会采用.java的文件去编写配置信息
> * SpringBoot工程在部署时，采用的是jar包的方式，内部自动依赖Tomcat容器，提供了多环境的配置
> * 后期要学习的微服务框架SpringCloud需要建立在SpringBoot的基础上





## 二、SpringBoot快速入门

### 2.1、快速构建SpringBoot

> Idea中创建SpringBoot项目，初始源可以选择https://start.aliyun.com
>
> 创建完成后，修改pom.xml，dependencies的artifactId结尾加上-web，并更新所需依赖
>
> 编写Controller
>
> 启动初始生成的Application，然后就可以在浏览器访问了

> 需要注意，第一次配置Maven项目，需要修改下载源为阿里云仓库

### 2.2、SpringBoot的目录结构

> 1. pom.xml文件
>    1. 指定了一个父工程：指定当前工程为SpringBoot，帮助我们声明了starter依赖的版本
>    2. 项目的元数据：包名，项目名，版本号
>    3. 指定了properties信息：指定了java的版本为1.8
>    4. 导入依赖：默认情况下导入spring-boot-starter，spring-boot-starter-test
>    5. 插件：spring-boot-maven-plugin

---

> 2. .gitignore文件：默认忽略的文件和目录

---

> 3. src目录

```
-src
  -main
  	-java
  		-包名
  		  启动类.java			  # 需要将controller类，放在启动类的子包中或者同级包下
    -resources					
      -static					# 存放静态资源
      -templates				# 存放模板页面
      application.properties	# SpringBoot提供的唯一的配置文件
  -test		# 测试用
```

### 2.3、SpringBoot的三种启动方式

> 1、运行启动类的main方法即可运行SpringBoot工程

---

> 2、采用maven的命令来运行SpringBoot工程
>
> mvn spring-boot:run

---

> 3、采用jar包的方式运行
>
> * 将当前项目打包成一个jar文件
>
>   ​     mvn clean package
>
> * 在powershell中通过java -jar jar文件路径 即可



## 三、SpringBoot常用注解

### 3.1、@Configuration和@Bean

> SpringBoot不推荐使用xml文件
>
> @Configuration注解相当于beans标签
>
> @Bean注解相当于bean标签
>
> ​     id = "方法名 | 注解中的name属性(优先级更高)"
>
> ​     class = "方法的返回结果的类型"

```java
import com.example.demo01.entity.User;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class UserConfig {

    @Bean
    public User user(){
        User u = new User();
        u.setId(1);
        u.setName("张三");
        return u;
    }

    /*
    * <beans>
        <bean id="user" class="com.example.demo01.entity.User" />
      </beans>
    * */
}
```

### 3.2、@SpringBootApplication

> @SpringBootApplication是一个组合注解，具体如下

---

> 1、@SpringBootConfiguration就是@Configuration注解，代表启动类就是一个配置类

---

> 2、@EnableAutoConfiguration帮助实现自动装配，当SpringBoot工程启动时，会运行一个SpringFactoriesLoader的类，加载META-INF/spring.factories配置类(已经开启的)，通过SpringFactoriesLoader中的load方法，以For循环的方式，逐个加载。
>
> 优点：无需编写大量的整合配置信息，只需要按照SpringBoot提供好的约定去整合即可
>
> 缺点：假设你导入了一个starter依赖，那么你就需要填写它必要的配置信息
>
> 手动关闭指定内容的自动装配：
>
> ​	以关闭Quartz为例：@SpringBootApplication(exclude = QuartzAutoConfiguration.class)

---

> 3、@ComponentScan就相当于<context:component-scan basePackage="包名" />，用于扫描注解

## 四、SpringBoot常用配置

### 4.1、SpringBoot的配置文件格式

> SpringBoot的配置文件支持properties和yml，还有json
>
> 推荐使用yml文件格式：
>
> 优势：
>
> 1. 可以根据换行和缩进帮助管理配置文件
> 2. 相比properties更轻量

### 4.2、多环境配置

> 在application.yml文件中添加一个配置项：
>
> spring:
>
>   profiles:
>
> ​    active: 环境名

---

> 在resource目录下，创建多个application-环境名.yml文件即可
>
> 比如application-dev.yml，application-prod.yml

---

> 在部署工程时，通过 java -jar jar文件路径 --spring.profiles.active=环境名    即可指定部署时使用哪个环境

### 4.3、引入外部配置文件信息

> 和传统的SSM方式一样，通过@Value的注解去获取properties或yml配置文件中的内容

----------------------------

> 如果在yml配置文件中需要编写大量的自定义配置，并且具有统一的前缀时，采用如下方式

```java
@ConfigurationProperties(prefix="aliyun")
@Component
@Data
public class AliyunProperties {
    
    private String xxxx;
    ...
        
}
```

```yml
# yml配置文件
aliyun:
  xxxx: some-content
  ...
```

### 4.4、热加载

> 1、导入依赖

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-devtools</artifactId>
	<optional>true</optional>
</dependency>
```

> 2、修改settings中的配置

![image-20210105102918651](https://i.loli.net/2021/01/11/2qXItiQKRgyDPEC.png)

> 3、修改内容后，直接build即可，不用重启项目工程

![image-20210105102852930.png](https://i.loli.net/2021/01/11/GbUZe9EkBnNAmph.png)

## 五、SpringBoot整合MyBatis

> 1、导入依赖

```xml
<!--        MySQL驱动-->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>

<!--        druid连接-->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.1.10</version>
</dependency>

<!--        mybatis-->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.3.2</version>
</dependency>
```



> 2、编写配置文件
>
> 连接数据库，自动生成实体类

![image-20210105110035154.png](https://i.loli.net/2021/01/11/g4J63LHXrhWCT8d.png)

```java
// 自动生成实体类后，修改其包路径
package com.example.demo01.entity;

public class Difficultydegree {

  private long difficultyId;
  private String description;


  public long getDifficultyId() {
    return difficultyId;
  }

  public void setDifficultyId(long difficultyId) {
    this.difficultyId = difficultyId;
  }


  public String getDescription() {
    return description;
  }

  public void setDescription(String description) {
    this.description = description;
  }

}
```

---

```java
// 准备Mapper接口
public interface DegreeMapper {

    List<Difficultydegree> findAll();
}

// 在启动类中添加注解，扫描Mapper接口所在的包
@MapperScan(basePackages = "com.example.demo01.mapper")
```

---

```xml
// 准备映射文件
<mapper namespace="com.example.demo01.mapper.DegreeMapper">

<!--    List<Difficultydegree> findAll();-->
    <select id="findAll" resultType="Difficultydegree">
        select * from difficultydegree
    </select>

</mapper>
```

```yml
# 修改yml文件配置
# mybatis配置
mybatis:
  mapper-locations: classpath:mapper/*.xml # 扫描映射文件
  type-aliases-package: com.example.demo01.entity # 配置别名扫描的包
  configuration:
    map-underscore-to-camel-case: true # 开启驼峰映射配置
    
# 连接数据库的信息
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql///problemdatabase
    username: root
    password: root
    type: com.alibaba.druid.pool.DruidDataSource
```



> 3、测试
>
> 进入mapper类文件，右键,goto-test，创建测试类

```java
class DegreeMapperTest extends Demo01ApplicationTests {

    @Autowired
    private DegreeMapper degreeMapper;

    @Test
    void findAll() {
        List<Difficultydegree> list = degreeMapper.findAll();
        for (Difficultydegree degree : list) {
            System.out.println(degree);
        }
    }
}
```

### 5.2、注解方式整合Mybatis

> 1、创建Mapper接口

```java
public interface PTypeMapper {

    List<Problemtype> findAll();

}
```

> 2、添加Mybatis注解
>
> 针对增删改查：@Insert, @Delete, @Update, @Select
>
> 仍旧需要在启动类中添加@MapperScan注解

```java
@Select("select * from problemtype")
List<Problemtype> findAll();

@Select("select * from problemtype where id = #{id}")
Problemtype selectById(@Param("id") Integer id);
```

> 3、测试，并显示执行的sql语句

```yml
# 设置显示执行的SQL语句
logging:
  level:
    com.example.demo01.mapper: DEBUG
```

```java
class PTypeMapperTest extends Demo01ApplicationTests {

    @Autowired
    private PTypeMapper mapper;

    @Test
    void findAll() {
        List<Problemtype> list = mapper.findAll();
        for (Problemtype problemtype : list) {
            System.out.println(problemtype);
        }
    }

    @Test
    void selectById() {
        Problemtype pt = mapper.selectById(2);
        System.out.println(pt.getDescription());
    }
}
```



### 5.3、SpringBoot整合分页助手

> 1、 导入依赖

```xml
<!--        PageHelper分页助手-->
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>1.2.10</version>
</dependency>
```

> 2、测试使用

```java
 @Test
    public void findByPage() {
        // 1、执行分页
        PageHelper.startPage(1, 2);

        // 2、执行查询
        List<Difficultydegree> list = mapper.findAll();

        // 3、封装PageInfo对象
        PageInfo<Difficultydegree> pageinfo = new PageInfo<>(list);

        // 4、输出结果
        for (Difficultydegree difficultydegree : pageinfo.getList()) {
            System.out.println(difficultydegree);
        }

    }
```





## 六、SpringBoot整合JSP







