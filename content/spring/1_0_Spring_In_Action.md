# 1.1 Spring In Action

    @RequestMapping(path="/design",                      // <1>
                produces="application/json")
    @CrossOrigin(origins="*")        // <2>
# PART1 FOUNDATIONAL SPRING
## 1 Getting started with Spring
### DI

At its core, Spring offers a container, often referred to as the `Spring application context`, that create and manages application components.
The action of writting beans together is based on a pattern known as `dependency injection`(DI).
Historically, the way you would guide Spring's application context to write beans together was with one or more XML files that described the components and their relationship to other components.

```xml
<bean id="inventoryService" class="com.example.InventoryService" />

<bean id="productService" class="com.example.ProductService">
    <constructor-arg ref="inventoryService" />	
</bean>
```

In recent version of Spring, however, a Java-based configuration is more common. The following Java-based configuration class is equivalent to the XML configuration:

```java
@Configuration
public class ServiceConfiguration {
	@Bean
	public InventoryServiceIn inventoryService() {
		return new InventoryService();
	}
	
	@Bean
	public ProductService productService() {
		return new ProductService(InventoryService());
	}
}
```

### The Spring Initializr

The Spring Initializr is both a browser-based web application and a REST API, which can produce a skeleton Spring project structure that you can flesh out with whatever functionality you want. Several ways to use Spring Initializr follow:

* From the web application at http://start.spring.io
* From the command line using the curl command
* From the command line using the Spring Boot command-line interface
* When creating a new project with Spring Tool Suite
* When creating a new project with IntelliJ IDEA
* When creating a new project with NetBeans

### Test

Testing is an important part of software development. Recognizing this, the Spring
Initializr gives you a test class to get started. The following listing shows the baseline
test class.

```java
package tacos;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class) // Uses the Spring runner
@SpringBootTest // A Spring Boot test
public class TacoCloudApplicationTests {
    @Test // The test method
    public void contextLoads() {
    }
}
```

### Testing the controller

```java
package tacos;
import static org.hamcrest.Matchers.containsString;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.view;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;

@RunWith(SpringRunner.class)
@WebMvcTest(HomeController.class) // Web test for HomeController
public class HomeControllerTest {
    @Autowired
    private MockMvc mockMvc; // Injects MockMvc	

    @Test
    public void testHomePage() throws Exception {
        mockMvc.perform(get("/"))      // Performs GET /
        .andExpect(status().isOk())    // Expects HTTP 200
        .andExpect(view().name("home")) // Expects home view
        .andExpect(content().string(    // Expects Welcome to...
        containsString("Welcome to...")));
    }
}
```
## 2 Developing web applications

### Validating form input

Spring supports Java's Bean Validation API(also known as JSR-303;https://jcp.org/en/jsr/detail?id=303)



## 3 Working with data

### Working with JdbcTemplate

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
```

For development purposes, an embedded database will be just fine. I've added the following dependency to the build:

```
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>
```

Domain

```java
@Data
@RequiredArgsConstructor
public class Ingredient {
    private final String id;
    private final String name;
    private final Type type;

    public static enum Type {
        WRAP, PROTEIN, VEGGIES, CHEESE, SAUCE
    }
}
```

Interface

```
public interface IngredientRepository {
    Iterable<Ingredient> findAll();
    Ingredient findOne(String id);
    Ingredient save(Ingredient ingredient);
}

```

implement

```java
public class JdbcIngredientRepository implements IngredientRepository {
    private JdbcTemplate jdbc;

    @Autowired
    public JdbcIngredientRepository(JdbcTemplate jdbc) {
        this.jdbc = jdbc;
    }

    @Override
    public Iterable<Ingredient> findAll() {
        return jdbc.query("select id, name, type, from Ingredient where id=?",
                this::mapRowToIngredient);
    }

    @Override
    public Ingredient findOne(String id) {
        return jdbc.queryForObject("select id, name, type, from Ingredient where id=?",
                new RowMapper<Ingredient>() {
                    @Override
                    public Ingredient mapRow(ResultSet rs, int i) throws SQLException {
                        return new Ingredient(
                                rs.getString("id"),
                                rs.getString("name"),
                                Ingredient.Type.valueOf(rs.getString("type"))
                        );
                    }
                }, id);
    }

    @Override
    public Ingredient save(Ingredient ingredient) {
        jdbc.update("insert into Ingredient (id, name, type) values (?,?,?)",
                ingredient.getId(),
                ingredient.getName(),
                ingredient.getType().toString());
        return ingredient;
    }

