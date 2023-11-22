<template>
	<view class="login">
		<!-- <image class="logo" :src="logo"></image> -->
		<text class="txt1">申请获取以下权限</text>
		<text class="txt2">获取你的登录信息</text>
		<u-button @click="login" hair-line="false" :ripple="true" ripple-bg-color="#e86158" :custom-style="btnStyle">授权登录</u-button>
		
		
		
		
		<view class="input_box">
			<uni-easyinput focus prefixIcon="person-filled" placeholder="请输入用户名" v-model="value" ></uni-easyinput>
			<uni-easyinput prefixIcon="locked-filled"  placeholder="请输入密码" type="password" v-model="password"></uni-easyinput>
		</view>
		<u-button class="login_btn" @click="login" size="medium" hair-line="false" :ripple="true" ripple-bg-color="#e86158" :custom-style="btnStyle">登录</u-button>
		<view class="register" @click="to_register">注册</view>
	
	</view>
</template>

<script>
	export default {
		data() {
			return {
				btnStyle: {
					color: "#f9f8e5",
					backgroundColor: '#ff8a89',
					
				},
				logo: "",
			};
		},
		onLoad() {
			// this.getSysInfo();
		},
		methods: {
			goBack() {
				uni.reLaunch({
					url: '/pages/index/index'
				});
			},
			
			to_register(){
					console.log("a");
					uni.navigateTo({
						url:"/pages/login/register"
					})
			},
			

			async login() {
				uni.showLoading({
					mask: true,
					title: '正在登录中'
				});
				var that = this;
				let code = await this.getLoginCode();
				
				uni.request({
					url: 'http://82.157.244.44:8000/auth/login/',
					method: 'POST',
					header: {
						'Content-Type': 'application/json',
					},
					data: {
						code: code,
					},
					success: (res) => {
						if (res.data.access) {
							uni.setStorageSync("hasLogin", true);
							uni.setStorageSync("token", res.data.access);
							// that.getUserInfo(); // 如果有获取用户信息的方法，可以调用
							uni.switchTab({
								url: '/pages/index/index'
							});
						}
						uni.hideLoading();
					},
					fail: (err) => {
						console.error(err);
						// 处理登录失败的逻辑，显示错误信息给用户
						uni.hideLoading();
					},
				});
			},
			// getUserInfo() {
			// 	this.$H.get("user/userInfo").then(res => {
			// 		uni.setStorageSync("userInfo", res.result)
			// 	})
			// },
			getLoginCode() {
				return new Promise((resolve, reject) => {
					uni.login({
						provider: 'weixin',
						success: function(loginRes) {
							resolve(loginRes.code);
						}
					});
				});
			},


		}
	}
</script>

<style lang="scss">
	.login {
		display: flex;
		flex-direction: column;
		padding: 100rpx;
	}

	.login .logo {
		width: 230rpx;
		height: 230rpx;
		margin: 50rpx auto;
	}

	.login .txt1 {
		margin-bottom: 10rpx;
	}

	.login .txt2 {
		color: #999;
		margin-bottom: 50rpx;
	}

	.login .txt3 {
		color: #8c8c8c;
		margin-top: 30rpx;
		text-align: center;
	}
	.input_box{
		text-align: center;
		margin-top: 160rpx;
	}
	.register{
		text-align: right;
		color: darkblue;
	}
	.login_btn{
		text-align: center;
		margin-top: 20rpx;
	}
	.uni-easyinput{
		margin-top: 30rpx;
	}
</style>