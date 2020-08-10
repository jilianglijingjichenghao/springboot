# 一、配置文件

## 1、获取yml文件(自动提示)

```java
@Component
@ConfigurationProperties(prefix="persion")
public class Persion {
    private String name;
    private Integer age;
    private Dog dog;
    private Map<String,Object> maps;
    private List<String> lists;

    public Dog getDog() {
        return dog;
    }
```

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-configuration-processor</artifactId>
   <optional>true</optional>
</dependency>
```

``` yml
persion:
    name: zhangsan
    age: 18
    maps: {k1: 11,k2: 33}
    lists:
      - kl
      - klk
      - sadf
      - dsf
    dog:
      name: 小狗
      age: 3
```

## 2、获取properties文件

``` properties
persion.name=张三顶顶顶
persion.age=12
persion.maps.k1=12
persion.maps.k2=op
persion.dog.name=小狗
persion.dog.age=12
persion.lists=a,dsf,df

```

获取中文乱码，是因为properties文件时ascii码,需要设置idea运行编码

<img src="C:\Users\JL\AppData\Roaming\Typora\typora-user-images\image-20200603111844078.png" alt="image-20200603111844078"  />

## 3、@ConfigurationProperties 和@Value区别

|                      | @ConfigurationProperties | @Value     |
| -------------------- | ------------------------ | ---------- |
| 功能                 | 批量注入配置文件中属性   | 一个个指定 |
| 松散绑定（驼峰命名） | 支持                     | 不支持     |
| SpEl                 | 不支持                   | 支持       |
| JSR303数据校验       | 支持                     | 不支持     |
| 复杂类型封装         | 支持                     | 不支持     |

## 4、@PropertySource和@ImportResource

### 4.1、@PropertySource 注入自己写的配置文件

```java
@PropertySource(value = {"classpath:persion.properties"})
```

### 4.2、@ImportResource 注入spring配置文件（主程序类上）

```java
@ImportResource(locations = {"classpath:beans.xml"})
```

springboot 推荐使用 @Bean，默认id为返回方法名

``` java
@Configuration
public class DemoApplication {
	@Bean
	public Hello Hello2(){
		System.out.println("给容器添加组件。。。。");
		return new Hello();
	}
}
```

## 5、占位符的使用

``` properties
persion.name=zhangsan${random.uuid}
persion.age=${random.int}
persion.maps.k1=12
persion.maps.k2=op
persion.dog.name=${persion.name}_小狗
persion.dog.age=${persion.agea:23}
persion.lists=a,dsf,df
```

${persion.agea:23} 如果找不到属性，可以给一个默认值

## 6、profile

### 1、多profile文件

文件名可以是 application${profile}.properties/yml

### 2、yml支持多文档块方式

``` yml
server:
  port: 8090
spring:
  profiles:
    active: prod
---
server:
  port: 8087
spring:
  profiles: dev
---
server:
  port: 8889
spring:
  profiles: prod

```



### 3、激活指定profile

1、在配置文件中指定 spring.profiles.active=prod

2、以命令行方式

```
--spring.profiles.active=dev
```

 3、虚拟机参数

``` 
-Dspring.profiles.active=dev
```

## 7、配置文件加载位置

-file:./config/

-file:./

-classpath:./config/

-classpath:./

优先级由高到低，高优先级会覆盖低优先级文件

springboot 会加载四个位置的主配置文件，**互补配置**

```properties
spring.config.location=D:/application.properties #打包的时候才会生效
server.servlet.context-path=/demo
```

# 二、 日志框架

## 1、spring boot 选用

SLF4j和logback

## 2、怎么使用

``` java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HelloWorld {
  public static void main(String[] args) {
    Logger logger = LoggerFactory.getLogger(HelloWorld.class);
    logger.info("Hello World");
  }
}
```

## 3、设置日志输出界别

``` java
Logger log = LoggerFactory.getLogger(getClass());
    @Test
    void contextLoads() {
		// 日志输出级别，由低到高
        log.trace("这是 trace 信息");
        log.debug("这是 debug 信息");
        log.info("这是 info 信息");
        log.warn("这是 warn 信息");
        log.error("这是 error 信息");
    }
