<template>
	<view class="container">
		<u-button @click="chat">聊一聊</u-button>
		<u-button @click="write_diary">心情记录</u-button>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				href: 'https://uniapp.dcloud.io/component/README?id=uniui'
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