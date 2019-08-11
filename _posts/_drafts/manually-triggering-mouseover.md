---
layout: post
title: "Manually Dispatching Events on Elements"
author: Andrew Kulpa
---

The other week I was developing a pdf previewer in Angular that displayed only upon the user's mouse entering an element. I figured this would be easy to tweak in browser using the dev tools. After all, modern browsers support inspecting elements and activating pseudo-classes like `:hover`, `:active`, `:focus`, and `:focus-within`. While that definitely is the case for those pseudo-classes that are supported, there are so many more that are not.

To accommodate for the lack of built-in pseudo-class triggering, there are a few options

```js

var event = new Event('mouseenter');
document.querySelector('.navbar-toggler-icon').dispatchEvent(event);

```