    private Ingredient mapRowToIngredient(ResultSet rs, int rowNum) throws SQLException {
        return new Ingredient(
                rs.getString("id"),
                rs.getString("name"),
                Ingredient.Type.valueOf(rs.getString("type"))
        );
    }
}
```

### Persisting data with Spring Data JPA

The Spring Data project is a rather large umbrella project comprised of several sub-projects, most of which are focused on data persistence with a variety of different data-base types. A few of the most popular Spring Data projects include these:

* Spring Data JPA—JPA persistence against a relational database
* Spring Data MongoDB—Persistence to a Mongo document database
* Spring Data Neo4j—Persistence to a Neo4j graph database
* Spring Data Redis—Persistence to a Redis key-value store
* Spring Data Cassandra—Persistence to a Cassandra database

#### Annotating the domain as entities

```java
@Data
@RequiredArgsConstructor
@NoArgsConstructor(access = AccessLevel.PRIVATE, force = true)
@Entity
public class Ingredient {
    @Id
    private final String id;
    private final String name;
    private final Type type;

    public static enum Type {
        WRAP, PROTEIN, VEGGIES, CHEESE, SAUCE
    }
}
```

## 4 Securing Spring

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

By doing nothing more than adding the security starter to the project build, you get the following security features:

* All HTTP request paths require authentication.
* No specific roles or authorities are required.
* There’s no login page.
* Authentication is prompted with HTTP basic authentication.
* There’s only one user; the username is user.

You’ll need to at least configure Spring Security to do the following:

* Prompt for authentication with a login page, instead of an HTTP basic dialog box.
* Provide for multiple users, and enable a registration page so new Taco Cloud customers can sign up.
* Apply different security rules for different request paths. The homepage and registration pages, for example, shouldn’t require authentication at all.

To get started, you’ll ease into it by writing the barebones configuration class shown in the following listing.

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
}
```

As it turns out, Spring Security offers several options for configuring a user store, including these:

* An in-memory user store
* A JDBC-based user store
* An LDAP-backed user store
* A custom user details service

No matter which user store you choose, you can configure it by overriding a configure() method defined in the WebSecurityConfigurerAdapter configuration base class. To get started, you’ll add the following method override to the SecurityConfig class:

`An in-memory user store`

```java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.inMemoryAuthentication()
          .withUser("buzz")
            .password("infinity")
            .authorities("ROLE_USER")
        .and()
          .withUser("woody")
            .password("bullseye")
            .authorities("ROLE_USER");
}
```

The in-memory user store is convenient for testing purposes or for very simple applications, but it doesn’t allow for easy editing of users. If you need to add, remove, or change a user, you’ll have to make the necessary changes and then rebuild and redeploy the application.

`A JDBC-based user store`

```java
public static final String DEF_USERS_BY_USERNAME_QUERY =
    "select username,password,enabled " +
    "from users " +
    "where username = ?";

public static final String DEF_AUTHORITIES_BY_USERNAME_QUERY =
    "select username,authority " +
    "from authorities " +
    "where username = ?";

public static final String DEF_GROUP_AUTHORITIES_BY_USERNAME_QUERY =
    "select g.id, g.group_name, ga.authority " + 
    "from groups g, group_members gm, group_authorities ga " +
    "where gm.username = ? " +
    "and g.id = ga.group_id " + 
    "and g.id = gm.group_id";

@Autowired
DataSource dataSource;

@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.jdbcAuthentication()
        .dataSource(dataSource)
        .usersByUsernameQuery(DEF_USERS_BY_USERNAME_QUERY)
        .authoritiesByUsernameQuery(DEF_AUTHORITIES_BY_USERNAME_QUERY)
        .passwordEncoder(new StandardPasswordEncoder("53cr3t"));
}
```

The passwordEncoder() method accepts any implementation of Spring Security’s PasswordEncoder interface. Spring Security’s cryptography module includes several such implementations:

* BCryptPasswordEncoder —Applies bcrypt strong hashing encryption
* NoOpPasswordEncoder —Applies no encoding
* Pbkdf2PasswordEncoder —Applies PBKDF2 encryption
* SCryptPasswordEncoder —Applies scrypt hashing encryption
* StandardPasswordEncoder —Applies SHA-256 hashing encryption

