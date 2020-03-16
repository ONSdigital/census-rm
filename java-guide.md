# Java Useful Reference

## The Basics

### Build & Dependency Management
We use a tool called Maven (command `mvn`) to build our projects, and a big part of this process is making sure that the right dependencies (libraries) are downloaded. Dependencies are divided into things that are needed for the application to run, and things which are only needed for testing.

Everything in the world of Maven is controlled by a file in the project root called `pom.xml`

Maven has the concept of a "parent". The parents of all the projects in RM is Spring Boot. By doing this, a lot of so-called "magic" is inherited from the parent.

Maven doesn't have the same level of dependency vulnerability checking and warning as something like Pipenv or NPM, so we must be very careful when adding new dependencies. Essentially... we have a few trusted sources, and we avoid adding obscure things at all costs.

Maven builds our code, tests our code and packages our code, as well as building a Docker image.

Maven does a lot for us, and it's also intimidatingly complex, so it's covered in more detail later on in this document.

### What the heck is this Spring thing anyway?
We use the Spring framework, which is absolutely massive, so it's important to only think about the bits which we use and find useful - try to ignore the size and complexity of the enormous Spring universe. The parts of Spring we're interested in are as follows:
- Spring Boot : This gives us a nice easy way of starting up our application, with various configuration options. It automates a lot of things.
- Spring Integration : This gives us a small part of what we need to process Rabbit messages
- Spring AMQP : This gives us the remaining Rabbit-specific parts of what we need to process and send Rabbit messages
- Spring Data JPA : This gives us what we need to Create, Read, Update and Delete (CRUD) persistent entities in the database
- Spring Web : This allows us to create RESTful APIs

As well as all the Spring magic which makes it easier for us to do things with APIs, our Postgres database and Rabbit, there is a core concept in Spring called *dependency injection* which is worth understanding. It enables us to test our classes in isolation, because we _inject_ mock versions of the things the class depends upon. Because we use something called 'autowiring' we also don't need to worry about wiring everything up ourselves - Spring does it automatically for us.

Spring is incredibly powerful, but also incredibly complicated if you try to look at _everything_ that it's capable of.

Spring is covered in detail later on in this document.

### Java Persistence API (JPA), ORMs and Hibernate
We use an Object-Relational Mapping (ORM) framework called Hibernate. The JPA classes built into Java abstract away our code from the fact that Hibernate is the framework we're using... in theory we could replace Hibernate with something else (e.g. TopLink) and our code would theoretically not have to change.

Anyway, the value of all this is that by marking a Java class as `@Entity` some magic happens to make it persistable into the database. When we create a new object, a new row is created in the DB. When we modify an object, the changes are saved to the database. We can, of course, also search for data that's stored in the database and view that as regular Java objects. Finally, we can also delete data out of the database using these 'magic' Java objects/classes. This is what Object-Relational Mapping is - it allows us to perform Create, Read, Update and Delete (CRUD) using regular Plain Old Java Objects (POJOs).

Our database access is actually a fusion of Spring Data, JPA and Hibernate, along with Postgres JDBC drivers, which all get fairly seemlessly and magically wired together into something which makes our life a lot easier.

### Other nonstandard-Java things which we use a lot
Mockito is used extensively to create mock versions of objects which can be 'injected' into classes which we are unit testing, which is to say we are wanting to test in complete isolation. Mockito allows us to control the behaviour of what should happen when the code we're testing does something with one of its dependencies, and it allows us to check to see what happened to the mock objects - did the things which we expected to happen actually happen?

JUnit is the industry-standard unit testing library, which works very well with Mockito.

Spring provides a mechanism to trigger code to be run on a timer or schedule, for example, every minute or every day at 4pm. We use this feature quite often to allow us to process batches of data, or do work on a repeating basis.

Spring also provides a very sophisticated transaction management system, which is integral to the way we process messages and API requests. Spring has the capability of ensuring that all the transactional infrastructure (i.e. the Postgres database & Rabbit) is included in the same transaction, which gets started automatically when we receive a message or API request. If we have an error, Spring will rollback the entire transaction across all the resources, such that the database changes will not take effect and any message will neither be consumed nor published.

Lombok is used to provide a lot of boring, repetitive 'boilerplate' Java code, such as the getters and setters which protect private class member variables. It can provide various kinds of constructors, builders and more complicated things like equals and hashcode methods (by adding `@EqualsAndHashCode` annotation to a class) which is hard to implement. Lombok 'magic' can mean that clicking to be taken to the definition of a method might not work properly, or your IDE might show errors if you don't have the Lombok plugin installed.

