<template>
	<view>
		<view >
		<button class="custom-button" @click="add_medicine">
			
			<view class="button-content">
			<text>添加药物提醒</text>
			<image src="/static/medicine/药丸.png"></image>
			</view>
		</button>
		
		<button @click="all_medicine">
		<view class="button-content">
		       <text>查看药物提醒</text>
		      <image src="/static/medicine/医药制造业.png"></image>
		</view></button>
		</view>
		
	</view>
</template>

<script>
	export default {
		data() {
			
		},
		methods: {
			
			add_medicine(){
				uni.navigateTo({
					url:"/pages/medicine/medicine_add"
				})
			},
			all_medicine(){
				uni.navigateTo({
					url:"/pages/medicine/medicine_all"
				})
			}
		}
	};
</script>

<style>
	button {
		padding: 10px 20px;
		background-color: #3498db;
		color: white;
		border: none;
		border-radius: 10px;
		cursor: pointer;
		outline: none;
		margin: 30rpx;
		/* 取消按钮点击时的默认边框样式 */
	}
	
	.button-container {
	  display: flex;
	  justify-content: center;
	  align-items: center;
	}
	

	
	.button-content {
	  display: flex;
	  flex-direction: column;
	  align-items: center;
	}
	
	
	text {
	  margin-bottom: 20px; /* 文字和图片之间的间距 */
	  font-size: 20rf
	}
	
	image {
	  width: 120px; /* 调整图片的宽度 */
	  height: 120px; /* 调整图片的高度 */
	}

</style>