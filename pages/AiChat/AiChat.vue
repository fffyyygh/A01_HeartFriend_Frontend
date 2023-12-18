<template>
	<view>
		<view class="chat-container">
			<scroll-view class="chat-messages" scroll-y>
				<view v-for="(message, index) in messages" :key="index" class="message">
					<!-- <view :class="message.type === 'sent' ? 'sent' : 'received'">
						{{ message.content }}
					</view> -->
					<view :class="message.type === 'sent' ? 'sent' : 'received'">
						<block v-if="message.type === 'received'">
							<!-- 使用v-if和v-else来处理换行 -->
							<view v-for="(line, lineIndex) in formatMessage(message)" :key="lineIndex">{{line}}</view>
						</block>
						<block v-else>
							{{message.content}}
						</block>
					</view>
				</view>
			</scroll-view>

			<view class="input-container">
				<input v-model="newMessage" type="text" placeholder="在这里输入您的消息" @confirm="sendMessage" />
				<button @tap="sendMessage">发送</button>
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
			formatMessage(message) {
				// 将 "\n\n" 替换为数组，以实现换行效果
				return message.content.split('\n\n');
			},
			async sendMessage() {
				if (this.newMessage.trim() !== '') {
					// Add the user's message to the chat history
					this.messages.push({
						content: this.newMessage.trim(),
						type: 'sent'
					});

					// Prepare the request payload
					const payload = JSON.stringify(this.messages);

					try {
						// Make a request to the backend API
						const response = await uni.request({
							url: 'http://82.157.244.44:8000/api/v1/chat/',
							method: 'POST',
							header: {
								'Content-Type': 'application/json',
								'Authorization': `Bearer ${uni.getStorageSync('token')}`
							},
							data: payload
						});
						console.log("response:::", response);
						// Update the chat history with the assistant's response
						this.messages.push({
							content: response[1].data.content,
							type: 'received'
						});
					} catch (error) {
						console.error('Error sending message:', error);
						// Handle the error as needed
					}

					// Clear the input field
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