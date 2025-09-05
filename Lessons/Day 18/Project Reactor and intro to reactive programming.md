# Project Reactor and intro to reactive programming
**[Project Reactor](https://projectreactor.io/)** is a fourth-generation reactive library built on the Reactive Streams specification for creating **non-blocking, asynchronous applications** on the *JVM*. It serves as a foundational technology for building efficient, scalable Java back-end systems that can handle *high-throughput* scenarios with *minimal resource consumption*.

## LC
### Video
> [!note]- Video
> <div class="iframe-container"> <iframe src="https://us02web.zoom.us/rec/share/uqQp02-kgUA4d-hXOZgfuTbHnlQnHP4D0TUKC_7pM9xewbKqWp85vXwxkJYBB9r0.TbpnnYGORI2VGFUi" frameborder="0" allowfullscreen></iframe> </div>

[Video link](https://us02web.zoom.us/rec/share/uqQp02-kgUA4d-hXOZgfuTbHnlQnHP4D0TUKC_7pM9xewbKqWp85vXwxkJYBB9r0.TbpnnYGORI2VGFUi)

### Repository
https://github.com/Guybrush3791/boolean-uk-2-fortnox-webflux-mono.git

## Core Purpose and Benefits
*Project Reactor* addresses a fundamental problem in traditional Java backend applications:
**Resource Efficiency**

In conventional *Spring MVC* applications using *Tomcat* (java server), each client request creates a new thread that often blocks while waiting for database queries or external API calls. 
This leads to *inefficient CPU utilization as threads busy-wait for responses*.

*Reactor* solves this by implementing a **push-based, non-blocking architecture** that:
- *Creates thread pools* equal to the *number of CPU cores available*
- Performs *blocking-aware scheduling* within the Java application
- Maintains *high CPU utilization* by avoiding thread blocking

## Key Components
The library provides *two main reactive types*:
- **`Mono<T>`** Represents a stream that can emit 0 or 1 element [[\^1](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html)]
- **`Flux<T>`** Represents a stream that can emit 0 to many elements [[\^2](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html)]

These types implement the reactive programming paradigm, which is "concerned with data streams and the propagation of change". Unlike traditional pull-based Iterator patterns, reactive streams are **push-based**, meaning data flows to subscribers when available rather than being requested.

## Real-World Backend Applications
Project Reactor is commonly used in Java backend development alongside:
- **Spring Boot** as the fundamental application framework
- **Spring WebFlux** for reactive HTTP communication
- **Reactor Kafka** for reactive message processing
- **R2DBC** for reactive database access

Companies like Trivago have successfully implemented Reactor in their backend systems to handle:
- **Fluctuating processing volumes** through non-blocking I/O and backpressure
- **API rate limiting** and request buffering
- **Resilient error handling** and propagation
- **Vertical and horizontal scalability**


## Backpressure Support

One of *Reactor's key features* is built-in **back-pressure handling**, which prevents system overload when producers generate data faster than consumers can process it. This is crucial for maintaining system stability and preventing buffer overflow in high-load scenarios.

*Project Reactor represents a paradigm shift from imperative to reactive programming*, requiring developers to think in terms of *data streams and transformations* rather than *sequential operations*. While this involves a learning curve, it enables building highly efficient, resilient back-end systems capable of handling modern scalability requirements.

---

# Links
![[Lessons/Day 18/__blocks/Links]]