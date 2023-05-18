# Spring Notes

# Section 1 – Spring Core 
### Objective 1.1 Introduction to Spring Framework 
- The Spring Framework is an open source, lightweight DI(Dependency injection) Container and Framework for building Java enterprise applications
- Apache 2 License
- Does not require Java EE application server
- Not Invasive. Implement interfaces and write code as POJOS
- Low overhead
- Serves as a lifecycle manager
- DI also called IOC (inversion of control)


## Objective 1.2 Java Configuration 
### 1.2.1 Define Spring Beans using Java code 
- Use @Configuration annotation and define @Beans inside that configuration
- pass configuration into argument of `Application.Run()`

1.2.2 Access Beans in the Application Context 

- Programmatically using an instance of application context

- `TransferService transferService = (TranserService) context.getBean(“transferService”)`
- `TransferService transferService = context.getBean(“transferService”, TransferService.class)`
- If type is unique, no need for bean id `TransferService transferService = context.getBean(TransferService.class)`

### 1.2.3 Handle multiple Configuration files 
- Use the `@Import` annotation to source beans from multiple configurations
- Seperation of concerns, keeps application code and infastructure code seperate as infrastructure may change depending on environment
### 1.2.4 Handle Dependencies between Beans 
- Use @Autorwired to inject beans defined elsewhere
- Avoid tramp beans, pass around actual dependency
### 1.2.5 Explain and define Bean Scopes 
- Scopes are...
- Singleton: One instance in application context used by all
- Singletons should be created with multithreading in mind. Shoud be either stateless, synchronized, or not singleton at all
- Prototype: New instance created everytime bean is referenced
- Session: New bean created once per user session (web environment only)
- Request: New bean created once per request (web environment only)
- Web Socket:
- Refresh Scope:
- Thread Scope: 




## Objective 1.3 Properties and Profiles 
### 1.3.1 Use External Properties to control Configuration 
- Environment:This Bean represents loaded properties from runtime environemnt
    - JVM System Properties `System.getProperty()`
    - System Environmental Variables `System.getenv()`
    - Java Properties files
- Environment bean can also get properties form property sources
    - `@PropertySource` annotation 
    - Prefixed with `classpath:, file:, or  http:`
-  `@Value` Annotation
    - Inject values from the environment without directly referencing the environment
### 1.3.2 Demonstrate the purpose of Profiles
- Profiles can be used to represent different
    - Environments (dev, test, uat, prod)
    - Implementation (jdbc, jpa)
    - Deployment platorm (On pre, cloud)
    
    Beans are included or excluded based on profile membership
- `@Profile` Annotation
- Profile must be activated at runtime
    - Via command line property

    `Dspring.profifles.active`
    - Progmatically
    
    `System.setProperty('spring.profiles.active', 'profile')`
    - Integration test only `@ActiveProfiles`
- `@Profile` can be combined with `@PropertySource` to specify configuration files per profile
### 1.3.3 Use the Spring Expression Language (SpEL) 
- 
## Objective 1.4 Annotation-Based Configuration and Component Scanning 
### 1.4.1 Explain and use Annotation-based Configuration 
- Annotating our business logic classes to be spring components `@Component`,  `@ComponentScan`
- Constructor Injection: Recommended best practice
    - Mandatory Dependencies
    - Dependencies can be immutabele
    - Consice (Pass several params at once)
- Method Injection: Also works, but not recommended
    - Circular Dependencies Possible
    - Dependencies are mutable
    - Could be verbose for several params
    - Inherited Automatically
- Field Injection: hard to unit test
- Beans to be autowired are required by default, but the `required` property can be set to false to avoid application throwing exception when bean is unavailible

    We can also use Optionals in java 8
- If two beans are possible canidates for autowiring, an exception will be thrown

    Use `@Qualifier` in this scenario. Only when two beans both implement the same interface
- Spring Will look for dependencies in the following order
    - Look for unique bean of required type
    - Use Qualifier if present
    - Try to find matching bean by name
- You can use `@Value` to set attributes
- Beans are initialzied normally on startup unless annotated as `@Lazy` then they will be intialized when dependency injected or `getBean()` is used