```

```properties
logging.level.com.example=trace
logging.file.name=E:/spring.log
#当前磁盘下，创建spring/log文件夹，使用spring.log默认日志
logging.file.path=/spring/log
#控制台日志输出格式
#%d{yyyy-MM-dd日期、%thread线程、%-5level级别、%logger{50}50个字符、%msg日志消息、%n换行
logging.pattern.console=%d{yyyy-MM-dd}[%thread] %-5level %logger{50} - %msg%n
#文件中日志输出格式
logging.pattern.file=%d{yyyy-MM-dd} == [%thread] == %-5level == %logger{50} == %msg%n
```

## 4、自定义日志

logback.xml

logback-spring.xml (高级设置)可以根据环境不同输出日志

```xml
<configuration>
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    　　　<encoder>
         <springProfile name="dev">
             <pattern>%d{yyyy-MM-dd}--->[%thread]---> %-5level %logger{50}---> %msg%n</pattern>
         </springProfile>
        <springProfile name="!dev">
            <pattern>%d{yyyy-MM-dd}===[%thread]=== %-5level %logger{50}===  %msg%n</pattern>
        </springProfile>
    　　　</encoder>
    </appender>
    <root level="INFO">
        <appender-ref ref="STDOUT" />
    </root>
    　　　　
</configuration>
```

# 三、web开发

## 1、thymeleaf

 ### 1.1、 pom添加

``` xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

 ###  1.2、将页面放在 templates

 ###  1.3、语法

页面提示添加 <html lang="en" xmlns:th="http://www.thymeleaf.org">

``` properties
Variable Expressions: ${...} 
Selection Variable Expressions: *{...} 
Message Expressions: #{...} 
Link URL Expressions: @{...}
Fragment Expressions: ~{...}
```

## 2、webjars(其他webjars，百度webjars添加maven)

pom 添加

```xml
  <dependency>
            <groupId>org.webjars</groupId>
            <artifactId>bootstrap</artifactId>
            <version>4.5.0</version>
        </dependency>
```

引用

 th:href="@{/webjars/bootstrap/4.5.0/css/bootstrap.css}" 

 th:src="@{/image/bootstrap-solid.svg}"

优点：换项目名称后，会自动添加项目名称

## 3、国际化

### 3.1、添加国际化文件

```xml
spring.messages.basename=i18n.login
```

页面引用

```jsp
th:text="#{login.name}、 [[#{login.remember}]]
```

### 3.2、国际化切换

实现 LocaleResolver 接口

``` java
public class MyLocaleResolver implements LocaleResolver {

    @Override
    public Locale resolveLocale(HttpServletRequest request) {
        String l = request.getParameter("l");
        Locale locale = Locale.getDefault();
        if (!StringUtils.isEmpty(l)) {
            String[] ls = l.split("_");
            locale = new Locale(ls[0],ls[1]);
        }
        return locale;
    }

    @Override
    public void setLocale(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Locale locale) {

    }
}

   @Bean
    public LocaleResolver localeResolver (){
       return new MyLocaleResolver();

   }
```

``` jsp
<a class="btn-sm" th:href="@{/login.html(l='zh_CN')}">中文</a>
<a class="btn-sm" th:href="@{/login.html(l='en_US')}">英文</a>
```

## 4、登录

### 4.1 禁用页面缓存

```xml
spring.thymeleaf.cache=false 页面ctrl+f9 重新加载页面
```

### 4.2 页面消息验证

```jsp
<p style="color: red" th:text="${msg}" th:if="${not #strings.isEmpty(msg)}"></p>
```

### 4.3 登录拦截

```java
public class LoginHandlerInter implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        Object user = request.getSession().getAttribute("loginUser");
        if (null == user){
            request.setAttribute("msg","你没有权限请先登录");
            request.getRequestDispatcher("/login.html").forward(request,response);
            return false;
        } else{
            return true;
        }

    }
    //注册
     public void addInterceptors(InterceptorRegistry registry) {
               registry.addInterceptor(new LoginHandlerInter()).addPathPatterns("/**").
                       excludePathPatterns("/","/login.html","/user/login");
           }
```

## 5 列表页面

### 5.1公共页面抽取

**th:insert** ：公共内容插入，有标签

**th:replace**：公共内容替换，没有标签div

**th:include**: 公共内容包含



```html
th:fragment="topbar"----- th:replace="dashboard::topbar"
id="sidebarMenu"    ----- th:replace="~{dashboard::#sidebarMenu}"
```

