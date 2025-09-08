# Error Handling
The core of *WebFlux error handling* is that errors are terminal signals in Reactor, handle them with reactive operators like **onErrorResume** and **onErrorReturn**, and complement them with centralized handling via `@ControllerAdvice` (annotation style) or `ErrorWebExceptionHandler`/`AbstractErrorWebExceptionHandler` (functional style).

## Reactor basics
- In Reactive Streams, an error is a terminal event that cancels the sequence --> downstream operators see the error unless explicitly handled, so recovery must be defined in the pipeline
- Prefer *transforming* or *recovering* from errors in-stream for resilience, then surface un-handled errors to a global handler to produce *HTTP responses consistently*


## Operator toolbox
- `onErrorResume`: switch to a fallback Publisher based on the `Throwable` (supports conditional handling by exception type or predicate for dynamic recovery)
```java file:ErrorController.java
@GetMapping(value = "/on-error-resume")  
public Mono<String> onErrorResumeDemo(@RequestParam(defaultValue = "timeout") String mode) {  

    Mono<String> upstream = switch (mode) {  
        case "timeout" -> Mono.delay(Duration.ofMillis(1500)) // [!note] wait for 1.5 seconds
                .then(Mono.error(new TimeoutException("Simulated timeout"))); // [!note] throws `TimeoutException`
        case "notfound" -> Mono.error(new IllegalArgumentException("Simulated not found"));  // [!note] throws `IllegalArgumentException`
        default -> Mono.error(new RuntimeException("Unexpected"));  // [!note] throw `RuntimeException`
    };  
  
    return upstream  
		.onErrorResume(err -> {  // [!note] the new exotic `catch`
		
			if (err instanceof TimeoutException)   
				return Mono.just("cache-value");  // [!note] fallback return value for `TimeoutException`

			if (err instanceof IllegalArgumentException)  
				return Mono.just("default-for-illegal-arg");  // [!note] fallback return value for `IllegalArgumentException`

			return Mono.error(err); // [!note] propagate unknown problems  
		});  
}
```

- `onErrorReturn`: emit a static fallback value and complete; best for simple defaults when context is not needed
```java file:ErrorController.java
@GetMapping("/on-error-return")  
public Mono<String> onErrorReturnDemo() {  
  
    return Mono.<String>error(new RuntimeException("boom"))  
		.onErrorReturn("static-default");  
}
```

- `onErrorMap`: map an exception to another type (e.g., domain to HTTP-friendly), deferring response mapping to a global handler
```java file:ErrorController.java
@ResponseStatus(code = org.springframework.http.HttpStatus.BAD_REQUEST)  // [!note] setup the proper **error code**
static class MyBadRequestException extends RuntimeException {  
    public MyBadRequestException(String message) { super(message); }  
}

@GetMapping(value = "/on-error-map")  
public Mono<String> onErrorMapDemo() {  
  
    return Mono.<String>error(new IllegalArgumentException("bad input"))  
		.onErrorMap(IllegalArgumentException.class, ex ->  // [!note] my a core exeption to custom one to fix **error code**
			new MyBadRequestException("Mapped: " + ex.getMessage())  
		);  
}
```
- `doOnError`: side-effect logging/metrics; does not consume or recover, error still propagates
```java file:ErrorController.java
@GetMapping(value = "/do-on-error")  
public Mono<String> doOnErrorDemo() {  
    return Mono.<String>error(new RuntimeException("to-log"))  
		.doOnError(ex -> log.warn("Observed error in pipeline: {}", ex.toString()));  // [!note] still erroring out --> no resume/return here 
}
```
- `onErrorContinue`: skip faulty element and continue for certain operators (*use carefully* due to semantics and potential surprises)
```java file:ErrorController.java
@GetMapping(value = "/on-error-continue", produces = MediaType.APPLICATION_NDJSON_VALUE)  
public Flux<String> onErrorContinueDemo() {  
    return Flux.just("1", "x", "2", "y", "3")  
		.map(s -> {  
			int v = Integer.parseInt(s);  // [!note] this will cause `NumberFormatException` for **non-numeric values**  
			if (v == 2) throw new IllegalStateException("unlucky-two");  // [!note] `IllegalStateException` demo
			return v;  
		})  
		.map(v -> "ok-" + v)  
		.onErrorContinue((ex, val) -> {  // [!note] observe and skip the incriminating element  
			log.info("Skipping value {} due to {}", val, ex.toString());  
		});  
}
```

