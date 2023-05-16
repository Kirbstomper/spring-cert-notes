# Spring Notes

## Section 1 – Spring Core 
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
### 1.6.4 Explain different types of Advice and when to use them 




## Section 2 – Data Management
## Objective 2.1 Introduction to Spring JDBC
### 2.1.1 Use and configure Spring’s JdbcTemplate
### 2.1.2 Execute queries using callbacks to handle result sets
### 2.1.3 Handle data access exceptions
## Objective 2.2 Transaction Management with Spring
### 2.2.1 Describe and use Spring Transaction Management
### 2.2.2 Configure Transaction Propagation
### 2.2.3 Setup Rollback rules
### 2.2.4 Use Transactions in Tests
## Objective 2.3 Spring Boot and Spring Data for Backing Stores
### 2.3.1 Implement a Spring JPA application using Spring Boot
### 2.3.2 Create Spring Data Repositories for JPA