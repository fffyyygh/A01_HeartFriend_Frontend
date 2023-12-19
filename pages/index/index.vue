<template>
	<view>
		<swiper class="swiper" :indicator-dots="true" :autoplay="true" :interval="3000" :duration="1000">
			<swiper-item class="swiper-item">
				<image class="swiper-image" mode="aspectFill" src="../../static/index-swiper/cat1.jpg"></image>
			</swiper-item>
			<swiper-item class="swiper-item">
				<image class="swiper-image" mode="aspectFill" src="../../static/index-swiper/cat2.jpeg"></image>
			</swiper-item>
			<swiper-item class="swiper-item">
				<image class="swiper-image" mode="aspectFill" src="../../static/index-swiper/cat3.jpg"></image>
			</swiper-item>
		</swiper>
		<view class="container">
			<view class="box">
				<image class="box-image" mode="aspectFit" src="../../static/index/robot.png"></image>
				<text
					class="encouragement-text">在这段疗愈之旅中，你并非孤身一人。勇敢尝试与AI展开对话，它可以成为你的倾诉伙伴，帮你理清思绪，缓解心情。科技的力量已经证明在心理治疗领域发挥着积极的作用。给予自己这个宝贵的机会，让对话成为你治愈过程中不可或缺的一环。</text>
				<u-button @click="go_chat" hair-line="false" :ripple="true" ripple-bg-color="#e86158"
					:custom-style="btnStyle">聊一聊</u-button>
			</view>
			<view class="box">
				<image class="box-image" mode="aspectFit" src="../../static/index/diary.png"></image>
				<text
					class="encouragement-text">每天都是新的开始，而记录下你的感受是迈向康复的一步。尝试将情绪、想法和反应写在日记中，这不仅有助于更好地了解自己，还有助于追踪变化。就像有人说的那样，文字是治愈的良药。日记是你独特的故事，每一页都是自我关爱的表现。</text>
				<u-button @click="go_diary" hair-line="false" :ripple="true" ripple-bg-color="#e86158"
					:custom-style="btnStyle">心情记录</u-button>
			</view>
		</view>
		<!-- 用于测试的按钮，点击后跳转到 user-home 页面 -->
		<button @click="goToUserHome">提升我自己为管理员</button>
	</view>
</template>

<script>
	export default {
		onLoad() {
			console.log("a");
			// 检查本地存储中是否有令牌
			const token = uni.getStorageSync('token');
			if (!token) {
				// 如果没有令牌，执行登录和注册流程（操作A）
				this.performLoginAndRegistration();
			} else {
				// 如果令牌存在，向后端请求用户信息
				this.requestUserInfo(token);
			}
		},
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
			goToUserHome() {
				uni.request({
					url: `http://82.157.244.44:8000/api/v1/forum/getAdmin/`,
					method: "POST",
					header: {
						'Authorization': `Bearer ${uni.getStorageSync('token')}`,
					},
					success: (res) => {
						console.log(res);
					}
				
				});
			},

			async getLoginCode() {
				return new Promise((resolve, reject) => {
					uni.login({
						provider: 'weixin',
						success: function(loginRes) {
							resolve(loginRes.code);
						},
					});
				});
			},
			async performLoginAndRegistration() {
				console.log("开始登录注册流程");
				// 在这里实现登录和注册流程
				try {
					uni.showLoading({
						mask: true,
						title: '正在登录中',
					});
					//正常code
					const code = await this.getLoginCode();
					// 注册测试用code
					// const code = "test12052128";
					console.log(code);
					// 发送登录请求
					const loginRes = await uni.request({
						url: 'http://82.157.244.44:8000/api/v1/user/login/',
						method: 'POST',
						header: {
							'Content-Type': 'application/json',
						},
						data: {
							code: code,
						},
					});
					// console.log(loginRes);
					// console.log(loginRes[1]);
					// console.log(loginRes[1].data);
					const res = loginRes[1].data;
					console.log(res);

					if (res.access) {
						uni.setStorageSync('has_Login', true);
						uni.setStorageSync('token', res.access);

						// 检查 'created' 标志并采取适当的操作
						if (res.created) {
							// 如果用户已注册，请求用户信息
							this.requestUserInfo(res.access);
						} else {
							// 如果用户未注册，跳转到注册页面
							uni.navigateTo({
								url: '/pages/login/register',
							});
						}
					}
				} catch (error) {
					console.error(error);
					// 处理登录失败的逻辑，显示错误信息给用户
				} finally {
					uni.hideLoading();
				}
			},
			async requestUserInfo(token) {
				console.log("开始请求用户信息");
				// 在这里实现使用令牌向后端请求用户信息的逻辑
				try {
					const userInfoRes = await uni.request({
						url: 'http://82.157.244.44:8000/api/v1/user/info/',
						method: 'GET',
						header: {
							'Authorization': `Bearer ${token}`,
						},
					});
					//console.log(userInfoRes[1].data);
					const userInfo = userInfoRes[1].data;
					//console.log(userInfo);
					// 在获取用户信息后，将其存储在本地存储中以备将来使用
					// uni.setStorageSync('userInfo', userInfo);
					// 修改为先判断本地存储中是否有userInfo，如果存在，则更新它，否则就存储新的 userInfo。
					let storedUserInfo = uni.getStorageSync('userInfo');
					if (storedUserInfo) {
						// 如果本地存储中已经有 userInfo，则更新它
						storedUserInfo = Object.assign({}, storedUserInfo, userInfo);
					} else {
						// 如果本地存储中没有 userInfo，则直接存储新的 userInfo
						storedUserInfo = userInfo;
					}
					// 存储 userInfo 到本地存储
					uni.setStorageSync('userInfo', storedUserInfo);

					// 做任何其他关于用户信息的操作
				} catch (error) {
					console.error(error);
					// 处理获取用户信息失败的逻辑
				}
			},
			go_chat() {
				uni.navigateTo({
					url: '/pages/AiChat/AiChat',
				});
			},
			go_diary() {
				uni.navigateTo({
					url: '/pages/diary/diary_index',
				});
			},
		},
	}
</script>

<style lang="scss">
	.container {
		padding: 20rpx;
		font-size: 14px;
		display: flex;
		justify-content: space-between;
	}

	.box {
		width: 48%;
		/* 设置盒子宽度为容器的48% */
		text-align: center;
	}

	.box-image {
		width: 100%;
		height: 150rpx;
		margin-top: 10px;
		/* 调整图片与文本之间的间距 */
	}

	.encouragement-text {
		margin-top: 10rpx;
	}

	.swiper {
		height: 400upx;
	}

	.swiper-item {
		display: block;
		text-align: center;
	}

	/*图片宽度设置100% ，高度300upx（设为auto图片无法显示）*/
	.swiper-image {
		width: 100%;
		height: 400upx;
	}
</style>