### 1.4.2 Discuss Best Practices for Configuration choices 
- `@Autowired` is not required if you have a default constuctor as there is nothing to annotate, or if you only have one non-default constructor. If you have more than one constuctor spring will use the zero argument constuctor by default and you have to specify which one you want used with the annotation.
- Component Scanning should be optimized to only include packages where you have components. BE SPECIFIC, otherwise scanning could take a long time and effect application startup times, as everything in the classpath is scanned
- When should you use Annotations vs Java Config?
    - Annotations: Sometimes no choice!. When using Sterotypes
    - Java Config: 
        - When it is desired to keep beans decoupled from spring due to legacy code or code used outside of spring runtime
        - When managing configuarion in a single place is an important issue
        - Can be used for all classes not just your own.

### 1.4.3 Use @PostConstruct and @PreDestroy 
- `@PostConstruct`
    - Called at startup after all dependencies have been injected
    - called after setter injection is preformed
- `@PreDestroy`
    - Called at shutdown prior to destroying bean instance
    - Only called if application shutsdown normally, won't happen if process dies or is killed
    - Useful for releasing resources and cleaning up
    - Not called by prototype beans

They can have any visibility, but must be void and take no arguments
These are NOT spring annotations, JSR-250/ `javax.annotation` . Spring just supports them
- Beans have access to setting lifecycle methods via property. Used for classes you don't write
    - `initMethod`
    - `destroyMethod`
### 1.4.4 Explain and use “Stereotype” Annotations 
- Specialized annotations used to denote a role used by spring, many are composite annotations including components. 
- Examples are `@Controller`, `@Repository`, `@Service`, `@Configuration`, `@Controller`, `@RestController`,etc
- Component scanning will scan for Sterotypes too
- They are meta-annotations, so you can create your own annotations using them too




## Objective 1.5 Spring Bean Lifecycle 
### 1.5.1 Explain the Spring Bean Lifecycle
- Initialization
    - Spring beans are created
    - Dependency injection occurs
    - Step 1: Load and process bean definitions
        - For every bean find and create its dependencies
        - Instanciate bean
        - Preform setter injection
        - Bean post processors
        - Bean is ready for use
    - Step 2: Preform bean creation
- Usage
    - Beans are availible for use in the application
- Destruction
    - Beans are released for garbage collection

- Bean initializer is an extension point
### 1.5.2 Use a BeanFactoryPostProcessor and a BeanPostProcessor 
- `BeanFactoryPostProcessor`
    - Applies transformations to bean definitions before objects are even created
    - Several Implementations provided by spring (reading properties, registering a custom scope)
    - You can write your own (not common), functional interface with single method `postProcessBeanFactory` Because this needs to run before everything else, define your bean as `static`

    - PropertiesSourcesPlaceholderConfigurer
        - Usually don't have to set this up yourself
        - if on spring 4.2 or below, you have to create one manually
        - You can also create one if you want to ignore certain propety sources
- `BeanPostProcessor`
    - Can modify bean instances in ANY way
    - Powerful enabling feature
    - Will run against every bean
    - Can modify a bean before (`BeforeInit`) and/or after(`AfterInit`) initiatilization 
    - Spring has a couple of implmentation, or you can write your own if you implment `BeanPostProcessor`
    - If implementing, make sure you return your bean after each of these methods or else you will lose it!

### 1.5.3 Explain how Spring proxies add behavior at runtime 
- Spring wraps your implementation in a proxy if you want some extra behavior around it
- Created during initialization phase by a `BeanPostProcessor`
- Adds behavior to your bean transparently
- Diffrent types of proxy
    - JDK Proxy (Interface based)
        - Also called dynamic proxies
        - API built into JDK
        - Requirements :(Java Interfaces)
        - ALL Interfaces proxied
    - CGLib Proxy (Subclass based)
        - NOT built into JDK
        - Included in Spring jars
        - Used when interface is not availible
        - Cannot be applied to final classes or methods
    
    SPRING BOOT ALWAYS GOES TO CGLib OVER JDK PROXY

### 1.5.4 Describe how Spring determines bean creation order 
- Step 1: Evaluate Dependencies for each bean
- Step 2: Get each dependency needed -> Create if need be -> See step 2
- You can force creation order using `@DependsOn`
### 1.5.5 Avoid issues when Injecting beans by type (SOLUTIONS)
- Return the implementation type, to give more information to the container
- Return a composite interface. Same as above, gives more information to container
- Aim to be "Sufficiently Expressive"
    - Retrun interfaces unless..
        - Multiple interfaces exist AND they are needed for dependency injection
    - Writing to interfaces is good practice

- Even if you use implementation types
    - Still use interfaces when injecting dependencies
    - Injecting implementation types is brittle, Dependency injection may fail if bean is proxied or a diffrent implementation is returned

