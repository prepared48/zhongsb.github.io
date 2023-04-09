---
layout: post
title:  "python基础—pip指定包安装目录"
date:   2021-06-06 09:11:54
categories: Python 
tags: Python 
mathjax: true
---

* content
{:toc}

有的时候我们安装了annconda环境，有很多的python环境，比如py36, py37, py27。此时，我们使用`pip`安装包的时候，经常可能安装在一个不知道的路径，或者不是我们期望安装的路径。

这就是本文要解决的问题了。





`application-integrationtest.properties`该配置文件包含持久层存储的配置细节：

pip 指定某个路径安装包

## 场景：

有的时候我们安装了annconda环境，有很多的python环境，比如py36, py37, py27。此时，我们使用`pip`安装包的时候，经常可能安装在一个不知道的路径，或者不是我们期望安装的路径。

这就是本文要解决的问题了。

## 方法一

指定安装`numpy`包到固定文件夹下，比如这里“文件夹”是安装路径

```python
    pip install -t 文件夹 numpy
```    

## 方法二

设置 pip 默认安装路径

找到 `site.py` 文件。（windows：可以通过自带的查找，或者使用 `everything`软件；Linux直接使用find命令即可）

我的目录：D:\program\Anaconda\envs\py36\Lib\site.py

修改 `USER_SITE` 和 `USER_BASE` 两个字段的值(之前是null).

```python
    #自定义依赖安装包的路径
    USER_SITE = null
    #自定义的启用Python脚本的路径
    USER_BASE = null
```

我这里修改为

```python
    USER_SITE = "D:\program\Anaconda\envs\py36\Lib\site-packages"
    USER_BASE = "D:\program\Anaconda\envs\py36\Scripts"
```

使用命令查看、验证

```python
    python -m site
```

结果

```python
	sys.path = [
	    'C:\\Users\\z2010',
	    'D:\\program\\Anaconda\\envs\\py36\\python36.zip',
	    'D:\\program\\Anaconda\\envs\\py36\\DLLs',
	    'D:\\program\\Anaconda\\envs\\py36\\lib',
	    'D:\\program\\Anaconda\\envs\\py36',
	    'D:\\program\\Anaconda\\envs\\py36\\Lib\\site-packages',
	    'D:\\program\\Anaconda\\envs\\py36\\Lib\\site-packages\\torchvision-0.2.1-py3.6.egg',
	    'D:\\program\\Anaconda\\envs\\py36\\Lib\\site-packages\\tqdm-4.28.1-py3.6.egg',
	    'D:\\program\\Anaconda\\envs\\py36\\Lib\\site-packages\\pyahocorasick-1.4.0-py3.6-win-amd64.egg',
	    'D:\\program\\Anaconda\\envs\\py36\\Lib\\site-packages\\win32',
	    'D:\\program\\Anaconda\\envs\\py36\\Lib\\site-packages\\win32\\lib',
	    'D:\\program\\Anaconda\\envs\\py36\\Lib\\site-packages\\Pythonwin',
	]
	USER_BASE: 'D:\\program\\Anaconda\\envs\\py36\\Scripts' (exists)
	USER_SITE: 'D:\\program\\Anaconda\\envs\\py36\\Lib\\site-packages' (exists)
	ENABLE_USER_SITE: True
```

pip安装到conda根目录了，需要安装py36

---

公众号：关注【哥妞】，了解技术，学会泡妞~

```
spring.datasource.url = jdbc:h2:mem:test
spring.jpa.properties.hibernate.dialect = org.hibernate.dialect.H2Dialect
```

如果我们想使用MySQL来进行集成测试，我们可以修改上述配置文件（`application-integrationtest.properties`）的值。集成测试的测试用例看起来像`Controller层`的单元测试。

```java
@Test
public void givenEmployees_whenGetEmployees_thenStatus200()
  throws Exception {

    createTestEmployee("bob");

    mvc.perform(get("/api/employees")
      .contentType(MediaType.APPLICATION_JSON))
      .andExpect(status().isOk())
      .andExpect(content()
      .contentTypeCompatibleWith(MediaType.APPLICATION_JSON))
      .andExpect(jsonPath("$[0].name", is("bob")));
}
```

区别是Controller层测试用例中，没有东西是模拟的，并且是执行端到端场景。

## 5 通过`@TestConfiguration`进行测试配置

