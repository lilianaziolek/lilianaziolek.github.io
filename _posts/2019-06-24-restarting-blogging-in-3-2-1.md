---
title:  "(Re)starting blogging in 3..2..1"
categories:
  - blog
---

Many many years ago, I had a tech blog ( [London Geekette](https://london-geekette.blogspot.com) ). I may at some point move the stuff from there here - if I deem it remotely relevant / up-to-date. In the meantime, it is a brand new world for me, and a brand new blog seemed fitting.

Also, to be frank, another part of the motivation was that writing (formatting) tech content in blogspot was a real PitA. This new way of writing in Markup, and committing to Github seemed SO much easier (and, of course, geekier). If you don't know what I'm talking about, check out [Github Pages](https://help.github.com/en/articles/using-jekyll-as-a-static-site-generator-with-github-pages).  Or, even easier, start from [Minimal Mistakes Template](https://mmistakes.github.io/minimal-mistakes/) which is the one currently on this blog. It's quite customizable (I'll play with it more later) and comes with a [nice example project](https://github.com/mmistakes/mm-github-pages-starter) - just copy and commit under your_github_user.github.io repo. The page will shortly be available under https://your_github_user.github.io - and you can also [add a custom domain](https://help.github.com/en/articles/using-a-custom-domain-with-github-pages), which I'll be doing later. And I'll also be playing a bit with the template.

Anyway, that's a small digression but I was quite excited about how easy it was to set it all up. :)


The brand new world I'm talking about is getting out of corporate environment of contracting for financial institutions and into a world of "indie hackers" - that is, going solo (well, almost, I have a +1 for this), building something from scratch, using any tech stack I want. Naturally, a lot of that stack is stuff I used before and liked. Despite (some) hate Java gets, I still consider it one I'm most productive in. I could, I'm sure, learn Kotlin, Clojure or Scala, or maybe take Go, or whatever else is the "cool new kid" - and they all have some benefits, definitely. But the honest truth is, I knew I had SO MUCH new stuff to learn, I just wanted to have a handful of reliable parts of my tech stack where I genuinely know what I'm doing and don't waste time googling something obvious.

There were 2 main areas I needed to get into - the first was front-end, and the other one was (or rather, will be, if all goes well), machine learning, and thus Python (rejected idea of going with R, sorry data scientists, too much out of my comfort zone). Therefore, this blog will contain the following mixture of topics, depending on what I'm working on and what I find challenging:

- **Java** and all stuff related: Spring, JVM, libraries/frameworks
- Front-end: our choice of poison is **Vue** with Nuxt.js and Vuetify (could not be happier with our choice, I'll get to why in a dedicated post soon)
- **Neo4j** as the database - again, quite happy how it worked so far, the lack of rigid schema really helped us make changes in data structure quickly which is nice when you're working with domain that you don't know so well initially
- At the time of writing, we run things on **Google Cloud Platform** (plus search in AWS, just because I was too lazy/busy to setup Elastic search myself) and fought some battles here already. This may or may not change - most of the project is 100% portable, there is one very isolated area which would be easy to rewrite, although obviously deployment would need to be re-done.  
- **Python & ML** - honestly, this is very very basic at the moment, and I won't be writing about it for quite some time. [OSBO](https://www.onestopbeauty.online) needs to grow first so that I have data to machine-learn on!

The main goal I have for this blog is, whenever talking about specific tech problem, to provide a fully working github repo with an example. I haven't yet decided how exactly it's going to work, if I'll share any code between different posts or not, but here is the challenge I often come across. Product you're using has feature A, and there are docs, and plenty examples for how to set-up feature A. Same product (or sometimes, for more fun, another) has feature B, and again, there's documentation, examples, happy days. The problems start when you try to take a bit of feature A and a bit of feature B and make them play with each other. And suddenly there are no examples, and nothing works, and you're left banging your head against the wall desperately googling for that one, just one example that uses both *at the same time*, AND is more than just a few code snippets which you don't know where to paste and when you try to put them in your project, still don't work.

I know exactly why many blogs are written this way. There is great benefit to learning things in isolation. It's easier to show, easier to understand, easier to find. Still, I sometimes think that more complex examples are missing (especially for relatively modern technologies) - and this is what I'd love to focus on. How this works out in practice - we'll see :)   

So, that's it folks in the way of introduction. If any of this sounds interesting, tune in and follow for updates :)