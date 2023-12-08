<template>
	<view class="user-profile">
		<view class="user-info">
			<image class="user-avatar" :src="getFullAvatarUrl(userInfo.avatar_url)" mode="aspectFill"></image>
			<text class="user-name">{{ userInfo.username }}</text>
			<!-- 根据性别显示不同图标 -->
			<image v-if="userInfo.gender === 'male'" src="/static/my/male.png" class="gender-icon"></image>
			<image v-else-if="userInfo.gender === 'female'" src="/static/my/female.png" class="gender-icon"></image>
			<image v-else-if="userInfo.gender === 'other'" src="/static/my/other.png" class="gender-icon"></image>
			<image v-else-if="userInfo.gender === 'secret'" src="/static/my/secret.png" class="gender-icon"></image>
			<!-- 添加关注按钮 -->
			<view class="follow-button" @click="followUser">关注</view>
		</view>
		<view class="info-section">
			<view>
				<text class="info-label">年龄：</text>
				<text class="user-age">{{ userInfo.age }}岁</text>
			</view>
			<view>
				<text class="info-label">个人介绍：</text>
				<text class="user-intro">{{ userInfo.self_intro }}</text>
			</view>
		</view>

		<!-- TODO：加一个显示该用户发的所有帖子的列表 -->

	</view>
</template>

<script>
	export default {
		data() {
			return {
				userInfo: {},
				uuid: '',
			};
		},
		onLoad(query) {
			this.uuid = query.uuid;
			console.log('this.uuid::::', this.uuid);
			this.getUserInfo(this.uuid);
		},
		methods: {
			getUserInfo(uuid) {
				uni.request({
					url: `http://82.157.244.44:8000/api/v1/user/query-info/?uuid=${uuid}`,
					method: 'GET',
					header: {
						'Authorization': `Bearer ${uni.getStorageSync('token')}`,
					},
					success: (res) => {
						console.log('用户信息请求成功:', res.data);
						this.userInfo = res.data;
					},
					fail: (err) => {
						console.error('用户信息请求失败:', err);
					},
				});
			},
			getFullAvatarUrl(relativeUrl) {
				return `http://82.157.244.44:8000${relativeUrl}`;
			},
			// 添加关注按钮点击事件
			followUser() {
				// 实现关注逻辑，你可以在这里调用后端接口等
				console.log('关注用户');
			},
		},
	};
</script>

<style scoped>
	.user-profile {
		padding: 20rpx;
	}

	.user-info {
		display: flex;
		align-items: center;
	}

	.user-avatar {
		width: 80px;
		height: 80px;
		border-radius: 50%;
		margin-right: 20rpx;
	}

	.user-name {
		font-size: 18px;
		font-weight: bold;
	}

	.info-section {
		margin-top: 20rpx;
	}

	.user-age,
	.user-intro {
		margin-top: 10rpx;
		color: #666;
		font-size: 14px;
	}

	.gender-icon {
		width: 30rpx;
		height: 30rpx;
		margin-left: 15rpx;
	}

	.follow-button {
		margin-left: auto; // 将关注按钮推到右侧
		background-color: #ff8a89;
		color: #f9f8e5;
		padding: 5px 10px;
		border-radius: 5px;
		cursor: pointer;
		width: 160rpx;
		text-align: center;
	}
</style>