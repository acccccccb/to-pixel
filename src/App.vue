<script setup>
    // This starter template is using Vue 3 <script setup> SFCs
    // Check out https://vuejs.org/api/sfc-script-setup.html#script-setup
    import Converter from './components/Converter.vue';
    import { defineComponent, ref } from 'vue';
    import { darkTheme, lightTheme } from 'naive-ui';
</script>

<template>
    <n-config-provider :theme="darkTheme">
        <n-layout content-style="min-height: 100vh">
            <n-layout-header class="header">To Pixel</n-layout-header>
            <n-layout-content class="content" content-style="width: 100%;">
                <n-grid
                    x-gap="10"
                    :y-gap="10"
                    :cols="24"
                    style="text-align: center"
                >
                    <n-gi :span="20" :offset="2">
                        <n-message-provider>
                            <Converter />
                        </n-message-provider>
                    </n-gi>
                </n-grid>
            </n-layout-content>
            <n-layout-footer class="footer">
                <a target="_blank" href="https://gitee.com/GLUESTICK/to-pixel">
                    Gitee
                </a>
                |
                <a target="_blank" href="https://github.com/acccccccb/to-pixel">
                    Github
                </a>
            </n-layout-footer>
        </n-layout>
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
        line-height: 80px;
        height: 80px;
        text-align: center;
        font-size: 2em;
    }
    .content {
        min-height: calc(100vh - 130px);
        display: flex;
        justify-content: center;
        align-items: center;
    }
    .footer {
        line-height: 50px;
        height: 50px;
        text-align: center;
    }
    .footer a {
        color: #fff;
    }
</style>
