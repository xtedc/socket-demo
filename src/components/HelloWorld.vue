<template>
  <div>
    <ul>
      <li v-for="(message, index) in messages" :key="index">{{ message }}</li>
    </ul>
    <div class="user-info">
      <div class="item" @click="onTransfer(user)" v-for="(user, index) in userList" :key="index">
        <img alt="" class="user-avatar" :src="user.url">
        <div class="user-name">{{ user.name }}</div>
        <div class="user-amount">￥{{ user.amount }}</div>
      </div>
    </div>
    <div class="transfer-record">
      <div class="item" v-for="(trigger, index) in triggers" :key="index">
        <div>{{ `${trigger.userName}向${trigger.toUserName}转账${trigger.amount}元` }}</div>
      </div>
    </div>
  </div>
</template>

<script>
import io from 'socket.io-client';

export default {
  data() {
    return {
      message: '',
      messages: [],
      avatarList: [
        'https://staging-statics.maiscrm.com/avatar/wechat/aHR0cHM6Ly90aGlyZHd4LnFsb2dvLmNuL21tb3Blbi92aV8zMi9RMGo0VHdHVGZUSTNaaWNYRkJIOHl2SUlqTkphNDEybEZpY0t5Yk43aWJJbnFMcFl4ZXRvdWRCOENzcjltSFEwWkVNeVBFaWF1bmg3QjhNaWI2MWdCVEVRaWNjQS8xMzI=?x-oss-process=image/resize,h_120,w_120',
        'https://staging-statics.maiscrm.com/avatar/wechat/aHR0cHM6Ly93ZXdvcmsucXBpYy5jbi93d3BpYy81MzU2MjFfZko5SS03dE1SdG1lNVUyXzE2NTkzMjQ5MzYvMA==?x-oss-process=image/resize,h_120,w_120',
        'https://staging-statics.maiscrm.com/avatar/wechat/aHR0cDovL3d4LnFsb2dvLmNuL21taGVhZC9PaWJSTmR0bEpka0cxaWFla3JyNDBQcGlhNnlwWFBvalA5cVVOb25ybkExWm9xYmpadGhvaWFjR1pBLzA=?x-oss-process=image/resize,h_120,w_120',
        'https://staging-statics.maiscrm.com/avatar/wechat/aHR0cDovL3d4LnFsb2dvLmNuL21taGVhZC9RM2F1SGd6d3pNNlpMbk4yVlI5M2NTZFYxSmJYSXN6SndxeWpaOXZJRmsxTWlhTzFPeUpiWTZBLzA=?x-oss-process=image/resize,h_120,w_120',
      ],
      names: ['小张', '小李', '小王', '小马', '小邓'],
      userList: [],
      triggers: []
    };
  },
  computed: {
    id() {
      return this.$route.query?.id || null
    },
    userInfo() {
      console.log(this.avatarList[Math.round(Math.random() * 4)]);
      return {
        name: this.$route.query?.name || this.names[Math.round(Math.random() * 4)],
        id: this.$route.query?.id || null,
        amount: 0,
        url: this.avatarList[Math.round(Math.random() * 3)]
      }
    }
  },
  mounted() {

    const socket = io('http://localhost:3000');
    socket.emit('login', this.userInfo)
    socket.on('login', data => {
      this.userList = data.userList || []
    })
    socket.on('previousMessages', (data) => {
      this.triggers = data
    })
    socket.on('trigger', (data) => {
      this.triggers.push(data)
    })
    socket.on('userList', userlist => {
      console.log(userlist)
      this.userList = userlist
    })
  },
  methods: {
    onTransfer(user) {
      if (user.id !== this.userInfo.id) {
        let sign = prompt(`向${user.name}转账`);

        if (sign) {
          const socket = io('http://localhost:3000');
          const trigger = {
            userName: this.userInfo.name,
            userId: this.userInfo.id,
            amount: sign,
            toUserName: user.name,
            toUserId: user.id,
          }
          socket.emit('trigger', trigger);
        }
      }
    }
  },
};
</script>
<style scoped lang="less">
.user-info {
  display: flex;
  margin-bottom: 20px;

  .item {
    width: 50px;
    height: 80px;
    margin-right: 10px;

    .user-avatar {
      width: 50px;
      height: 50px;
    }
  }
}
</style>