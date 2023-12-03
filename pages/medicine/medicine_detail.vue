<template>
	<view class="diary-container">
		<view class="diary-item">
			<view class="item-row">
				<text class="item-label">药名：</text>
				<text class="item-value">{{ diary.name }}</text>
			</view>
			<view class="item-row">
				<text class="item-label">数量：</text>
				<text class="item-value">{{ diary.amount }}{{ diary.unit }}</text>
			</view>
			<view class="item-row">
				<text class="item-label">开始日期：</text>
				<text class="item-value">{{ diary.start_date }}</text>
				
			</view>
			<view class="item-row">
				<text class="item-label">结束日期：</text>
				<text class="item-value">{{ diary.finish_date }}</text>
			</view>
			<view class="item-row">
				<text class="item-label">下次取药日期：</text>
				<text class="item-value">{{ diary.next_pick_date }}</text>
			</view>
			<view class="item-row">
				<text class="item-label">服药时间：</text>
				
			</view>
			
			
				<view v-for="(time, index) in diary.select_time" :key="index" class="item-row">
				            <view class="item-value">{{ time }}</view>
							
				</view>
			
			
			

			<view class="item-row">
				<text class="item-label">备注：</text>
				<text class="item-value">{{ diary.note }}</text>
			</view>
		</view>
		<button @click="change">更改</button>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				diary: {
					amount: "1",
					finish_date: "2023-12-21",
					name: "药名",
					next_pick_date: "2023-12-13",
					note: "在饭后服用",
					select_time:["09:01", "11:02"],
					start_date: "2023-12-06",
					unit: "瓶",
				},

			}
		},
		methods: {
			change() {
				uni.navigateTo({
					url: "/pages/medicine/medicine_change"
				})
			}
		}
	}
</script>

<style>
	/* 样式可以根据需求调整 */
	.diary-container {
		padding: 20px;
		
	}

	.diary-item {
		border: 1px solid #ccc;
		padding: 10px;
		margin-bottom: 10px;
		background-color: azure;
	}

	.item-row {
		display: flex;
		align-items: center;
		margin-bottom: 5px;
	}

	.item-label {
		font-weight: bold;
		margin-right: 5px;
		font-size: 20px;
	}

	.item-value {
		flex: 1;
		font-size: 20px;
	}

	button {
		margin-top: 20px;
		padding: 10px 20px;
		background-color: #007bff;
		color: white;
		border: none;
		border-radius: 5px;
		cursor: pointer;
	}

	button:hover {
		background-color: #0056b3;
	}
</style>