## Objective 1.6 Aspect Oriented Programming 
### 1.6.1 Explain the concepts behind AOP and the problems that it solves 
- AOP enables modularization of cross-cutting concerns
    - Lets the code for a cross-cutting concern exist in a single place, in a module. In Java a class represents this module
    - Cross-Cutting Concern: General Functionality needed in many places in your application
        - Logging/Tracing
        - Transaction Management
        - Security
        - Error Handling
        - Custom Business Rules
        - Caching
        - Preformance Monitoring
    - Failure to modularize results in code tangling (coupling of concerns with non-concern code) and code scattering (Same concern in multiple modules)

- Join Point: A point of execution in the program such as a method call or exception thrown
- Pointcut: An expression that selects one or more Join Pointss
- Advice: Code to be executed at each Join Point
- Aspect: A module that encapsulates Pointcuts and Advice
- Weaving: Technique by which aspects are combined with main code
- AOP Proxy: Enhanced class that stands in place of your original



`@Aspect` Annotatation: AspectJ annotation, not spring specific. So also annotation with `@Component`

`@Before` Defines the advice

`@EnableAspectJAutoProxy` must annotate configuraton 
`@Import` the include the aspect configuration

- Aspects are applied by
    - Spring creates a proxy, weaving aspect and target
    - Proxy implements target interface
    - All calls routed through proxy interceptor
    - Matching advice is executed
    - Target method is executed

Joinpoint Parameter provides context about intercepted point

### 1.6.2 Implement and deploy Advices using Spring AOP 
- 
### 1.6.3 Use AOP Pointcut Expressions 
- Spring AOP uses AspectJ's pointcut expression language; supports a partial subset
- `execution(methodPattern)`
- Can be chained together

    `execution(methodPattern) || execution(methodPattern)`
- Method Pattern

    `[Modifiers] ReturnType [ClassType] MethodName(Arguments) [Throws Exception Type]`
    
    Wildcards:
    - '*' matches once
    - '.." matches zero or more

    Restrict By Class:
    - Includes subclasses
    - Ignored if a different implementation is used

    Restrict by interface
    - More flexible, works if implementation changes

    Use annotations
    - Matches if annotation is present

### 1.6.4 Explain different types of Advice and when to use them 
- `@Before`
- `@AfterReturning`
    - `returning` attribute 
- `@AfterThrowing`
    - `throwing` attribute
    - Only invokes after right exception type is thrown
    - Will not stop exception from propgating, but can throw a new one
    - Use `@Around` if you want to stop from propgating
- `@After`
    - Called regardless of if an exception has been thrown or not
- `@Around`
    - Inherits from JoinPoint and adds `proceed()` method
    - Example was on caching 





# Section 2 – Data Management
## Objective 2.1 Introduction to Spring JDBC
### 2.1.1 Use and configure Spring’s JdbcTemplate
- Autowire JDBC template in
- can work with 
    - Simple Types
    - Generic Maps
    - Domain Objects
- Can `query()` with or without bind variables `?`
- Can use `update()` when inserting rows `?` still applies
    - Returns the number of rows modified
- Any non select statement should use update
### 2.1.2 Execute queries using callbacks to handle result sets
- JdbcTemplate can return each row of a resultset as a map
    - Expecting single row? `queryForMap()`
    - Expecting multiple rows? `queryForList()`
    - Useful for ad-hoc reporting/testing as not mapped to an object
- Callbacks let us handle mapping from Map to Object
    - ORM might be easier for this if object is complex
    - `RowMapper` interface: single method to impl `mapRow`
        - Works for both single and multi row queries
        - Parameterized to define return type
    - `ResultSetExtractor`
        - used for processing entire resultset at once
        - Used like an iterator
    - `RowCallbackHandler` : another handler that writes to alternative destinations
### 2.1.3 Handle data access exceptions
- Spring always throws unchecked exceptions
- `SqlException` (NOT USED BY SPRING)
    - Checked exception
    - too general, one for each database error
    - Class knows you are using JDBC
    - Tight coupling
    - can usually not be handled when thrown
- Spring throws `DataAccessException` which is unchecked
    - Hides what data library you are using
    - Actually a higharchy of sub-exceptions
    - Consistant across all data access technologies

## Objective 2.2 Transaction Management with Spring
### 2.2.1 Describe and use Spring Transaction Management
- 1: Declare a `PlatformTransactionManager` bean
- 2: Declare the transactional methods
    -  Using annotations or Programmatic
    - Can mix and match
