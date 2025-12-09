<script setup>
    import { ref, reactive, onMounted } from "vue";
    import axios from "axios";

    axios.defaults.baseURL = 'http://127.0.0.1:8000'

    const messageList = ref([]);
    const replyingTo = ref(null);
    const replyContent = ref('');
    const replyUser = ref({
        name: '',
        email: ''
    });

    const formatDateTime = (timestamp) => {
        if (!timestamp) return '';
            const date = new Date(timestamp);
            // 处理时区偏移（东八区）
            date.setMinutes(date.getMinutes() - date.getTimezoneOffset());
        
        const padZero = (num) => num.toString().padStart(2, '0');
        const year = date.getFullYear();
        const month = padZero(date.getMonth() + 1);
        const day = padZero(date.getDate());
        const hours = padZero(date.getHours());
        const minutes = padZero(date.getMinutes());
        const seconds = padZero(date.getSeconds());

        return `${year}-${month}-${day} ${hours}:${minutes}:${seconds}`;
    };

    onMounted(async () => {
        try {
            const res = await axios.get("/list");
            if (res.data.code === 200) {
                messageList.value = res.data.data.map(item => reactive({
                    id: item.id,
                    name: item.name,
                    email: item.email,
                    message: item.message,
                    good: Number(item.good),
                    bad: Number(item.bad),
                    mid: item.mid || 0,
                    timestamp: formatDateTime(item.timestamp),
                    replies: []
                }));
                const topLevelComments = messageList.value.filter(item => item.mid === 0);
                const secondLevelComments = messageList.value.filter(item => item.mid !== 0);
                
                topLevelComments.forEach(topComment => {
                    topComment.replies = secondLevelComments.filter(
                        reply => reply.mid === topComment.id
                    );
                });
                
                messageList.value = topLevelComments;
            }
        } catch (err) {
            console.error("加载留言失败：", err);
        }
    });

    const addgood = async (item) => {
        const newGoodCount = Number(item.good) + 1;
        item.good = newGoodCount;

        try {
            const res = await axios.post("/update-good", {
                id: item.id,
                good: newGoodCount
            });
            if (res.data.code !== 200) {
                item.good = newGoodCount - 1;
                console.error("点赞同步失败：" + res.data.error);
            }
            console.log(`ID${item.id}点赞数更新成功：`, newGoodCount);
        } catch (err) {
            item.good = newGoodCount - 1;
            console.error(`ID${item.id}点赞更新失败：`, err);
        }
    };

    const addbad = async (item) => {
        const newbadCount = Number(item.bad) + 1;
        item.bad = newbadCount;

        try {
            const res = await axios.post("/update-bad", {
                id: item.id,
                bad: newbadCount
            });
            if (res.data.code !== 200) {
                item.bad = newbadCount - 1;
                console.error("点踩同步失败：" + res.data.error);
            }
            console.log(`ID${item.id}点踩数更新成功：`, newbadCount);
        } catch (err) {
            item.bad = newbadCount - 1;
            console.error(`ID${item.id}点踩更新失败：`, err);
        }
    };

    const toggleReply = (commentId) => {
        if (replyingTo.value === commentId) {
            replyingTo.value = null;
            replyContent.value = '';
        } else {
            replyingTo.value = commentId;
        }
    };

    const submitReply = async (parentId) => {
        if (!replyUser.value.name.trim()) {
            console.warn('请输入您的昵称');
            return;
        }
        if (!replyUser.value.email.trim()) {
            console.warn('请输入您的邮箱');
            return;
        }
        if (!replyContent.value.trim()) {
            console.warn('请输入回复内容');
            return;
        }
        try {
            const response = await axios.post('/reply', {
                name: replyUser.value.name,
                email: replyUser.value.email,
                message: replyContent.value,
                mid: parentId
            });
            if (response.data.message === 'OK') {
                console.log('回复成功！');
                const parentComment = findCommentById(parentId);
                if (parentComment) {
                    const now = new Date();
                    parentComment.replies.push(reactive({
                        id: Date.now(),
                        name: replyUser.value.name,
                        email: replyUser.value.email,
                        message: replyContent.value,
                        good: 0,
                        bad: 0,
                        mid: parentId,
                        timestamp: formatDateTime(now.toISOString())
                    }));
                }
                replyingTo.value = null;
                replyContent.value = '';
            } else {
                console.error('回复失败：' + (response.data.error || '未知错误'));
            }
        } catch (error) {
            console.error('回复失败：', error);
        }
    };

    const findCommentById = (id) => {
        for (const comment of messageList.value) {
            if (comment.id === id) return comment;
            const found = comment.replies.find(reply => reply.id === id);
            if (found) return found;
        }
        return null;
    };
</script>

