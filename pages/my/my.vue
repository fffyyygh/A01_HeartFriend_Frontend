<template>
	<view>
		<view class="head">
			<block v-if="has_login">
				<view class="userinfo" @click="goUser">
					<u-avatar :src="userInfo.avatar_url"></u-avatar>
					<view class="username">
						<text>{{ userInfo.username }}</text>
						<text class="sub-txt">{{ userInfo.self_intro }}</text>
					</view>

					<image v-if="userInfo.gender === 'male'" src="/static/my/male.png" class="gender-icon"></image>
					<image v-else-if="userInfo.gender === 'female'" src="/static/my/female.png" class="gender-icon">
					</image>
					<image v-else-if="userInfo.gender === 'other'" src="/static/my/other.png" class="gender-icon">
					</image>
					<image v-else-if="userInfo.gender === 'secret'" src="/static/my/secret.png" class="gender-icon">
					</image>

					<u-icon name="arrow-right" class="arrow-right"></u-icon>
				</view>
			</block>
			<block v-else>
				<view class="btn-login">
					<u-button type="default" shape="circle" @click="wxLogin" plain>登录</u-button>
				</view>
			</block>
			<u-grid :col="3" :border="false" style="margin: 20rpx 0;">
				<u-grid-item @click="to_my_fans">
					<text>{{ fansNum}}</text>
					<view class="grid-text">粉丝</view>
				</u-grid-item>
				<u-grid-item @click="to_my_focus">
					<text>{{ focusNum }}</text>
					<view class="grid-text">关注</view>
				</u-grid-item>
				<u-grid-item @click="to_my_posts">
					<text>{{ postNum }}</text>
					<view class="grid-text">帖子</view>
				</u-grid-item>

			</u-grid>
		</view>
		<view class="block-wrap">
			<view class="block-title">我的服务</view>
			<u-grid :col="1" :border="false" style="margin: 20rpx 0;">

				<u-grid-item @click="goLike" class="service-item">
					<image class="gn-icon" src="/static/my/like-icon.png"></image>
					<view class="grid-text">我的点赞</view>
				</u-grid-item>

				<u-grid-item @click="goDislike" class="service-item">
					<image class="gn-icon" src="/static/my/dislike-icon.png"></image>
					<view class="grid-text">我的点踩</view>
				</u-grid-item>

				<u-grid-item @click="goReport" class="service-item">
					<image class="gn-icon" src="/static/my/report-icon.png"></image>
					<view class="grid-text">我的举报</view>
				</u-grid-item>

				<u-grid-item v-show="if_admin" @click="goAdmin" class="service-item">
					<image class="gn-icon" src="/static/my/admin-icon.png"></image>
					<view class="grid-text">管理论坛</view>
				</u-grid-item>
			</u-grid>
		</view>
		<button @click="goToUserHome">提升我自己为管理员</button>

	</view>

</template>

