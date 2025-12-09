<script setup>
    import Comments from './components/Comments.vue';
    import SendMessage from './components/SendMessage.vue';
    
</script>

<template>
    <div class="background">
        <h1>留言板</h1>
        <h2>发送留言</h2>
        <SendMessage></SendMessage>
        <div class="title-section">
        <h2>留言</h2>
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