### 5.2 高亮显示点击菜单

```jsp
th:class="${activeUri=='emps'?'nav-link active':'nav-link'}">
引用
th:replace="common/bar::#sidebarMenu(activeUri='emps')"
```

### 5.3 日期格式化

```properties
spring.mvc.format.date=yyyy-MM-dd
```

页面展示

``` html
<tr th:each="emp:${emps}">
    <td th:text="${emp.id}"></td>
    <td th:text="${emp.name}"></td>
    <td th:text="${emp.sex}==0?'男':'女'"></td>
    <td th:text="${emp.email}"></td>
    <td th:text="${emp.department.depment}"></td>
    <td th:text="${#dates.format(emp.birth, 'yyyy-MM-dd')}"></td>
    <td>
        <a class="btn btn-sm btn-primary" th:href="@{/emp/}+${emp.id}">编辑</a>
        <a class="btn btn-sm btn-danger" th:href="@{/emp/}+${emp.id}">删除</a>
    </td>
</tr>
```



## 6、添加删除

```html
<a class="btn btn-sm btn-primary" th:href="@{/emp/}+${emp.id}">编辑</a>
<button th:attr="deluri=@{/emp/}+${emp.id}" class="btn btn-sm btn-danger deleteBtn" type="submit">删除</button>

<form id="delForm" method="post">
     <input type="hidden" name="_method" value="DELETE">
</form>

<script>
  $(".deleteBtn").click(function(){
  $("#delForm").attr("action",$(this).attr("deluri")).submit();
  return false;
  })
</script>

```

**删除：需要注意配置，否则会出现405**

```properties
spring.mvc.hiddenmethod.filter.enabled=true
```

## 7、定制错误页面

**需要配置，不然页面不显示信息message和exception**(springboot2.X)

```properties
server.error.include-exception=true 
server.error.include-message=always
spring.mvc.view.prefix=/WEB-INF/
spring.mvc.view.suffix=.jsp
```

### 7.1 ErrorMvcAutoConfiguration.class

```java
DefaultErrorAttributes
    status
    exception
    message
    errors
    timestamp
    error
    path
    trace
```

### 7.2 页面

error/4xx.html、5.xx.html

```html
<h1>status:[[${status}]]</h1>
<h2>timestamp:[[${timestamp}]]</h2
```

#### 1）手机端定制json数据

```java
@ResponseBody
@ExceptionHandler(UserException.class)
public String getException(){
    Map<String ,Object> map = new HashMap();
    map.put("code",500);
    map.put("message","用户不存在");
    return map.toString();
}
// 没有自适应浏览器
```

#### 2）转到/error响应自适应效果

```java
@ExceptionHandler(UserException.class)
public String getException(Exception e,HttpServletRequest request){
    Map<String ,Object> map = new HashMap();
    map.put("code",500);
    map.put("message",e.getMessage());
    request.setAttribute("javax.servlet.error.status_code",500);
    return "forward:/error";
}
```

#### 3) 携带定制错误信息

```java
@Component
public class MyErrorAtributes extends DefaultErrorAttributes {
    @Override
    public Map<String, Object> getErrorAttributes(WebRequest webRequest, ErrorAttributeOptions options) {
        Map<String, Object> map = super.getErrorAttributes(webRequest, options);
        map.put("aa","aaa");
        Map<String,Object> ex = (Map<String,Object>)webRequest.getAttribute("ex", 0);
        map.put("ext",ex);
        return map;
    }
}
@ExceptionHandler(UserException.class)
public String getException(Exception e,HttpServletRequest request){
    Map<String ,Object> map = new HashMap();
    map.put("code",500);
    map.put("message","user notExit");
    request.setAttribute("javax.servlet.error.status_code",500);
    request.setAttribute("ex",map);
    return "forward:/error";
}
```

## 8、嵌入式servlet容器配置

### 1）配置

```java
@Bean
public WebServerFactoryCustomizer<ConfigurableWebServerFactory> webServerFactoryCustomizer(){
    return new WebServerFactoryCustomizer<ConfigurableWebServerFactory>(){
        @Override
        public void customize(ConfigurableWebServerFactory factory) {
            factory.setPort(8888);
        }
    };
}
```

### 2）注册三大组件