We use a GoDaddy logging library because it allows us to easily switch to JSON logging, and to attach important relevant information to anything we log by simply using the `with()` method - it will log a 'snapshot' view of any more complicated objects which you need to see.

We use a library called Orika which automates the process of copying the data from an object of one Class to a new object of a different Class - useful, for example where we have a class which is used for database persistence, and we also have another class which is used as a Data Transfer Object (DTO) for Rabbit messages and REST API requests/responses.


### Standard Java Features Worthy of Mention
Java's multithreading/parallel processing has become increasingly easier to use in the later versions of Java. Although we don't use a great deal of multithreading, it's worth understanding that we use a couple of different mechanisms of doing multithreading: the `Executor` class and the `parallel()` function.

Java has numerous lambda functions which are convenient, especially when working with lists, maps, sets etc. For example, to get a list where certain items have been filtered out, is as easy as using the `filter()` lambda function and then `collect()` the results into a new list.

Putting a public static method into a class is a very powerful pattern for helper functions, where there are "no side-effects" - for example, if I need to calculate a value from some input(s) but I don't need to do any database access or use any other kinds of dependencies, this works really well in a re-usable helper function... don't use Spring unless you actually need Spring to inject dependencies into your class. If your class's functions don't use any dependencies, then Spring shouldn't be responsible for it - just make it a Plain Old Java Object (POJO) with public static methods.


## Diving Deeper

### Maven
There aren't any websites which are particularly helpful about Maven, in the way that we use it at the ONS. We have combined a number of Maven features to create a fairly standardised project setup. It's worth going over those in this section so that you know what Maven is doing for us.

Essentially, we are feeding Java code into Maven at one end of the sausage machine, and an "executable JAR file" is created as an "artifact". What this means is that a zip file containing all the compiled Java code plus all the dependencies, is created, and this zip file (which has the extension `.jar`) can be executed by typing `java -jar <JAR file name>` and your application will start.

By default, Maven will compile your Java code and zip it all up into a JAR file. That's about all it will do. We have used lots of other Maven features to make it do loads of really useful things for us.

One of the most important jobs of Maven is to deal with the libraries that our projects depend upon, which means downloading them and storing them into your local Maven repository, so that they can be used. Dependencies can be marked with `<scope>test</scope>` which means they're only needed for testing and should not be included in the final JAR file.

After all the code successfully compiles, Maven runs all our unit tests. We use JUnit for our unit testing, so any class in the `src/test` folder which has a method annotated with `@Test` will be executed. If any unit tests fail, then the Maven build will stop.

A plugin called `docker-compose-maven-plugin` uses Docker Compose to start instances of Postgres, Rabbit and other things we might need, which have been configured to run on special testing ports, which enables our integration tests to run.

Another plugin we then use is `maven-failsafe-plugin` which we use to run our integration tests. Any test class which ends with the suffix `IT` will be run during the integration testing phase.

The `spring-boot-maven-plugin` turns our compiled Java into an executable JAR. It's necessary to tell this plugin the class which contains the `main()` function, which will be run when Java starts up.

Then, we have used an open-source plugin from Spotify called `dockerfile-maven-plugin` which we use to build a Docker image from the Dockerfile and the JAR file which will have been created in the `target` directory. It's this Docker image which is uploaded to the Google Container Registry (GCR) and will be used by Kubernates to execute our application on the Google Cloud Platform. This plugin only creates the image in your _local_ respository - it doesn't push it up to the GCR.

