---
title:  Intercepting back button on mobile in Vue/Nuxt/Vuetify apps
categories:
  - front-end
tags: 
  - vuejs
  - nuxt
  - javascript

---

# Problem statement
The first version of OSBO was not particularly mobile-friendly. Thanks to great work done in Vuetify and Nuxt, once we started paying more attention to being mobile-friendly, the transition wasn't difficult and fairly quickly we had a page that worked quite well on mobile.  
Or so we thought. The very first test with a "real user" has shown us that on mobile clicking back button is a very strong urge when trying to close full-screen popups - for example when we show a zoomed-in image of a product. Since we're just in a browser, back button takes the user to the previous page, rather than closing the pop-up. This can be very frustrating - you are on a product page, you look at a product image, you click back - and suddenly you're back on the products list page. We decided we needed to intercept back button, and if any pop-up is open, close it instead. Simple?

Unfortunately, it is easier said than done. There isn't really such thing as "listening to back button event" in Javascript.

Another complication is that we don't want to intercept back button on desktop - only where the users are likely to be on a touch screen - that is, on mobiles and tablets.

# Device detection
This is quite a touchy subject. Unfortunately, there still isn't a 100% reliable method to do this that would work both server- and client-side. Remember, we have SSR and we want to serve correct HTML immediately - before we get to the browser and can question its capabilities or execute any clever Javascript. On the server we can only really rely on one thing - User-Agent. Yes, we know it's not 100% reliable, but there simply doesn't seem to be a better way (happy to be corrected - feel free to comment if you know a more reliable way to detect mobiles/tablets *during SSR rendering*).