```java
@Bean
public ServletRegistrationBean myservlet(){
    ServletRegistrationBean servletRegistrationBean = new ServletRegistrationBean(new MyServerlet(),"/myServlet");
    return servletRegistrationBean;
}
@Bean
public FilterRegistrationBean myFilter(){
    FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean();
    filterRegistrationBean.setFilter(new MyFilter());
    filterRegistrationBean.setUrlPatterns(Arrays.asList("/hello","/myFilter"));
    return filterRegistrationBean;
}
@Bean
public ServletListenerRegistrationBean myListener(){
    ServletListenerRegistrationBean bean = new ServletListenerRegistrationBean();
    bean.setListener(new MyListener());
    return bean;
}
```

### 3)   其他嵌入式servlet组件

```xml
tomcat:
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
jetty:长连接
<!--<dependency>
   <artifactId>spring-boot-starter-jetty</artifactId>
   <groupId>org.springframework.boot</groupId>
</dependency>-->
undertow:并发
<dependency>
   <artifactId>spring-boot-starter-undertow</artifactId>
   <groupId>org.springframework.boot</groupId>
</dependency>
```



# 3、Docker安装

### VirtualBox 安装centos7.iso

###  1)安装时需要选择网络

### 2)配置网络

![image-20200706175417030](C:\Users\JL\AppData\Roaming\Typora\typora-user-images\image-20200706175417030.png)

``` shell
service network start #重启服务
ip addr #然后可以通过putty连接



```

### 3) 安装docker

``` shell
uname -r #查看内核
yum update #更新内核 大于3.10
yum install docker#安装
[root@localhost ~]# systemctl start docker #启动docker
[root@localhost ~]# docker -v #查看docker版本
Docker version 1.13.1, build 64e9980/1.13.1
[root@localhost ~]#  systemctl enable docker #开机启动
[root@localhost ~]# docker search mysql #查询镜像
[root@localhost ~]# docker pull mysql #下载镜像（：5.5 可以加版本号下载）
[root@localhost ~]# docker images 查看镜像
[root@localhost ~]# docker rmi d404d78aa797 删除镜像，加id
```

镜像地址： https://hub.docker.com/ 

```shell
#启动容器
[root@localhost ~]# docker run --name mytomcat -d tomcat:latest
#查看运行中的容器
[root@localhost ~]# docker ps
#停止容器
[root@localhost ~]# docker stop 7d05b79960bc
#查看所有容器
[root@localhost ~]# docker ps -a
#启动容器
[root@localhost ~]# docker start 7d05b79960bc
#移除容器
[root@localhost ~]# docker rm 7d05b79960bc
#启动了一个做了映射端口的tomcat
[root@localhost ~]# docker run -p 8888:8080 -d tomcat
```

# mysql

``` shell
#启动mysql
[root@localhost ~]# docker run -p 3306:3306 --name mysql01 -e MYSQL_ROOT_PASSWORD=123456 -d mysql

```

# 4、jdbc连接mysql

## 1)配置连接

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://192.168.0.172:3306/mysql
    username: root
    password: 123456
```

## 2）自动执行建表语句

```yaml
 initialization-mode: always
    schema:
    - classpath:aa.sql
```

## 3)Druid接入

### 3.1加入依赖

```xml
 <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.23</version>
        </dependency>
```

### 3.2 改变类型为druid

```yaml
    type: com.alibaba.druid.pool.DruidDataSource
    filters: stat(不加上没有显示sql监控)
```

### 3.3 实现类

```java
@Configuration
public class DruidConfig {
    @Bean
    @ConfigurationProperties(prefix = "spring.datasource")
    public DruidDataSource durid(){
        return new DruidDataSource();
    }
    @Bean
    public ServletRegistrationBean statViewServlet(){
        ServletRegistrationBean bean = new ServletRegistrationBean(new StatViewServlet(),"/druid/*");
        Map<String,String> paraeters = new HashMap<>();
        paraeters.put("loginUsername","admin");
        paraeters.put("loginPassword","123456");
        paraeters.put("allow","");  //默认就是允许所有访问
        paraeters.put("deny","");		//默认访问
        bean.setInitParameters(paraeters);
        return bean;
    }
    @Bean
    public FilterRegistrationBean webStatFilter(){
        FilterRegistrationBean bean = new FilterRegistrationBean();
        Map<String,String> paraeters = new HashMap<>();
        bean.setFilter(new WebStatFilter());
        bean.setUrlPatterns(Arrays.asList("/*"));
        paraeters.put("profileEnable","/*");
        paraeters.put("exclusions","*.css,/druid/*");
        bean.setEnabled(true);
        bean.setInitParameters(paraeters);

        return bean;
    }
}
```

### 3.4 访问效果

![image-20200710131647847](C:\Users\JL\AppData\Roaming\Typora\typora-user-images\image-20200710131647847.png)

## 4) mybatis配置文件使用

### 4.1）mapper文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.boot.data.EventMapper">  #接口
    <select id="getEvent" resultType="com.example.boot.data.Event"> #实体类
        select * from event where db = #{db}
    </select>

</mapper>
```

