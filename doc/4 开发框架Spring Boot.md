# 4 开发框架Spring Boot

### 4.3.4 多个环境的配置文件
1. 格式application-{profile}.properties
application-test.properties——测试环境
application-dev.properties——开发环境
application-prod.properties——生产环境
2. 第一种使用方法在application.perperties中指定spring.profiles.active即可，值为profile
第二种直接指定$ java -jar springbootdemo.jar -- spring.profiles.active=dev

### 4.4 运行状态监控Actuator
1. 使用方法
第一、加入依赖spring-boot-starter-actuator
第二、配置management.port和management.security.enabled
第三、可以根据Actuator端口信息，启动程序后访问状态监控访问。常见如端口号/health，/env全部环境属性，/info获取应用程序的信息,/bean注入了哪些bean.如果要通过actuator执行shutdown要先配置endpoints.shutdown.enabled=true配置会提示找不到
$curl -X POST http:localhost:9091/shutdown
2. 解释/health内容。该API接口提供的信息是由一个或多个健康指示器提供的健康信息组合成，理解为每个"键"为一个指示器，通常能连上为UP，否则为DOWN
```
{
	"status":"UP",
	"diskSpace":{
		"status":"UP",
		"total":392461021184,
		"free":381625602048,
		"threehold":10485760
	}
}
```
第四、另外也可以通过另一个依赖使用shell连接。spring-boot-starter-remote-shell,每次输出一个链接shell的密码，随机，用户名时user，端口2000
`ssh user@localhost -p 2000`回车后输入密码即可
只可查4个特有的shell命令，/beans 等

### 4.5 SpringBoot整合JPA

1. Java项目开发中提到JPA一般时指使用Hibernate的实现，因为在Java的ORM框架中，只有Hibernate实现的最好
2. 使用方法：一、加入依赖spring-boot-starter-web引入Web功能的起步以来、JPA的起步依赖spring-boot-starter-data-jpa、MySQL数据库的连接器的依赖Mysql-connector-java
二、在application.properties配置文件中配置数据库信息
三、创建model实体类，并且加入注解@Entity @Id @Column @GeneratedValue并配置好是否为一、为空在字段上
```
@Column(nullable=false,unique=true)
private String username;
```
四、创建DAO层，UserDao接口继承JpaRepository的接口，方法格式时findByUsername,传入参数username
五、创建Service层，直接调用UserDao接口即可
六、创建Controller，@RequestMapping @RestController @PathVariable注解可以获取RESTful风格的Url路径上的参数
>参考[restful风格，restcontroller与controller ](https://www.cnblogs.com/softidea/p/5884772.html)
>理解：1、restful风格，一个网址就是一个资源，其形式类似于http://xxx.com/xx/{id}/{id} ,反之原来放个取得数据和添加数据的网址都是http://xxx.com/xx，方法为get和post
>2、@RestController=@Controller+@RequestBody，@Controller需要的匹配度要更高，对应填充html的代码，通常返回的是视图。而@RestController返回的是对象，若无html渲染，则解析为JSON

### 4.6 SpringBoot整合Redis

1. 下载redis
2. 加入起步依赖spring-boot-starter-data-redis
3. application.yml中配置Redis数据源，如host、port=6379默认、数据库配置信息，配置Pool连接数等
4. RedisDao数据操作层，通过@Repository注解来注入SpringIoC容器中，该类是通过RedisTemplate来访问Redis的。这个类可以配置redis config例如设置String类型的数据过期时间等
```
@Repository
public class RedisDao{
	@Autowired
	private StringRedisTemplate template;
	public void setKey(String key,String value){
		valueOperations<String,String> ops=template.opsForValue();
		ops.set(key,value,1,TimeUnit.MINUTES);//1分钟过期
	}
	public String getValue(String key){
		ValueOperations<String,String> ops=this.template.opsForValue();
		return ops.get(key);
	}
}
测试
redisDao.setKey("name","forezp");
redisDao.setKey("age","17");
logger.info(redisDao.getValue("name"));
```

### 4.7 SpringBoot整合Swagger2，搭建Restful API在线文档
1. 引入起步依赖，springfox-swagger2和springfox-swagger-ui
2. Swagger2是一个强大在线API文档的框架，提供了在线文档的查阅和测试功能
3. 写一个配置类Swagger2，需要注入一个Docket的Bean，包括了apiInfo，即基本API文档的描述信息，以及包扫描的基本包名等信息

>[HTTP协议中POST、GET、HEAD、PUT等请求方法及相应值得含义](https://blog.csdn.net/qq_36183935/article/details/80570062)
>简要记：HEAD只请求页面的首部
>PUT从客户端向服务端传送的数据取代指定的文档的内容，一般用于更新。而TRACE可通过回传回来的TRACE报文分析请求报文是否最终原始相同，以用于判断中间是否被修改。
>PUT和POST区别在于当 一个方法重复执行，对于PUT后一个请求会把前一个请求覆盖掉，而POST不会覆盖 。所以PUT用于更新，POST用于增加资源