The preceding code uses StandardPasswordEncoder . But you can choose any of the other implementations or even provide your own custom implementation if none of the out-of-the-box implementations meet your needs. The PasswordEncoder interface is rather simple:

```java
public interface PasswordEncoder {
    String encode(CharSequence rawPassword);
    boolean matches(CharSequence rawPassword, String encodedPassword);
}
```

No matter which password encoder you use, it’s important to understand that the password in the database is never decoded. Instead, the password that the user enters at login is encoded using the same algorithm, and it’s then compared with the encoded password in the database. That comparison is performed in the Password-Encoder ’s matches() method.

`An LDAP-backed user store`

To configure Spring Security for LDAP-based authentication, you can use the ldap-Authentication() method. This method is the LDAP analog to jdbcAuthentication() .The following configure() method shows a simple configuration for LDAP authen-tication:

```java
@Override
protected void configure(AuthenticationManagerBuilder auth)
throws Exception {
    auth
        .ldapAuthentication()
        .userSearchFilter("(uid={0})")
        .groupSearchFilter("member={0}");
}
```

`A custom user details service`

1. DEFINING THE USER DOMAIN AND PERSISTENCE

   ```java
   @Entity
   @Data
   @NoArgsConstructor(access = AccessLevel.PRIVATE, force = true)
   @RequiredArgsConstructor
   public class User implements UserDetails {
   
    private static final long serialVersionUID = 1L;
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
   
    private final String username;
    private final String password;
    private final String fullname;
    private final String street;
    private final String city;
    private final String state;
    private final String zip;
    private final String phoneNumber;
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return Arrays.asList(new SimpleGrantedAuthority("ROLE_USER"));
    }
   
    @Override
    public String getPassword() {
        return password;
    }
   
    @Override
    public String getUsername() {
        return username;
    }
   
    @Override
    public boolean isAccountNonExpired() {
        return true;
    }
   
    @Override
    public boolean isAccountNonLocked() {
        return true;
    }
   
    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }
   
    @Override
    public boolean isEnabled() {
        return true;
    }
   }
   ```

   With the User entity defined, you now can define the repository interface:

   ```java
   public interface UserRepository extends CrudRepository<User, Long> {
   	User findByUsername(String username);
   }
   ```

   

2. CREATING A USER - DETAILS SERVICE

   ```java
   @Service
   public class UserRepositoryUserDetailsService implements UserDetailsService {
       private UserRepository userRepo;
   
       @Autowired
       public UserRepositoryUserDetailsService(UserRepository userRepo) {
           this.userRepo = userRepo;
       }
       @Override
       public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
           User user = userRepo.findByUsername(username);
           if (user !=null) {
               return user;
           }
           throw  new UsernameNotFoundException("User '" + username + "' not found");
       }
   }
   ```

   You do, however, still need to configure your custom user details service with Spring Security. Therefore, you’ll return to the configure() method once more:

   ```java
   @Configuration
   @EnableWebSecurity
   public class SecurityConfig extends WebSecurityConfigurerAdapter {
       @Autowired
       private UserRepositoryUserDetailsService userDetailsService;
   
       @Override
       protected void configure(AuthenticationManagerBuilder auth) throws Exception {
           auth.userDetailsService(userDetailsService)
                   .passwordEncoder(encoder());
       }
   
       @Override
       protected void configure(HttpSecurity http) throws Exception {
           http.authorizeRequests()
                   .antMatchers("/design", "/orders")
                   .hasRole("ROLE_USER")
                   .antMatchers("/", "/**").permitAll();
       }
   
       @Bean
       public PasswordEncoder encoder() {
           return new StandardPasswordEncoder("53cr3t");
       }
   }
   ```

   

3. REGISTERING USERS

   ```java
   @Data
   public class RegistrationForm {
       private String username;
       private String password;
       private String fullname;
       private String street;
       private String city;
       private String state;
       private String zip;
       private String phone;
   
       public User toUser(PasswordEncoder passwordEncoder) {
           return new User(
                   username, passwordEncoder.encode(password),
                   fullname, street, city, state, zip, phone);
       }
   }
   ```

### Securing web requests

Among the many things you can con-
figure with HttpSecurity are these:

