<template>
	<view>
		<view v-for="(focus, index) in my_focus" :key="index" class="post-item">
			<!-- 用户信息 -->
			<view class="user-info">
				<!-- 头像 -->
				<image class="user-avatar" :src="my_focus[index].avatar_url" mode="aspectFill" @click="goToUserHome_fan(index)"></image>
				<!-- 用户名和关注按钮 -->
				<view class="user-details">
					<view class="user-details-top">
						<text class="user-name">{{ my_focus[index].username }}</text>
						<button :class="{'follow-button': if_focus[index], 'unfollowed-button': !if_focus[index]}"
							@click="follow_click(index)">{{ if_focus[index] ? '已关注' : '关 注'}}</button>
					</view>
					<text class="user-intro">{{ my_focus[index].self_intro }}</text>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				my_focus: [],
				if_focus: [],

			}
		},
		onLoad() {
			this.get_all_fans();
		},
		methods: {
			goToUserHome_fan(index) {
				const fanUuid = this.my_focus[index].uuid;
				uni.navigateTo({
					url: `/pages/user-home/user-home?uuid=${fanUuid}`,
				});
			},
			follow_click(index) {
				console.log("a");
				uni.request({
					url: "http://82.157.244.44:8000/api/v1/user/follow-unfollow/",
					method: "POST",
					header: {
						'Authorization': `Bearer ${uni.getStorageSync('token')}`,
					},
					data: {
						"uuid": this.my_focus[index].uuid,
					},
					success: (res) => {
						console.log("关注状态改变", res.data);
					}

				});

				this.$set(this.if_focus, index, !this.if_focus[index]);
				console.log(this.if_focus[index]);
			},


			get_all_fans() {
				uni.request({
					url: 'http://82.157.244.44:8000/api/v1/user/followers/', // 后端接口地址
					method: 'GET',
					header: {
						'Authorization': `Bearer ${uni.getStorageSync('token')}`,
					},
					success: (res) => {
						console.log('数据接收成功:', res.data);
						this.my_focus = res.data.followers
						for (const user of this.my_focus) {
							user.avatar_url = "http://82.157.244.44:8000" + user.avatar_url;
							this.if_focus.push(true);
						}
					},
					fail: (err) => {
						console.error('数据发送失败:', err);
					}
				});
			}

		}
	}
</script>

<style>
	.post-item {
		background-color: #fff;
		margin-bottom: 20rpx;
		padding: 20rpx;
		border-radius: 10rpx;
		box-shadow: 0 2rpx 4rpx rgba(0, 0, 0, 0.1);
	}


	.user-info {
		display: flex;
		align-items: center;
		margin-bottom: 10rpx;
	}

	.user-avatar {
		width: 40px;
		height: 40px;
		border-radius: 50%;
		margin-right: 10px;
	}

	.user-name {
		font-weight: bold;
	}


	.user-info {
		display: flex;
		align-items: center;
		margin-bottom: 20px;
		/* 增加一些间距 */
	}

	.user-avatar {
		width: 50px;
		/* 增加头像尺寸 */
		height: 50px;
		border-radius: 50%;
		margin-right: 15px;
		/* 增加头像和用户名之间的间距 */
	}

	.user-name {
		font-weight: bold;
		font-size: 16px;
		/* 调整字体大小 */
		color: #333;
		/* 修改文字颜色 */
	}

	.user-intro {
		font-size: 14px;
		/* 较小的字体大小 */
		color: #666;
		/* 修改文字颜色 */
		margin-top: 5px;
		/* 控制自我介绍与用户名的间距 */
	}

	.user-info {
		display: flex;
		align-items: flex-start;
		/* 顶部对齐 */
		margin-bottom: 20px;
		/* 增加间距 */
	}

	.user-avatar {
		width: 50px;
		height: 50px;
		border-radius: 50%;
		margin-right: 15px;
	}

	.user-name {
		font-weight: bold;
		font-size: 16px;
		color: #333;
	}

	.user-details {
		display: flex;
		flex-direction: column;
	}

	.user-intro {
		font-size: 14px;
		color: #666;
	}

	.user-info {
		display: flex;
		align-items: flex-start;
		margin-bottom: 20px;
	}

	.user-avatar {
		width: 50px;
		height: 50px;
		border-radius: 50%;
		margin-right: 15px;
	}

	.user-name {
		font-weight: bold;
		font-size: 16px;
		color: #333;
	}

	.follow-button {
		display: flex;
		align-items: center;
		justify-content: center;
		align-self: flex-start;
		padding: 8px 16px;
		border: none;
		background-color: darkgray;
		color: white;
		font-size: 30rpx;
		height: 80rpx;
		width: 200rpx;
		margin-right: 20rpx;
	}

	.unfollowed-button {
		display: flex;
		align-items: center;
		justify-content: center;
		align-self: flex-start;
		padding: 8px 16px;
		border: none;
		background-color: steelblue;
		color: #fff;
		font-size: 30rpx;
		height: 80rpx;
		width: 200rpx;
		margin-right: 20rpx;
	}


	.user-details {
		display: flex;
		flex-direction: column;
		flex: 1;
		/* 使用户介绍占据剩余空间 */
	}

	.user-details-top {
		display: flex;
		align-items: center;
		justify-content: space-between;
		/* 将用户名和按钮放在同一行 */
		margin-bottom: 6px;
		/* 调整按钮与介绍之间的间距 */
	}
</style>