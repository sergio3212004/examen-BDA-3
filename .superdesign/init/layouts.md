# Layouts

There is no reusable application shell yet. `App.vue` is the root and only renders the starter demo.

## `src/App.vue`

```vue
<script setup lang="ts">
import HelloWorld from './components/HelloWorld.vue'
</script>

<template>
  <HelloWorld />
</template>
```

## `src/main.ts`

```ts
import { createApp } from 'vue'
import './style.css'
import App from './App.vue'

createApp(App).mount('#app')
```
