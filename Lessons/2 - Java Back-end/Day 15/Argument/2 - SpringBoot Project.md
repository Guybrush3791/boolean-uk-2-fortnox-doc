# SpringBoot Project
## Create new project from [start.spring.io](https://start.spring.io/)
Create a new project through [start.spring.io](https://start.spring.io/) UI or via `cli` and give it proper name
```sh file:"New SpringBoot project download command" hlt:|${project-name}
curl https://start.spring.io/starter.zip               \
  -d type=gradle-project                               \
  -d language=java                                     \
  -d bootVersion=3.5.5                                 \
  -d baseDir=${project-name}                           \
  -d groupId=org.booleanuk                             \
  -d artifactId=${project-name}                        \
  -d name=${project-name}                              \
  -d description="Demo project for Spring Boot"        \
  -d packageName=org.booleanuk.demo                    \
  -d packaging=jar                                     \
  -d javaVersion=21                                    \
  -d dependencies=web,devtools,postgresql,data-jpa     \
  -o ${project-name}.zip
```
## Setup
### Dependencies & Properties
The *OAuth2* require *resource server* dependency, while in the property file makes sure the proper *server port* and *uri of the issuer* are in place
```gradle file:build.gradle group:setting
dependencies {
	
	// ...
	
	// oauth2 server  
	implementation 'org.springframework.boot:spring-boot-starter-oauth2-resource-server'
}
```
```yaml file:application.yaml group:setting hlt:2|4000,12|http:1
server:  
  port: 4000 # [!note] This MUST match Keycloak settings
  #...
  
spring:  
  # ...
  security:  
    oauth2:  
      client:  
        resourceserver:  
          jwt:  
            issuer-uri: http://localhost:8080/realms/springboot-realm-1 # [!note] This MUST match Keycloak settings
```

### Security
Create a class named `SecurityConfig` in package `${main-pkg}.security` and define the main security rules
```java file:SecurityConfig.java hlt:11|\"/public/**\",12,22|localhost:8080,spring:1
@Configuration  
public class SecurityConfig {  
  
    @Bean  
    SecurityFilterChain web(HttpSecurity http) throws Exception {  
  
        return http  
                .csrf(AbstractHttpConfigurer::disable)  
                .sessionManagement(sm -> sm.sessionCreationPolicy(SessionCreationPolicy.STATELESS))  
                .authorizeHttpRequests(auth -> auth  
                        .requestMatchers("/public/**").permitAll()  // [!note] Public available base URL 
						.anyRequest().authenticated()				// [!note] Everything else must be authenticated
                )  
                .oauth2ResourceServer(OAuth2ResourceServerConfigurer::jwt)  
                .build();  
    }  
  
    @Bean  
    JwtDecoder jwtDecoder() {  
    
        return NimbusJwtDecoder.withJwkSetUri( 
	        "http://localhost:8080/realms/springboot-realm-1/protocol/openid-connect/certs" // [!note] Make sure this reflect the Keycloak setting
	    ).build(); 
    }  
}
```

We now have *2 main base URL*:
- `/public/**` --> whatever start with `/public` will be available *without authentication*
- everything else --> *require authentication*

## Project Implementation
Now you have project properly tied up with *Keycloak* login, you can now define you routes accordingly to the access you want grant to each end-point

### Simple CRUD
Define a *JPA* `Entity`, `EntityDto` and `Repository` to access database in read and write modes

```java file:Employee.java group:employee
@Data  
@NoArgsConstructor  
@AllArgsConstructor  
@Entity  
@Table(name = "employees")  
public class Employee {  
  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    private int id;  
  
    private String firstName;  
    private String lastName;  
    private String location;  
    private String email;  
  
    public Employee(EmployeeDto employeeDto) {  
  
        setFirstName(employeeDto.getFirstName());  
        setLastName(employeeDto.getLastName());  
        setLocation(employeeDto.getLocation());  
        setEmail(employeeDto.getEmail());  
    }  
}
```
```java file:EmployeeDto.java group:employee
@Data  
@NoArgsConstructor  
@AllArgsConstructor  
public class EmployeeDto {  
  
    private String firstName;  
    private String lastName;  
    private String location;  
    private String email;  
}
```
```java file:EmployeeRepository.java group:employee
public interface EmployeeRepository extends JpaRepository<Employee, Integer> {  
}
```

### Service layer
The **service layer** in Spring Boot holds and orchestrates your _business logic_, acting as the bridge between *REST controllers* and *repositories*. It defines the set of operations your application exposes, coordinates transactions, and keeps controllers and persistence code decoupled.
```java file:EmployeeService hlt:1,4,5
@Service // [!note] Mark service classes as `@Service`
public class EmployeeService {  
  
    @Autowired  
    private EmployeeRepository employeeRepository; // [!note] Use repository in hire
  
    public List<Employee> getAllEmployees() {  
  
        return employeeRepository.findAll();  
    }  
  
    public Employee getEmployeeById(int id) {  
  
        return employeeRepository.findById(id).orElse(null);  
    }  
  
    public Employee save(Employee employee) {  
  
        return employeeRepository.save(employee);  
    }  
  
    public Employee delete(Employee employee) {  
  
        employeeRepository.delete(employee);  
  
        return employee;  
    }  
}
```
> [!note] You can write the database access logic in hire, and use this class, instead of `Repository` interface, in all your *controllers*. <br /> Most of the main methods are just an interface to repository's methods.

### Controller
Now it's time to define the `RestController` to access data from outside world. The best way to decide which route is protected and which one is not, is to define *2 separate controllers* and make one of them *whole public* and the other one *whole reserved* .
```java file:EmployeePublicController.java group:employee-controller hlt:2|public,5,6
@RestController  
@RequestMapping("public/employees") // [!note] Make sure the public controller is on `public` route
public class EmployeePublicController {  
  
    @Autowired  
    private EmployeeService employeeService; // [!note] Use `services` instead of `repositories` in controller for better code re-use
  
    @GetMapping  
    public List<Employee> getAllEmployees() {  
  
        return employeeService.getAllEmployees();  
    }  
  
    @GetMapping("{id}")  
    public ResponseEntity<Employee> getEmployeeById(@PathVariable int id) {  
  
        Employee e = employeeService.getEmployeeById(id);  
  
        if (e == null) throw new ResponseStatusException(HttpStatus.NOT_FOUND, "Employee not found");  
  
        return ResponseEntity.ok(e);  
    }  
}
```
```java file:EmployeeAdminController.java group:employee-controller hlt:2|employees,5,6
@RestController  
@RequestMapping("employees") // [!see] This controller is whole protected
public class EmployeeAdminController {  
  
    @Autowired  
    private EmployeeService employeeService; // [!note] Use service instead of repository in controller for better code re-use
  
    @PostMapping  
    public ResponseEntity<Employee> createEmployee(@RequestBody EmployeeDto employeeDto) {  
  
        Employee e = new Employee(employeeDto);  
        e = employeeService.save(e);  
  
        return ResponseEntity.ok(e);  
    }  
  
    @PutMapping("{id}")  
    public ResponseEntity<Employee> updateEmployee(  
            @PathVariable int id,  
            @RequestBody EmployeeDto employeeDto  
    ) {  
  
        Employee e = employeeService.getEmployeeById(id);  
  
        e.setFirstName(employeeDto.getFirstName());  
        e.setLastName((employeeDto.getLastName()));  
        e.setEmail(employeeDto.getEmail());  
        e.setLocation(employeeDto.getLocation());  
  
        return new ResponseEntity<Employee>(employeeService.save(e), HttpStatus.CREATED);  
    }  
  
    @DeleteMapping("{id}")  
    public ResponseEntity<Employee> deleteEmployee(@PathVariable int id) {  
  
        Employee e = employeeService.getEmployeeById(id);  
        employeeService.delete(e);  
  
        return ResponseEntity.ok(e);  
    }  
}
```
> [!note] With this configuration all *read* operations are accessible as *guest*, while the *write* ones are available only for *authenticated users*

---

# Links
![[Lessons/2 - Java Back-end/Day 15/__block/Links]]