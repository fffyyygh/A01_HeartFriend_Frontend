<template>
	<view>
		<button @click="navigateBack">返回</button>
		
		  <view class="container">  
		    <view class="chat-log">  
		      <text v-for="message in messages" :key="message.id">{{message.text}}</text>  
		    </view>  
		    <view class="chat-input">  
		      <input type="text" v-model="inputText" @input="sendMessage" placeholder="请输入消息内容">  
		      <button @click="sendMessage">发送</button>  
		    </view>  
		  </view>  
		
	</view>
</template>

<script>
	export default {
		methods: {
			navigateBack() {
				// 调用uni-app的导航返回方法
				uni.navigateBack({
					delta: 1 // 返回的页面数，这里是返回上一页，你可以根据实际情况调整
				});
			}
		}
	};
</script>

<style>

</style>