* Requiring that certain security conditions be met before allowing a request to be served
* Configuring a custom login page
* Enabling users to log out of the application
* Configuring cross-site request forgery protection

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
        .antMatchers("/design", "/orders")
        .hasRole("ROLE_USER")
        .antMatchers("/", "/**").permitAll();
}
```

In this case, you specify two security rules:

* Requests for /design and /orders should be for users with a granted authority of ROLE_USER .
* All requests should be permitted to all users.

Table 4.1 Configuration methods to define how a path is to be secured

| Method | What it does |
| ------ | ------------ |
|access(String)|Allows access if the given SpEL expression evaluates to true|
|anonymous() |Allows access to anonymous users|
|authenticated() |Allows access to authenticated users|
|denyAll() |Denies access unconditionally|
|fullyAuthenticated() |Allows access if the user is fully authenticated (not remembered)|
|hasAnyAuthority(String...) |Allows access if the user has any of the given authorities|
|hasAnyRole(String...) |Allows access if the user has any of the given roles|
|hasAuthority(String) |Allows access if the user has the given authority|
|hasIpAddress(String) |Allows access if the request comes from the given IP address|
|hasRole(String) |Allows access if the user has the given role|
|not() |Negates the effect of any of the other access methods|
|permitAll() |Allows access unconditionally|
|rememberMe() |Allows access for users who are authenticated via remember-me|

Table 4.2 Spring Security extensions to the Spring Expression Language

| Security expression | What it evaluates to |
| ------------------- | -------------------- |
|authentication| The user’s authentication object|
|denyAll |Always evaluates to false|
|hasAnyRole(list of roles) |true if the user has any of the given roles|
|hasRole(role) |true if the user has the given role|
|hasIpAddress(IP address) |true if the request comes from the given IP address|
|isAnonymous() |true if the user is anonymous|
|isAuthenticated() |true if the user is authenticated|
|isFullyAuthenticated() |true if the user is fully authenticated (not authenticated with remember-me)|
|isRememberMe() |true if the user was authenticated via remember-me|
|permitAll |Always evaluates to true|
|principal |The user’s principal object|

This sets up a security filter that intercepts POST requests to /logout. Therefore, to provide logout capability, you just need to add a logout form and button to the views in your application:

```java
.and()
.logout()
.logoutSuccessUrl("/")
```

When the user clicks the button, their session will be cleared, and they will be logged out of the application. By default, they’ll be redirected to the login page where they can log in again. But if you’d rather they be sent to a different page, you can call logoutSucessFilter() to specify a different post-logout landing page.

#### Preventing cross-site request forgery

It’s possible to disable CSRF support, but I’m hesitant to show you how. CSRF protec-tion is important and easily handled in forms, so there’s little reason to disable it. But if you insist on disabling it, you can do so by calling disable() like this:

```java
.and()
	.csrf()
		.disable()
