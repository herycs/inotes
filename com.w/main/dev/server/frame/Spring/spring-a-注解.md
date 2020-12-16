# 注解

## Bean

### 声明为Bean

- @Service    将业务层（Service层）的类标识为Spring容器中的Bean
- @Controller  将控制层的类标识为Spring容器中的Bean
- @Repository  将数据访问层（Dao层）的类标识为Spring容器中的Bean
- @Component  将一个类标识为Spring容器中的Bean。---泛指

### 作用域

| **作用域**        | **说明**                                                     |
| ----------------- | ------------------------------------------------------------ |
| singleton（单例） | 默认值。该Bean（类）在Spring容器中只有一个实例，无论引用/获取这个Bean多少次，都指向同一个对象。singleton适用于无会话状态的Bean（如Dao组建、Service组件）。 在spring容器启动时就创建实例，且程序运行过程中，这个Bean只创建、使用一个实例。由spring容器负责管理生命周期，实例的创建、销毁均由spring容器负责。 |
| prototype（多例） | 每次获取该Bean的实例时，都会创建一个新的实例。在需要时（从容器中获取该bean的时）才创建该bean的一个实例。由spring容器负责创建实例，创建好之后交给调用者，由调用者负责后续处理，spring容器不再管理这个实例。 |
| request           | web中使用。在一次HTTP请求中，获取的是该Bean的同一个实例，该实例只在此次HTTP请求中有效。新的HTTP请求，会创建新的实例。 |
| session           | web中使用。在一次HTTP session中获取的是该Bean的同一个实例（一个session中只创建此bean的一个实例），创建实例只在本次HTTP session中有效。新的session，会创建新的实例 |
| globalSession     | 在一个全局的HTTP session中，获取到的是该Bean的同一个实例。只在使用portlet上下文时有效。 |
| application       | 为每个ServletContext对象创建一个实例。只在web相关的ApplicationContext中有效。 |
| websocket         | 为每个websocket对象创建一个实例。只在web相关的ApplicationContext中有效。 |



## 其他

### Json与实体

@ResponseBody

@RestController = @Controller与@ResponseBody同时加在类上

要让@ResponseBody在类上也起作用，需要在springmvc配置文件中加上<mvc:annotation-driven />这一行配置才可以。

### URL与处理器

@RequestMapping

处理控制器与URL映射

①将 @RequestMapping 注解在类中方法上，而Controller类上不添加 @RequestMapping 注解，这时的请求 URL 是相对于 Web 根目录。

②将 @RequestMapping 注解在Controller 类上，这时类的注解是相对于 Web 根目录，而方法上的是相对于类上的路径。

@RequestMapping 中的 value 和 path 属性（这两个属性作用相同，可以互换），@PathVariable 将 URL 中的占位符绑定到控制器的处理方法的参数中，占位符使用{}括起来。

## 各层常用注解

### Controller

类：

@Controller | @RestController
@CrossOrigin
@RequestMapping("/code")
@ResponseBody

```java
@Controller | @RestController
@CrossOrigin
@RequestMapping("/code")
@ResponseBody
public class ValidateCode {

    @RequestMapping(value = "/validateCode.do")
    public void getVerify(HttpServletRequest request, HttpServletResponse response) throws IOException {
        //...
    }
    @RequestMapping(value = "/examine/{articleId}", method= RequestMethod.PUT)
	public Result examine(@PathVariable("articleId") String articleId){
		//...
	}
    @RequestMapping(value="/search",method = RequestMethod.POST)
    public Result findSearch( @RequestBody Map searchMap){
        //...
    }
}
```

### 全局异常处理

```java
@ControllerAdvice
public class BaseExceptionHandler {
    @ExceptionHandler(value = Exception.class)
    @ResponseBody
    public Result handlerError(HttpServletRequest req, Exception e) {
        // MessageBean一个前后台交互的Json实体类，包括success，msg，data属性
        if (e instanceof  SizeLimitExceededException){
            return new Result(StateCode.FAILED, "文件大小超过限制");
        }
        System.out.println(e.getMessage());
        return new Result(StateCode.FAILED, "操作失败");
    }
}
```

### Dao

```java
@Repository
public interface AnnouncementDao {
    @Insert("insert into announcement(annu_title)  values( #{announcement.annu_title} )")
    int addAnnouncement(@Param("announcement") Announcement announcement) throws SQLException;
}
```

### Service

```java
@Service("annoService")
public class AnnouncementServiceImpl implements AnnouncementService {

    @Autowired
    private AnnouncementDao announcementDao;

    @Override
    public int addAnno(Announcement announcement) throws Exception{
        int result = announcementDao.addAnnouncement(announcement);
        return result;
    }
}

```

### Pojo

```java
@Entity
@Table(name="tb_article")
public class Article implements Serializable{
    @Id
    private String id;//ID
    private String columnid;//专栏ID
    private String userid;//用户ID
    private String title;//标题
}
```

### Application

```java
@SpringBootApplication
@EnableEurekaClient
public class ArticleApplication {
    public static void main(String[] args) {
        SpringApplication.run(ArticleApplication.class, args);
    }
    @Bean
    public Utils utils(){
        return new Utils(1);
    }
}
```