### 4.2) 引入映射文件

```yaml
mybatis:
  mapper-locations: classpath:/mybatis/mapper/*.xml
```

### 4.3)编写接口类和controller

```java
public interface EventMapper {
    public Event getEvent(String id);
}


@RestController
public class EventContraller {
    @Autowired
    EventMapper eventMapper;
    @GetMapping("/emp/{db}")
    public Event getEmp(@PathVariable("db") String db){
        return eventMapper.getEvent(db);
    }
}
```

`加上注解`

```java
@MapperScan("com.example.boot.data")
```

[访问地址](http://localhost:8080/emp/sas)



## 5) JPA

### 5.1)配置jpa

```ymal
 jpa:
    hibernate:
      ddl-auto: update #自动创建或者更新表
    show-sql: true #显示sql
```

### 5.2)实体类

```java
@Entity
@Table(name = "city")
public class City {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;
    @Column(name = "province_id")
    private int provinceId;
    @Column(name = "city_name")
    private String cityName;
    @Column(name = "description")
    private String description;
```

### 5.3)继承JpaRepository

``` java
public interface CityRepository extends JpaRepository<City,Integer> {
}
```

### 5.4)新增和查询

```java
@RestController
public class CityController {
    @Autowired
    CityRepository cityRepository;
    @GetMapping("/city/{id}")
    public City getCity(@PathVariable("id") int id){
        return cityRepository.getOne(id);
    }
    @GetMapping("/city")
    public City sageCity(City city){
        return cityRepository.save(city);
    }
}
```

[新增](http://localhost:8080/city?cityName=aa&description=bb)

查询实体类加上 @JsonIgnoreProperties(value={"hibernateLazyInitializer","handler","fieldHandler"})

# 5、上传下载

```java
@Controller
@RequestMapping("/file/")
public class Upload {
    @RequestMapping("/aa")
    public String file(){
        return "upload";
    }
    @RequestMapping("/upload")
    @ResponseBody
    public String upload (@RequestParam("file") MultipartFile file) {
        // 获取原始名字
        String fileName = file.getOriginalFilename();
        // 获取后缀名
        // String suffixName = fileName.substring(fileName.lastIndexOf("."));
        // 文件保存路径
        String filePath = "d:/upload/";
        // 文件重命名，防止重复
        fileName = filePath + UUID.randomUUID() + fileName;
        // 文件对象
        File dest = new File(fileName);
        // 判断路径是否存在，如果不存在则创建
        if(!dest.getParentFile().exists()) {
            dest.getParentFile().mkdirs();
        }
        try {
            // 保存到服务器中
            file.transferTo(dest);
            return "上传成功";
        } catch (Exception e) {
            e.printStackTrace();
        }
        return "上传失败";
    }
    @RequestMapping("download")
    public void download(HttpServletResponse response) throws Exception {
        // 文件地址，真实环境是存放在数据库中的
        File file = new File("D:\\upload\\a.txt");
        // 穿件输入对象
        FileInputStream fis = new FileInputStream(file);
        // 设置相关格式
        response.setContentType("application/force-download");
        // 设置下载后的文件名以及header
        response.addHeader("Content-disposition", "attachment;fileName=" + "a.txt");
        // 创建输出对象
        OutputStream os = response.getOutputStream();
        // 常规操作
        byte[] buf = new byte[1024];
        int len = 0;
        while((len = fis.read(buf)) != -1) {
            os.write(buf, 0, len);
        }
        fis.close();
    }
}
```

