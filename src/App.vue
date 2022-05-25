<script setup>
    // This starter template is using Vue 3 <script setup> SFCs
    // Check out https://vuejs.org/api/sfc-script-setup.html#script-setup
    import Converter from './components/Converter.vue';
    import { defineComponent, ref } from 'vue';
    import { darkTheme } from 'naive-ui';
</script>

<template>
    <n-config-provider :theme="darkTheme">
        <n-message-provider>
            <n-layout content-style="min-height: 100vh">
                <n-layout-header>to-pixel</n-layout-header>
                <n-layout-content content-style="padding: 24px">
                    <Converter />
                </n-layout-content>
                <n-layout-footer>powered by vue</n-layout-footer>
            </n-layout>
        </n-message-provider>
    </n-config-provider>
</template>
<script></script>

<style>
    #app {
        font-family: Avenir, Helvetica, Arial, sans-serif;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        color: #2c3e50;
    }
    html,
    body,
    #app {
        padding: 0;
        margin: 0;
        height: 100%;
    }
</style>
