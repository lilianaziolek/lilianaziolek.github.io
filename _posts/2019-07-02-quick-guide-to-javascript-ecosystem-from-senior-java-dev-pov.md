---
title:  "A (super)quick guide to VueJS ecosystem - from senior Java dev point of view"
categories:
  - high-level
tags: 
  - javascript
  - java
  - vuejs
  - nuxt
---

# JavaScript ecosystem in a nutshell
At the end of 2016 or beginning of 2017 I came across this [blog post](https://hackernoon.com/how-it-feels-to-learn-javascript-in-2016-d3a717dd577f). It was at a point when I was beginning to think of building [OSBO](https://www.onestopbeauty.online), and I knew that this would involve finally leaving my Java/backend-only fortress that I happily occupied throughout my whole carrier, and moving at least somewhat into the "enemy grounds". This blog post was funny, but also in some ways petrifying. It confirmed all my worries about what it will look like - to have to do any front-end work. It sounded simply crazy.

Now, 2.5 years later and many many lines of Vue code later, I want to make the front-end world a little bit less intimidating for folks who are like me (back then). Competent / senior Java devs, who for this reason or other (choice or circumstances) did not get much experience doing (much) front-end work, and are not really sure where to start.

# Java ecosystem in a nutshell
When you stop and think for a moment, Java world is also way more than just Java once it comes to anything more than HelloWorld. I used to mentor a few junior developers, and I recently felt a little bit sorry for the steep learning curve they must face. If you join a modern project these days, from day one you'll probably come across a number of the following names (in no particular order):  
Maven / Gradle; Spring, Spring JDBC, Spring MVC, Spring Boot, Spring Cloud, Spring ... ; Hibernate; Lombok, Guava, Apache Commons; Jackson, GSON, Jaxb; Spark; Camel; JMS; Tomcat, Jetty, Netty; Eureka, Hystrix, Ribbon; JUnit, Mockito, AssertJ, Cucumber; Slf4j, Logback, Log4j; Docker  
*Not to mention*: traditional DBs + SQL; MongoDB; Elasticsearch; Cassandra; Neo4j; Couchbase; Kafka; Ehcache, ...  
*And also*: AWS, Google Cloud Platform, Azure - all with their respective hundreds of products.  
And that's just stuff from the top of my head, a tip of the iceberg. There is just so much more.

Most of us don't really think about it because we're already familiar with this stack. We add tools and frameworks as and when we need them, we learn another thing, we move on. It's when you look at all of this in one place, from the perspective of a newbie, that you realise the number of moving parts involved.

And with this small detour I arrive at a confession: I honestly don't know why I thought JavaScript world would be any different :)

# Making sense of both
Luckily, a lot of things map conceptually fairly easily to what we're already familiar with, and the rest makes sense logically. Our stack at the moment consists of Vue/Nuxt/Vuetify and as such, I'll go from that perspective.  
 So without further ado:

- **Vue** - the mapping into Java world is not always going to be obvious, and I think Vue vs React vs Angular is one of these things that is not strictly translatable. Maybe the closest would be Java vs Kotlin vs Clojure vs Scala vs... - you still have the underlying core (JVM + bytecode) just as with web frameworks you have browsers, HTTP, HTML, CSS, JavaScript/Typescript at the core of what eventually runs.
- **NodeJS** - think: Tomcat/Jetty and the likes. Just as much as you don't need them for every single Java app, once you hit any more sophisticated/dynamic projects, you'll most likely use it.
- **Nuxt** - this is like a swiss-army knife of the Vue world. It's what Spring Boot is for Java. Opinionated framework, and you better stick to the conventions - but when you do it can save so, SOOOO much time. It integrates a number of other goodies, from VueX, Vue Router, to webpack, and loads of other things, and Just Works. I love it. All the following comes for free (otherwise it will be up to you to make these things play nicely together)
    - **Vuetify** - a material-design components library. Vue itself is mostly about "language" to describe your app. Think loops, and conditionals, and structure. Vuetify is what brings you out-of-the-box nicely styled buttons, tables, iterators, tabs and many many other building blocks so that your page can look pretty. You could use Vue with pure HTML/CSS, or many other components libraries, or some simple CSS layers above - it's all up to personal taste. We found Vuetify extremely beginner friendly so if you're not a CSS Ninja, you can't go wrong starting here.
    - **VueX** - state management library, kind of like an in-memory globally available cache for Vue apps. You'll probably need it for pretty much any app more complex than a static page with very little data.
    - **VueRouter** - a bit like Spring MVC/Controllers routes - basically, indicates which bit of your code is responsible for which part of your app
    - **SSR vs client mode vs statically rendered content** - this deserves its own post really, to go into nitty-gritty details, but for now there is one thing to understand. Nuxt gives you three options to run Vue:
        - *statically rendered site*, meaning you write your code in Nuxt+Vue, and then you create a beautiful static page, i.e. there is no Node.js, you just serve plain HTML/CSS/Javascript, even from something like S3. Think, a static HTML on your hard drive.
        - *full SPA (Single Page Application) mode*, that is, your app is delivered as a pretty much empty shell to the browser, and the browser executes Javascript to dynamically create HTML/DOM
        - *universal mode* - the first hit to your page will be executed on the Node.js server (thus name: SSR, server-side-rendering), and then subsequent pages/routes within this client's session (to be precise: until someone closes/reopens tab, or clicks refresh) will be handled by the browser  
      
  The benefit of generated static site is pretty obvious - it's much easier to serve. However, you won't be able to use it for heavily dynamic/data driven apps. If you can't use that, what's the benefit of universal/SSR mode vs SPA? In short: SEO. These days search bots are much much better with Javascript than they used to. If you have a page with just a bit of JS on it, you'll probably still get the page index just fine. Unfortunately, in our experience, with anything more sophisticated, when you drive your page from quite a few data-calls, bots don't wait long enough/process everything sufficiently to index it correctly.  
  Nuxt makes it incredibly easy to use SSR, and when we realised we needed SSR, that was the point when we started using Nuxt as without it we were in a World-Of-Pain.
    - **Axios** - Axios =~ Spring RestTemplate. A neat library to make HTTP calls. Nicely integrated into Nuxt so that you can access it anywhere you need it with very little configuration.
    - **PWA (Progressive Web App)** - according to Google, *A Progressive Web App (PWA) is a web app that uses modern web capabilities to deliver an app-like experience to users.* Nuxt comes with a module which makes creating PWA easier. (We're only at the beginning of the journey here but I may write more about it later on)
    - **npm/yarn + webpack** - I roll it into one point even though these are independent technologies - because to me, it all fits into "how do I manage my dependencies and build the thing". That is, Maven/Gradle equivalent. The centre here is package.json (think: build.gradle / pom.xml). The webpack part is not something you need to think about much when you use Nuxt - so we don't - but you can configure it quite a bit when needed.
  - **Babel** - kind of related to above. would you be happy to be stuck on Java 1.4 or 5.0 and not to be able to use all the stuff that came in Java 6, 7, 8... ? (rhetorical question) Not surprisingly, JavaScript folks are not to keen on being stuck on some old JavaScript syntax either. But unlike in Java world, you have very little control over what environment (browser) your code will run in. So in some ways, Babel is kind of like a clever Java compiler, which converts your brand-new-code into something that an old JVM... I mean old browser... can still understand. Neat. Oh, and the best thing? If you use Nuxt, all the magic happens without you even thinking about it.  
  BTW, have you noticed I keep using "JavaScript" here - in fairness, I should probably say JS, EcmaScript, TypeScript... - but that would just confuse things at this stage, so let's stick to JS as a mental shortcut, knowing it's not strictly just that.
  - **Eslint + Prettifier** - sort of like Findbugs, PMD and code style checkers in Java world. We actually don't have them turned on as they were extremely noisy in default config, and I didn't have the time to fine-tune it - but it's something on my (never-ending) TODO list. 
  - **Jest and Cypress** - testing testing testing. Jest is like JUnit, Cypress we find useful for high-level/functional testing. Many options out there, these seemed to agree with us most.
  
  And, frankly, that's it! That's all you need to know to start your journey with Vue/Vuetify/Nuxt. Yes, of course there is way way more, especially when you start looking under the hood a bit more, or have unusual requirements - but it is entirely possible to get productive just being vaguely familiar with the above. It's all you need to build an app, and not just a super simple Hello World!
  
# BONUS 1: Why VueJS and not React or Angular?          
I get this question a lot from my dev friends so might just as well address it here, once. Angular was easy - I absolutely hate Google's tendency to just abandon projects, and I'm convinced they'll do it again, so I didn't even look into it any further than that. To be frank, I have nothing against React per se - maybe except that it's made by an evil evil company that I prefer to keep at arms length. But otherwise, I'm sure it's a brilliant piece of tech. So why not?

Our project is built by two people, myself and a brand new developer, a person who at the start of the project could claim as much experience as doing an HTML website in Dreamweaver. I looked into React first, but the whole "Javascript only" attitude scared me a bit. Even for me, getting a simple app only just a bit beyond "Hello World" was not a 5 minute job, I didn't understand what was happening. The fact that Vue has this neat concept of combining HTML (structure) + CSS (style) + Javascript (actions) into components seemed much easier to grasp for a newbie, and it makes an awful lot of sense to me. There is also a great choice of really basic materials about HTML and CSS. You can learn more gradually. React? It just felt like too steep a curve to start with.
  
*A fun fact*: when we first started, because I was so "hardcore Java" we didn't even use Nuxt. We didn't use Node.js. We started with all rolled into single app, a Spring Boot with a bit of FreeMarker sprinkled with plain Vue. Times of Javascript libraries served from Webjars. Then adding Vue Router and VueX manually. It was fun times, I've learned a lot about the stack that way - but it's not necessarily a way I'd recommend if you value your time ;) I think the React docs might be a bit better now, but back then, it was really pushing you down the full-stack route, and I simply wasn't ready for that.

So here we are. I did not regret this decision at any point. Yes, having React skills would probably be a bit more practical from perspective of "more jobs out there" - but otherwise, we are very happy with how Vue works.

# BONUS 2: What are the gotchas?
## Environment
So far, there is one major "gotcha" that really bugs me about Nuxt/Vue combo and is something that as a back-end dev you're likely to trip on. The concept of "build-once-deploy-multiple-times". This is something really tricky to do at the moment, and it involves a bunch of hacks rather than a neat, standard solution.  
In your usual Java app (not going too crazy with sth like Spring Cloud Config Server), you'll often have externalised config in the form of properties/yml files, and/or passing in environment variables. It's the latter that will likely give you an infinite amount of grief because *environment variables in certain parts of Nuxt are baked in at build time*. Let me repeat that. Nuxt/Webpack build takes your environment variables **during build time** and bakes them into the generated resources. They are not taken from the environment at runtime.

What makes it more confusing is that this is not 100% the case for all of your app / use cases. There is a [plugin](https://www.npmjs.com/package/nuxt-env) for Nuxt that allows you to read and use runtime environment variables. A good rule of thumb is: if you're using something in your own code, in your components - you'll be fine using runtime $env variables. However, and this is where things are getting nasty, if you're using a 3-rd party Nuxt plugin or module (e.g. for google analytics) and it's configured in nuxt.config.js - you're stuffed. There is currently no elegant way for you to use environment variables for this purpose. It's extra confusing as nuxt.config.js is run twice - during build, and then on your (built) server startup. So if you have something like:
```javascript
console.log("Full environment we're running in: " + JSON.stringify(process.env));
``` 
at the beginning of your nuxt.config.js, then it might SEEM like the env variable is set correctly. Except, by the time this code is run, the variable in your config has already been hardcoded to the value that was present during the build.  
It's even more (!) confusing because if you run in dev mode (the one you will usually use during testing on localhost) everything will work because build and run are effectively the same process - so setting an environment variable for this process will work just fine. 

Yuck. This makes running things in Docker / cloud non-trivial, and effectively forces you to rebuild (at least part of) the app when you deploy (or use one of many possible hacks, which I may go into in a future post). I really hope that Nuxt team will find a neater solution at some point as at the moment, it feels really bad.

## Reactivity
When you start using Vue, it may take a little bit of time to get your head around how exactly Vue's *magic* reactivity works. We used to have cases where we were trying to use a dynamic value, and it was not updating the view the way we expected it to. It doesn't happen to us anymore, so I think now we intuitively grasped how reactivity works - but in the past it wasn't always obvious. If people come up with any examples of reactivity not working, I think I could try work out why, and perhaps break it down into more intuitive rules/way of looking at it.
 
# CODE
Technically, there isn't much of code to show here. Nuxt has a great generator for a skeleton project, all you need to do (after installing yarn and node.js), is run:
```bash
yarn create nuxt-app plain-nuxt-app
```
It will ask you a couple of questions about what you want included in your project. An example with choices equivalent to what we have in our project can be found in [examples/plain-nuxt-app](/examples/plain-nuxt-app)
The linting configuration that comes enabled by default is super strict so you might want to skip it if you are only just starting - otherwise you can get some scary looking confusing errors and warnings.

And that's all for today, folks. If any of the points or topics above is particularly interesting, please comment/request more info below! 
