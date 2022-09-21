[↑ Back to Spring Boot Notes](Contents.md)  
[← Home](/README.md)

# [Lite RX API Hands On](https://tech.io/playgrounds/929/reactive-programming-with-reactor-3/Intro)

## Introduction to Reactive Programming

Reactor 3 is a library built around the [Reactive Streams](https://www.reactive-streams.org/) specification

The goal behind Reactive Programming is to be fully asynchronous and non-blocking in a more readable and maintainable manner
than Callback based APIs adn Future types

### Reactive Stream Sequence

1. Publisher (the source) produces data
   - Does nothing until the Subscriber has subscribed (registered)
2. Subscriber consumes the data
   - Data is pushed to the Subscriber via the Publisher

![Reactive Stream Sequence](../../Utilities/Images/ProjectReactor/reactive-stream-sequence.png)

Reactor adds **operators** to the Reactive Stream Sequence. **Operators** are chained together to describe what processing
to apply at each stage to the data.

Apply an **operator** returns a new intermediate Publisher
- Subscriber to the operator upstream 
- Publisher to the operator downstream

The final form of the data ends up in the final Subscriber that defines what to do from a user perspective.

Needed for creating a Reactive Stream Publisher or Subscriber
- Code must comply with the spec and pass TCK
  - [TCK = Technology Compatibility Kit](https://en.wikipedia.org/wiki/Technology_Compatibility_Kit)
- Favor using an existing library like Reactor 3, RxJava, Akka Streams, Vert.x or Ratpack

## Flux 

**Flux** is a Reactive Streams Publisher that can be used to generate, transform and orchestrate Flux sequences via **operators**

Flux can emit 0 to _n_ elements via `onNext`events  

- Completes via `onComplete` terminal events
- Errors via `onError`terminal events

**Note:** if no terminal event is triggered, the Flux is infinite

![Flux Sequence](../../Utilities/Images/ProjectReactor/flux-sequence.png)

### Flux Example

```
Flux.fromIterable(getSomeLongList())
    .delayElements(Duration.ofMillis(100))
    .doOnNext(serviceA::someObserver)
    .map(d -> d * 2)
    .take(3)
    .onErrorResumeWith(errorHandler::fallback)
    .doAfterTerminate(serviceM::incrementTerminate)
    .subscribe(System.out::println);
```

## Mono

**Mono** is a Reactive Streams Publisher that can be used to generate, transform and orchestrate Mono sequences via **operators**

It is a specialization of Flux that can emit **at most 1 element**
- Valued (complete with element)
- Empty (complete without element)
- Failed (error)

![Mono Sequence](../../Utilities/Images/ProjectReactor/mono-sequence.png)

`Mono<Void>`is something that can be utilized when only the completion signal is interesting

### Mono Example

```
Mono.firstWithValue(
        Mono.just(1).map(integer -> "foo" + integer),
        Mono.delay(Duration.ofMillis(100)).thenReturn("bar")
    )
    .subscribe(System.out::println);
```

## StepVerifier

A `StepVerifier` comes from the `reactor-test` artifact and is capable of subscribing to any Publisher and assert expectations.
This is something that would be utilized with **unit testing**.

An instance of `StepVerifier` can be created via `.create()`, configured via a [DSL](https://docs.spring.io/spring-integration/docs/5.1.0.M1/reference/html/java-dsl.html)
for setting expectations, and finish with a single terminal expectation (completion, error, cancellation...)

When utilizing `StepVerifier` some form of `verify()` method **needs to be used**. If it is not used, the `StepVerifier` won't
subscribe to the sequence and nothing will be asserted.

### StepVerifier Example

```
StepVerifier.create(T<Publisher>).{expectations...}.verify()
```

## Transform

Reactor has several operators that can be used to transform data. This means that when the subscriber receives data, it
then can take that data and transform it into something else. 

[Here's an overview of ways to transform existing data with Reactor](https://projectreactor.io/docs/core/release/reference/#which.values)