<template>
    <div class="comments-list">
        <div class="message" v-for="item in messageList" :key="item.id">
            <h3 class="name">{{ item.name }}</h3>
            <p class="email">{{ item.email }} </p>
            <p class="content">{{ item.message }}</p>
            <span class="timestamp">{{ item.timestamp }}</span>
            <div class="button-group">
                <button class="like-btn" @click="addgood(item)">👍{{ item.good }}</button>
                <button class="dislike-btn" @click="addbad(item)">👎{{ item.bad }}</button>
                <button class="reply-btn" @click="toggleReply(item.id)">回复</button>
            </div>
            <div class="reply-input" v-if="replyingTo === item.id">
                <p class="reply">回复：</p>
                <div class="information">
                    <div class="form-control">
                        <input type="value" v-model="replyUser.name" required="">
                        <label>
                            <span style="transition-delay:0ms">昵</span><span style="transition-delay:50ms">称</span>
                        </label>
                    </div>
                    <div class="form-control">
                        <input type="value" v-model="replyUser.email" required="">
                        <label>
                            <span style="transition-delay:0ms">邮</span><span style="transition-delay:50ms">箱</span>
                        </label>
                    </div>
                    <!-- <p>昵称：</p>
                    <input 
                        v-model="replyUser.name" 
                        placeholder="请输入昵称" 
                        class="reply-name"
                    >
                    <p>邮箱：</p>
                    <input 
                        v-model="replyUser.email" 
                        placeholder="请输入邮箱" 
                        class="reply-email"
                    > -->
                </div>
                <div class="message-1">
                    <div class="form-control-message" style="width: auto!important;">
                        <textarea type="value" v-model="replyContent" required=""></textarea>
                        <label>
                            <span style="transition-delay:0ms">留</span><span style="transition-delay:50ms">言</span>
                        </label>
                    </div>
                    <!-- <p>回复：</p>
                    <textarea 
                        v-model="replyContent" 
                        placeholder="请输入回复内容" 
                        rows="3"
                    </textarea> -->
                    <div class="reply-actions">
                        <button class="reply-btn" @click="submitReply(item.id)">发送回复</button>
                        <button class="cancle-btn" @click="toggleReply(item.id)">取消</button>
                    </div>
                </div>
            </div>

            <div class="replies">
                <div class="message reply" v-for="reply in item.replies" :key="reply.id">
                    <h4>{{ reply.name }}</h4>
                    <p class="email">{{ reply.email }}</p>
                    <p class="content">{{ reply.message }}</p>
                    <span class="timestamp">{{ reply.timestamp }}</span>
                    <div class="button-group">
                        <button class="like-btn" @click="addgood(reply)">👍{{ reply.good }}</button>
                        <button class="dislike-btn" @click="addbad(reply)">👎{{ reply.bad }}</button>
                    </div>
                </div>
            </div>
        </div>
        <div class="empty-tip" v-if="messageList.length === 0">
            暂无留言，快来抢沙发！
        </div>
    </div>
</template>

<style scoped>
    .reply{
        font-size: large;
        font-weight: bold;
    }
    .reply-actions{
        justify-self: right;
    }
    .reply-actions button {
        padding: 5px 10px;
        margin: 2px;
        border: none;
        cursor: pointer;
        transition: background-color 0.2s;
    }
    .reply-btn {
        border-radius: 10px;
    }
    .cancle-btn{
        border-radius: 10px;
    }
    .message{
        display:grid ;
    }
    .information {
        display: flex;
        gap: 10px;
    }
    .message-1 {
        display: grid;
        gap: 10px;
    }
    .timestamp {
        font-size: 12px;
        color: #666;
        margin-left: 10px;
        justify-self: right;
    }
    .empty-tip{
        display: flex;
        padding-top: 10px;
        justify-content: center;
        align-items: center;
    }
    .message{
        background-color: var(--color-background);
        border-radius: 10px;
        padding: 10px;
        box-shadow: 0 4px 10px rgba(0,0,0,0.2);
        margin:5px;
    }
    .name{
        font-weight: bold;
    }
    .email{
        font-size: 0.8em;
        color: #666;
        margin-left: 10px;
    }
    .button-group{
        justify-self: right;
    }
    .button-group button {
        padding: 5px 10px;
        margin: 2px;
        border: none;
        cursor: pointer;
        transition: background-color 0.2s;
    }
    .like-btn{
        border-radius: 10px;
        color: #666;
        background-color: #98FB98;
    }
    .dislike-btn{
        border-radius: 10px;
        color: #666;
        background-color: #ffb3a7;
    }
    .like-btn:active{
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
        background-color: #7fe77f;
    }
    .dislike-btn:active{
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
        background-color: #dc8d80;
    }
    .reply-btn{
        border-radius: 10px;
    }
    .form-control {
        position: relative;
        margin: 20px 0 40px;
        width: 190px;
    }
    .form-control input {
        background-color: transparent;
        border: 0;
        border-bottom: 2px #666 solid;
        display: block;
        width: 100%;
        padding: 15px 0;
        font-size: 18px;
        color: var(--color-text);
        font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
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
        color: var(--color-text);
        border-bottom: 2px #666 solid;
        display: block;
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

    Button {
        border: none;
        border-radius: 20px;
        padding: 10px 20px;
        font-size: 16px;
        color: #ffffff;
        background-color: #007bffcf;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
        align-items: center;
        justify-content: center;
    }
    .reply-btn:active {
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
        background-color: #005cbf;
    }
    .cancle-btn:active {
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
        background-color: #005cbf;
    }
    .replies .reply {
    border-left: 2px solid #666;
    box-sizing: border-box;
    }
</style>