> [!note] `slf4j` logger instance
> `{java} private static final Logger log = LoggerFactory.getLogger(ErrorController.class);`

## Choosing resume vs return
- Use `onErrorReturn` for a constant, *trivial default* when errors are not contextual
- Use `onErrorResume` when *recovery depends on the error* or *needs a new Publisher*

## Map `Exception` to `ResponseEntity`
`@RestControllerAdvice` is a global, cross‑cutting mechanism that lets *annotation‑based WebFlux controllers* centralize exception handling in one place, so exceptions thrown *from any controller* method are *intercepted and mapped to consistent HTTP responses and bodies*. It combines `@ControllerAdvice` with `@ResponseBody`, meaning handler methods return serialized error bodies (e.g., `ProblemDetail` or `JSON`) rather than views, and are applied after any controller‑local `@ExceptionHandler` methods.

```java file:GlobalExceptionHandler.java hlt:7,20-24,36
@Order(1)  
@RestControllerAdvice  
public class GlobalExceptionHandler extends ResponseEntityExceptionHandler {  
  
    private static final Logger log = LoggerFactory.getLogger(GlobalExceptionHandler.class);  
  
    @ExceptionHandler(MyBadRequestException.class)  // [!note] match custom exception and map it to a `ProblemDetail` object
    public ResponseEntity<ProblemDetail> handleResourceNotFound(MyBadRequestException ex,  
                                                                ServerWebExchange exchange) {  
  
        ProblemDetail pd = ProblemDetail.forStatusAndDetail(HttpStatus.NOT_FOUND, ex.getMessage());  
  
        pd.setTitle("Resource Not Found");  
        pd.setProperty("timestamp", Instant.now().toString());  
        pd.setProperty("path", exchange.getRequest().getPath().value());  
  
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(pd);  
    }  
  
    @ExceptionHandler({   // [!note] map multiple exception at once
        MethodArgumentNotValidException.class,   
        BindException.class,   
        IllegalArgumentException.class 
    })  
    public ResponseEntity<ProblemDetail> handleValidation(Exception ex, ServerWebExchange exchange) {  
  
        ProblemDetail pd = ProblemDetail.forStatusAndDetail(HttpStatus.NOT_FOUND, ex.getMessage());  
  
        pd.setTitle("Validation Failed");  
        pd.setType(URI.create("https://example.com/problems/validation"));  
        pd.setProperty("path", exchange.getRequest().getPath().value());  
  
        return ResponseEntity.badRequest().body(pd);  
    }  
  
    @ExceptionHandler(Throwable.class)  // [!note] map all not-mapped exception (the more specific the more precedence)
    public ResponseEntity<ProblemDetail> handleAny(Throwable ex, ServerWebExchange exchange) {  
  
        log.error("Unhandled exception for {} {}", exchange.getRequest().getMethod(), exchange.getRequest().getURI(), ex);  
  
        ProblemDetail pd = ProblemDetail.forStatus(HttpStatus.INTERNAL_SERVER_ERROR);  
  
        pd.setTitle("Internal Server Error");  
        pd.setDetail("An unexpected error occurred");  
        pd.setProperty("path", exchange.getRequest().getPath().value());  
  
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(pd);  
    }  
}
```
> [!note] `ProblemDetail`
> `ProblemDetail` is a standardized error payload based on *RFC 7807* that represents* HTTP API errors* with a consistent JSON structure containing fields like `type`, `title`, `status`, `detail`, and `instance` for machine-readable and human-friendly diagnostics.

## Error handling in `Service`
The best practice is to let the service layer throw meaningful, *domain-specific exceptions* and handle translation to *HTTP* in a centralized *controller advice*. Controllers stay thin and focused on *HTTP concerns*, services enforce business rules and raise exceptions when those rules fail.
```java file:RaiderService.java hlt:6
public Mono<Raider> findById(int id) {  
  
    return Mono.fromCallable(() -> repository.findById(id))  
            .subscribeOn(Schedulers.boundedElastic())  
            .flatMap(Mono::justOrEmpty)  
            .switchIfEmpty(Mono.error(new MyBadRequestException("Raider not found")) // [!note] hnadle not found state with custom exception, with `GlobalExceptionHandler`
    );  
}
```

# Links
![[Lessons/2 - Java Back-end/Day 19/__blocks/Links]]