<script>
	export default {
		onShow() {
			this.userInfo = uni.getStorageSync('userInfo');
			this.getIfAdmin();
			this.get_all_post();
			this.get_all_focus();
			this.get_all_fans();

		},
		data() {
			return {
				userInfo: uni.getStorageSync('userInfo'),
				has_login: true,
				postNum: "",
				focusNum: "",
				fansNum: "",
				if_admin: false,
			};
		},
		methods: {

			
			goToUserHome() {
				uni.request({
					url: `https://vx.mikumikumi.xyz/api/v1/forum/getAdmin/`,
					method: "POST",
					header: {
						'Authorization': `Bearer ${uni.getStorageSync('token')}`,
					},
					success: (res) => {
						console.log(res);
					}
				
				});
			},
			getIfAdmin() {
				uni.request({
					url: `https://vx.mikumikumi.xyz/api/v1/user/info/`,
					method: "GET",
					header: {
						'Authorization': `Bearer ${uni.getStorageSync('token')}`,
					},

					success: (res) => {
						if (res.data.is_forum_admin)

						{
							this.if_admin = true;
						} else {
							this.if_admin = false;
						}

					},
					fail: (err) => {
						console.log("a");
					}
				})
			},

			goUser() {
				uni.navigateTo({
					url: "/pages/login/register"
				})
			},
			to_my_posts() {
				uni.navigateTo({
					url: "/pages/my/my_posts"
				})
			},


			to_my_focus() {
				uni.navigateTo({
					url: "/pages/my/my_focus"
				})
			},

			to_my_fans() {
				uni.navigateTo({
					url: "/pages/my/my_fans"
				})
			},



			get_all_post() {
				uni.request({
					url: `https://vx.mikumikumi.xyz/api/v1/forum/posts/getUserPosts/?offset=0&limit=20`, // 后端接口地址
					method: 'GET',
					header: {
						'Authorization': `Bearer ${uni.getStorageSync('token')}`,
					},
					data: {
						"uuid": this.userInfo.uuid,
					},

					success: (res) => {
						this.postNum = res.data.count;
					},
					fail: (err) => {
						console.error('数据发送失败:', err);
					}
				});


			},


			get_all_focus() {
				uni.request({
					url: 'https://vx.mikumikumi.xyz/api/v1/user/following/', // 后端接口地址
					method: 'GET',
					header: {
						'Authorization': `Bearer ${uni.getStorageSync('token')}`,
					},
					success: (res) => {
						this.focusNum = res.data.following.length;
					},
					fail: (err) => {
						console.error('数据发送失败:', err);
					}
				});

			},


			get_all_fans() {
				uni.request({
					url: 'https://vx.mikumikumi.xyz/api/v1/user/followers/', // 后端接口地址
					method: 'GET',
					header: {
						'Authorization': `Bearer ${uni.getStorageSync('token')}`,
					},
					success: (res) => {
						//console.log('数据接收成功:', res.data);	
						this.fansNum = res.data.followers.length;
					},
					fail: (err) => {
						console.error('数据发送失败:', err);
					}
				});

			},
			goAdmin() {
				uni.navigateTo({
					url: "/pages/my/admin",
				})
			},
			goLike() {
				uni.navigateTo({
					url: "/pages/my/post_like",
				})
			},
			goDislike() {
				uni.navigateTo({
					url: "/pages/my/post_dislike",
				})
			},
			goReport() {
				uni.navigateTo({
					url: "/pages/my/report_my",
				})
			}

		}
	}
</script>

<style lang="scss" scoped>
	.head {
		padding: 20rpx;
		background-color: #F9E8D9;

		.sub-txt {
			font-size: 24rpx;
			color: #616161;
			display: block;
			display: -webkit-box;
			-webkit-box-orient: vertical;
			-webkit-line-clamp: 1;
			overflow: hidden;
		}

		margin-bottom: 20rpx;
	}

	.userinfo {
		display: flex;
		align-items: center;
		padding: 20rpx;
	}

	.userinfo .username {
		display: flex;
		flex-direction: column;
		margin-left: 20rpx;
	}

	.grid-text {
		color: #999;
		font-size: 12px;
		margin-bottom: 20rpx;
	}

	.userinfo u-avatar {
		margin-right: 20rpx;
	}

	.userinfo .arrow-right {
		margin-left: auto;
	}

	.btn-login {
		margin: 40rpx 0;
	}

	.gn-icon {
		width: 60rpx;
		height: 60rpx;
		margin-bottom: 20rpx;
	}

	/*服务按钮*/
	.btn-wrap {
		display: flex;
		margin-top: 30rpx;
	}

	.btn-wrap .btn-contact {
		background-color: #fff;
		margin-left: 15rpx;
		margin-right: 15rpx;
		padding: 20rpx;
		line-height: unset;
		font-size: 12px;
		color: #999;
	}

	.btn-wrap .btn-contact:active {
		background-color: #ffe838;
	}

	.btn-wrap .btn-contact .txt {
		margin-top: 20rpx;
	}

	.btn-wrap .btn-contact::after {
		border: unset;
		position: unset;
	}

	.icon-size {
		font-size: 50rpx;
	}

	.block-wrap {
		background-color: #fff;
		border-radius: 20rpx;
		margin: 20rpx;
		overflow: hidden;

		.block-title {
			background-color: #fff;
			padding: 20rpx;
		}

		.service-item {
			display: flex;
			align-items: center;
			justify-content: space-between;
			padding: 20rpx;
			border-bottom: 1px solid #ccc; // Add border between items

			.gn-icon {
				width: 60rpx;
				height: 60rpx;
				margin-right: 20rpx; // Adjust spacing between icon and text
			}

			.grid-text {
				flex-grow: 1; // Allow text to take remaining space
				color: #999;
				font-size: 26rpx; // Adjust font size as needed
			}
		}
	}

	.gender-icon {
		width: 30rpx;
		height: 30rpx;
		margin-left: 15rpx;
	}
</style>