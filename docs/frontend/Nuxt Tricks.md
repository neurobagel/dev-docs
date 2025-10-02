---
title: Nuxt Tricks
tags: [Frontend]

---

## Injecting HTML head attributes

`<head>` is the default place for some things like meta tags, 
but also for linking external js scripts, for example from 
a CDN. In Nuxt, we can inject e.g. a JS script from a CND into
the `<head>` tag of all attributes using the `nuxt.config.js` file:

```js
head: {
    title: 'My Tool name',
    script: [
      {
        src: 'https://my.cdn.com/happyscript',
      },
    ],
```

[See the official docs here](https://v2.nuxt.com/docs/configuration-glossary/configuration-head/)

## Using environment variables on the client side

By default, reading from a `.env` file (with ` in nuxt means that the variables declared in the `.env` file get exposed to the server **AND** the client. This isn't a good thing, because you might have some secrets in there that you don't want to share with the client.

What's worse: when you provide an environment variable by declaring it in the environment when you call `nuxt`, it does not get exposed to the client! In other words, `.env` and `export MYVAR=hello` behave differently.

For this reason, Nuxt has deprecated reading environment variables using the dotenv module: https://v2.nuxt.com/docs/configuration-glossary/configuration-env/. Instead we now explicitly set which variables (whether from .env or shell variable) should be only visible to the server, and which should visibile to the server and the client (see [here](https://v2.nuxt.com/docs/configuration-glossary/configuration-runtime-config) and [here](https://stackoverflow.com/questions/67703133/how-to-use-env-variables-in-nuxt-2-or-3)).


```js
export default {
  publicRuntimeConfig: {
    myPublicVariable: process.env.PUBLIC_VARIABLE, // will be visible on the server AND the client
  },
  privateRuntimeConfig: {
    myPrivateToken: process.env.PRIVATE_TOKEN // will be visiblye ONLY on the server
  }
}
```

## Routing
Routing in Nuxt is done by wrapping `vue-router` (see [docs](https://v2.nuxt.com/docs/get-started/routing/)). In Annotation tool in particular routing is done using the `to` prop of `b-nav-item` in `tool-navbar` component and `b-button` in `next-page` component.