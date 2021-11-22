# Quarkus Concurrency in Practice

<object data="https://eformat.github.io/quarkus-concurrency/pdf/Quarkus%20Concurrency%20in%20Practice.pdf" type="application/pdf" width="1000px" height="800px">
    <embed src="https://eformat.github.io/quarkus-concurrency/pdf/Quarkus%20Concurrency%20in%20Practice.pdf">
        <p>This browser does not support PDFs. Please download the PDF to view it: <a href="https://eformat.github.io/quarkus-concurrency/pdf/Quarkus%20Concurrency%20in%20Practice.pdf">Download PDF</a>.</p>
    </embed>
</object>

## Code snippets

Generate app
```bash
quarkus create app --maven --java --no-wrapper -x quarkus-resteasy-reactive,quarkus-resteasy-reactive-jackson,quarkus-vertx org.acme:quarkus-concurrency:1.0.0-SNAPSHOT
```
Add dep
```bash
<dependency>
    <groupId>io.smallrye.reactive</groupId>
    <artifactId>smallrye-mutiny-vertx-web-client</artifactId>
    <version>2.15.1</version>
</dependency>
```
Slide.3
```java
@Path("/hello")
public class ReactiveGreetingResource {

    @GET
    @Path("imperative")
    @Produces(MediaType.TEXT_PLAIN)
    public String imperativeHello(@PathParam("name") String name) {
        System.out.println(Thread.currentThread().getName());
        return greeting(name);
    }

    @GET
    @Path("reactive")
    @Produces(MediaType.TEXT_PLAIN)
    public CompletionStage<String> reactiveHello(@PathParam("name") String name) {
        System.out.println(Thread.currentThread().getName());
        return CompletableFuture.supplyAsync(() -> greeting(name)).minimalCompletionStage();
    }

    private String greeting (String name) {
        return "Hello " + (null == name || name.isBlank() ? "World !" : name);
    }

}
```
Slide.16
```java
@Path("/handleit")
public class ReactiveGreetingResource {

    @GET
    public void handleit() {
        Multi.createFrom().range(0,10)
                .onSubscription().invoke(sub -> System.out.println("Received Subscription: " + sub))
                .onRequest().invoke(req -> System.out.println("Got a request: " + req))
                .select().where((i -> i % 2 == 0))
                .onItem().transform(i -> i * 100);
    }

}
```

```java
@Path("/handleit")
public class ReactiveGreetingResource {

    @GET
    public void handleit() {
        Multi.createFrom().range(0,10)
                .onSubscription().invoke(sub -> System.out.println("Received Subscription: " + sub))
                .onRequest().invoke(req -> System.out.println("Got a request: " + req))
                .select().where((i -> i % 2 == 0))
                .onItem().transform(i -> i * 100)
                .subscribe().with(
                        i -> System.out.println("i: " + i)
                );
    }

}
```
Slide.19
```java
@Path("/hello")
public class ReactiveGreetingResource {

    @GET
    @Produces(MediaType.TEXT_PLAIN)
    public String hello() {
        System.out.println(Thread.currentThread().getName());
        return "Hello RESTEasy Reactive";
    }

}
```

```java
@Path("/hello")
public class ReactiveGreetingResource {

    @GET
    @Produces(MediaType.TEXT_PLAIN)
    public Uni<String> hello() {
        System.out.println(Thread.currentThread().getName());
        return Uni.createFrom().item("Hello RESTEasy Reactive");
    }

}
```

```java
@Path("/hello")
public class ReactiveGreetingResource {

    @GET
    @Produces(MediaType.TEXT_PLAIN)
    @Blocking    
    public Uni<String> hello() {
        System.out.println(Thread.currentThread().getName());
        return Uni.createFrom().item("Hello RESTEasy Reactive");
    }

}
```
Slide.24
```java
@Path("/hello")
public class ReactiveGreetingResource {

    @GET
    @Produces(MediaType.TEXT_PLAIN)
    public Uni<String> hello() throws InterruptedException {
        System.out.println(Thread.currentThread().getName());
        Thread.sleep(10000);
        return Uni.createFrom().item("Hello RESTEasy Reactive");
    }

}
```
Slide.25
```java
@Path("/hello")
public class ReactiveGreetingResource {

    @GET
    @Produces(MediaType.TEXT_PLAIN)
    public Uni<String> hello() throws InterruptedException {
        System.out.println(Thread.currentThread().getName());
        return Uni.createFrom().item("Hello RESTEasy Reactive").onItem().delayIt().by(Duration.ofSeconds(10));
    }

}
```
Slide.26
```java
@Path("/hello")
public class ReactiveGreetingResource {

    @GET
    @Produces(MediaType.TEXT_PLAIN)
    public Uni<String> hello() throws InterruptedException {
        System.out.println(Thread.currentThread().getName());
        Uni.createFrom().item("Mutiny").emitOn(Infrastructure.getDefaultWorkerPool()).subscribe().with(
                item -> System.out.println(item + ":" + Thread.currentThread().getName())
        );
        return Uni.createFrom().item("Hello RESTEasy Reactive");
    }

}
```
Slide.27
```java
@Path("/hello")
public class ReactiveGreetingResource {

    @Inject
    EventBus bus;

    @GET
    @NonBlocking
    @Produces(MediaType.TEXT_PLAIN)
    public void hello() throws InterruptedException {
        System.out.println(Thread.currentThread().getName());
        Arrays.asList("Hello", "Aloha", "Konichiwa").forEach(
                item -> bus.<String>request("hi", item)
        );
    }

    @ConsumeEvent(value = "hi", blocking = true)
    public void doSomething(String greeting) {
        System.out.println(greeting + ":" + Thread.currentThread().getName());
    }

}
```
Slide.28
```java
package org.acme;

import io.smallrye.mutiny.Uni;
import io.vertx.core.json.JsonObject;
import io.vertx.mutiny.core.Vertx;
import io.vertx.mutiny.ext.web.client.HttpResponse;
import io.vertx.mutiny.ext.web.client.WebClient;

import javax.inject.Inject;
import javax.ws.rs.GET;
import javax.ws.rs.Path;

@Path("/joke")
public class ReactiveGreetingResource {

    @Inject
    Vertx vertx;

    private final String URL = "https://api.chucknorris.io/jokes/random";

    @GET
    public Uni<JsonObject> chuckNorrisJokes() {
        return WebClient.create(vertx).getAbs(URL).send()
                .onItem().transform(HttpResponse::bodyAsJsonObject);
    }

}
```
