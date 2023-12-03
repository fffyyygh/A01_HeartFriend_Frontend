<template>
	<view>
		<view class="chat-container">
		    <scroll-view class="chat-messages" scroll-y>
		      <view v-for="(message, index) in messages" :key="index" class="message">
		        <view :class="message.type === 'sent' ? 'sent' : 'received'">
		          {{ message.content }}
		        </view>
		      </view>
		    </scroll-view>
		
		    <view class="input-container">
		      <input v-model="newMessage" type="text" placeholder="Type your message here" @confirm="sendMessage" />
		      <button @tap="sendMessage">Send</button>
		    </view>
		  </view>
	</view>
</template>

<script>
	export default {
		 data() {
		    return {
		      messages: [],
		      newMessage: ''
		    };
		  },
		  methods: {
		    sendMessage() {
		      if (this.newMessage.trim() !== '') {
		        this.messages.push({
		          content: this.newMessage.trim(),
		          type: 'sent' // or 'received' depending on the sender
		        });
		        this.newMessage = '';
		      }
		    }
		  }
	};
</script>

<style>
.chat-container {
  height: 100vh;
  display: flex;
  flex-direction: column;
}
.chat-messages {
  flex: 1;
  padding: 10px;
  overflow-y: auto;
}
.message {
  margin-bottom: 10px;
}
.sent {
  background-color: #DCF8C6;
  align-self: flex-end;
  border-radius: 5px;
  padding: 8px;
}
.received {
  background-color: #E5E5EA;
  align-self: flex-start;
  border-radius: 5px;
  padding: 8px;
}
.input-container {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 10px;
}
input {
  flex: 1;
  padding: 8px;
  border-radius: 5px;
  border: 1px solid #ccc;
  margin-right: 10px;
}
button {
  padding: 8px 15px;
  border-radius: 5px;
  background-color: #007AFF;
  color: white;
  border: none;
}
</style>