- 3: add `@EnableTransactionManagement` to a configuration class

`@Transactional` annotation applied to methods/classes/interfaces that are atomic units of work

Spring proxies

Rollsback if method throws a runtime exception
- Transaction context bound to current thread
- JdbcTemplate used in a transactional method uses that context automatically
- Can get it manually `DataSourceUtils.getConnection(dataSource)`
- Can be combined at class and method level for overriding

### 2.2.2 Configure Transaction Propagation
- 7 levels of propogation
- REQUIRED: defaults, use current transaction. Create new one if it  does not exist
- REQUIRES_NEW: Creates a new one all the time, suspending the current transaction if one exists
- Rules enforced by a proxy, remember if a method is called internal in a class its proxy won't be used
### 2.2.3 Setup Rollback rules
- By defaults only if runtime exception
- `rollbackFor` and `noRollbackFor` attributes on `@Transactional`
### 2.2.4 Use Transactions in Tests
- Annotate test method with `@Transactional`
- Transactions will be rolled back after testing!
- use `@Commit` if you do want something to commit 
## Objective 2.3 Spring Boot and Spring Data for Backing Stores
### 2.3.1 Implement a Spring JPA application using Spring Boot
- use `spring-boot-starter-data-jpa`
- Spring autoconfigures
    - DataSource
    - EntityManagerFactoryBean
    - JPATransactionManager
- Configuration exists for dialect used, auto-creating tables, showing sql, any other hibernate proeprty
- `@EntityScan` annotation in application.java
### 2.3.2 Create Spring Data Repositories for JPA
- Annotate entity classes
    - `@Entity`
    - `@Table` for table name
    - `@Id`
    - `@GeneratedValue(strategy)`

Other annotations exist for diffrent data stores

- Extend from a repository interface 
    - `Repository<T, ID>`
- You can then define your own methods usinger finders or `@Query`
    - findBy[datamemeberofclass][op]
    - `@Query` uses query lanaguage of underlying product
- Spring will automatically scan everything from package application class exists in or you can use `@EnableJPARepositories(basePackage)`

# Section 3 – Spring MVC
## Objective 3.1 Web Applications with Spring Boot
### 3.1.1 Explain how to create a Spring MVC application using Spring Boot
- Web Servelet Apporach
    - Traditional
    - Based on Java EE servlet
    - 
- Reactive Approach
    - Newer
    - More efficient
    - non blocking
- Springboot supports embedded servlet containers
    - can also still deploy by war
- `spring-boot-starter-web`

### 3.1.2 Describe the basic request processing lifecycle for REST requests
- Request is sent to dispatcher servlet
    - handles converting the message
- Dispatch servlet then calls controller with dispatch request
- controlle returns data to the dispatch servlet
- Data is converted again, response is sent back out
- We only deal with controller and view
### 3.1.3 Create a simple RESTful controller to handle GET requests
- `@Controller`
- `@GetMapping`
- `@ResponseBody`
- `@RestController` can be used to ommit response body

- Controller method arguments like Locale, principle, http session can also be used as a parameter in requests

- `@RequestParam` gets from `?paramname`
- `@PathVariable` gets from `\pathvariableName` 
    - NO need for annotation value if parameter name matches parameter name

### 3.1.4 Configure for deployment
- Just build it, use embedded tomcat container
- To use another servlet container exclude tomcat then include it in depenencies
- If you want to run in jar/war via web container
    - extend Application class with `SpringBootServletInitializer`
    - Specify configuration classes to use
    - Define a main method (if running from jar)
- Mark Tomcat dependencies as provided when building WARS for traditional containers
- Building project by `mvn package` or `gradle assemble` produces 2 jars, one fat and one with just app code

## Objective 3.2 REST Applications
### 3.2.1 Create controllers to support the REST endpoints for various verbs
- Use diffrent verbs
- GET, PUT, POST, DELETE, PATCH
### 3.2.2 Utilize RestTemplate to invoke RESTful services
- Used for builing rest client applications
- Supports all HTTP methods
- Create using 
    - `new RestTemplate()`
    - Autowire `RestTemplateBuilder` in spring class
- You can get responseEntity instead of body if we want to deal with HTTP status codes and headers (`getForEntity()`)
- You can use RequestEntity to send requests too!
- RestTemplate is not deprecated, but it also won't evolve. WebClient is the new hotness to support new stuff like streaming


