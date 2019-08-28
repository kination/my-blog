---
layout: post
title: Basic data flow on Vue.js components
date: 2019-06-21
description: 
tags:
- vue
- vuejs
- frontend
- component
- typescript
permalink: vuejs-component-basic-data-flow
---

It has been 'bout a month using `Vue.js` on production level. I've used `AnguarJS 1.x` in first web development and work with `React` last 1 year. And first impression of `Vue.js` is, it is very easy to start on. Not because it has similarities with frameworks I've used, it has great document for beginners, and it is not fragmentized like `React`. For example, state management is almost unified with `Vuex`. I'm not sure this is good or bad, but it make to develop clear for the starters like me.


## Making component
Component are re-usable Vue instance, something like 'function' or 'module' in programming. It can be customized button, input, table, or mixed stuff such as 'button with input', 'table with title', etc.

Actually you can put all of these in single page without grouping several parts as a component. But good thing of separating these, are making this re-usable. Like making useful utilities inside code project, making re-usable component helps reducing code and easy to maintain.

So, here is the component planning to work on.
![custom_input](/assets/post_img/vuejs-component-basic-data-flow/input-label-iconic.png)

This is not the exact result, but it is enough to refer. I'll try to wrap up features inside this as single component.
In this image, you can see feature as:
- input
- label on top
- icon on left

Let's go on.

And one more, I'll not talk about css styles. Let's focus on component here.


## Basic form
Let's name this component as `custom-icon-input`.

What properties will be needed for this component? It will require item as icon image, label text, input placeholder for UI. And more, it needs to receive icon touch events, and model to connect. So it will be like:

```html
<template>
    <div>
        <!-- label -->
        <label>{{ label }}</label>
        <!-- icon -->
        <div v-on:click="iconClick" >
            <img src="iconSrc" />
        </div>
        <!-- input -->
        <input
            :name="inputName"
            :placeholder="inputPlaceholder"
            v-model="inputModel" />
    </div>
</template>
```

and class part:

```ts
import { Component, Prop } from 'vue-property-decorator'

@Component
export default class CustomIconInput extends Vue {
  @Prop() label: string
  @Prop() iconSrc: string
  @Prop() iconClick: Function

  @Prop() inputPlaceholder: string
  @Prop() inputModel: string

  // ...   

}
```

This component will be used outside as:

```html
<custom-icon-input
    icon-type="(app-icon-type)"
    :icon-click="(app-icon-click-event)"
    input-label="(app-input-label)"
    input-name="(app-input-name)"
    input-placeholder="(app-input-placeholder)"
    :input-model="(app-input-model)"
/>
```

There are still several things to do. But first let's check how it will work.


## '[Vue warn]: Avoid mutating a prop directly since the value will be overwritten...'
If we implement like above, this will be the first hurdle.

![vue warning](/assets/post_img/vuejs-component-basic-data-flow/prop-mutate-warn.png)

There are lot of documents treating about this, so you can find more from Google or StackOverflow.

Anyway, main reason of this is, because you are trying to change parent object `(app-input-model)` in child component(custom-icon-input). In this component `<input>` is feature of child component. It will change `inputModel` when inputting text, and this `inputModel` is equal to `(app-input-model)`.

Seems it was allowed before, but now considered as anti-pattern in new rendering mechanism, whenever the parent component re-renders, the child component’s local changes will be overwritten. It seems to avoid two-way bindings(like other modern web frameworks like `React`, `Angular`).


### How to avoid it?

To send changed property to parent component, we need to send 'event' using `$emit`.

This is fixed component template:
```html
<template>
    <div>
        <!-- label -->
        <label>{{ label }}</label>
        <!-- icon -->
        <div v-on:click="iconClick" >
            <img src="iconSrc" />
        </div>
        <!-- input -->
        <input
            :name="inputName"
            :placeholder="inputPlaceholder"
            v-bind:value="inputModel"
            v-oninput="$emit('update:input-model', $event.target.value)" />
    </div>
</template>
```

Instead of using `v-model`, it uses `v-bind` and `v-on`. Because:
```html
<input v-model="inputModel" />
```

works same as
```html
<input
  v-bind:value="inputModel"
  v-on:input="inputModel=$event.target.value" />
```

But in this case, it directly changes `inputModel` in child, and it causes this warning. So instead of changing property directly, we can use `$emit('update:xxx', value)` to send 'update event' to parent, to make variable change on parent side.

```html
<input
  v-bind:value="inputModel"
  v-on:input="$emit('update:input-model', $event.target.value)" />
```

This means to send 'update input-model' event to parent, with value from '$event.target.value'.

Now you could check warning has been disappeared.


### .sync
This is the part to use component from parent.

```html
<custom-icon-input
    icon-type="(app-icon-type)"
    :icon-click="(app-icon-click-event)"
    input-label="(app-input-label)"
    input-name="(app-input-name)"
    input-placeholder="(app-input-placeholder)"
    :input-model.sync="(app-input-model)"
/>
```

You can see `.sync` in back of model. This is new on `Vue.js 2.3.0+`.

When child component sends event via `$emit`, parent will receive and update its local data property:
```js
$emit('update:input-model', $event.target.value)
```

to update this event logically you need to add update event on component.
```html
<custom-icon-input
    v-bind:input-model="(app-input-model)"
    v-on:update:input-model="(app-input-model) = $event"
/>
```

But by using `.sync`, it can be done more simple:
```html
<custom-icon-input
    v-bind:input-model.sync="(app-input-model)"
/>
```


## Accept click events
We also need click events for icon. But as in same reason, we need to send 'clicked' event to parent, to make this handled by parent side. One more time, child cannot effect parent directly.

```html
<template>
    <div>
        <!-- label -->
        <label>{{ label }}</label>
        <!-- icon -->
        <div v-on:click="$emit('icon-click')" >
            <img src="iconSrc" />
        </div>
        <!-- input -->
        <!-- ... -->
</template>
```

By changing as above, clicking image div will trigger function connected with `icon-click` in parent side. In this template it will call `(app-icon-click-event)`.

```html
<custom-icon-input
    icon-type="(app-icon-type)"
    v-on:icon-click="(app-icon-click-event)"
    input-label="(app-input-label)"
    input-name="(app-input-name)"
    input-placeholder="(app-input-placeholder)"
    :input-model.sync="(app-input-model)"
/>
```


## Reference
* <https://vuejs.org>
* <https://michaelnthiessen.com/avoid-mutating-prop-directly/>
