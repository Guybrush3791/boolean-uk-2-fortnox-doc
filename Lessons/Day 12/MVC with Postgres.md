---
argument: SpringBoot database integration
section: springboot, database
lesson count: "1"
ex count: "1"
---
# MVC with Postgres
## LC

### Dependencies

```gradle
// postgres driver
implementation("org.postgresql:postgresql:42.7.7")

// lombok
implementation("org.projectlombok:lombok:1.18.38")
annotationProcessor 'org.projectlombok:lombok:1.18.38'
testAnnotationProcessor 'org.projectlombok:lombok:1.18.38'
```
## Lesson
### Lombok

Project Lombok is an annotation-based Java library that integrates with editors and build tools to automatically generate boilerplate code, such as getters, setters, constructors, toString(), equals(), and hashCode() methods, during compilation.[[1](https://projectlombok.org/)]

Its primary purpose is to reduce repetitive coding, minimize lines of code, enhance readability and maintainability, and allow developers to focus on business logic rather than mundane tasks, by injecting the necessary bytecode into .class files without altering the source code.[[2](https://auth0.com/blog/a-complete-guide-to-lombok/)]

Here's a list of the most commonly used Lombok annotations, ordered roughly by popularity and frequency of mention across sources:

- **@Data** a composite annotation that bundles `@Getter`, `@Setter`, `@ToString`, `@EqualsAndHashCode`, and `@RequiredArgsConstructor` for creating fully functional data classes
- **@Getter**/**@Setter** generate *getter* and *setter* methods for fields, often used individually or at the class level to avoid manual implementation
- **@Builder** implements the builder pattern for constructing complex or immutable objects in a fluent, readable way
- **@ToString** automatically generates a `toString()` method that includes field values
- **@EqualsAndHashCode** overrides `equals()` and `hashCode()` methods based on class fields
- **@Value** creates immutable classes by combining `@Getter`, `@ToString`, `@EqualsAndHashCode`, and `@AllArgsConstructor`, with fields marked as final
- **@AllArgsConstructor**/**@NoArgsConstructor**/**@RequiredArgsConstructor** generate constructors for all fields, no arguments, or required (final or `@NonNull`) fields, respectively

Less common but notable annotations include `@With` for immutable field updates and others like `@SneakyThrows` for specific scenarios. Usage depends on project needs, such as *Hibernate* integration where pitfalls like improper `equals`/`hashCode` generation should be avoided.

### Postgres in Java 
[Repository link](https://github.com/boolean-uk/java-api-mvc-with-postgres-workshop.git)
![[Repository/Day 12/Theory/1 - MVC with Postres/README]]
## Exercise
[Repository link](https://github.com/boolean-uk/java-api-mvc-with-postgres.git)
![[Repository/Day 12/Ex/1 - MVC with Postgres/README]]