```

Again, I caution you not to disable CSRF protection, especially for production applications.

### Knowing your user

To achieve the desired connection between an Order entity and a User entity, you need to add a new property to the Order class:

```java
@Data
@Entity
@Table(name="Taco_Order")
public class Order implements Serializable {
    ...
    @ManyToOne
    private User user;
    ...
}
```

There are several ways to determine who the user is. These are a few of the most
common ways:

* Inject a Principal object into the controller method
* Inject an Authentication object into the controller metho
* Use SecurityContextHolder to get at the security context
* Use an @AuthenticationPrincipal annotated method

For example, you could modify processOrder() to accept a java.security.Principal as a parameter. You could then use the principal name to look up the user from a UserRepository :

```java
@PostMapping
public String processOrder(@Valid Order order, Errors errors,
SessionStatus sessionStatus,
Principal principal) {
    ...
    User user = userRepository.findByUsername(
    principal.getName());
    order.setUser(user);
    ...
}
```

This works fine, but it litters code that’s otherwise unrelated to security with security code. You can trim down some of the security-specific code by modifying processOrder() to accept an Authentication object as a parameter instead of a Principal :

```java
@PostMapping
public String processOrder(@Valid Order order, Errors errors,
SessionStatus sessionStatus,
Authentication authentication) {
    ...
    User user = (User) authentication.getPrincipal();
    order.setUser(user);
    ...
}
```

With the Authentication in hand, you can call getPrincipal() to get the principal object which, in this case, is a User . Note that getPrincipal() returns a java.util.Object , so you need to cast it to User .
Perhaps the cleanest solution of all, however, is to simply accept a User object inprocessOrder() , but annotate it with @AuthenticationPrincipal so that it will bethe authentication’s principal:

```java
@PostMapping
public String processOrder(@Valid Order order, Errors errors,
SessionStatus sessionStatus,
@AuthenticationPrincipal User user) {
    if (errors.hasErrors()) {
    return "orderForm";
    }
    order.setUser(user);
    orderRepo.save(order);
    sessionStatus.setComplete();
    return "redirect:/";
}
```

There’s one other way of identifying who the authenticated user is, although it’s a bit messy in the sense that it’s very heavy with security-specific code. You can obtain an Authentication object from the security context and then request its principal like this:

```java
Authentication authentication =
SecurityContextHolder.getContext().getAuthentication();
User user = (User) authentication.getPrincipal();
```

### Summary
* Spring Security autoconfiguration is a great way to get started with security, but most applications will need to explicitly configure security to meet their unique security requirements.
* User details can be managed in user stores backed by relational databases, LDAP, or completely custom implementations.
* Spring Security automatically protects against CSRF attacks.
* Information about the authenticated user can be obtained via the Security-Context object (returned from SecurityContextHolder.getContext() ) or injected into controllers using @AuthenticationPrincipal .

# PART2 INTEGRATED SPRING
## 5 Working with configuration properties
## 6 Creating REST service
### Enabling hypermedia

Hardcoding API URLs and using string manipulation on API client code makes the client code brittle. Hypermedia as the Engine of Application State, or HATEOAS, is means of creating self-describing APIs wherein resource returned from an API contain links to related resource. To enable hypermedia in the Application API, you'll need to add the spring HATEOAS starter dependency to the build:
    
    spring-boot-starter-hateoas


### Enabling data-backed services

To start using Spring Data REST, artu add the following dependency to your build:

    spring-boot-starter-data-restart

## 7 Consuming REST service
art
art API with
art
* RestTemplate-
* Traversion-A hyperlink-aware, synchronous REST client provided by Spring HATEOAS. Inspired from a JavaScript library of the same name.
* WebClient

### 

### Navigating REST APIs with Traversion

## 8 Sending messages asynchronously

### 8.1 Sending messages with JMS

Before you can use JMS, you must add a JMS client to your project’s build. With Spring Boot, that couldn’t be any easier. All you need to do is add a starter dependency to the build. First, though, you must decide whether you’re going to use Apache ActiveMQ, or the newer Apache ActiveMQ Artemis broker.
If you’re using ActiveMQ, you’ll need to add the following dependency to your project’s pom.xml file:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-activemq</artifactId>
</dependency>
```

If ActiveMQ Artemis is the choice, the starter dependency should look like this:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-artemis</artifactId>
</dependency>
```

Table 8.1 Properties for configuring the location and credentials of an Artemis broker

| Property | Description |
| -------- | ----------- |
|spring.artemis.host| The broker’s host|
|spring.artemis.port| The broker’s port|
|spring.artemis.user| The user to use to access the broker (optional)|
|spring.artemis.password| The password to use to access the broker (optional)|

For example, consider the following entry from an application.yml file that might be
used in a non-development setting:

```yaml
spring:
	artemis:
        host: artemis.tacocloud.com
        port: 61617
        user: tacoweb
        password: l3tm31n
```

If you were to use ActiveMQ instead of Artemis, you’d need to use the ActiveMQ-specific properties listed in table 8.2.
Table 8.2 Properties for configuring the location and credentials of an ActiveMQ broker

| Property | Description |
| -------- | ----------- |
|spring.activemq.broker-url| The URL of the broker|
|spring.activemq.user| The user to use to access the broker (optional)|
|spring.activemq.password| The password to use to access the broker (optional)|
|spring.activemq.in-memory| Whether or not to start an in-memory broker (default: true)|
Notice that instead of offering separate properties for the broker’s hostname and port, an ActiveMQ broker’s address is specified with a single property, spring.activemq.broker-url . The URL should be a tcp:// URL, as shown in the following YAML snippet:

```yaml
spring:
    activemq:
        broker-url: tcp://activemq.tacocloud.com
        user: tacoweb
        password: l3tm31n
