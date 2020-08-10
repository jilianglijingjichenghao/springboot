# spring boot 高级

## 1、缓存

### 1）@Cacheable

```java
@EnableCaching //主程序
```

```java
//查询
@Cacheable(cacheNames = "getCity",key = "#id") //contrl或者service
```

```java
cacheNames/value //指定缓存组件的名称
key //缓存数据使用的key，可以用它来指定  key = "#root.methodName+'['+#id+']'"
condition = "#id>1
unless = "#a0==2"
```

自定义key

```java
@Configuration
public class MykeygConfig {
    @Bean("mykeygen")
    public KeyGenerator mykey(){
       return  new KeyGenerator(){

            @Override
            public Object generate(Object o, Method method, Object... objects) {
                return method+"[["+ Arrays.asList(objects).toString()+"]]";
            }
        };
    }
}
```

### 2)  @CachePut

```java
 @CachePut(cacheNames = "city",key = "#result.id") //更新
```

**注意：cacheNames 必须和查询的一致，不然不会更新缓存**

### 3）@CacheEvict

```java
@CacheEvict(cacheNames = "city",key = "#id") //清空缓存 
allEntries = true //清空所有
beforeInvocation=false//默认是在方法执行之后执行
keyGenerator = "mykeygen"//自定义key    
```

### 4）@Caching和@CacheConfig

```java
//复杂的缓存配置
@Caching(
    cacheable = {
        @Cacheable(/*value = "city",*/key="#name")
    },put = {
        @CachePut(/*value = "city",*/key = "#result.description"),
        @CachePut(/*value = "city",*/key = "#result.id")
    }
)
//全局的配置
@CacheConfig(cacheNames = "city")
```

## 2、Redis 安装（6379端口）

### 1) [常用命令](http://www.redis.cn/commands.html#set)

### 2) Template

```java
@Autowired
RedisTemplate redisTemplate;

@Autowired
StringRedisTemplate stringRedisTemplate;
//保存对象序列化
@Bean
public RedisTemplate<Object, City> redisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
    RedisTemplate<Object, City> template = new RedisTemplate();
    template.setConnectionFactory(redisConnectionFactory);
    Jackson2JsonRedisSerializer serializer = new Jackson2JsonRedisSerializer(City.class);
    template.setDefaultSerializer(serializer);
    return template;
}
```

### 3) redis 加入序列化

```java
@Bean
public RedisCacheManager cacheManager(RedisConnectionFactory redisConnectionFactory) {
    RedisCacheConfiguration cacheConfiguration =
        RedisCacheConfiguration.defaultCacheConfig()
        .entryTtl(Duration.ofDays(1))   // 设置缓存过期时间为一天
        .disableCachingNullValues()     // 禁用缓存空值，不缓存null校验
        .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new
                                                                                        GenericJackson2JsonRedisSerializer()));     // 设置CacheManager的值序列化方式为json序列化，可加入@Class属性
    return RedisCacheManager.builder(redisConnectionFactory).cacheDefaults(cacheConfiguration).build();     // 设置默认的cache组件
}
```

### 4) 多个类，需要写多个template

```java
@Bean
public RedisTemplate<Object, Event> redisTemplateEvent(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
    RedisTemplate<Object, Event> template = new RedisTemplate();
    template.setConnectionFactory(redisConnectionFactory);
    Jackson2JsonRedisSerializer serializer = new Jackson2JsonRedisSerializer(Event.class);
    template.setDefaultSerializer(serializer);
    return template;
}
```

## 3、Rabbitmq 

### 3.1 安装

### 1）下载镜像,启动镜像

```shell
[root@localhost ~]# docker pull rabbitmq:3.7.26-management
[root@localhost ~]# docker run -d -p 5672:5672 -p 15672:15672 --name myrabbitmq docker.io/rabbitmq:3.7.26-management
```

### 2）浏览器访问

用户密码（guest）

![image-20200717104842941](C:\Users\JL\AppData\Roaming\Typora\typora-user-images\image-20200717104842941.png)

