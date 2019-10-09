<template>
<div id="app" class="w-full h-screen">
  <weather></weather>
</div>
</template>

<script>

import Weather from './components/Weather.vue'

export default {
    name: 'app',
    components: {
        Weather
    }
}
</script>