```

Whether you choose Artemis or ActiveMQ, you shouldn’t need to configure these properties for development when the broker is running locally. 

If you’re using ActiveMQ, you will, however, need to set the spring.activemq.in-memory property to false to prevent Spring from starting an in-memory broker.An in-memory broker may seem useful, but it’s only helpful when you’ll be consuming messages from the same application that publishes them (which has limited usefulness).
Instead of using an embedded broker, you’ll want to install and start an Artemis(or ActiveMQ) broker before moving on. Rather than repeat the installation instructions here, I refer you to the broker documentation for details:
* Artemis—https://activemq.apache.org/artemis/docs/latest/using-server.html
* ActiveMQ—http://activemq.apache.org/getting-started.html#GettingStarted-Pre-InstallationRequirements

With the JMS starter in your build and a broker waiting to ferry messages from one application to another, you’re ready to start sending messages.

#### Sending messages with JmsTemplate

JmsTemplate is the centerpiece of Spring’s JMS integration support. Much like Spring’s other template-oriented components, JmsTemplate eliminates a lot of boiler-plate code that would otherwise be required to work with JMS. Without JmsTemplate ,you’d need to write code to create a connection and session with the message broker, and more code to deal with any exceptions that might be thrown in the course of sending a message. JmsTemplate focuses on what you really want to do: send a message.
JmsTemplate has several methods that are useful for sending messages, including the following:

```java
// Send raw messages
void send(MessageCreator messageCreator) throws JmsException;
void send(Destination destination, MessageCreator messageCreator) throws JmsException;
void send(String destinationName, MessageCreator messageCreator) throws JmsException;
// Send messages converted from objects
void convertAndSend(Object message) throws JmsException;
void convertAndSend(Destination destination, Object message) throws JmsException;
void convertAndSend(String destinationName, Object message) throws JmsException;
// Send messages converted from objects with post-processing
void convertAndSend(Object message, MessagePostProcessor postProcessor) throws JmsException;
void convertAndSend(Destination destination, Object message, MessagePostProcessor postProcessor) throws JmsException;
void convertAndSend(String destinationName, Object message, MessagePostProcessor postProcessor) throws JmsException;
```

As you can see, there are really only two methods, send() and convertAndSend() ,each overridden to support different parameters. And if you look closer, you’ll notice that the various forms of convertAndSend() can be broken into two subcategories. In trying to understand what all of these methods do, consider the following breakdown:
* Three send() methods require a MessageCreator to manufacture a Message object.
* Three convertAndSend() methods accept an Object and automatically convert that Object into a Message behind the scenes.
* Three convertAndSend() methods automatically convert an Object to a Message , but also accept a MessagePostProcessor to allow for customization of the Message before it’s sent.

Moreover, each of these three method categories is composed of three overriding methods that are distinguished by how the JMS destination (queue or topic) is specified:
* One method accepts no destination parameter and sends the message to a default destination.
* One method accepts a Destination object that specifies the destination for the message.
* One method accepts a String that specifies the destination for the message by name.

```java
private JmsTemplate jms;

@Autowired
public JmsOrderMessagingService(JmsTemplate jms) {
    this.jms = jms;
}

@Override
public void sendOrder(Order order) {
    jms.send(new MessageCreator() {
        @Override
        public Message createMessage(Session session) throws JMSException {
            return session.createObjectMessage(order);
        }
    });
}
```





```java
@Override
public void sendOrder(Order order) {
    jms.send(session -> session.createObjectMessage(order));
}
```

### 8.2 Working with RabbitMQ and AMQP

### 8.3 Messaging with Kafka

### 8.4 Summary
* Asynchronous messaging provides a layer of indirection between communicating applications, which allows for looser coupling and greater scalability.
* Spring supports asynchronous messaging with JMS, RabbitMQ, or Apache Kafka.
* Applications can use template-based clients ( JmsTemplate , RabbitTemplate , or KafkaTemplate ) to send messages via a message broker.
* Receiving applications can consume messages in a pull-based model using the same template-based clients.
* Messages can also be pushed to consumers by applying message listener annotations ( @JmsListener , @RabbitListener , or @KafkaListener ) to bean methods.

## 9 Integrating Spring

# PART3 REACTIVE SPRING

## Introducing Reactor
As we develop application code, there are two style of code we can write: imperative and reactive:

* Imperative code is a lot like that absurd hypothetical newspaper subscription. It's a serial set of tasks, each running one at a time, each after the previous task. Date is processed in bulk and can't be handed over to the next task until the previous task has completed its work on the bulk of data.
* Reactive code is a lot like a real newspaper subscription. A set of tasks is defined to process data, but those tasks can run in parallel. Each task can process subsets of the data, handing it off to the next task in line while it continues to work on another subset of the data.



# PART4 CLOUD-NATIVE SPRING

# PART5 DEPLOYED SPRING


