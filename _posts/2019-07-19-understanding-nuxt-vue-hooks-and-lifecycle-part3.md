---
title:  "Understanding Nuxt & Vue hooks and lifecycle (part 3)"
categories:
  - front-end
tags: 
  - vuejs
  - nuxt
  - javascript

---

This is part 3 of mini-series - Understanding Nuxt & Vue hooks and lifecycle - a quick reference table to refresh memory. 

If you've missed the previous parts:
* [Part 1 here](https://tech.onestopbeauty.online/front-end/understanding-nuxt-vue-hooks-and-lifecycle-part1/) - which explains each of the mechanisms in more details, 
* [Part 2 here](https://tech.onestopbeauty.online/front-end/understanding-nuxt-vue-hooks-and-lifecycle-part2/) - which shows each of the mechanisms on an example app,
* [Quick guide to Vue and Nuxt from Java dev PoV](https://tech.onestopbeauty.online/high-level/quick-guide-to-javascript-ecosystem-from-senior-java-dev-pov/). 

I'm not adding modules to this table because, as explained in Parts 1 & 2, the module code only gets executed on Nuxt startup. Of course module code might initialise/attach various hooks - but then they'd follow the rules below.

                
What               | SSR (1st page)     | Client (1st page) | Client (Next pages) | Notes                                         | Example usage
-------------------|--------------------|-------------------|---------------------|-----------------------------------------------|---------------
beforeCreate       |:heavy_check_mark:  |:heavy_check_mark: |:heavy_check_mark:   | Does not have access to component's *this* (does not exist yet) | If you're not using Nuxt: getting/preparing any data that is required by the component. With Nuxt, fetch/asyncData is easier 
created            |:heavy_check_mark:  |:heavy_check_mark: |:heavy_check_mark:   | Has access to component's data, but not DOM (no `this.$refs`) | (in client mode) generate and attach extra styles to document; process data/props with extra logic (could also be in computed prop) 
mounted            |:x:                 |:heavy_check_mark: |:heavy_check_mark:   | First hook with access to data and DOM        |
plugins (dual mode)|:heavy_check_mark:  |:heavy_check_mark: |:x:                  | use inject to make plugins available globally | globally shared functionality, e.g. this.$user.isLoggedIn (goes to store behind the scenes)     
plugins (client)   |:x:                 |:heavy_check_mark: |:x:                  | use inject to make plugins available globally | actions that need to be performed once per visitor (client-side), e.g. setting up authorisation tokens
plugins (server)   |:heavy_check_mark:  |:x:                |:x:                  | use inject to make plugins available globally | actions that need to be performed once per visitor (server-side) 
nuxtServerInit     |:heavy_check_mark:  |:x:                |:x:                  | used for VueX initialisation                  | fetch globally used data, e.g. elements for navigation menu or other configuration from API 
middleware         |:heavy_check_mark:  |:x:                |:heavy_check_mark:   | can be attached globally, or just to some pages | automatic redirects for certain pages - e.g. when content moved, or if user tries to access protected page when not logged in
asyncData / fetch  |:heavy_check_mark:  |:x:                |:heavy_check_mark:   | only executed for pages, not components       | fetch data (into store or component) required on certain route