# Section 4 – Testing
## Objective 4.1 Testing Spring Applications
### 4.1.1 Write tests using JUnit 5
- JUnit Jupiter
- `@Test`
- `@BeforeEach`
- `@BeforeAll`
- `@Disabled`
### 4.1.2 Write Integration Tests using Spring
- Integration Test: Tests multiple units working together
- Unit test already show they work when apart
- Test application classes in contex of surrounding infrastructure
- Test Containers is sick
- Based on `TestContext` framework: Defines a ApplicationContext for your test to use
-`@ExtendWith(SpringExtension.class)`
- `@ContextConfiguration(classes = {TestConfig.class})`
- Then you can inject and use bean as usual! 
- `@SpringJunitConfig` combines both
- Can autowire via test method argument
- You can also include configuration as a inner class
    - `@Configuration`, `@Import` on inner class
- Multiple tests share same beans. Use `@DirtiesContext` to make new context between each test
- `@TestPropertiesSource`
    - Set `propertes` attribute
    - Set `location` attribute to pass properties file
    - Defaults to `[classname].properties`
### 4.1.3 Configure Tests using Spring Profiles
- `@ActiveProfiles` annotation
- `@Profile` annotation on beans comes into play here
### 4.1.4 Extend Spring Tests to work with Databases
- Use `@Sql` to prepopulate with a sql script before test
- Use a in-memory database (h2)
- `@Sql` on method will override class level. 
    - You can also `setExecutionPhase` property to run after the method
    - `config` attribute lets you set what happens if the script fails to run, commentPrefix and seperator
    - Default is whatever comes from `@Sql` on class level else `FAIL_ON_ERROR`
## Objective 4.2 Advanced Testing with Spring Boot and MockMVC
### 4.2.1 Enable Spring Boot testing
- Include `spring-boot-starter-test` dependency
- Built on top of spring testing framework
- `@SpringBootTest` and other annotations
- `@MockBean`
- AssertJ and Hamcrest for asserts/matching
- Mockito
- JsonASsert
- JsonPath
### 4.2.2 Perform integration testing
- Uses `@SpringBootTest`
- Autoconfigures a `TestRestTemplate`
    - Uses relative path
    - Fault tolerant so you can test errors
    - Use `RestTemplateBuilder` to customize
- Provides support for diffrent webEnvironment Modes
- Embedded server can be started by testing framework
### 4.2.3 Perform MockMVC testing
- Provides first class support for testing Spring MVC code
- Processes request through DispatcherServlet
- Without running a web container to test
- Autowire `mockMvc`
- Set webEnvironment to `MOCK`
- `MockMvcRequestBuilders` and `MockMvcRequestMatcher`
- Arguments to `mockMvc.Preform()` determine the action
    - `preform()` returns `ResultsAction` object which you can chain `expects()` on
    - `andDo()`, `andReturn()` can do things with `MvcResult` object 
### 4.2.4 Perform slice testing
- Preform isolated testing within a slice of application
- Requires mocking dependencies
- Use `@WebMvcTest(controllerName.class)` to only create beans relevant to that controller
- use `@MockBean`: Spring mock annotation, mockitos won't take the application context into account
- `@DataJpaTest`
    - Useful when a test only focuees on JPA components
    - AutoConfigures `TestEntityManager`
    - Uses an embedded im-memory database


# Section 5 – Security
## Objective 5.1 Explain basic security concepts
- Principal: User device or system that preforms an action
- Authentication: Establishing principals credentials are valid
- Authorization: Deciding if pricipal is allowed to access a resource
- Authority: Permission or credential enabling access
- Secured Resource: Resource being secured
## Objective 5.2 Use Spring Security to configure Authentication and Authorization
- Setup Filter Chain
    - 
- Configure security (Authorization) rules
- Setup web authentication

`AuthenticationProvider` and `UserDetailsService`

InMemory, Database

Can implement custom AuthenticationProvider

- Password hashing is:
    - one way
    - supports multiple encoding schemes
    - supports multiple schemees (hash is prefixed with scheme in database)
    - `DelegatingPasswordEncoder` does this all!
    - BCrypt is the default
- Enable http authentication with `httpBasic()` in filter chain
- Form login, and form logout is also configured in filter chain
## Objective 5.3 Define Method-level Security
- Spring uses AOP for method level security
- `@EnableMethodSecurity`
- `@PreAuthorize`
- `@PostAuthorize` Useful for if you need to query from database before knowing if someone can access something


