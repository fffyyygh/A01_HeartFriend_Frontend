<template>
	<view class="register">
		<button class="face_box" open-type="chooseAvatar" @chooseavatar="getFaceImg">
			<image :src="avatar_url" class="face"></image>
		</button>
		<view class="face_btn">点击上传头像</view>
		<uni-forms class="register_box" label-position="top" :modelValue="userInfo" @submit="register"
			:rules="registerRules" ref="register_form">
			<uni-group>
				<uni-forms-item name="username" label="昵称" required>
					<uni-easyinput type="nickname" prefixIcon="auth-filled" placeholder="请输入昵称" @blur="bindblur"
						@input="bindinput" v-model="userInfo.username"></uni-easyinput>
				</uni-forms-item>
				<uni-forms-item name="self_intro" label="个人介绍" required>
					<uni-easyinput prefixIcon="person-filled" placeholder="请输入个人介绍"
						v-model="userInfo.self_intro"></uni-easyinput>
				</uni-forms-item>
				<uni-forms-item name="gender" label="性别" required>
					<uni-data-checkbox v-model="userInfo.gender" :localdata="sexs" />
				</uni-forms-item>
				<uni-forms-item name="age" label="年龄" required>
					<uni-easyinput type="number" placeholder="请输入年龄" v-model="userInfo.age"></uni-easyinput>
				</uni-forms-item>
			</uni-group>
			<u-button @click="register" :custom-style="btnStyle">提交</u-button>
		</uni-forms>

	</view>
</template>

