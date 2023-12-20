<template>
	<view>
		<view >
		<button  @click="all_report">
			<view class="button-content">
			<text>所有举报信息</text>
			</view>
		</button>
		
		<button @click="handel_report">
		<view class="button-content">
		       <text>已处理举报</text>
		</view></button>
		
		<button @click="not_handel_report">
		<view class="button-content">
		       <text>未处理举报</text>
		</view></button>
		</view>
		
	</view>
</template>

<script>
	export default {
		data() {
			return{
				
			};
		},
		methods: {
			
			all_report(){
				uni.navigateTo({
					url:"report_all"
				})
			},
			handel_report(){
				uni.navigateTo({
					url:"report_handel"
				})
			},
			not_handel_report(){
				uni.navigateTo({
					url:"report_unhandel"
				})
			},
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

</style>