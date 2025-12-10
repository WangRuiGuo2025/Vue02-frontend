<script setup>
    import Comments from './components/Comments.vue';
    import SendMessage from './components/SendMessage.vue';
    import { onMounted,ref } from 'vue';
    import axios from "axios";
    axios.defaults.baseURL = 'http://127.0.0.1:8000'
    const maxId = ref(0);
    onMounted(async () => {
        try {
            const res = await axios.get("/number");
            if (res.data.code === 200) {
                maxId.value = res.data.data;
            }
        } catch (err) {
            console.error("获取最大ID失败：", err);
        }
    });
</script>

<template>
    <div class="background">
        <h1>留言板</h1>
        <h2>发送留言</h2>
        <SendMessage></SendMessage>
        <div class="title-section">
        <h2>留言({{ maxId }})</h2>
        </div>
        <Comments></Comments>
    </div>
</template>

<style scoped>
    h1{
        justify-self: center;
        padding : 20px;
    }
    h2{
        justify-self: center;
        padding : 10px;
    }
</style>
