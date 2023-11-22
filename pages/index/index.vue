<template>
	<view class="container">
		<u-button @click="chat" hair-line="false" :ripple="true" ripple-bg-color="#e86158"
			:custom-style="btnStyle">聊一聊</u-button>
		<u-button @click="write_diary" hair-line="false" :ripple="true" ripple-bg-color="#e86158"
			:custom-style="btnStyle">心情记录</u-button>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				btnStyle: {
					color: "#f9f8e5",
					backgroundColor: '#ff8a89',
					marginTop: '20rpx',

				},
			}
		},
		methods: {
			chat() {
				uni.navigateTo({
					url: "/pages/AiChat/AiChat"
				})
			},
			write_diary() {
				uni.navigateTo({
					url: "/pages/diary/diary"
				})
			}

		}
	}
</script>

<style lang="scss">
	.container {
		padding: 20px;
		font-size: 14px;
		line-height: 24px;
	}
</style>