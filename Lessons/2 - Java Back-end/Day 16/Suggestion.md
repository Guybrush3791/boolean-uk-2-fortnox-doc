# Suggestion
## Legenda
🔴 --> *ERROR*
🟡 --> *SUGGESTION*

## 🔴 Use proper package tree
```sh file:"Proper package folder manager"
.
├── controller
│   └── employee
├── model
│   ├── dto
│   ├── jpa
│   ├── repository
│   └── service
└── security
```

## 🟡 Proper documentation syntax (method and class level)
```java file:OrderAdminController.java
/**
 *
 * Create a new order. Accepts an order payload and returns the created resource or its identifier.
 * Uses method security so only users with required roles can access.
 * 
 * <p>HTTP</p>
 *     - Method: POST
 *     - Path: /api/public/orders
 *     - Request body: JSON representation of OrderRequestDto
 *     - Success: 201 Created with response body containing the saved order or its ID
 *     - Errors: 400 Bad Request for validation failures; 401/403 for unauthorized/forbidden access
 * 
 * <p>Security</p>
 * Requires hasAnyRole('USER','ADMIN'); method is protected with @PreAuthorize and evaluated before invocation.
 * 
 * @param orderRequestDto the request payload containing order details; must be valid and non-null
 * @return ResponseEntity<?> with HTTP 201 on success and the created order (or identifier) in the body
 * @throws org.springframework.web.bind.MethodArgumentNotValidException if the request body fails validation
 */
@PostMapping
public ResponseEntity<?> createOrder(/*...*/) {
	// ...
}
```
- comments at *class* and *method* level like this is one should be treated like inner documentation, it should explain scope of *class*/*method*, incoming paramenters, return response and eventually exception
> [!note] Once you proper format your comments, they will appear as suggestion in most IDEs
> ![[Pasted image 20250904111403.png]]

## 🔴 Return datatype
As a rule of thumb, follow this minimal framework:
- method not returning any data --> `{java} ResponseEntity<Void>`
```java
@PutMapping("{id}")
public ResponseEntity<Void> disableUser(/*...*/) { /

	// ...
	return ResponseEntity.ok();
}
```
- method returning exactly one data type --> `{java} ResponseEntity<ConcreteType>`
```java
@GetMapping
public ResponseEntity<List<Product>> getAllProducts() {
	return ResponseEntity.ok(productService.findAll());
}
```
- method returning more then one data type (eg: data or error) --> `{java} ResponseEntity<?>`
```java
@GetMapping
public ResponseEntity<?> getAllProducts() {
	
	Product p = // ...
	ProductDto pDto = // ...
	
	// ...
	
	if (/*...*/)
		return ResponseEntity.ok(p)
	else
		return ResponseEntity.ok(pDto)
}
```

## 🟡 Dependency Injection (DI) update
In the new versions of *SpringBoot* (like the one we are using at the moment) the suggest way to *inject* dependencies is *constructor* instead of the `@Autowired` annotation shown in class

**PREFER THIS**
```java file:MainController
@RestController  
@RequestMapping("api")  
public class MainController {  
  
    private final MainService mainService;  
  
    public MainController(MainService mainService) {  
  
        this.mainService = mainService;  
    }
}
```
> [!note] Make sure you make your *injected variable* `final` for immutability

**OVER THIS**
```java file:MainController
@RestController  
@RequestMapping("api")  
public class MainController {  
  
	@Autowired
    private MainService mainService;  
}
```

## 🟡 Avoid very small `Controller`
Even if in class we used `EntityAdminController` & `EntityPublicController` you don't need 2 controllers, if they are simple enough; just add `public` into the mapped **URL**

## 🔴 Avoid using `Object` as datatype as much as possible
Using `Object` as datatype is the best way to lose control on type of data, use proper *DTO* instead, even on *query*
```java
@Query(/*...*/)  
List<CustomerValueDto> getCustomerTotalValues(int id);
```

## 🔴 Come back with `id` all the time
Especially when you try to infer some values (like value per customer, for example), remember to come back with `id` handle, in order to have a reference for each items. 

## 🔴 Get Customers Value
Java approach
```java
public List<CustomerDto.CustomerValueDto> getCustomersValue() {
        
	return customerRepo.findAll().stream()
		.map(customer -> {
		
			long sum = customer.getOrderList().stream()
				.flatMap(order -> order.getProductList().stream())
				.mapToLong(Product::getPrice)
				.sum();
				
			return new CustomerDto.CustomerValueDto(customer, sum);
		})
		.collect(java.util.stream.Collectors.toList());
}
```
SQL approach
```SQL
SELECT c.id, c.name, sum(p.price) AS "value"
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id 
LEFT JOIN order_product op ON o.id = op.order_id 
LEFT JOIN products p ON op.product_id = p.id
GROUP BY c.id 
WHERE c.id = ?
```