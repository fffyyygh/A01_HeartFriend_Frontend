<template>
	<view class="register">
		<button class="face_box" open-type="chooseAvatar" @chooseavatar="getFaceImg">
			<image :src="avatar_url" class="face"></image>
			
			<view class="face_btn">点击上传头像</view>
		</button>
		
		
		<uni-forms class="register_box" @submit="register">
			<uni-group>
			<uni-forms-item name="username" required>
				<uni-easyinput  focus prefixIcon="person-filled" placeholder="请输入用户名" v-model="value"></uni-easyinput>
			</uni-forms-item>
			<uni-forms-item name="nickname"required>
				<uni-easyinput  type="nickname" prefixIcon="auth-filled" placeholder="请输入昵称" v-model="value"></uni-easyinput>
			</uni-forms-item>
			<uni-forms-item name="phone_number"required>
				<uni-easyinput  prefixIcon="phone-filled" placeholder="请输入手机号" v-model="value"></uni-easyinput>
			</uni-forms-item>
			<uni-forms-item name="password"required>
				<uni-easyinput  prefixIcon="locked" placeholder="请输入密码" type="password" v-model="password"></uni-easyinput>
			</uni-forms-item>
			<uni-forms-item name="password2"required>
			    <uni-easyinput  prefixIcon="locked-filled" placeholder="请再次输入密码" type="password"
					v-model="password"></uni-easyinput>
			</uni-forms-item>
			</uni-group>
			<button form-type="submit">注册</button>
		</uni-forms>

		
		<view class="register" @click="to_login">已有账号，去登陆</view>

	</view>
</template>

<script>
	export default {
		data() {
			return {
				avatar_url: "../../static/默认_头像.png"
			}
		},
		methods: {
			to_login() {
				uni.navigateTo({
					url: "/pages/login/login"
				})
			},
			getFaceImg(event) {
				console.log(event);
				let self = this;
				
				this.avatar_url = event.detail.avatarUrl;
				console.log(this.avatar_url)
			},
			register(){
				console.log("a")
			}

		}
	}
</script>

<style>
	.face {
		width: 150rpx;
		height: 150rpx;
	}

	.face_box {
		text-align: center;
		background-color: #fff;
		border-color: #fff;
		border-width: 0rpx;
	}

	.register {
		display: flex;
		flex-direction: column;
		padding: 100rpx;
		background-color: #fff;
	}

	.uni-easyinput {
		margin-top: 10rpx;
	}

	.register_box {}

	.register {
		text-align: right;
		color: darkblue;
	}
	.face_btn{
		text-align: center;
	}
	
</style>