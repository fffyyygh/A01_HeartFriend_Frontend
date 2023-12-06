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
			<u-grid :col="3" :border="false" style="margin: 20rpx 0;" @click="toNav">
				<u-grid-item>
					<text>{{ userInfo.fans || 0}}</text>
					<view class="grid-text">粉丝</view>
				</u-grid-item>
				<u-grid-item>
					<text>{{ userInfo.follow || 0 }}</text>
					<view class="grid-text">关注</view>
				</u-grid-item>
				<u-grid-item @click="to_my_posts">
					<text>{{ userInfo.postNum || 0 }}</text>
					<view class="grid-text">帖子</view>
				</u-grid-item>

			</u-grid>
		</view>
		<view class="block-wrap">
			<view class="block-title">我的服务</view>
			<u-grid :col="1" :border="false" style="margin: 20rpx 0;" @click="toNav">

				<u-grid-item index="/pages/my/post?type=2">
					<image class="gn-icon"></image>
					<view class="grid-text">我的帖子</view>
				</u-grid-item>

				<u-grid-item index="/pages/my/post?type=1">
					<image class="gn-icon"></image>
					<view class="grid-text">我的点赞</view>
				</u-grid-item>

				<u-grid-item index="/pages/my/user?type=1">
					<image class="gn-icon"></image>
					<view class="grid-text">我的关注</view>
				</u-grid-item>

				<u-grid-item index="/pages/my/user?type=2">
					<image class="gn-icon"></image>
					<view class="grid-text">我的设置</view>
				</u-grid-item>


			</u-grid>
		</view>

		<button @click="exit">退出登陆</button>
	</view>

</template>

<script>
	export default {
		onShow() {
			console.log('my.vue is shown');
			this.userInfo = uni.getStorageSync('userInfo');
		},
		data() {
			return {
				userInfo: uni.getStorageSync('userInfo'),
				has_login: true
			};
		},
		methods: {
			exit() {
				uni.navigateTo({
					url: "/pages/login/login"
				})
			},
			goUser() {
				uni.navigateTo({
					url: "/pages/login/register"
				})
			},
			to_my_posts(){
				uni.navigateTo({
					url:"/pages/my/my_posts"
				})
			}
		}
	}
</script>

<style lang="scss" scoped>
	.head {
		padding: 20rpx;
		background-color: #ffe838;

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
	}

	.gender-icon {
		width: 30rpx;
		height: 30rpx;
		margin-left: 15rpx;
	}
</style>