在前文中我们看到，增加了注解`@SpringBootTest`的类会启动整个应用上下文，这也意味着我们可以通过`@Autowire `注入任何通过`component`扫描的类到我们的测试类中：

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class EmployeeServiceImplIntegrationTest {

    @Autowired
    private EmployeeService employeeService;

    // class code ...
}
```

然而，我们也许想要避免启动整个应用程序，而只是启动一个特殊的测试配置。我们可以通过`@TestConfiguration`注解实现它。使用这个注解的方式有两种。一种方式是，我们可以在内部类的地方使用该注解来注入我们想要通过`@Autowire`注入的类。

```java
@RunWith(SpringRunner.class)
public class EmployeeServiceImplIntegrationTest {

    @TestConfiguration
    static class EmployeeServiceImplTestContextConfiguration {
        @Bean
        public EmployeeService employeeService() {
            return new EmployeeService() {
                // implement methods
            };
        }
    }

    @Autowired
    private EmployeeService employeeService;
}
```

另一种方式是，我们可以创建分开的测试配置类，而不是内部类：

```java
@TestConfiguration
public class EmployeeServiceImplTestContextConfiguration {
    
    @Bean
    public EmployeeService employeeService() {
        return new EmployeeService() { 
            // implement methods 
        };
    }
}
```

带`@TestConfiguration`注解的配置类会被`componet`扫描排除在外，因此我们需要在所有我们想要使用`@Autowired`的测试类中清晰的导入该类。我们可以通过`@Import`注解来实现：

```java
@RunWith(SpringRunner.class)
@Import(EmployeeServiceImplTestContextConfiguration.class)
public class EmployeeServiceImplIntegrationTest {

    @Autowired
    private EmployeeService employeeService;

    // remaining class code
}
```

## 6 通过 @MockBean 模拟

Service 层代码是依赖于持久层代码的：

```java
@Service
public class EmployeeServiceImpl implements EmployeeService {

    @Autowired
    private EmployeeRepository employeeRepository;

    @Override
    public Employee getEmployeeByName(String name) {
        return employeeRepository.findByName(name);
    }
}
```

然后，在测试Service层的时候，我们并不需要或者关心持久层是怎么实现的。理想情况下，我们应该可以在没有连接完整持久层代码的情况下，编写和测试Service层代码。

为了实现这样的解耦，==我们可以使用 Spring Boot Test 提供的 Mocking 支持来做到==。

让我们瞟一眼测试类的框架先：

```java
@RunWith(SpringRunner.class)
public class EmployeeServiceImplIntegrationTest {

    @TestConfiguration
    static class EmployeeServiceImplTestContextConfiguration {
 
        @Bean
        public EmployeeService employeeService() {
            return new EmployeeServiceImpl();
        }
    }

    @Autowired
    private EmployeeService employeeService;

    @MockBean
    private EmployeeRepository employeeRepository;

    // write test cases here
}
```

为了检查该Service类，我们需要有个一已经创建好并且可以通过 `@Bean` 可获得的Service类实例，这样我们才可以通过`@Autowired`在测试类中注入该Service类。我们可以通过`@TestConfiguration`注解来实现。

这里另一个有趣的事情是使用`@MockBean`。它会创建一个`EmployeeRepository`模拟类，它可以被用来替换真正的`EmployeeRepository`.

```java
@Before
public void setUp() {
    Employee alex = new Employee("alex");

    Mockito.when(employeeRepository.findByName(alex.getName()))
      .thenReturn(alex);
}
```

启动完成之后，测试用例就简单了：

```java
@Test
public void whenValidName_thenEmployeeShouldBeFound() {
    String name = "alex";
    Employee found = employeeService.getEmployeeByName(name);
 
     assertThat(found.getName())
      .isEqualTo(name);
 }
```

## 7 通过`@DataJpaTest`注解集成测试

我们将使用`Employee`实体，它有两个属性：id和name：

```java
@Entity
@Table(name = "person")
public class Employee {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    @Size(min = 3, max = 20)
    private String name;

    // standard getters and setters, constructors
}
```

这是使用 Spring Data JPA的持久层类：

```java
@Repository
public interface EmployeeRepository extends JpaRepository<Employee, Long> {

    public Employee findByName(String name);

}
```

这是持久层代码。现在让我们继续往下编写测试代码。首先，我们创建测试类的基本框架：

```java
@RunWith(SpringRunner.class)
@DataJpaTest
public class EmployeeRepositoryIntegrationTest {

    @Autowired
    private TestEntityManager entityManager;

    @Autowired
    private EmployeeRepository employeeRepository;