<script>
	export default {
		onLoad() {
			console.log("加载服务器里的userInfo");
			this.requestUserInfo(uni.getStorageSync('token'));
		},
		data() {
			return {
				btnStyle: {
					color: "#f9f8e5",
					backgroundColor: '#527853'
				},
				avatar_url: uni.getStorageSync('userInfo').avatar_url,
				// 单选数据源
				sexs: [{
					text: '男',
					value: "male"
				}, {
					text: '女',
					value: "female"
				}, {
					text: '保密',
					value: "secret"
				}, {
					text: '其他',
					value: "other"
				}],
				userInfo: uni.getStorageSync('userInfo'),
				registerRules: {
					// 对username字段进行必填验证
					username: {
						rules: [{
								required: true,
								errorMessage: '请填写昵称',
							},
							{
								minLength: 1,
								maxLength: 9,
								errorMessage: '{label}长度在 {minLength} 到 {maxLength} 个字符',
							}
						],
						label: '昵称'
					},
					age: {
						rules: [{
							required: true,
							errorMessage: '请选择年龄',
						}, ],
						label: '年龄',
					},
				}
			}
		},
		methods: {
			bindblur(e) {
				console.log(e.detail.value);
				this.userInfo.username = e.detail.value; // 获取微信昵称
			},
			bindinput(e) {
				console.log(e.detail.value);
				this.userInfo.username = e.detail.value; // 获取微信昵称
			},
			to_login() {
				uni.navigateTo({
					url: "/pages/login/login"
				})
			},
			async getFaceImg(event) {
				try {
					let self = this;
					this.avatar_url = event.detail.avatarUrl;
					console.log('this.avatar_url:', this.avatar_url);

					// Set the avatar_url in the userInfo object
					this.userInfo.avatar_url = event.detail.avatarUrl;

					const tempImageUrl = this.avatar_url;
					console.log('tempImageUrl:', this.avatar_url);

					// Use uni.getImageInfo to get the local path of the image
					const imageInfo = await uni.getImageInfo({
						src: tempImageUrl,
					});

					console.log('imageInfo:', imageInfo);
					console.log('imageInfo[1].path:', imageInfo[1].path);

					// Step 2: Save the image locally
					wx.getFileSystemManager().saveFile({
						tempFilePath: imageInfo[1].path,
						success: async (saveRes) => {
							const savedFilePath = saveRes.savedFilePath;
							console.log(savedFilePath);
							console.log("图片地址")
							// Step 3: Upload the saved file to the server
							uni.uploadFile({
								url: 'http://82.157.244.44:8000/api/v1/user/upload-avatar/',
								filePath: savedFilePath,
								name: 'avatar_url',
								header: {
									'content-type': 'multipart/form-data',
									'Authorization': `Bearer ${uni.getStorageSync('token')}`,
									// Add any other headers if needed
								},
								success: (uploadRes) => {
									const data = JSON.parse(uploadRes.data);
									console.log('Upload successful:', data);

									// Step 4: Request user info after the upload is complete
									this.requestUserInfo(uni.getStorageSync('token')).then(
										() => {
											this.avatar_url = uni.getStorageSync(
												'userInfo').avatar_url;
											console.log('更新后的this.avatar_url', this
												.avatar_url);
										});
								},
								fail: (uploadErr) => {
									console.error('Upload failed:', uploadErr);
									// Handle the upload failure
								},
							});
						},
						fail: (saveErr) => {
							console.error('Save file failed:', saveErr);
							// Handle the save failure
						},
					});
				} catch (error) {
					console.error('Error in getFaceImg:', error);
				}
			},

			async requestUserInfo(token) {
				console.log("开始请求用户信息");
				console.log('register userInfo', this.userInfo);
				// 在这里实现使用令牌向后端请求用户信息的逻辑
				try {
					const userInfoRes = await uni.request({
						url: 'http://82.157.244.44:8000/api/v1/user/info/',
						method: 'GET',
						header: {
							'Authorization': `Bearer ${token}`,
						},
					});
					console.log(userInfoRes[1].data);
					const userInfo = userInfoRes[1].data;
					console.log(userInfo);
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
					this.userInfo = storedUserInfo;
					console.log('请求后 userInfo', this.userInfo);

					// 做任何其他关于用户信息的操作
				} catch (error) {
					console.error(error);
					// 处理获取用户信息失败的逻辑
				}
			},
			async register() {
				// 表单验证
				try {
					await this.$refs.register_form.validate();
				} catch (err) {
					console.log('表单错误信息：', err);
					// 在这里处理表单验证失败的逻辑
					return; // 如果表单验证失败，不进行后续的请求
				}

				// 表单验证通过，发送注册请求到后端
				try {
					console.log(uni.getStorageSync('token'));
					console.log('开始发送注册请求');
					const res = await uni.request({
						url: 'http://82.157.244.44:8000/api/v1/user/info/',
						method: 'PATCH',
						header: {
							'Authorization': `Bearer ${uni.getStorageSync('token')}`,
							'Content-Type': 'application/json',
						},
						data: this.userInfo,
					});
					console.log(res);
					// 检查后端返回的注册信息，如果成功则进行进一步操作
					if (res[1].data) {
						console.log('注册成功！');
						console.log('注册返回的信息res[1].data', res[1].data);
						await this.requestUserInfo(uni.getStorageSync('token'));
					} else {
						console.error('注册失败，后端返回:', res.data);
						// 在这里处理注册失败的逻辑
					}
				} catch (error) {
					console.error('注册请求失败:', error);
					// 在这里处理请求失败的逻辑
				}
				uni.navigateBack({
					delta: 1
				});
			}
		}
	}
</script>

<style>
	.face_box {
		/* text-align: center; */
		background-color: #fff;
		/* border: none;
		border-color: #fff;
		outline: none; */
	}

	button::after {
		border: 0; // 或者 border: none;
	}

	.face {
		width: 150rpx;
		height: 150rpx;
	}

	.face_btn {
		text-align: center;
		color: darkgray;
	}

	.register {
		display: flex;
		flex-direction: column;
		padding: 50rpx;
		background-color: #fff;
	}

	.uni-easyinput {
		margin-top: 10rpx;
	}

	.register_box {}

	.registerbtn {
		background-color: #ff8a89;
	}
</style>