# Reactive streams and error handling

**[Project Reactor](https://projectreactor.io/)** implements the **Reactive Streams specification**, enabling the creation of **non-blocking, asynchronous, and resilient applications** on the *JVM*. A key part of any reactive system is not only transforming and consuming *data streams*, but also *handling errors* gracefully without breaking the flow of data processing.

Error handling in reactive programming is fundamentally different from traditional imperative error handling. Instead of **try-catch blocks at execution time**, reactive systems provide **declarative error handling operators** that allow developers to control failure recovery, fallback strategies, and propagation in a *stream-oriented context*.

## LC
### Video
> [!note]- Video
> <div class="iframe-container"> <iframe src="https://us02web.zoom.us/rec/share/edgdTslkdBVYal2bIlBi_CHacHssVoRoC1uY9bkn-z2NCbiHRFcqslVCtmLcFSy7.OLZi3_l9gM5ACn7N" frameborder="0" allowfullscreen></iframe> </div>

[Video link](https://us02web.zoom.us/rec/share/edgdTslkdBVYal2bIlBi_CHacHssVoRoC1uY9bkn-z2NCbiHRFcqslVCtmLcFSy7.OLZi3_l9gM5ACn7N)

### Repository
https://github.com/Guybrush3791/boolean-uk-2-fortnox-webflux-flux-and-error.git

## Core Purpose and Benefits
Reactive Streams aim to address two critical aspects of high-throughput systems:
- **Controlled Data Flow (Back-pressure)** ensures that producers don’t overwhelm consumers by dynamically adjusting demand
- **Error Resilience** errors are modeled as signals in the stream rather than as disruptive exceptions. This allows the *stream* to remain  composable  and predictable

Compared to traditional Java methods where exceptions bubble up and crash the execution chain. In Project Reactor every *Publisher* (`Flux`/`Mono`) can emit three types of signals:
1. **Next (`onNext`)** → Data event
2. **Error (`onError`)** → Terminal failure
3. **Complete (`onComplete`)** → Successful termination

This separation makes error recovery *a first-class concern*
## Key Components for Error Handling
With `Flux<T>` and `Mono<T>`, developers can handle errors *reactively* using operators such as:
- **`onErrorReturn(value)`** --> provides a default fallback value when an error occurs
- **`onErrorResume(error -> Publisher)`** --> switches to a different reactive sequence for recovery
- **`onErrorMap(error -> newError)`** --> transforms exceptions into domain-specific types
- **`retry(n)`** --> attempts a failed operation again up to `n` times
- **`doOnError(callback)`** --> executes side-effects such as logging before propagating the error

These operators prevent unexpected crashes and **keep the pipeline responsive and robust**

## Real-World Back-end Applications
Error handling is central in **Spring Boot WebFlux** applications using reactive streams. Common scenarios include:
- **Resilient API calls** if an upstream service fails, `onErrorResume` can redirect requests to a *fallback mechanism* or *cached response*
- **Reactive Database Access (`R2DBC`)** errors during queries can be wrapped in `onErrorMap` to produce meaningful custom exceptions for higher layers
- **Message Processing (`Reactor Kafka`/`RabbitMQ`)** stream `retries` and `retryWhen` operators help recover from transient message broker issues without message loss
- **Fault-Tolerant Micro-services** combining back-pressure with fallback streams allows systems to degrade gracefully under load instead of failing completely

## Back-pressure and Error Propagation
*Back-pressure* and *error signals* are tightly connected in *Reactive Streams*. If a **consumer cannot keep up**, the producer is **signaled to slow down**, preventing buffer overflows.

When errors *do occur*, they are:
- Delivered via `onError` as a *terminal event* (the stream stops unless *explicitly recovered*)
- Optionally transformed into a *fallback sequence* (non-terminal continuation via *recovery operators*)

This approach gives fine-grained control over failure propagation, aligning with modern **resilient and reactive system design principles**.

## Paradigm Shift
Developers moving from *imperative programming* to *reactive error handling* need to adjust their mindset:
- Instead of *throw/catch*, think in terms of *error signals*
- Instead of breaking pipelines, design *fallback strategies*
- Instead of ignoring overloads, let back-pressure regulate flow

By embracing these practices, back-end systems built with *Project Reactor* and *Spring WebFlux* achieve **greater stability, scalability, and predictability** under both normal and failure conditions.

# Links
![[Lessons/Day 19/__blocks/Links]]