### 3）如果不能访问，需要设置

````shell
[root@localhost ~]# docker exec -it myrabbitmq /bin/bash
root@26f779641db5:/# rabbitmq-plugins enable rabbitmq_management

````

**direct**:     完全匹配

**fanout**： 全部发送

**topic**：	按照字符匹配(.#  *.）



### 4)   添加交换机

![image-20200717110709311](C:\Users\JL\AppData\Roaming\Typora\typora-user-images\image-20200717110709311.png)

### 5）添加消息

![image-20200717110730786](C:\Users\JL\AppData\Roaming\Typora\typora-user-images\image-20200717110730786.png)

### 6）绑定关系

![image-20200717110814275](C:\Users\JL\AppData\Roaming\Typora\typora-user-images\image-20200717110814275.png)

### 7）发送消息

![image-20200717111613870](C:\Users\JL\AppData\Roaming\Typora\typora-user-images\image-20200717111613870.png)

### 8)接收消息，清空前一次

![image-20200717134310978](C:\Users\JL\AppData\Roaming\Typora\typora-user-images\image-20200717134310978.png)

### 3.2 springboot与rabbitmq

### 1）添加配置文件

```properties
spring.rabbitmq.host=192.168.0.142
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
```

### 2）测试

```java
@Autowired
RabbitTemplate rabbitTemplate;
@Test
void contextLoads() {
    Map<String,Object> map = new HashMap<>();
    map.put("msg","这是一个消息");
    map.put("data", Arrays.asList("aa",12,false));
    rabbitTemplate.convertAndSend("exchange.direct","atguigu.news",map);
}
```

### 3）接收消息

```java
@Test
void recive(){
    Object o = rabbitTemplate.receiveAndConvert("atguigu.news");
    System.out.println(o.getClass());
    System.out.println(o);
}
```

### 4)序列化消息

```java
@Configuration
public class MyConvent {
    @Bean
    public MessageConverter myConverter(){
       return new Jackson2JsonMessageConverter();
    }
}
```

### 5) 监听消息

```java
@RabbitListener(queues = "atguigu.news")
public void getMsg(Book book){
    System.out.println("收到消息："+book);
}
//主程序加上 @EnableRabbit
```

### 6）代码创建

```java
@Autowired
AmqpAdmin amqpAdmin;
@Test
public void create(){
    //amqpAdmin.declareExchange(new DirectExchange("aaa"));
    //amqpAdmin.declareQueue(new Queue("bb"));
    amqpAdmin.declareBinding(new Binding("bb", Binding.DestinationType.QUEUE,"aaa","gg",null));
}
```

## 4、ElasticSearch

### 1）下载安装镜像

```shell
[root@localhost ~]# docker pull elasticsearch:7.8.0
[root@localhost ~]#  docker run -e ES_JAVA_OPTS="-Xms256m -Xmx256m" -d -p 9200:9200 -p 9300:9300 --name ES0 121454ddad72

docker run -d -e ES_JAVA_POTS="-Xms256m -Xmx256m"  -e "discovery.type=single-node" -p 9200:9200 -p 9300:9300 --name ES0 f29a1ee41030
# 查看报错日志
$cat /proc/sys/vm/max_map_count
65530
 
#设置最大内存为262144
$sysctl -w vm.max_map_count=262144
 
#此时查看
$cat /proc/sys/vm/max_map_count
262144

dd if=/dev/zero of=swapfile bs=1024 count=500000
mkswap swapfile
swapon swapfile

```

### 2) postman测试

https://www.elastic.co/guide/cn/elasticsearch/guide/current/highlighting-intro.html

![image-20200720100619666](C:\Users\JL\AppData\Roaming\Typora\typora-user-images\image-20200720100619666.png)

### 3)jest

#### 1）引入依赖,注释掉spring-data

```xml
        <!--<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
        </dependency>-->
        <!-- https://mvnrepository.com/artifact/io.searchbox/jest -->
        <dependency>
            <groupId>io.searchbox</groupId>
            <artifactId>jest</artifactId>
            <version>2.4.0</version>
        </dependency>
```

#### 2)加上实体类，调用测试

```java
	//实体类
    @JestId
    private Integer id;
    private String name;
    private Integer age;
    @Autowired
    JestClient jestClient;
	//保存
    @Test
    public void contextLoads() throws IOException {
        User user = new User();
        user.setId(1);
        user.setName("张三");
        user.setAge(32);
        Index index = new Index.Builder(user).index("aa").type("info").build();
        jestClient.execute(index);
    }
	//查询
 	@Test
    public void search() throws IOException {
        String query ="{\n" +
                "    \"query\" : {\n" +
                "        \"bool\": {\n" +
                "            \"must\": {\n" +
                "                \"match\" : {\n" +
                "                    \"name\" : \"张三\" \n" +
                "                }\n" +
                "            }\n" +
                "        }\n" +
                "    }\n" +
                "}";
        Search search = new Search.Builder(query).addIndex("aa").addType("info").build();
        SearchResult result = jestClient.execute(search);
        System.out.println(result.getJsonString());
    }
```

[访问地址](http://192.168.0.137:9200/aa/info/1)

### 4)spring_data_elasticsearch

```xml
//elasticsearch 版本需要一致
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>19.0</version>
</dependency>
```

#### 1）添加实体类

```java
//indexName代表所以名称,type代表表名称
@Document(indexName = "wantu_notice_info", indexStoreType = "vv")
public class Notice {

    //id
    @JsonProperty("auto_id")
    private Long id;

    //标题
    @JsonProperty("title")
    private String title;

    //公告标签
    @JsonProperty("exchange_mc")
    private String exchangeMc;

    //公告发布时间
    @JsonProperty("create_time")
    private String originCreateTime;

    //公告阅读数量
    @JsonProperty("read_count")
    private Integer readCount;
}
```

#### 2)添加接口

```java

@Component
public interface NoticeRepository extends ElasticsearchRepository<Notice, Long> {

}
```

#### 3)添加控制类

```java
public class NoticeController {
    @Autowired
    private NoticeRepository nticeRepository;

    @GetMapping("/save")
    public Object save(long id, String title){

        Notice article = new Notice();
        article.setId(id);
        article.setReadCount(123);
        article.setTitle(title);
        return nticeRepository.index(article);
    }
    /**
     * @param title   搜索标题
     * @param pageable page = 第几页参数, value = 每页显示条数
     */
    @GetMapping("/search")
    public Object search(String title ,@PageableDefault() Pageable pag){

        //按标题进行搜索
        QueryBuilder builder = QueryBuilders.matchQuery("title", title);
        //如果实体和数据的名称对应就会自动封装，pageable分页参数
        Iterable<Notice> listIt =  nticeRepository.search(builder, pag);
        //Iterable转list
        List<Notice> list= Lists.newArrayList(listIt);

        return list;
    }
    @RequestMapping("/all")
    public List<Notice> all() throws Exception {
        Iterable<Notice> data = nticeRepository.findAll();
        List<Notice> ds = new ArrayList<>();
        for (Notice d : data) {
            ds.add(d);
        }
        return ds;
    }
    @RequestMapping("/id")
    public Object findid(long id) throws Exception {
        return nticeRepository.findById(id).get();


    }
}
```

## 5、任务

### 1）异步任务

添加两个注解

```java
@Async // service
@EnableAsync//主程序
```

### 2）定时任务

```java
@Scheduled(cron = "0 * * * * MON-THU" ) //service
@EnableScheduling //主程序
表达式
    ，枚举（0，1，2，3，4）
    - 区间（0-4）
    / 步长（0/4）每四秒
```

### 3）发送邮件

#### 1)添加依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```

#### 2) 邮箱配置

密码为授权码

![image-20200723113725584](C:\Users\JL\AppData\Roaming\Typora\typora-user-images\image-20200723113725584.png)

```properties
spring.mail.username=894318744@qq.com
spring.mail.password=ltfuyqvrdxmnbfeg
spring.mail.host=smtp.qq.com
spring.mail.properties.mail.smtp.ssl.enable=true
```

#### 3）测试

```java
@Autowired
JavaMailSender javaMailSender;
@Test
public void bb(){
    SimpleMailMessage message = new SimpleMailMessage();
    message.setText("aaaaa");
    message.setSubject("bbbb");
    message.setFrom("894318744@qq.com");
    message.setTo("1509182970@qq.com");
    javaMailSender.send(message);
}
```

#### 4)带附件的邮件

```java
@Test
public void cc() throws MessagingException {

    MimeMessage mimeMessage = javaMailSender.createMimeMessage();
    MimeMessageHelper helper = new MimeMessageHelper(mimeMessage,true);

    //SimpleMailMessage message = new SimpleMailMessage();
    helper.setText("<b style='color:red'>aaaaa</b>",true);
    helper.setSubject("bbbb");
    helper.setFrom("894318744@qq.com");
    helper.setTo("1509182970@qq.com");

    helper.addAttachment("qpms.log.2020-07-20",new File("C:\\Users\\JL\\Documents\\qpms.log.2020-07-20"));
    helper.addAttachment("未标题-1.jpg",new File("C:\\Users\\JL\\Documents\\未标题-1.jpg"));
    javaMailSender.send(mimeMessage);
}
```

## 6、安全

### 1）登录&认证&授权

```java
//1.继承WebSecurityConfigurerAdapter
//2.重写configure http 给请求路径加角色
//3.重写configure AuthenticationManagerBuilder 给用户授权（注意密码需要加密）

@EnableWebSecurity
public class MySecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        //super.configure(http);
        http.authorizeRequests().antMatchers("/").permitAll()
            .antMatchers("/1/**").hasRole("VIP1")
            .antMatchers("/2/**").hasRole("VIP2")
            .antMatchers("/3/**").hasRole("VIP3");
        //没有权限到登录页面
        http.formLogin();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        //super.configure(auth);
        auth.inMemoryAuthentication().passwordEncoder(new BCryptPasswordEncoder()).
            withUser("zhangsan").password(new BCryptPasswordEncoder().encode("123456")).roles("VIP1","VIP2")
            .and().withUser("lisi").password(new BCryptPasswordEncoder().encode("123456")).roles("VIP2","VIP3")
            .and().withUser("wangwu").password(new BCryptPasswordEncoder().encode("123456")).roles("VIP1","VIP3");
    }
}
```

### 2）注销

```java
//注销
http.logout().logoutSuccessUrl("/"); //成功后到首页，请求提交必须是post
```

### 3) 页面判断

```xml
<dependency>
    <groupId>org.thymeleaf.extras</groupId>
    <artifactId>thymeleaf-extras-springsecurity4</artifactId>
    <version>3.0.4.RELEASE</version>
</dependency>
<!--spring boot 2.0.7-->
```

```html
<!DOCTYPE html>
<html lang="en"
      xmlns:th="https://www.thymeleaf.org"
      xmlns:sec="https://www.thymeleaf.org/thymeleaf-extras-springsecurity4">

<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<div sec:authorize="!isAuthenticated()">
    <a th:href="@{/login}">请登录</a>
</div>
<div sec:authorize="isAuthenticated()">

    <h2><span sec:authentication="name"> </span>你好，你拥有的角色
        <span sec:authentication="principal.authorities"></span> </h2>
    <form th:action="@{/logout}" method="post">
        <input type="submit" value="注销">
    </form>
</div>

<table>

    <ul sec:authorize="hasRole('VIP1')">
        <li>初级</li>
        <li><a href="/1/1">1</a></li>
        <li><a href="/1/2">2</a></li>
        <li><a href="/1/3">3</a></li>
    </ul>
    <ul sec:authorize="hasRole('VIP2')">
        <li>高级</li>
        <li><a href="/2/1">1</a></li>
        <li><a href="/2/2">2</a></li>
        <li><a href="/2/3">3</a></li>
    </ul>
    <ul sec:authorize="hasRole('VIP3')">
        <li>绝世</li>
        <li><a href="/3/1">1</a></li>
        <li><a href="/3/2">2</a></li>
        <li><a href="/3/3">3</a></li>
    </ul>

</table>
</body>
</html>
```

### 4）自己的登录页

```java
http.formLogin().loginPage("/index");
//默认post请求，否则要指定loginProcessingUrl("/login")

<form th:action="@{/index}" method="post">
//记住我
http.rememberMe().rememberMeParameter("rember");
```

## 7、分布式

### 1）安装zookeeper

```shell
[root@localhost ~]# docker run --name zook01 -p 2181:2181 --restart always -d 2bb455f2234b

```

### 2）添加依赖

```xml
 <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>2.7.3</version>
        </dependency>
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo</artifactId>
            <version>2.7.3</version>
        </dependency>
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-dependencies-zookeeper</artifactId>
            <version>2.7.3</version>
            <exclusions>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-log4j12</artifactId>
                </exclusion>
            </exclusions>
            <type>pom</type>
        </dependency>
```

### 3）配置

```yaml
dubbo:
  application:
    name: sea-provider-log
  registry:
    protocol: zookeeper
    address: 192.168.0.147:2181
  scan:
    base-packages: com.example.ticket
```



### 4)代码调用

```java
import org.apache.dubbo.config.annotation.Service;

@Service
public class TicketServiceImpl implements TicketService{
    @Override
    public String aa() {
        System.out.println("dfasdsdfadsf");
        return "aa";
    }
}
import org.apache.dubbo.config.spring.context.annotation.EnableDubbo;
// 主程序加@EnableDubbo
import org.apache.dubbo.config.annotation.Reference;
@Reference(check=false)
TicketService ticketService;
@Test
public void aa(){
    System.out.println(ticketService.aa());
}
```

## 8、Eureka

![image-20200730103159887](C:\Users\JL\AppData\Roaming\Typora\typora-user-images\image-20200730103159887.png)

### 1) 注册中心

配置

```yaml
server:
  port: 8761
eureka:
  instance:
    hostname: eureka
  client:
    register-with-eureka: false #不注册自己
    fetch-registry: false #不从eureka获取
```

注解

```java
@EnableEurekaServer
```

[访问地址](http://localhost:8761/)

### 2) 提供者

配置

```yaml
server:
  port: 8002
eureka:
  instance:
    prefer-ip-address: true
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

service controller

### 3）消费者

配置

```yaml
spring:
  application:
    name: user
eureka:
  instance:
    prefer-ip-address: true
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
server:
  port: 8888
```

controller

```java
@RestController
public class UserController {
    @Autowired
    RestTemplate restTemplate;

    @GetMapping("/aa")
    public String aa(){
        String s = restTemplate.getForObject("http://PROVID/ticket", String.class);
        return s;
    }
}
```

主程序

```java
@EnableDiscoveryClient //发现服务
@SpringBootApplication
public class UserApplication {

    public static void main(String[] args) {
        SpringApplication.run(UserApplication.class, args);
    }
    @LoadBalanced //负载均衡
    @Bean
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }
}
```

没有加载依赖需要加上

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <mainClass>com.example.provid.ProvidApplication</mainClass>//启动类

    </configuration>
    <executions>
        <execution>
            <goals>
                <goal>repackage</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

## 9、热部署

添加依赖(ctrl+f9)

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
</dependency>
```

## 10、监控管理

### 1）、添加依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### 2）、配置

```properties
management.endpoints.web.exposure.include=*
management.server.port=8081
```

[访问地址](http://localhost:8081/actuator/beans)

## 11、自定义

```java
@Component
public class MyAppHealthIndictor implements HealthIndicator {
    @Override
    public Health health() {
      return Health.down().withDetail("msg","服务器错误").build();

    }
}
```



