---
title:  "Understanding Nuxt & Vue hooks and lifecycle (part 1)"
categories:
  - front-end
tags: 
  - vuejs
  - nuxt
  - javascript

---

# Remember, Young Padawan: DRY 
One of the software development principles that we get taught very early in our dev careers is DRY - **Don't Repeat Yourself**. It's a good thing too, as there isn't much worse than trying to crawl through a massive codebase trying to find all the copy-pasted instances of the same piece of logic.

When we first started with Vue (and later Nuxt) I wasn't always sure where to put certain bits of code, like getting data from the server, or checking if the user is logged in. Enter: the topic of this mini series. I will start with a quick recap of what mechanisms are available in Vue/Nuxt landscapes, I'll follow with examples of situations when each of these might be useful, I'll point out places where we got it wrong so that you don't have to, and summarise the whole thing with a little reference table.

One of the trickiest aspects was to reconcile how the situation varies between SSR and client-side, and there were a few cases we had to work out why things would work on refresh but not route change, or vice-versa. We sometimes got it wrong when various hooks/methods get called and, more importantly, when they don't get called. The information is usually somewhere in the docs (plus, the documentation has massively improved in the last year or so) - but I think it's nice to have it collected all in one place. 

# Recap: Vue lifecycle
Vue documentation has an [excellent diagram](https://vuejs.org/v2/guide/instance.html#Lifecycle-Diagram) showing the order/situations in which Vue component methods are called. Unfortunately, it does not clearly mention a few important things (as it's related more to how Nuxt operates in universal mode, than to pure Vue).
 * *Only **beforeCreate** and **created** are called during SSR* (as well as on the client). All the other methods (most importantly: mounted, which is one used quite often in examples) are only called on the client. So, if you have a piece of logic that needs to be executed during SSR, mounted (which, otherwise, is often a good place for some extra logic) is not a good place for this. 
 * *beforeCreate* does not have access to the components props/data because **this** (the component reference) is still undefined. 
 * *created*, does have access to **this**, including data and props, but does not have access to DOM. How does it matter? If you ever want to use e.g. [this.$refs](https://vuejs.org/v2/api/#ref), they are not initialised at this point. They will only be processed (visible) in mounted. Which does not run in SSR.
  
# Recap: Nuxt-specific tools
Note: many of the following methods accept [Nuxt context](https://nuxtjs.org/api/context/) as one of the parameters.
## Plugins
[Plugins](https://nuxtjs.org/guide/plugins) are bits of code that get executed **once or twice** per visitor, before Vue.js app instance gets created. You can have plugin that's executed on both server and client side (thus twice total), or only on one side. Nuxt has useful convention that any plugin called XXX.client.js get executed only on client side, and YYY.server.js is only in SSR. Additionally, Nuxt makes available an **inject** method that allows you to make shared code/functionality available in vue instances/components, Nuxt context, and/or VueX store. A popular plugin is Axios, which allows you to access a shared Axios instance e.g. via this.$axios. Similarly, you can create your own plugin and access it, e.g. via this.$eventBus. 
## Modules
A [module](https://nuxtjs.org/guide/modules/) code is executed on Nuxt startup (i.e. **once during the lifetime of your Node.js server**). Modules extend nuxt functionality - for example they can automatically add and configure a plugin. It is NOT executed in browser/on each page, or even on the server for each client accessing your page. Therefore, modules are not a good place for any code that should be executed for each visitor. Unless, of course, your Nuxt module adds code into one of the hooks that do get executed for each visitor - but the module code itself would run just once, to initialise certain hooks.
## nuxtServerInit in store/index.js
One of the first actions executed in SSR (only) is [nuxtServerInit](https://nuxtjs.org/guide/vuex-store#the-nuxtserverinit-action). It is executed just **once** for each visitor to your website (when they first navigate to your website, or when they hit refresh). It is a good place to put Axios calls to get some commonly used data and put it in store.
## Middleware
[Middleware](https://nuxtjs.org/guide/routing#middleware) is executed before rendering each page (before route is loaded), regardless if you're on server- or client-side. You can have global middleware (configured in nuxt.config.js), or localised middleware, attached only to certain layouts and/or pages. It's important to know that middleware is only executed once before render - i.e. on first hit to the page it will be executed in SSR only. On subsequent pages/routes it will be executed on the client only. It does not get called on both client and server for the same page.
## Mixins
[Mixins](https://vuejs.org/v2/guide/mixins.html) are extensions to components, pages, or layouts. They have access to the whole component that they are mixed into - so they can use this.$route, this.$store, and anything else you'd be able to call in the component. They are very useful to extract common functionality (including hooks like mounted) that for some reason cannot be extracted as standalone components. In simple terms, they behave in the same way as if you had copy-pasted the mixin code into each component where it's used.
## asyncData & fetch
Both [asyncData](https://nuxtjs.org/guide/async-data) and [fetch](https://nuxtjs.org/api/pages-fetch) methods are executed before the component is initialised and so do not have access to **this**. Both can be used to get some data from an API to populate the component. Both are **only* executed for pages (NOT components). Both take Nuxt context as parameter. Both will be executed on server-side on first load, and on client side for subsequent route changes. (*Note*: there are some subtle caveats here as to when these are called or not which I'll get into in a separate post)
 * **asyncData** should return a promise, or use async/await - but in either case, the result returned will be integrated into **data** part of the component
 * **fetch**, on the other hand, should be used for data intended for VueX store - it does not need to return anything and should instead commit to store any required data. It can use async/await.
## Bonus: watch route
None of asyncData or fetch get triggered in some specific situations when only route params change. For this situation, you may need to watch the route to refresh data - or change your router configuration. More details in a separate post.

Official Nuxt documentation has a [useful diagram](https://nuxtjs.org/guide#schema) showing the order in which things are called. Let's go into a bit more details into what it means in relation to typical user-app interaction.

# Example
The code for this post (and all the more detailed follow-ups in this series) can be found on [Github]().

In the next post (or a few) in this series, I'll go through what exactly happens as the user navigates through the app, and will point out various techniques and gotchas related to the tools covered above. 
