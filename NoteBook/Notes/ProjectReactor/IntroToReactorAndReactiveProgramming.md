[↑ Back to Project Reactor Notes](Contents.md)  
[← Home](/README.md)

# Intro to Reactor and Reactive Programming

A hands on intro course developed by Project Reactor  
🔗 [Link to course](https://tech.io/playgrounds/929/reactive-programming-with-reactor-3/Intro)

## Contents

[Introduction to Reactive Programming](#introduction-to-reactive-programming)  
[Flux](#flux)  
[Mono](#mono)  
[StepVerifier](#stepverifier)  
[Transform](#transform)  
[Merge](#merge)  
[Request](#request)  
[Error](#error)  
[Adapt](#adapt)
[Other Operations](#other-operations)  

### 📝 Note 

Each topic has a set of exercises, if utilizing the course they will appear on each topics page. If you are utilizing just the notebook here are links to the GitHub repo conatining the exercises and solutions. 

🔗 [Intro to Reactor Topics Exercises](https://github.com/reactor/lite-rx-api-hands-on/tree/techio_course/src/main/java/io/pivotal/literx)  
🔗 [Intro to Reactor Topics Solutions](https://github.com/reactor/lite-rx-api-hands-on/tree/solution/src/main/java/io/pivotal/literx)  

---

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

```java
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

It is a specialization of **Flux** that can emit **at most 1 element**
- Valued (complete with element)
- Empty (complete without element)
- Failed (error)

![Mono Sequence](../../Utilities/Images/ProjectReactor/mono-sequence.png)

`Mono<Void>` is something that can be utilized when only the completion signal is interesting

### Mono Example

```java
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

```java
StepVerifier.create(T<Publisher>).{expectations...}.verify()
```

## Transform

Reactor has several operators that can be used to transform data. This means that when the subscriber receives data, it
then can take that data and transform it into something else. 

#### 🔗 [Transform Overview](https://projectreactor.io/docs/core/release/reference/#which.values)

## Merge

Merging sequences is utilized when listening for values from multiple Publishers, merging the data retrieved and returning
a single Flux. 

**Note:** Examples of ways to merge data from multiple publishers can be found [using the link above](#🔗-transform-overview)
and looking for the bullet point stating `"I want to combine publishers..."` 

## Request 

**Backpressure** a feedback mechanism that allows a **Subscriber** to signal to its **Publisher** how much data it is prepared to process, limiting the rate at which the **Publisher** produces data. 

**AKA:** A way that the **Subscriber** can inform the **Publisher** how much data it can consume

Backpressure is configured at the **Subscription** level.

`subscribe()` - creates a **Subscription**  
`cancel()` - cancels the flow of data  
`request(long)` - tunes demand of data

**Request Example:** `request(Long.MAX_VALUE)` - **Publisher** will emit data at its fastest pace due to request demand is essentially unbound. 

🔗 [Backpressure Overview](https://projectreactor.io/docs/core/release/reference/#reactive.backpressure)  
🔗 [Subscribe Method Examples](https://projectreactor.io/docs/core/release/reference/#_subscribe_method_examples)  
🔗 [Peeking into a Sequence](https://projectreactor.io/docs/core/release/reference/#which.peeking)

## Error

Reactor ships with several tools that can be used to handle, recover from and even retry a new **Subscription**. The main goal with handling errors still stands, catch them and handle them gracefully.

🔗 [Error Handling Operators Overview](https://projectreactor.io/docs/core/release/reference/#_error_handling_operators)  
🔗 [Handling Errors Overview](https://projectreactor.io/docs/core/release/reference/#_error_handling_operators)

## Adapt

Reactor 3 has the ability to interact with [RxJava3](https://reactivex.io/intro.html) without having to utilize a library inbetween to translate. This can help with projects that are utilizing RxJava3 to leverage Reactor 3 with less complexity and re-work. 

### `Flux` ↔️ `Flowable`

```java
// Adapt Flux to RxJava Flowable
Flowable<User> fromFluxToFlowable(Flux<User> flux) {
	return Flowable.fromPublisher(flux);
}

// Adapt RxJava Flowable to Flux
Flux<User> fromFlowableToFlux(Flowable<User> flowable) {
	return Flux.from(flowable);
}
```

🔗 [`Flux.from()`](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#from-org.reactivestreams.Publisher-)  
🔗 [`Flowable.fromPublisher()`](https://reactivex.io/RxJava/3.x/javadoc/io/reactivex/rxjava3/core/Flowable.html#fromPublisher-org.reactivestreams.Publisher-)

### `Flux` ↔️ `Flowable` ↔️ `Observable`

```java
// Adapt Flux to RxJava Observable
Observable<User> fromFluxToObservable(Flux<User> flux) {
	return Flowable.fromPublisher(flux).toObservable();
}

// Adapt RxJava Observable to Flux
Flux<User> fromObservableToFlux(Observable<User> observable) {
	return Flux.from(observable.toFlowable(BackpressureStrategy.BUFFER));
}
```

🔗 [`Observable`]()  
🔗 [`Flowable.toObservable()`](https://reactivex.io/RxJava/3.x/javadoc/io/reactivex/rxjava3/core/Flowable.html#toObservable--)  
🔗 [`Observable.toFlowable()`](https://reactivex.io/RxJava/3.x/javadoc/io/reactivex/rxjava3/core/Observable.html#toFlowable-io.reactivex.rxjava3.core.BackpressureStrategy-)  
🔗 [`BackpressureStrategy`](http://reactivex.io/RxJava/3.x/javadoc/io/reactivex/rxjava3/core/BackpressureStrategy.html)

### `Mono` ↔️ `Single`

```java
// Adapt Mono to RxJava Single
Single<User> fromMonoToSingle(Mono<User> mono) {
	return Single.fromPublisher(mono);
}

// Adapt RxJava Single to Mono
Mono<User> fromSingleToMono(Single<User> single) {
	return Mono.from(Flowable.fromSingle(single));
}
```

🔗 [`Single`](https://reactivex.io/RxJava/3.x/javadoc/io/reactivex/rxjava3/core/Single.html)  
🔗 [`Mono.from()`](https://reactivex.io/RxJava/3.x/javadoc/io/reactivex/rxjava3/core/Single.html)  
🔗 [`Flowable.fromSingle()`](https://reactivex.io/RxJava/3.x/javadoc/io/reactivex/rxjava3/core/Flowable.html#fromSingle-io.reactivex.rxjava3.core.SingleSource-)  
🔗 [`Single.fromPublisher()`](https://reactivex.io/RxJava/3.x/javadoc/io/reactivex/rxjava3/core/Single.html#fromPublisher-org.reactivestreams.Publisher-) 

### `Mono` ↔️ `CompletableFuture`

```java
// Adapt Mono to Java 8+ CompletableFuture
CompletableFuture<User> fromMonoToCompletableFuture(Mono<User> mono) {
	return mono.toFuture();
}

// Adapt Java 8+ CompletableFuture to Mono
Mono<User> fromCompletableFutureToMono(CompletableFuture<User> future) {
	return Mono.fromFuture(future);
}
```

🔗 [`Mono.toFuture`](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#toFuture--)  
🔗 [`Mono.fromFuture`](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#fromFuture-java.util.concurrent.CompletableFuture-)  

## Other Operations

Reactor 3 has a wide variety of other operations under it's toolbelt outside of what was already covered. 

🔗 [Which operator do I need?](https://projectreactor.io/docs/core/release/reference/index.html#which-operator)

### Zip Operator

🔗 [`Flux.zip()`](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#zip-java.util.function.Function-int-org.reactivestreams.Publisher...-)

In this example you can see that the `Flux.zip()` operator takes in the 3 `Flux<Strings>` that are passed into the function.

Following the `Flux.zip()` operator a `.flatMap()` is utilized to create a new `Flux<User>` to be returned. When creating the User there are arguements using `.getT1()` and then 'T' followed by a different number. This is how to retreve information from a Tuple which is what is returned by the `.zip()` operator.

```java
// Create a Flux of user from Flux of username, firstname and lastname.
Flux<User> userFluxFromStringFlux(Flux<String> usernameFlux, Flux<String> firstnameFlux, Flux<String> lastnameFlux) {
	return Flux.zip(
		usernameFlux, 
		firstnameFlux, 
		lastnameFlux)
		.flatMap(
			info -> 
			Flux.just(
				new User(
					info.getT1(),
					info.getT2(),
					info.getT3())));
}
```

### Fastest Mono

🔗 [`Mono.firstWithValue()`](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#firstWithValue-java.lang.Iterable-)

In this example the goal is to retrieve a `Mono` that returns it's value first or faster. Utilizing the `Mono.firstWithValue()` operator it will return whichever provided `Mono` returns its value fastest.

```java
// Return the mono which returns its value faster
Mono<User> useFastestMono(Mono<User> mono1, Mono<User> mono2) {
	return Mono.firstWithValue(mono1, mono2);
}
```

### Fastest Flux

🔗 [`Flux.firstWithValue()`](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#firstWithValue-java.lang.Iterable-)

Similar to the [example above](#fastest-mono) - the goal is to retrieve the first or fastest element returned by a `Flux`. Utilizing the `Flux.firstWithValue()` operator it will select which `Flux` is the fastest or first to emit a value.

```java
// Return the flux which returns the first value faster
Flux<User> useFastestFlux(Flux<User> flux1, Flux<User> flux2) {
	return Flux.firstWithValue(flux1, flux2);
}
```

### Flux Completion

🔗[`Flux.ignoreElements()`](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#ignoreElements--)  
🔗[`Flux.then()`](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#then--)

In this example there is a scenario where you're not interested in the elements of a `Flux`, but want the completion of the sequence represented as a `Mono<Void>`. This is accomplished by utilizing the `Flux.then()` operator.

```java	
// Convert the input Flux<User> to a Mono<Void> that represents the complete signal of the flux
Mono<Void> fluxCompletion(Flux<User> flux) {
	return flux.then();
}
```

If there was a need to maintain the `Flux` type the operator `Flux.ignoreElements()` could be utilized.

### Null aware user to mono

🔗[`Mono.justOrEmpty()`](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#justOrEmpty-T-)

Reactor 3 is not a fan of `null` values, I mean no one really is. In this example we have a situation where the arguement being passed into the function is nullable. Which means there is a chance it may be `null`.  
The goal is to return a `Mono<User>` however still return a valid `Mono` if the argument is `null`. Utilizing `Mono.justOrEmpty()` is able to accomplish that goal.

```java
// Return a valid Mono of user for null input and non null input user (hint: Reactive Streams do not accept null values)
Mono<User> nullAwareUserToMono(User user) {
	return Mono.justOrEmpty(user);
}
```

### Otherwise if empty
 
🔗[`Mono.defaultIfEmpty()`](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#defaultIfEmpty-T-)  
🔗[`Mono.switchIfEmpty()`](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#switchIfEmpty-reactor.core.publisher.Mono-)

In this example there is chance that the `Mono` argument could be empty. If it is empty, the goal is to have it return a default value instead of an empty `Mono`. This is accomplished by utilizing the `Mono.defaultIfEmpty()` operator.

```java
// Return the same mono passed as input parameter, expect that it will emit User.SKYLER when empty
Mono<User> emptyToSkyler(Mono<User> mono) {
	return mono.defaultIfEmpty(User.SKYLER);
}
```

There is a similar operator that can be utilized if there was a need for another sequence instead of another value. That operator is `Mono.switchIfEmpty()`.

### Collect to list

🔗 [`Flux.collectList()`](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#collectList--)

In this example the goal is to collect all values provided by a `Flux` into a `Mono<List>` containing all the values of the `Flux`. To accomplish this the `Flux.collectList()` operator is utilized.

```java
// Convert the input Flux<User> to a Mono<List<User>> containing list of collected flux values
Mono<List<User>> fluxCollection(Flux<User> flux) {
	return flux.collectList();
}
```

## Reactive to Blocking

At times there may be cases where portions of a project will be non-reactive, but could utilize a portion of code that is reactive. 

This is something that should be avoided; however, there is a solution available if it's absolutely needed.

### Value from Mono

🔗 [`Mono.block()`](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#block--)

In this example a `Mono` is being passed into a function that returns a non-reactive type. This is to represent utilizing reactive components with non-reactive components. This is accomplished by the `Mono.block()` operator.

```java
// Return the user contained in that Mono
User monoToValue(Mono<User> mono) {
	return mono.block();
}
```

### Flux to Iterable

🔗 [`Flux.toIterable()`](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#toIterable--)

Similar to the [example above](#value-from-mono) a `Flux` is being passed into a function returning a non-reactive type. This is acccomplished by the `Flux.toIterable()` operator.

```java
// Return the users contained in that Flux
Iterable<User> fluxToValues(Flux<User> flux) {
	return flux.toIterable();
}
```

## Blocking to Reactive

This covers another scenario of non-reactive code interacting with reactive code. The non-reactive code in this case is a `BlockingRepository` (example - JDBC connection to a database).

The best approach is to isolate blocking parts of your code into their own execution contex via a `Scheduler`. This allows to keep efficiency of the rest of the pipeline high and only creating extra threads when needed.

### Slow Publisher

🔗 [`Flux.defer()`](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#defer-java.util.function.Supplier-)  
🔗 [`Flux.subscribeOn()`](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#subscribeOn-reactor.core.scheduler.Scheduler-)  
🔗 [`Schedulers.boundedElastic()`](https://projectreactor.io/docs/core/release/api/reactor/core/scheduler/Schedulers.html#boundedElastic--)  

In this example a `Flux` is created out of a `Blocking Repository`. What takes place is that the `Flux` is defered until it is suscribed to, which is when the repository completes its operations. This allows a pool of threads that grows on demand.

```java
// Create a Flux for reading all users from the blocking repository deferred until the flux is subscribed, and run it with a bounded elastic scheduler
Flux<User> blockingRepositoryToFlux(BlockingRepository<User> repository) {
	return Flux.defer(() -> Flux.fromIterable(repository.findAll()))
				.subscribeOn(Schedulers.boundedElastic());
}
```

### Slow Subscriber

🔗 [`Flux.publishOn()`]()

In this example a non-reactive repository save is the what's creating a slow `Subscriber`. The save is isolated into its own excicution, and the `Mono.then()` operator is utilized for knowing if succuss or failure. 

```java
// Insert users contained in the Flux parameter in the blocking repository using a bounded elastic scheduler and return a Mono<Void> that signal the end of the operation
Mono<Void> fluxToBlockingRepository(Flux<User> flux, BlockingRepository<User> repository) {
	return flux
			.publishOn(Schedulers.boundedElastic())
			.doOnNext(repository::save)
			.then();
}
```
