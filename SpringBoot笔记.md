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







## 四、SpringBoot常用配置









## 五、SpringBoot整合MyBatis







## 六、SpringBoot整合JSP







