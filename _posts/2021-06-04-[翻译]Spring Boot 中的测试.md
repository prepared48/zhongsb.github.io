---
layout: post
title:  "[翻译]Spring Boot 中的测试"
date:   2021-06-04 09:11:54
categories: JAVA SpringBoot
tags: JAVA 测试 SpringBoot
mathjax: true
---

* content
{:toc}

如何进行单元测试、集成测试是程序猿们逃脱不掉的宿命。在Spring Boot 中怎么进行集成测试呢？

且听我慢慢道来～




原文地址：https://www.baeldung.com/spring-boot-testing

## 1 概览

在这个教程中，我们会带你看看如果使用 Spring Boot 中的框架编写测试用例。内容会覆盖单元测试，也会有在执行测试用例前会启动 Spring 上下文的集成测试。如果你是使用 Spring Boot 的新手，查看链接：[Spring Boot 介绍](https://www.baeldung.com/spring-boot-start)。

扩展阅读：[探索 Spring Boot TestRestTemplate](https://www.baeldung.com/spring-boot-testresttemplate)、[Spring Boot @RestClientTest快速导航](https://www.baeldung.com/restclienttest-in-spring-boot)、[在Spring Beans中注入 Mockito Mocks](https://www.baeldung.com/injecting-mocks-in-spring)

## 2 项目启动

我们要使用的应用程序是一个api，这个api会提供一些关于Employee表的基本操作（增删改查）。这是一个典型的分层框架——API调用从controller层到service层，最后到持久层。

## 3 Maven 依赖

首先增加测试依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
    <version>2.2.6.RELEASE</version>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>test</scope>
</dependency>
```

`spring-boot-starter-test`包是包含测试所需要的大部分元素的主要依赖项。`H2`数据库是一个内存数据库。它不需要我们配置和启动一个真正的数据库，因此在测试场景下方便了开发人员。

### 3.1 JUnit4

Spring Boot 2.4 中，JUnit 5’s vintage engine 包已经从`spring-boot-starter-test`中被移除了。如果我们想用  `JUnit4` 写测试用例，我们需要添加下述依赖项。

```xml
<dependency>
    <groupId>org.junit.vintage</groupId>
    <artifactId>junit-vintage-engine</artifactId>
    <scope>test</scope>
    <exclusions>
        <exclusion>
            <groupId>org.hamcrest</groupId>
            <artifactId>hamcrest-core</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

## 4 通过 @SpringBootTest 进行集成测试

就像标题所说，集成测试焦点是整合应用程序的不同层（controller层、service层以及持久层）。这也意味着没有 mocking 参与其中。

理想情况下，我们应该把单元测试和集成测试分开，并且不应该和单元测试一起运行。我们可以通过使用不同的配置文件来实现这个分离。为什么要这么做呢？因为一般集成测试比较消耗时间并且有可能需要真正的数据库（不是内存数据库）来执行。

然而在本文中，我们不关注这个，我们关注的是，使用内存数据库H2持久化存储。

集成测试需要启动一个容器来执行测试用例。因此需要一些额外的设置——这些在 Spring Boot 中都很容易。

```java
@RunWith(SpringRunner.class)
@SpringBootTest(
  SpringBootTest.WebEnvironment.MOCK,
  classes = Application.class)
@AutoConfigureMockMvc
@TestPropertySource(
  locations = "classpath:application-integrationtest.properties")
public class EmployeeRestControllerIntegrationTest {

    @Autowired
    private MockMvc mvc;

    @Autowired
    private EmployeeRepository repository;

    // write test cases here
}
```

当我们需要启动整个容器时，`@SpringBootTest`注解是很有用的。这个注解会创建测试用例中需要的应用上下文（ApplicationContext）。

我们可以`@SpringBootTest`注解的`webEnvironment`属性来配置运行时环境；我们可以在这里使用`WebEnvironment.MOCK`，这样整个容器会以模拟servlet 环境来运行。

然后，`@TestPropertySource`注解帮助我们配置在测试用例中使用的配置文件地址。需要注意的是，这个注解配置的配置文件会覆盖存在的`application.properties`配置文件。

`application-integrationtest.properties`该配置文件包含持久层存储的配置细节：

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



