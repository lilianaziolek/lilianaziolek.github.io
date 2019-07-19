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

<table>
<tr>
<th>What</th>
<th>SSR (1st page)</th>
<th>Client (1st page)</th>
<th>Client (Next pages)</th>
<th>Notes</th>
<th>Example usage</th>
</tr>

<tr>
<td>beforeCreate</td>
<td>:heavy_check_mark:</td>
<td>:heavy_check_mark:</td>
<td>:heavy_check_mark:</td>
<td>Does not have access to component's *this* (does not exist yet)</td>
<td>If you're not using Nuxt: getting/preparing any data that is required by the component. With Nuxt, fetch/asyncData is easier</td>
</tr>

<tr>
<td>created</td>            
<td>:heavy_check_mark:</td> 
<td>:heavy_check_mark:</td> 
<td>:heavy_check_mark:</td>  
<td>Has access to component's data, but not DOM (no `this.$refs`)</td> 
<td>(in client mode) generate and attach extra styles to document; process data/props with extra logic (could also be in computed prop)</td>
</tr>

<tr>
<td>mounted</td>            
<td>:x:</td>               
<td>:heavy_check_mark:</td> 
<td>:heavy_check_mark:</td>  
<td>First hook with access to data and DOM</td>
<td>DOM operations, client-side operations such as subscribing to events</td>
</tr>

<tr>
<td>plugins (dual mode)</td>
<td>:heavy_check_mark:</td>     
<td>:heavy_check_mark:</td> 
<td>:x:</td>                 
<td>use inject to make plugins available globally</td> 
<td>globally shared functionality, e.g. this.$user.isLoggedIn (goes to store behind the scenes)</td>
</tr>

<tr>
<td>plugins (client)</td>   
<td>:x:</td>               
<td>:heavy_check_mark:</td> 
<td>:x:</td>                 
<td>use inject to make plugins available globally</td> 
<td>actions that need to be performed once per visitor (client-side), e.g. setting up authorisation tokens</td>
</tr>

<tr>
<td>plugins (server)</td>   
<td>:heavy_check_mark:</td> 
<td>:x:</td>                
<td>:x:</td>                 
<td>use inject to make plugins available globally</td> 
<td>actions that need to be performed once per visitor (server-side)</td>
</tr>

<tr>
<td>nuxtServerInit</td>     
<td>:heavy_check_mark:</td> 
<td>:x:</td>                
<td>:x:</td>                 
<td>used for VueX initialisation</td>                  
<td>fetch globally used data, e.g. elements for navigation menu or other configuration from API</td>
</tr>

<tr>
<td>middleware</td>         
<td>:heavy_check_mark:</td>
<td>:x:</td>                
<td>:heavy_check_mark:</td>  
<td>can be attached globally, or just to some pages</td> 
<td>automatic redirects for certain pages - e.g. when content moved, or if user tries to access protected page when not logged in</td>
</tr>

<tr>
<td>asyncData / fetch</td>  
<td>:heavy_check_mark:</td>
<td>:x:</td>                
<td>:heavy_check_mark:</td>  
<td>only executed for pages, not components</td>       
<td>fetch data (into store or component) required on certain route</td>
</tr>

</table>
                


