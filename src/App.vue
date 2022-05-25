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
                <n-layout-header class="header">To Pixel</n-layout-header>
                <n-layout-content class="content" content-style="padding: 24px">
                    <Converter />
                </n-layout-content>
                <n-layout-footer class="footer">powered by vue</n-layout-footer>
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
    .header {
        line-height: 50px;
        height: 50px;
        text-align: center;
        font-size: 2em;
    }
    .content {
        min-height: calc(100vh - 100px);
    }
    .footer {
        line-height: 50px;
        height: 50px;
        text-align: center;
    }
</style>
