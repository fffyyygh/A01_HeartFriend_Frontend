<template>
	<view class="register">
		<button class="face_box" open-type="chooseAvatar" @chooseavatar="getFaceImg">
			<image :src="avatar_url" class="face"></image>
		</button>
		<view class="face_btn">点击上传头像</view>
		<uni-forms class="register_box" :modelValue="userData" @submit="register" :rules="registerRules"
			ref="register_form">
			<uni-group>
				<uni-forms-item name="username" label="用户名" required>
					<uni-easyinput focus prefixIcon="person-filled" placeholder="请输入用户名"
						v-model="userData.username"></uni-easyinput>
				</uni-forms-item>
				<uni-forms-item name="nickname" label="昵称" required>
					<uni-easyinput type="nickname" prefixIcon="auth-filled" placeholder="请输入昵称" @blur="bindblur"
						@input="bindinput" v-model="userData.nickname"></uni-easyinput>
				</uni-forms-item>
				<uni-forms-item name="gender" label="性别" required>
					<uni-data-checkbox v-model="userData.gender" :localdata="sexs" />
				</uni-forms-item>
				<uni-forms-item name="phone_number" label="手机号" required>
					<uni-easyinput prefixIcon="phone-filled" placeholder="请输入手机号"
						v-model="userData.phone_number"></uni-easyinput>
				</uni-forms-item>
				<uni-forms-item name="password" label="密码" required>
					<uni-easyinput prefixIcon="locked" placeholder="请输入密码" type="password"
						v-model="userData.password"></uni-easyinput>
				</uni-forms-item>
				<uni-forms-item name="password2" label="密码" required>
					<uni-easyinput prefixIcon="locked-filled" placeholder="请再次输入密码" type="password"
						v-model="userData.password2"></uni-easyinput>
				</uni-forms-item>
			</uni-group>
			<u-button @click="register" :custom-style="btnStyle">注册</u-button>
			<u-button @click="to_login" :custom-style="btnStyle2">已有账号，去登陆</u-button>
		</uni-forms>

	</view>
</template>

<script>
	export default {
		data() {
			return {
				btnStyle: {
					color: "#f9f8e5",
					backgroundColor: '#ff8a89'
				},
				btnStyle2: {
					marginTop: '20rpx',
					color: "#f9f8e5",
					backgroundColor: '#454545'
				},
				avatar_url: "../../static/默认_头像.png",
				// 单选数据源
				sexs: [{
					text: '男',
					value: 0
				}, {
					text: '女',
					value: 1
				}, {
					text: '保密',
					value: 2
				}],
				userData: {
					username: '',
					nickname: '',
					gender: 2,
					phone_number: '',
					password: '',
					password2: '',
					avatar_url: "../../static/默认_头像.png"
				},
				registerRules: {
					// 对username字段进行必填验证
					username: {
						// username 字段的校验规则
						rules: [
							// 校验 name 不能为空
							{
								required: true,
								errorMessage: '请填写用户名',
							},
							// 对name字段进行长度验证
							{
								minLength: 1,
								maxLength: 9,
								errorMessage: '{label}长度在 {minLength} 到 {maxLength} 个字符',
							}
						],
						// 当前表单域的字段中文名，可不填写
						label: '用户名'
					},
					// 对nickname字段进行必填验证
					nickname: {
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
					// 对phone_number字段进行必填验证
					phone_number: {
						rules: [{
								required: true,
								errorMessage: '请填写手机号',
							}
							// {
							// 	pattern: /^[1][3-9][0-9]{9}$/,
							// 	errorMessage: '请输入有效的{label}',
							// }
						],
						label: '手机号'
					},
					// 对password字段进行必填验证
					password: {
						rules: [{
								required: true,
								errorMessage: '请填写密码',
							},
							// Add other rules as needed for the password field
						],
						label: '密码'
					},
					// 对password2字段进行必填验证
					password2: {
						rules: [{
								required: true,
								errorMessage: '请填写密码',
							},
							// Add other rules as needed for the password field
						],
						label: '密码'
					},
				}
			}
		},
		methods: {
			bindblur(e) {
				console.log(e.detail.value);
				this.userData.nickname = e.detail.value; // 获取微信昵称
			},
			bindinput(e) {
				console.log(e.detail.value);
				this.userData.nickname = e.detail.value; // 获取微信昵称
			},
			to_login() {
				uni.navigateTo({
					url: "/pages/login/login"
				})
			},
			getFaceImg(event) {
				console.log(event);
				let self = this;

				this.avatar_url = event.detail.avatarUrl;
				console.log(this.avatar_url);
				// Set the avatar_url in the userData object
				this.userData.avatar_url = event.detail.avatarUrl;
				console.log('Avatar URL:', this.userData.avatar_url);
				//获取到的头像地址是临时地址，所以拿到地址以后我们要将图片上传到自己的服务器上面
			},
			register() {
				this.$refs.register_form.validate().then(res => {
					console.log('表单数据信息：', res);
					console.log('User Data:', this.userData);
					// You can send the data to your backend API here once it's available
					// For example, you can use uni.request to make an HTTP request to your backend
					// uni.request({
					//   url: 'your_backend_api_url',
					//   method: 'POST',
					//   data: this.userData,
					//   success(res) {
					//     console.log('Backend Response:', res.data);
					//   },
					//   fail(err) {
					//     console.error('Error sending data to backend:', err);
					//   }
					// });
				}).catch(err => {
					console.log('表单错误信息：', err);
				})
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