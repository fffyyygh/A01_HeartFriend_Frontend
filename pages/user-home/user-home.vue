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
			<button :class="{'followed-style': isFollowed, 'unfollowed-style': !isFollowed}" @click="follow_click">{{ isFollowed ? '已关注' : '关注' }}</button>
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
				isFollowed: false,
			};
		},
		onLoad(query) {
			this.uuid = query.uuid;
			console.log('this.uuid::::', this.uuid);
			this.getUserInfo(this.uuid);
		},
		methods: {
			follow_click(){
				console.log("a");
				
				
				uni.request({
					url:"http://82.157.244.44:8000/api/v1/user/follow-unfollow/",
					method:"POST",
					header: {
						'Authorization': `Bearer ${uni.getStorageSync('token')}`,
					},
					data:{
						"uuid":this.uuid,
					},
					success: (res)=> {
						console.log("关注成功", res.data);
						this.isFollowed = ! this.isFollowed;
					}
					
				});
			},
			
			get_if_followed(){
				uni.request({
					url:"http://82.157.244.44:8000/api/v1/user/following/",
					method:"GET",
					header: {
						'Authorization': `Bearer ${uni.getStorageSync('token')}`,
					},
					success: (res) =>{
						let follow = res.data.following;
						let uuid= this.userInfo.uuid;
						this.isFollowed = follow.some(item => item.uuid === uuid);
					},
					fail: (err)=> {
						console.log("a");
					}
				})
			
			},
			
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
						this.get_if_followed();
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

	.unfollowed-style{
		margin-left: auto; // 将关注按钮推到右侧
		background-color: #ff8a89;
		color: #f9f8e5;
		padding: 5px 10px;
		border-radius: 15px;
		cursor: pointer;
		width: 160rpx;
		font-size: 10;
		text-align: center;
	}
	
	.followed-style{
		margin-left: auto; // 将关注按钮推到右侧
		background-color: white;
		color: #ff8a89;
		padding: 5px 10px;
		border-radius: 15px;
		border:2px solid #ff8a89;
		cursor: pointer;
		width: 160rpx;
		text-align: center;
	}
	
	
</style>