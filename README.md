# Learn Reactive Programming with Spring Framework 5!
## Introduction to Reactive Programming

- [reactivemanifesto](https://www.reactivemanifesto.org/)
- Reactive != fast!!
- A typical CRUD type application will not see much, if any performance improvement
- Best used for streaming type applications
- Reactive Manifesto - Responsive
  - The system responds in a timely manner.
  - Problems may be detected quickly and dealt with effectively.
  - Responsive systems provide rapid and consistent response times.
- Reactive Manifesto - Resilient
  - System stays responsive in the face of failure.
  - Resilience is achieved by replication, containment, isolation, and delegation.
  - Failures are contained within each component.
  - Parts of the system can fail, without compromising the system as a whole.
  - Recovery of each component is delegated to another.
  - High-availability is ensured by replication where necessary.
 - Reactive Manifesto - Elastic
   - Reactive Systems can react to changes in the input rate by increasing or decreasing resources
allocated to service inputs.
- Reactive Manifesto - Message Driven
  - Reactive Systems rely on asynchronous message passing to establish a boundary between
components.This ensures loose coupling, isolation, and location transparency.
  - Message passing enables load management, elasticity, and flow control.
  - Non-blocking communication allows recipients to only consume resources while active, leading to less system overhead.
- Reactive Programming
  - Reactive Programming focuses on non-blocking, asynchronous execution - a key characteristic of Reactive Systems.

### What is Reactive Programming?
- Common Use Cases
  - External Service Calls
  - Highly Concurrent Message Consumers
  - Spreadsheets
  - Abstraction Over Asynchronous Processing
  - Abstract whether or not your program is synchronous or asynchronous 
- Features of Reactive Programming
  - Data Streams
  - Asynchronous
  - Non-blocking
  - Backpressure
  - Failures as Messages
- Asynchronous
  - Events are captured asynchronously.
  - A function is defined to execute when an event is emitted.
  - Another function is defined if an error is emitted.
  - Another function is defined when complete is emitted.
- Non-Blocking
  - The concept of using non-blocking is important.
  - In Blocking, the code will stop and wait for more data (ie reading from disk, network, etc)
  - Non-blocking in contrast, will process available data, ask to be notified when more is available, then continue.
  
- Back Pressure
  - The ability of the subscriber to throttle data
- Failures as Messages
  - Exceptions are not thrown in a traditional sense.
  - Would break processing of stream.
  - Exceptions are processed by a handler function.
  
### Reactive Streams
- Goal is to create a standard for asynchronous stream processing with non-blocking back pressure.
- Reactive Streams is a set of 4 interfaces which define the API:
  - Publisher, Subscriber, Subscription, Processor
- On April 30th, 2015, version 1.0.0 of Reactive Streams was released.
- Under JEP-266, Reactive Streams is now part of the Java 9 JDK
- On August 9th, 2017 version 1.0.1 of Reactive Streams was released.
- Spring Reactive Types (both types implement the Reactive Streams Publisher interface):
  - Mono - is a publisher with zero or one elements in data stream
  - Flux - is a publisher with zero or MANY elements in the data stream
  
### Mono Operations

```java
//subscribe - starts the execution
  @Test
    void getByIdMapFunction(){
        Mono<Person> personMono = personRepository.getById(1);

        personMono.map(person -> {
            System.out.println(person.toString());
            return person.getFirstName();
        }).subscribe(firstName -> {
            System.out.println("from map: " + firstName);
        });
    }
```
### Flux Operations
```java
@Test
    void testFluxToListMono() {
        Flux<Person> personFlux = personRepository.findAll();

        Mono<List<Person>> personListMono = personFlux.collectList();

        personListMono.subscribe(list -> {
           list.forEach(person -> {
               System.out.println(person.toString());
           });
        });
    }
```
### Filtering Flux objects
```java
   @Test
    void testFindPersonById() {
        Flux<Person> personFlux = personRepository.findAll();

        final Integer id = 3;

        Mono<Person> personListMono = personFlux.filter(person ->
                person.getId() == id).next();

        personListMono.subscribe(person -> {
            System.out.println(person.toString());
        });
    }
```

### Reactive StepVerifier
- Used for tests
- Part of reactor-test

```java
 @Test
 void getByIdSubscribe() {
        Mono<Person> personMono = personRepository.getById(1);

        StepVerifier.create(personMono)
                .expectNextCount(1) //how many elements 
                					//will go through the stream
                .verifyComplete();

        personMono.subscribe(person -> {
            System.out.println(person.toString());
        });
    }
    
    @Test
    void testFluxSubscribe() {
        Flux<Person> personFlux = personRepository.findAll();

        StepVerifier.create(personFlux)
                .expectNextCount(4)
                .verifyComplete();

        personFlux.subscribe(person -> {
            System.out.println(person.toString());
        });
    }
```
## Netflux Reactive Example
### Service
```java
@Service
@RequiredArgsConstructor
public class MovieServiceImpl implements MovieService {

    private final MovieRepository movieRepository;

//this generates a MovieEvent every second
    @Override
    public Flux<MovieEvent> streamMovieEvents(String id) {
        return Flux.<MovieEvent>generate(movieEventSynchronousSink -> {
            movieEventSynchronousSink.next(new MovieEvent(id, new Date()));
        }).delayElements(Duration.ofSeconds(1));
    }
}
```
### Boostraping
```java
@RequiredArgsConstructor
@Component
public class InitMovies implements CommandLineRunner {

    private final MovieRepository movieRepository;

    @Override
    public void run(String... args) throws Exception {

        movieRepository.deleteAll().thenMany(
                Flux.just("Silence of the Lambdas", "AEon Flux", "Enter the Mono<Void>", "The Fluxxinator",
                                "Back to the Future", "Meet the Fluxes", "Lord of the Fluxes")
                        .map(Movie::new)
                        .flatMap(movieRepository::save)
        ).subscribe(null, null, () -> {
            movieRepository.findAll().subscribe(System.out::println);
        });
    }
}
```