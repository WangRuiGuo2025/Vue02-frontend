<script setup>
    import { ref } from 'vue'
    import axios from 'axios'
    axios.defaults.baseURL = 'http://127.0.0.1:8000'
    axios.defaults.headers.post['Content-Type'] = 'application/json'
    const formData = ref({
        name: '',
        email: '',
        message: ''
    })
  const sendMessage = async () => {
    if (!formData.value.name.trim()) {
        console.log('请填写昵称！')
        return
    }
    if (!formData.value.email.trim()) {
        console.log('请填写留言内容！')
        return
    }
    if (!formData.value.message.trim()) {
        console.log('请填写留言内容！')
        return
    }
    try {
        const response = await axios.post('/message', {
        name: formData.value.name,
        email: formData.value.email,
        message: formData.value.message
        })
        if (response.data.message === 'OK') {
            console.log('留言发送成功！')
            formData.value = {
                name: '',
                email: '',
                message: ''
            }
            location.reload();
        } else {
            console.log('留言发送失败：' + (response.data.detail || '未知错误'))
        }
    } catch (error) {
        console.error('发送失败：', error)
        console.log('网络错误或接口异常，请稍后重试！')
    }
  }
</script>

<template>
  <div class="background">
    <div class="sendMessage">
        <div class="information">
            <div class="form-control">
                <input type="value" v-model="formData.name" required="">
                <label>
                    <span style="transition-delay:0ms">昵</span><span style="transition-delay:50ms">称</span>
                </label>
            </div>
            <!-- <input type="text"  placeholder="请输入昵称"> -->
            <!-- <input type="text" v-model="formData.email" placeholder="请输入邮箱"> -->
            <div class="form-control">
                <input type="value" v-model="formData.email" required="">
                <label>
                    <span style="transition-delay:0ms">邮</span><span style="transition-delay:50ms">箱</span>
                </label>
            </div>
        </div>
        <div class="message">
            <!-- <textarea v-model="formData.message" placeholder="请输入留言内容" rows="3"></textarea> -->
            <div class="form-control-message" style="width: auto!important;">
                <textarea type="value" v-model="formData.message" required=""></textarea>
                <label>
                    <span style="transition-delay:0ms">留</span><span style="transition-delay:50ms">言</span>
                </label>
            </div>
            <button @click="sendMessage()">发送</button>
        </div>
    </div>
  </div>
</template>

<style scoped>
    h1 {
        align-items: center;
        justify-content: center;
    }
    .information {
        display: flex;
        gap: 10px;
    }
    .message {
        display: grid;
        gap: 10px;
    }
    .background{
        background-color: var(--color-background);
        border-radius: 10px;
        padding: 10px;
        box-shadow: 0 4px 10px rgba(0,0,0,0.2);
    }
    .form-control {
        position: relative;
        margin: 20px 0 40px;
        width: 190px;
    }
    .form-control input {
        background-color: transparent;
        border: 0;
        color: var(--color-text);
        border-bottom: 2px #666 solid;
        display: block;
        width: 100%;
        padding: 15px 0;
        font-size: 18px;
        font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
    }
    Button {
        border: none;
        border-radius: 20px;
        padding: 10px 20px;
        font-size: 16px;
        color: #ffffff;
        background-color: #007bffcf;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
        display: flex;
        align-items: center;
        justify-content: center;
    }
    Button:active {
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
    }
    Button:active {
        background-color: #005cbf;
    }
    .form-control input:focus,
    .form-control input:valid {
        outline: 0;
        border-bottom-color: lightblue;
    }

    .form-control label {
        position: absolute;
        top: 15px;
        left: 0;
        pointer-events: none;
    }

    .form-control label span {
        display: inline-block;
        font-size: 18px;
        min-width: 5px;
        transition: 0.3s cubic-bezier(0.68, -0.55, 0.265, 1.55);
    }

    .form-control input:focus+label span,
    .form-control input:valid+label span {
        color: lightblue;
        transform: translateY(-30px);
    }

    .form-control-message {
        position: relative;
        margin: 20px 0 40px;
    }
    .form-control-message textarea {
        background-color: transparent;
        border: 0;
        border-bottom: 2px #666 solid;
        display: block;
        color: var(--color-text);
        width: 100%;
        padding: 15px 0;
        font-size: 18px;
        font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
    }

    .form-control-message textarea:focus,
    .form-control-message textarea:valid {
        outline: 0;
        border-bottom-color: lightblue;
    }

    .form-control-message label {
        position: absolute;
        top: 15px;
        left: 0;
        pointer-events: none;
    }

    .form-control-message label span {
        display: inline-block;
        font-size: 18px;
        min-width: 5px;
        transition: 0.3s cubic-bezier(0.68, -0.55, 0.265, 1.55);
    }

    .form-control-message textarea:focus+label span,
    .form-control-message textarea:valid+label span {
        color: lightblue;
        transform: translateY(-30px);
    }

</style>