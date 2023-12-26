<template>
	<view>
		<view class="chat-container">
			<scroll-view class="chat-messages" scroll-y>
				<view v-for="(message, index) in messages" :key="index" class="message">
					<view :class="message.type === 'sent' ? 'sent' : 'received'">
						<block v-if="message.type === 'received'">
							<!-- 使用 v-for 遍历文本行 -->
							<view v-for="(line, lineIndex) in formatMessage(String(message.content))" :key="lineIndex">
								{{ line }}
								<br />
							</view>
						</block>
						<block v-else>
							{{ message.content }}
						</block>
					</view>
				</view>

			</scroll-view>

			<view class="input-container">
				<input v-model="newMessage" type="text" placeholder="在这里输入您的消息" @confirm="sendMessage" />
				<button  :disabled="disable" @tap="sendMessage">发送</button>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				messages: [],
				newMessage: '',
				disable: false, 
			};
		},
		methods: {
			formatMessage(message) {
				const contentString = typeof message === 'string' ? message : String(message);
				console.log('message:::', contentString);
				const lines = contentString.split('\\n');
				console.log('Split Result:::', lines);
				return lines;
			},

			async sendMessage() {
				this.disable = true;
				if (this.newMessage.trim() !== '') {
					// Add the user's message to the chat history
					this.messages.push({
						content: this.newMessage.trim(),
						type: 'sent'
					});

					// Prepare the request payload
					const payload = JSON.stringify(this.messages);
					this.newMessage = '';
					
					try {
						// Make a request to the backend API
						const response = await uni.request({
							url: 'https://vx.mikumikumi.xyz/api/v1/chat/',
							method: 'POST',
							header: {
								'Content-Type': 'application/json',
								'Authorization': `Bearer ${uni.getStorageSync('token')}`
							},
							data: payload
						});
						console.log("response:::", response);
						// Update the chat history with the assistant's response
						const assistantResponse = {
							content: response[1].data.content.replace(/^"(.+)"$/,
								'$1'), // Remove surrounding quotes
							type: 'received'
						};
						this.messages.push(assistantResponse);
					} catch (error) {
						console.error('Error sending message:', error);
						// Handle the error as needed
					}

					// Clear the input field
					this.disable = false;
					
				}
			}
		}
	};
</script>

<style>
	.chat-container {
		height: 100vh;
		display: flex;
		width: 700rpx;
		flex-direction: column;
	}

	.chat-messages {
		flex: 1;
		padding: 10px;
		overflow-y: auto;
		overflow-x: hidden;
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
		white-space: pre-line;
		background-color: #E5E5EA;
		align-self: flex-start;
		border-radius: 5px;
		padding: 8px;
	}

	.input-container {
		width: 700rpx;
		box-sizing: border-box;
		display: flex;
		align-items: center;
		justify-content: space-between;
		padding: 10px;
		margin-bottom: 40rpx;
	}

	input {
		flex: 1;
		padding: 8px;
		border-radius: 5px;
		border: 1px solid #ccc;
		margin-right: 10px;
	}

	button {
		width: auto;
		margin-right: -20px;
		border-radius: 5px;
		background-color: #527853;
		color: white;
		border: none;
	}
</style>