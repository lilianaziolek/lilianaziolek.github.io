

Vue lifecycle methods:
https://vuejs.org/v2/guide/instance.html#Lifecycle-Diagram

beforeCreate SSR & client
created SSR & client
rest - client only

plugins- run before instantiating Vue.js app (SRR/client or both) (once per client, not before every page/route)
inject - to make available in vue instances, context, and/or store

module - executed on nuxt startup, stuff that extends nuxt functionality, e.g. automatically add and configure a plugin - it is NOT executed in browser/on each page, or even for each client

middleware - run before rendering page(s) - called once SSR (only!) and then on subsequent routes on the client

mixins - extensions to components, or pages, or layouts => can be executed e.g. on mounted or other usual hooks 

nuxtServerInit in store/index.js

fetch

asyncData




https://github.com/nuxt/nuxt.js/issues/2521

https://nuxtjs.org/guide#schema
middlewareServer // nuxt will call this for every route
plugin // nuxt will call this for every route
nuxtServerInit (store) // nuxt will call this for every route
middleware from 'nuxtConfig' // nuxt will call this for every route
middleware from 'layout'
middleware from 'page'
asyncData from 'pages'
fetch from 'pages' (store)


* Incoming Request
* nuxtServerInit
  Store action
* middleware
  1. nuxt.config.js
  2. matching layout
  3. matching page & children
* validate()
  Pages & children
* asyncData() & fetch()
  Pages & children
* Render
* Navigate
  <nuxt-link>
  (points back to middleware)

| Tables        | Are           | Cool  |
| ------------- |:-------------:| -----:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |
