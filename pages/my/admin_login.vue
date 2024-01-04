<template>
	<div class="login-page">
		<input v-model="username" type="text" placeholder="请输入管理员用户名" />
		<input v-model="password" type="password" placeholder="请输入密码" />
		<button class="btn" @click="submitForm">提交</button>
		<!-- <button @click="goback">回首页</button> -->
	</div>
</template>

<script>
	export default {
		data() {
			return {
				username: '',
				password: ''
			};
		},
		methods: {
			goback(){
				uni.switchTab({
					url: "/pages/my/my"
				})
			},
			submitForm() {
				// 在这里执行提交操作，比如发送请求给服务器进行验证
				// 你可以使用uni.request或者其他方法发送HTTP请求

				// 示例：假设使用uni.request发送POST请求
				uni.request({
					url: 'https://vx.mikumikumi.xyz/api/v1/forum/admin/login/',
					method: 'POST',
					data: {
						"openID": this.username,
						"password": this.password
					},
					success: (res) => {
						if(res.statusCode===200){
							console.log('登录成功', res);
							let storedtoken = uni.getStorageSync('token');
							this.requestUserInfo(res.data.access);
						}
						else{
							uni.showToast({
								title: '账号或密码错误',
								duration: 1000,
								icon:"error"
							});
						}
						
						

					},
					fail: (err) => {
						console.error('登录失败', err);
					}
				});
			},
			async requestUserInfo(token) {
				console.log("开始请求用户信息");
				// 在这里实现使用令牌向后端请求用户信息的逻辑
				try {
					const userInfoRes = await uni.request({
						url: 'https://vx.mikumikumi.xyz/api/v1/user/info/',
						method: 'GET',
						header: {
							'Authorization': `Bearer ${token}`,
						},


					});
					if (userInfoRes[1].statusCode === 200) { //console.log(userInfoRes[1].data);
						const userInfo = userInfoRes[1].data;

						console.log("aaaa", userInfoRes[1]);
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
						let storedtoken = uni.getStorageSync('token');
						uni.setStorageSync('old_token', storedtoken);
						uni.setStorageSync('token', token);
					}
					uni.showToast({
						title: '管理员登陆成功',
						duration: 1000,
						icon:"success"
					});
					
					

					// 做任何其他关于用户信息的操作
				} catch (error) {
					console.error("token失效", error);
					// 处理获取用户信息失败的逻辑
				}
			},
		}
	};
</script>

<style scoped>
	.login-page {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		height: 100vh;
	}

	input {
		margin-bottom: 10px;
		padding: 8px;
		border-radius: 4px;
		border: 1px solid #ccc;
		width: 400rpx;
	}

	.btn {
		display: flex;
		justify-content: center;
		align-items: center;
		width: 300rpx;
		height: 80rpx;
		margin-top: 20rpx;
		padding: 12px 20px;
		border-radius: 5px;
		background-color: #527853;
		color: #fff;
		text-align: center;
		font-size: 18px;
		/* 调整字体大小 */
		font-weight: bold;
		text-decoration: none;
		box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
		transition: transform 0.3s ease, background-color 0.3s ease, box-shadow 0.3s ease;
	}
</style>