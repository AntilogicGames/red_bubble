<template>
    <div id="app">
        <RedBubble/>
    </div>
</template>

<script>
import RedBubble from './components/RedBubble.vue'

export default {
    name: 'App',
    components: {
        RedBubble
    }
}
</script>

<style>
    body {
        background-image: url('assets/back.gif');
        background-repeat: no-repeat;
        background-size: cover;
        overflow: hidden;
    }
</style>