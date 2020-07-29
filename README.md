# Logux Vuex

<img align="right" width="95" height="148" title="Logux logotype"
     src="https://logux.io/branding/logotype.svg">

Logux is a new way to connect client and server. Instead of sending
HTTP requests (e.g., AJAX and GraphQL) it synchronizes log of operations
between client, server, and other clients.

* **[Guide, recipes, and API](https://logux.io/)**
* **[Chat](https://gitter.im/logux/logux)** for any questions
* **[Issues](https://github.com/logux/logux/issues)**
  and **[roadmap](https://github.com/logux/logux/projects/1)**
* **[Projects](https://logux.io/guide/architecture/parts/)**
  inside Logux ecosystem

This repository contains [Vuex] compatible API on top of the [Logux Client].

The current version is for Vue 3 and Vuex 4. It doesn’t work with Vue 2.
But if you still need for Vue 2 support, use [0.8 version from a separate branch](https://github.com/logux/vuex/tree/0.8).

[Vuex]: https://vuex.vuejs.org
[Logux Client]: https://github.com/logux/client
[logux.io]: https://logux.io/

## Install

```sh
npm install @logux/vuex vuex
```
or
```sh
yarn add @logux/vuex vuex
```

## Usage

See [documentation] for Logux API.

[documentation]: https://github.com/logux/docs

```js
import { CrossTabClient, createStoreCreator } from '@logux/vuex'

const client = new CrossTabClient({
  server: process.env.NODE_ENV === 'development'
    ? 'ws://localhost:31337'
    : 'wss://logux.example.com',
  subprotocol: '1.0.0',
  userId: 'anonymous',
  token: ''
})

const createStore = createStoreCreator(client)

const store = createStore({
  state: {},
  mutations: {},
  actions: {},
  modules: {}
})

store.client.start()

export default store
```

## Subscription

### `useSubscription`

```html
<template>
  <h1 v-if="isSubscribing">Loading</h1>
  <h1 v-else>{{ user.name }}</h1>
</template>

<script>
import { toRefs, computed } from 'vue'
import { useStore, useSubscription } from '@logux/vuex'

export default {
  props: {
    userId: String
  },
  setup (props) {
    let store = useStore()
    let { userId } = toRefs(props)

    let channels = computed(() => [`user/${userId}`])
    let isSubscribing = useSubscription(channels)

    let user = computed(() => store.state.users[userId])

    return {
      user,
      isSubscribing
    }
  }
})
</script>
```

### `loguxComponent`

```html
<template>
  <logux-component :channels="channels" v-slot="{ isSubscribing }">
    <h1 v-if="isSubscribing">Loading</h1>
    <h1 v-else>{{ user.name }}</h1>
  </logux-component>
</template>

<script>
import { toRefs, computed } from 'vue'
import { useStore, loguxComponent } from '@logux/vuex'

export default {
  components: {
    loguxComponent
  },
  props: {
    userId: String
  },
  setup (props) {
    let store = useStore()
    let { userId } = toRefs(props)

    let user = computed(() => store.state.users[userId])
    let channels = computed(() => [`users/${ userId }`])

    return {
      user,
      channels
    }
  }
}
</script>
```

## Using with Typescript

Place the following code in your project to allow this.$store to be typed correctly:

```ts
// shims-vuex.d.ts

import { LoguxVuexStore } from '@logux/vuex'

declare module '@vue/runtime-core' {
  // Declare your own store states.
  interface State {
    count: number
  }

  interface ComponentCustomProperties {
    $store: LoguxVuexStore<State>
  }
}
```
