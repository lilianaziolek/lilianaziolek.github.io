---
title:  "A (super)quick guide to Javascript ecosystem - from senior Java dev point of view"
categories:
  - javascript, java, high-level
---

# JavaScript ecosystem in a nutshell
At the end of 2016 I came across this [blog post](https://hackernoon.com/how-it-feels-to-learn-javascript-in-2016-d3a717dd577f). It was at a point when I was beginning to think of building a startup, and I knew that this would involve me finally leaving my Java/backend-only fortress that I happily occupied throughout my whole carrier, and moving at least somewhat into the "enemy grounds". This blog post was funny, but also in some ways petrifying. It confirmed all my worries about what it will look like - to have to do any front-end work. It sounded simply crazy.

Now, almost 3 years later and many many lines of Vue code later, I want to make the front-end world a little bit less intimidating for folks who are like me (back then). Competent / senior Java devs, who for this reason or other (choice or circumstances) did not get much experience doing (much) front-end work, and are not really sure where to start.

# Java ecosystem in a nutshell
When you stop and think for a moment, Java world is also way more than just Java once it comes to anything more than HelloWorld. I used to mentor a few junior developers, and I recently felt a little bit sorry for the steep learning curve they must face. If you join a modern project these days, from day one you'll probably come across a number of the following names (in no particular order):\
Maven / Gradle; Spring, Spring JDBC, Spring MVC, Spring Boot, Spring Cloud, Spring ... ; Hibernate; Lombok, Guava, Apache Commons; Jackson, GSON, Jaxb; Spark; Camel; JMS; Tomcat, Jetty, Netty; Eureka, Hystrix, Ribbon; JUnit, Mockito, AssertJ, Cucumber; Slf4j, Logback, Log4j; Docker\
*Not to mention*: traditional DBs + SQL; MongoDB; Elasticsearch; Cassandra; Neo4j; Couchbase; Kafka; Ehcache, ...\
*And also*: AWS, Google Cloud Platform, Azure - all with their respective hundreds of products.\
And that's just stuff from the top of my head, in reality there is just so much more.

Most of us don't really think about it because we're already familiar with this stack. We add tools and frameworks when we need them, we learn another thing, we move on. It's when you look at all of this in one place that you realise the number of moving parts involved.

And with this small detour I arrive at a confession: I honestly don't know why I thought JavaScript world would be any different :)

# Making sense of both
Luckily, a lot of things map conceptually fairly easily to what we're already familiar with, and the rest make sense logically. Our stack at the moment consists of Vue/Nuxt/Vuetify and as such, I'll go from that perspective.
 So without further ado:

- **Vue** - the mapping into Java world is not always going to be obvious, and I think Vue vs React vs Angular is one of these things that is not strictly translatable. Important thing to know is that whilst the main framework can change, a lot of the underlying tools and concepts I'll mention further on do not. 
- **NodeJS**
- **Nuxt** - this is like a swiss-army knife of the Vue world. It's what Spring Boot is for Java. Opinionated framework, and you better stick to the conventions - but when you do it can save so, SOOOO much time. It integrates a number of other goodies, from VueX, Vue Router, to webpack, and loads of other things, and Just Works. I love it. We didn't start with Nuxt, and whilst the journey without it has taught me a lot, if time is precious - use Nuxt. All the following comes for free (otherwise it will be up to you to make these things play nicely together)
  - **VueX** - state management library, kind of like an in-memory globally available cache for Vue apps. You'll probably need it for pretty much any app more complex than a static page with very little data.
  - **VueRouter** - a bit like Spring MVC / Controller routes - basically, indicates which bit of your code is responsible for which part of your app
  - **SSR vs client mode**      