To avoid reinventing the wheel, we decided to use a Nuxt module: **[nuxt-device-detect](https://www.npmjs.com/package/nuxt-device-detect)**. It exposes a set of flags, like isMobile, isTablet, isWindows, etc. via an object injected into Nuxt context and Vue instances. It is therefore possible to call something like: `this.$device.isMobileOrTablet` and get a true/false value, depending on the user agent. It works on both client and server side, which is great.

To include it in your project you only need to change two files:

package.json
```json
{
    "dependencies" : {
      "nuxt-device-detect": "~1.1.5"
    }
}
```  

nuxt.config.js
```javascript
{
  modules: [
   'nuxt-device-detect',
  ]
}
```

Then in any of your vue files you can start having conditional logic, for example:

```html
<template>
    <section>
        <div v-if="$device.isMobileOrTablet">
            <touchscreen-navbar></touchscreen-navbar>
        </div>
        <div v-else>
            <desktop-navbar></desktop-navbar>
        </div>
    </section>
</template>
```

Neat!

# Intercept back button on mobile and close popup instead
As mentioned, there isn't an event you could subscribe in Javascript that would communicate that back button is pressed. However, there is a fairly simple workaround.
1. We need to track if we have a popup open or not. If no popups are open, we should act as normal, i.e. navigate back. If any popups are open AND we are on mobile/tablet, then we will not navigate back and close the pop-up instead.
2. We need to hook into Vue router to get notified that the route is about to change (go back to previous page). We can achieve this by implementing `beforeRouteLeave` and/or `beforeRouteUpdate`. When Vue Router triggers event ("we're about to leave the current page") we can react and prevent this from happening, if appropriate. 

We should make sure we don't end up with similar code duplicated all over the place. We decided to use a combination of [eventbus plugin](https://tech.onestopbeauty.online/front-end/simple-eventBus-in-nuxt/) and 2 complementing mixins.

## Subscribe to notifications about open popups
When the page component is mounted, we subscribe to be notified about any open popups. If a popup is not open and the user presses back button, we will let the app go back.

We will create a mixin and then import it in any **page** that needs to have this functionality.

mobileBackButtoPageMixin.js (subscription part)
```javascript
export const mobileBackButtonPageMixin = {
    data() {
        return {
            dialogOpen: false
        }
    },
    mounted() {
        if (this.$device.isMobileOrTablet) {
            this.$eventBus.$on("dialogOpen", () => {
                this.dialogOpen = true;
            });
            this.$eventBus.$on("dialogClosed", () => {
                this.dialogOpen = false;
            });
        }
    },
    beforeDestroy() {
        //always remember to unsubscribe
        if (this.$device.isMobileOrTablet) {
            this.$eventBus.$off('dialogOpen');
            this.$eventBus.$off('dialogClosed');
        }
    }
    ...
};
```

## Notify from child components that a dialog is open
In every component that can open a popup we need to send a notification (`dialogOpen`) to eventBus, this will allow tracking if any popups are open. Additionally, the component needs to subscribe to `closeAllDialogs` so that a request that dialog is closed can be made (when back button is pressed on mobile). 
 Again, we will use a mixin.

mobileBackButtonDialogComponentMixin.js
```javascript
export const mobileBackButtonDialogComponentMixin = {
    methods: {
        notifyDialogStateViaEventBus(open) {
            if (open) {
                this.$eventBus.$emit('dialogOpen');
                this.$eventBus.$on("closeAllDialogs", () => {
                    this.closeAllDialogs();
                });
            } else {
                this.$eventBus.$emit('dialogClosed');
                this.$eventBus.$off("closeAllDialogs");
            }
        }
    },
};
```

This mixin needs to be imported into components:

```javascript
    import {mobileBackButtonDialogComponentMixin} from "@/mixins/mobileBackButtonDialogComponentMixin";

    export default {
    mixins: [mobileBackButtonDialogComponentMixin],
    ...
    }
```

On top of that, you need to add a watch for the property that controls the visibility of the popup. For example, if in template you have something like this:
`<v-dialog v-model="popupVisible">`
then in the component you will need to add this:
```javascript
watch: {
    popupVisible: 'notifyDialogStateViaEventBus'
},
methods: {
    closeAllDialogs(){
        this.popupVisible = false;
    }
}
```
Since we use Vuetify, our popups are v-dialogs, but this technique would work with any other component framework.

Note that you can have more than one popup in a component - you would simply add another watch, and another property set to false in "closeAllDialogs" method. 

## Intercept back navigation, close popup instead if appropriate
To get notified about route changes, we need to add two methods to the page that contains the popup (**important** - this has to be a page, and not a component).

mobileBackButtoPageMixin.js (intercept part)
```javascript
export const mobileBackButtonPageMixin = {
    ...
    beforeRouteUpdate(to, from, next) {
        if (this.$device.isMobileOrTablet && this.dialogOpen) {
            this.$eventBus.$emit('closeAllDialogs');
            next(false);
        } else {
            next();
        }
    },
    beforeRouteLeave(to, from, next) {
        if (this.$device.isMobileOrTablet && this.dialogOpen) {
            this.$eventBus.$emit('closeAllDialogs');
            next(false);
        } else {
            next();
        }
    }
};
```

Once `beforeRouteUpdate` and `beforeRouteLeave` hooks get called, we can check if we should stop navigating to previous page (are we on mobile/tablet and do we have open popups?). If we need to close popups instead, we issue an event to communicate it to components containing popups (`this.$eventBus.$emit('closeAllDialogs');`). Then `next(false)` tells Nuxt and VueRouter that navigation should not happen. Next() tells it to proceed as usual.

# Summary
This post shows you how to intercept mobile back button navigation and close a dialog instead. The example uses Nuxt and Vuetify - it is possible to apply the same concept without having Nuxt, and of course it would also work with a component framework other than Vuetify.

 As usual, the full working code can be found in [github](https://github.com/lilianaziolek/blog-examples/tree/eventBusAndBackButtonIntercept/dry-examples) - make sure to use the `eventBusAndBackButtonIntercept` branch . To test it, make sure you switch a sample mobile phone e.g. in chrome devtools *before* loading the page, so that correct UserAgent is sent.
 
 P.S. There is a cute little surprise in the popup, I hope you like it :dog: