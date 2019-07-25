---
title:  "How to build a simple event bus in Vue & Nuxt?"
categories:
  - front-end
tags: 
  - vuejs
  - nuxt

---

This post will be short and sweet, as it's really just a prep for another one coming up soon (intercepting back button on mobile in Vue/Nuxt apps).

# The problem
[Event bus, related to publish-subscribe pattern](https://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern) is a fairly fundamental concept in software development. If you have not heard of it, I'd recommend reading the wikipedia entry to understand the rest of the post.

In short, event bus allows you to decouple various parts of the system which somehow depend on things (events) happening in another part of the system. As an example, think of a situation where user being logged in should trigger a fetch of extra data in certain components.  
Some people might argue that with Vue's reactivity and VueX an event bus is not necessary. It is true to some degree - in that these two mechanisms greatly reduce the need for any explicit publish/subscribe to happen. However, in my opinion, whilst you could try to always just use computed properties or watches, the event bus might be in some cases a much simpler, and well-known pattern. As a developer, it's good to have various tools and choose them depending on what produces the easiest, most readable and maintainable code.

# Vue $on/$emit/v-on
Vue comes with an inbuilt event bus / publish-subscribe mechanism. Any Vue instance exposes a few related methods, including: [`$on`](https://vuejs.org/v2/api/#vm-on) and [`$emit`](https://vuejs.org/v2/api/#vm-emit).

## Reminder: Local events
Usually, we are using the $emit method and v-on directive for communication between parent and child components.

For example, in a child component consisting of a dialog (`ComponentPart.vue`) with a close button, we could have the following:

```html
<v-btn @click="$emit('close')">
    <v-icon>close</v-icon>
</v-btn>
```

And then the following in the parent component:
```html
<v-dialog v-model="dialog" >
    <component-part @close="dialog = false"></component-part>
</v-dialog>
```

Note that the `@close` is just a shortcut for `v-on:close`. (Can you guess what happens inside v-btn that allows us to write `@click`?)

## event bus plugin
event bus is using the same mechanism, except that we need to get hold of an instance of a component available globally, and instead of using `v-on`, we will use `$on`. As we covered in [previous series of posts](https://tech.onestopbeauty.online/front-end/understanding-nuxt-vue-hooks-and-lifecycle-part3/), to do something for each visitor and do it only once, on the client, we can create a plugin. This will initialise our event bus.

*eventBus.client.js*
```javascript
import Vue from 'vue'

const eventBus = new Vue();
//this helps WebStorm with autocompletion, otherwise it's not needed
Vue.prototype.$eventBus = eventBus;

export default ({app}, inject) => {
    inject('eventBus', eventBus);
}
```

## Example usage:
Let's say that in our VueX store we have communication with the back-end that gets kicked off after user logs in (simulated here just by having a Login button) and retrieves user details, e.g. telling us if the user is admin. Once we know if user is admin, we want to fetch some extra admin data to display in a component. With the $eventBus, it would look like so:

### Notify when user details change 
store/user.js
```javascript
export const state = () => ({
  userDetails: {
    admin: false
  },
});

export const mutations = {
  reverseUserDetails(state) {
    state.userDetails = {admin: !state.userDetails.admin};
  }
};
export const actions = {
  async fetchUserDetails({commit}) {
    // normally we'd have an axios call here, it would call our API to get user details
    // here I'm just hardcoding the userDetails to values opposite to what they were
    // every time when you "Login" and fetchUserDetails is called you will switch between admin and non-admin
    commit("reverseUserDetails");

    this.$eventBus.$emit("userDetailsChanged");
  }
};
```

### Subscribe to the event in the respective component
components/AdminDataDemo.vue
```html
<template>
  <div>
    <span v-if="isAdmin">{{adminData.someValue}}</span>
    <span v-else>Current user is not admin</span>
  </div>
</template>

<script>
  import {mapState} from 'vuex';

  export default {
    name: "AdminDataDemo",
    computed: {
      ...mapState({
        isAdmin: state => state.user.userDetails.admin,
        adminData: state => state.admin.adminData
      })
    },
    created() {
      //this listener is not needed in SSR-mode
      if (process.client) {
        console.log("Subscribing to know when userDetails change");
        this.$eventBus.$on("userDetailsChanged", () => {
          console.log("We were notified that user details changed, reacting, admin: " + this.isAdmin);
          if (this.isAdmin) {
            this.$store.dispatch('admin/fetchAdminData')
          } else {
            this.$store.dispatch('admin/removeAdminData')
          }
        });
      }
    },
    beforeDestroy() {
      //make sure to always unsubscribe from events when no longer needed
      console.log("Switching off userDetails listener");
      this.$eventBus.$off("userDetailsChanged");
    }
  }
</script>
```

### Admin data refresh

```javascript
export const state = () => ({
  adminData: {}
});

export const mutations = {
  setAdminData(state, value) {
    state.adminData = value
  }
};
export const actions = {
  async fetchAdminData({commit}) {
    // normally we'd have an axios call here, it would call our API to get some data specific to admin.
    // here we're just setting something random
    commit("setAdminData",{someValue: Math.random()});
  },
  async removeAdminData({commit}) {
    // if a user logs out, or stops being an admin, we want to remove existing adminData
    commit("setAdminData", {});
  }
};

```


### What's the benefit?
You could argue that user.js could dispatch to admin.js directly and make it get the extra data directly - but this would mean that, potentially, you would be fetching admin data even when component that requires it is not active. Also, you'd be coupling the getting general user details with admin functionality.

In this very simple case, you could also monitor user.js store state and react when `userDetails.admin` value changes. I hope this simple example show how this can be used for more complicated scenarios. I will show one such scenario (intercepting back button on mobile) in the next post.

# Full code
As always, a fully working project with this example is located in [Github](https://github.com/lilianaziolek/blog-examples/tree/eventBusAndBackButtonIntercept/dry-examples) - note it is just a branch on the project I used so far.

# Other notes:
* In Nuxt context, you could simply use `this.$root`, as it's the shared root Vue instance. However, I am a huge fan of communicating your intent in code as clearly as possible, so I chose to create a very simple plugin with a meaningful name.

* My sample code always has a lot of console.log statements so that, if you run it, you can quickly and easily see on the console what's happening. If using this code in an actual application, remove all of that to avoid excessive noise, or replace with proper logging framework (if you use it).