    // write test cases here

}
```

`@RunWith(SpringRunner.class)`注解提供一个Spring Boot Test 特性和JUnit中间的一个桥梁。当我们需要在JUnit测试类中使用Spring Boot 测试的特性的时候，这个注解就有用了。

`@DataJpaTest` 注解提供了持久层测试类的一些标准设置：

- 配置H2数据库，一个内存数据库
- 设置Hibernate，SPring Data，和DataSource
- 执行@EntityScan
- 打开SQL日志记录

为了继续数据库操作，我们需要在数据库中添加一些记录。为了设置这些数据，我们可以使用`TestEntityManager`。

Spring Boot `TestEntityManager` 是标准`JPA EntityManager`的替代方案，标准`JPA EntityManager`提供了编写测试时常用的方法。

`EmployeeRepository`是我们要进行测试的组件。现在我们编写我们第一个测试用例；

```java
@Test
public void whenFindByName_thenReturnEmployee() {
    // given
    Employee alex = new Employee("alex");
    entityManager.persist(alex);
    entityManager.flush();

    // when
    Employee found = employeeRepository.findByName(alex.getName());

    // then
    assertThat(found.getName())
      .isEqualTo(alex.getName());
}
```

在上述测试用例中，我们通过`TestEntityManager`往数据库中插入一条`Employee`记录，然后就通过命名API读取这条记录。`assertThat`来自于`Assertj`库，它与Spring Boot捆绑在一起。

## 8 通过`@WebMvcTest`进行单元测试

Controller层依赖Service层；简单起见，我们添加一个简单的方法：

```java
@RestController
@RequestMapping("/api")
public class EmployeeRestController {

    @Autowired
    private EmployeeService employeeService;

    @GetMapping("/employees")
    public List<Employee> getAllEmployees() {
        return employeeService.getAllEmployees();
    }
}
```
由于我们只关注Controller层代码，自然地，我们可以在单元测试中模拟Service层：

```java
@RunWith(SpringRunner.class)
@WebMvcTest(EmployeeRestController.class)
public class EmployeeRestControllerIntegrationTest {

    @Autowired
    private MockMvc mvc;

    @MockBean
    private EmployeeService service;

    // write test cases here
}
```

要测试Controller层，我们可以使用在大部分情况下，`@WebMvcTest`只会启动单个Controller类。我们可以和`@MockBean`注解一起使用来提供任何需要依赖的模拟实现。。它将为我们的单元测试自动配置Spring MVC基础结构。

在大部分情况下，`@WebMvcTest`只会启动单个Controller类。我们可以和`@MockBean`注解一起使用来提供任何需要依赖的模拟实现。

`@WebMvcTest`会自动配置`MockMvc`，它提供了一种强力的方式来简化测试MVC controller层的方式，而不需要启动一个完整的 HTTP 服务器。

测试类如下：

```java
@Test
public void givenEmployees_whenGetEmployees_thenReturnJsonArray()
  throws Exception {
    
    Employee alex = new Employee("alex");

    List<Employee> allEmployees = Arrays.asList(alex);

    given(service.getAllEmployees()).willReturn(allEmployees);

    mvc.perform(get("/api/employees")
      .contentType(MediaType.APPLICATION_JSON))
      .andExpect(status().isOk())
      .andExpect(jsonPath("$", hasSize(1)))
      .andExpect(jsonPath("$[0].name", is(alex.getName())));
}
```

get()方法调用可以被其他与HTTP相对应的方法替换，如put()、 post()等。请注意，我们还在请求中设置内容类型。
MockMvc是很灵活的，我们可以用它创建任何请求。

## 9 自动配置测试

Spring Boot的自动配置注释的一个惊人特性是，它有助于加载完整应用程序的某些部分和代码库的特定测试层。

除了上述提供的注解，这里还有一些被广泛使用的注解列表：

- @WebFluxTest:我们可以使用`@WebFluxTest`注解来测试Spring WebFlux控制器。它经常与`@MockBean`一起使用，为所需的依赖项提供模拟实现。
- @JdbcTest:我们可以使用`@JdbcTest`注释来测试JPA应用程序，但它只用于只需要数据源的测试。该注释配置一个内存内嵌入式数据库和一个`JdbcTemplate`。
- @JooqTest
- @DataMongoTest

...

你可以读到关于这些注解的更多文章，并继续优化集成测试，[优化Spring集成测试](https://www.baeldung.com/spring-tests)。

## 10 结论

在本文中，我们深入探讨了在Spring Boot中进行测试，并展示了怎么更有效的编写测试用例。

所有本文的源码都可以在这里找到，[github](https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-testing)。源码包含很多其他示例和不同的测试用例。

其他阅读：【Guide to Testing With the Spring Boot Starter Test】https://rieckpil.de/guide-to-testing-with-spring-boot-starter-test/