# Section 6 – Spring Boot
## Objective 6.1 Spring Boot Feature Introduction
### 6.1.1 Explain and use Spring Boot features
- Opinionated view of spring platform and third party libraries
- Supports many diffrent project types
- Handles most low level setup for you
- Gives you cool stuff like health checks, containerization, embedded servers, externalized configuration

### 6.1.2 Describe Spring Boot dependency management
- Depenencies are managed through parents and starters
- You can exclude/include diffrent versions of stuff
- Spring boot parent pom determines version of spring
- Starters bring in multiple coordinated dependencies, including transitive dependencies

## Objective 6.2 Spring Boot Properties and Autoconfiguration
## 6.2.1 Describe options for defining and loading properties
- Properties files
- Environment Variables
- JVM properties
- Profiles
- Supports YAML also

- Multiple profile specific properties can be in same file use `---` to 
indicate logical file

- Loads properties in this order (older can be overwritten)
    - Default properties (specified by setting SpringApplication.setDefaultProperties).
    - @PropertySource annotations on your @Configuration classes. Please note that such property sources are not added to the Environment until the application context is being refreshed. This is too late to configure certain properties such as logging.* and spring.main.* which are read before refresh begins.
    - Config data (such as application.properties files).
    - A RandomValuePropertySource that has properties only in random.*.
    - OS environment variables.
    - Java System properties (System.getProperties()).
    - JNDI attributes from java:comp/env.
    - ServletContext init parameters.
    - ServletConfig init parameters.
    - Properties from SPRING_APPLICATION_JSON (inline JSON embedded in an environment variable or system property).
    - Command line arguments.
    - properties attribute on your tests. Available on @SpringBootTest and the test annotations for testing a particular slice of your application.
    - @TestPropertySource annotations on your tests.
    - Devtools global settings properties in the $HOME/.config/spring-boot directory when devtools is active.
## 6.2.2 Utilize auto-configuration
- `@EnableAutoConfiguration`
- Spring boot will automatically create some beans it things you need
- Use @SpringBootApplication is common
- Uses spring context and classpath to autoconfigure stufff for you
## 6.2.3 Override default configuration
- You can overrider defaults by...
    - Setting some spring boot properties
    - Explicitly define some beans so spring boot wont
    - Explicity disable some auto configuration
        - `exclude` property in `@EnableAutoConfiguration`
        - Set property `spring.autoconfigure.exclude`
    - Change dependencies and their versions
        - May be needed due to copliance
        - Vunerabilities
        - Bugs in given version
## Objective 6.3 Spring Boot Actuator
### 6.3.1 Configure Actuator endpoints
- include `spring-boot-starter-actuator`
- enabled: endpoint is created and bean exists in application context
    - Default: all endpoints enabled except for `shutdown`
- exposed: endpoint is acessable via HTTP or JMX
    - HTTP: Only `health` is exposed by default
        - http only availible when using Spring MVC, webflux, or Jersey
    - JMX: All enabled endpoints exposed by default
        - JMX only availible when `spring.jmx.enabled = true`
- Configure exposed endpoints with `management.endpoints.web.exposure.include` property

### 6.3.2 Secure Actuator HTTP endpoints
- Secure `HealthEndpoint.class` or acutator path using spring security 
### 6.3.3 Define custom metrics
- Custom metrics are measured using Micrometer classes
    - `Counter`
    - `Gauge`
    - `Timer`
    - `DistributionSummary`: Provides a count, total, and max value for its metric
- Classes are registered or created with `MeterRegistery` bean
- Custom metric names listed on `actuator/metrics` endpoint
- Hiearchital Metrics
    - Often follow a naming schem with key/value atributtes in the name followed by periods
    - ex `http.method.<method-value>.status.<status-values>`
    - Consistant naming is hard to acheive
    - Adding new attributes can break exisiting queries
- Dimensional Metrics
    - Metrics are tagged
    - ex `http?tag=method:get&tag=status:200
    - Flexable naming convention
    - ADding new attributes is easy

- You can create a metric and register it
- Or you can use annotations like `@Timed` to avoid mixxing concerns
### 6.3.4 Define custom health indicators 
- Create a class that implements `HealthIndicator`
    - implement `health()` method
- Or extend `AbstractHealthIndicator`
    - override `doHealthCheck()` method
- Use `Health() builder to return health status
    - YOu can pass details `Health().down().withDetail(details).build()`
- You can also group health indicators in configuration
    - `management.endpoint.health.group.[groupname].include=[indicators]`