If you want to do further reading about Maven, the [homepage for the project is located here](http://maven.apache.org/)

I would advise against using the raw Maven reference documentation, because most of the 'clever stuff' we do with Maven comes from the Spring Boot Starter Parent project and the plugins we've used. While the documentation here might not seem very exhaustive, it does constitute most of what you need to know Maven is able to do, and to delve deeper into Maven's vast complexity would probably only serve to confuse and intimidate.

### Spring Boot
Spring's 'quick start` guide might be worth having a play with. [You can find it here](https://spring.io/quickstart)

The next guide/tutorial you might try [is this one from Baeldung](https://www.baeldung.com/spring-boot-start). Baeldung has the very best tutorials/guides out of all the sites on the web.

### Spring "Starter" Dependencies
By adding a "starter" dependency into your Spring Boot project, a lot of magic happens. Because Spring Boot aims to make everything auto-configure and wire itself, it's possible to add any "starter" dependencies that you want, and Spring Boot will detect those dependencies being present and provide you with useful error messages, if - for example - you forgot to include something important like a database driver or DB connection details.

Our projects use a combination of "starter" dependencies. For example, our REST API projects include both `spring-boot-starter-data-jpa` for database access and `spring-boot-starter-web` in order to handle requests from the web.

The starting point for each starter project is listed here:
- [Spring Boot Starter Web](https://spring.io/guides/gs/serving-web-content/)
- [Spring Boot Starter Data JPA](https://spring.io/guides/gs/accessing-data-jpa/)
- [Spring Boot Starter AMQP (RabbitMQ)](https://spring.io/guides/gs/messaging-rabbitmq/)
- [Spring Boot Starter Integration](https://spring.io/guides/gs/integration/)

Then, (some of) the better guides by Baeldung is listed here:
- [Spring Web](https://www.baeldung.com/building-a-restful-web-service-with-spring-and-java-based-configuration)
- [Spring Data JPA](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)
- [Spring Integration](https://www.baeldung.com/spring-integration)

When it comes to the combination of Spring Integration with AMQP and Rabbit, finding good guides is very hard. [This very technical reference guide](https://docs.spring.io/spring-integration/reference/html/amqp.html) is probably the best place to start, but it's quite complex and intimidating.

### Spring (in general)
It's useful to understand the function of Dependency Injection which is the basic core functionality of plain old Spring. [There's this document](https://www.baeldung.com/spring-core-annotations) which describes some of the annotations you can use. Most of the dependency injection that we need is covered by the other guides - for example, mostly we just want to be able to read/write from the database, send/receive messages and handle web requests, and as such, you should be able to pick up 'just about enough' Spring from those other guides, and never really bother too much about the details of the core of Spring.

### Java Persistence API (JPA) and Spring Data JPA
The Java Persistence API is useless on its own, because it's just an API. Hibernate is framework which provides an _implementation_ compatible with JPA, but we don't want to lock our code directly to Hibernate, so we use Spring Data JPA to abstract ourselves away from the underlying technology. In fact, we use everything in its default configuration mode, which means we're using Hibernate anyway, but all that is hidden away from us.

JPA provides the annotations to apply to Java classes and member variables, to mark them as persistent. Spring Data JPA provides Repository classes, which can do all the standard Create, Read, Update and Delete (CRUD) operations. We need both these things in combination, with Hibernate 'behind the scenes' to make our application capable of persisting data to the database.

JPA also provides a special query language called JPQL, which is 'vendor agnostic' - so, in theory we could move from Postgres to another database vendor and we would only have to change a single 'dialect' setting in order for our application to function.

Spring Data JPA provides a much more convenient way of querying the database, which is a kind of 'magic' where the name of a function creates the required query. So, if I had a table full of data about, for example, users, including their name and email address, I could write a method called `findByEmail(String email)` and Spring 'magic' would implement the query for me.

(This reference guide)[https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods] is invaluable for finding out the kinds of database queries you can do without having to write any code - just write the correctly named method and 'Spring magic' happens.

### Mockito
There should be lots of good example code in the project already to follow, but (this guide)[https://www.baeldung.com/mockito-annotations] is a very good starting point.

### Everything else
Google, click the top StackOverflow link, look for the reply with the most upvotes, double check that the reply isn't _really old_. There is far too much complexity in the assembled mass of open source technologies which constitutes RM to exhaustively cover everything. Unfortunately, it's not obvious just how much 'magic' is happening behind the scenes, because the point of the tech stack that we've chosen is to hide the plumbing and the chore of having to deal with things at a low level. The best approach is to embrace the stack we've chosen, because it should enable you to concentrate on writing the business logic and creating new features, and not having to worry about wiring everything together, or what pattern to follow, because the projects follow well-established conventions in terms of where you 'put stuff'.

Try not to be intimidated by the tech stack. It's complicated when you scratch the surface, because it's extremely mature. Somebody, somewhere, sometime, already had the problem that you needed to solve, and the stack has an easy solution... you just need to know where to look. Naturally, those who have had some time to work with the Java/Spring/Hibernate holy trinity will be able to provide some direction, and those with the most experience should be able to guide you, no matter what you want to build - don't feel as though you need to be self-reliant and trawl the vast amount of information captured here. This is just some light reading for your own pleasure and enjoyment.

