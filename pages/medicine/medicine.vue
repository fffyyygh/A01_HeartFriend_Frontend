<template>
	<view>
		<view class="button-container">
			<button class="custom-button" @click="add_medicine">
				<view class="button-content">
					<text>添加药物提醒</text>
					<image src="/static/medicine/药丸.png"></image>
				</view>
			</button>

			<button class="custom-button" @click="all_medicine">
				<view class="button-content">
					<text>查看药物提醒</text>
					<image src="/static/medicine/医药制造业.png"></image>
				</view>
			</button>
			<view class="focuslink" @click="focus_pic">
			      <text class="link-text">只有关注才能正常获取，点击链接跳转</text>
			</view>
		</view>
		

	</view>
</template>

<script>
	export default {
		data() {
			return {

			};
		},
		methods: {
			

			add_medicine() {
				uni.navigateTo({
					url: "/pages/medicine/medicine_add"
				})
			},
			all_medicine() {
				uni.navigateTo({
					url: "/pages/medicine/medicine_all"
				})
			},
			focus_pic(){
				uni.navigateTo({
					url:"medicine_focuspic"
				})
			}
		}
	};
</script>

<style>
	button {
		height: 400rpx;
		width: 700rpx;
		padding: 10px 20px;
		background-color: #527853;
		color: white;
		border: none;
		border-radius: 10px;
		cursor: pointer;
		outline: none;
		margin: 30rpx;
		/* 取消按钮点击时的默认边框样式 */
	}

	.button-container {
		height: 100%;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
	}

	.button-content {
		display: flex;
		flex-direction: column;
		align-items: center;
	}
	.focuslink {
	    margin-top: 20rpx; /* Adjust the top margin as needed */
	    font-size: 28rpx; /* Adjust the font size */
	    color: #527853; /* Change the text color */
	    text-decoration: underline; /* Underline the text */
	    cursor: pointer;
	    /* Other styles as per your design */
	  }
	
	  .link-text {
	    padding: 8rpx; /* Add padding to the text */
	  }


	text {
		margin-bottom: 20px;
		/* 文字和图片之间的间距 */
		font-size: 32rpx;
		font-weight: bold;
	}

	image {
		width: 120px;
		/* 调整图片的宽度 */
		height: 120px;
		/* 调整图片的高度 */
	}
</style>