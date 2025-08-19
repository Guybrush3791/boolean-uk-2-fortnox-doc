---
argument: intro TDD, domain modelling
section: java core
lesson count: "1"
ex count: "1"
---
# Intro to TDD and Domain Modelling
**Test-Driven Development (TDD)** is a software development methodology where tests are written before the actual code, following a repetitive cycle called **Red-Green-Refactor**. This approach reverses the traditional development process by starting with test cases that define the desired behavior of the code.

![[TDD - Red-Green-Refactor|1200]]
## LC
### Lesson

| Classes         | Methods                        | Scenario                                                                                        | Outputs                  |
| --------------- | ------------------------------ | ----------------------------------------------------------------------------------------------- | ------------------------ |
| `CohortManager` | `search(List<String>, String)` | If incoming cohorts list and cohort name is valid and the name is present in at least 1 cohort. | `List<String>` not empty |
|                 |                                | - empty cohorts list<br>- empty cohort name<br>- cohort name not present in the list            | `List<String>` empty     |

---

```
As a supermarket shopper,
So that I can pay for products at checkout,
I'd like to be able to know the total cost of items in my basket.
```



| Classes  | Methods                       | Scenario                                | Outputs       |
| -------- | ----------------------------- | --------------------------------------- | ------------- |
| `Basket` | `pay(List<Product> products)` | If more then 0 valid product is present | `float` price |
|          |                               | No product at all                       | `null`        |

---

```
As an organised individual,
So that I can evaluate my shopping habits,
I'd like to see an itemised receipt that includes the name and price of the products
I bought as well as the quantity, and a total cost of my basket.
```


| Classes    | Methods                                 | Scenario                              | Outputs                                   |
| ---------- | --------------------------------------- | ------------------------------------- | ----------------------------------------- |
| `Shopping` | `getUserHabbit(List<Product> products)` | If user has bought more then 10 items | products list with frequency and quantity |
|            |                                         | if user has bough less then 10 items  | `-1`                                      |

### Repositories
- [TDD domain model](https://github.com/Guybrush3791/java-tdd-domain-modelling.git)
- [Scrabble Ex](https://github.com/Guybrush3791/java-scrabble-challenge.git)
## Lesson
[Repository link](https://github.com/boolean-uk/java-tdd-domain-modelling.git)
![[Repository/Day 3/Theory/1 - java tdd domain modelling/README|Java TDD Domain Modelling]]

## Exercise
[Repository link](https://github.com/boolean-uk/java-tdd-todo-list)
![[Repository/Day 3/Ex/1 - java tdd todo list/README|Java TDD ToDo List]]