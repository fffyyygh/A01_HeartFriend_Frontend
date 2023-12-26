<template>
	<view class="diary-container">
		<view class="diary-item">
			<view class="item-row">
				<text class="item-label">药名：</text>
				<text class="item-value">{{ medicine.name }}</text>
			</view>
			<view class="item-row">
				<text class="item-label">数量：</text>
				<text class="item-value">{{ medicine.amount }}{{ medicine.unit }}</text>
			</view>
			<view class="item-row">
				<text class="item-label">开始日期：</text>
				<text class="item-value">{{ medicine.start_date }}</text>
				
			</view>
			<view class="item-row">
				<text class="item-label">结束日期：</text>
				<text class="item-value">{{ medicine.finish_date }}</text>
			</view>
			<view class="item-row">
				<text class="item-label">下次取药日期：</text>
				<text class="item-value">{{ medicine.next_pick_date }}</text>
			</view>
			<view class="item-row">
				<text class="item-label">服药时间：</text>
				
			</view>
			
			
				<view v-for="(time, index) in medicine.select_time" :key="index" class="item-row">
				            <view class="item-value">{{ time }}</view>
							
				</view>
			
			
			

			<view class="item-row">
				<text class="item-label">备注：</text>
				<text class="item-value">{{ medicine.note }}</text>
			</view>
		</view>
		<button @click="change">更改</button>
	</view>
</template>

<script>

	export default {
		data() {
			return {
				medicine: {},

			}
		},
		methods: {
			onLoad(query) {
				const medicineId = query.medicineId;
				console.log(medicineId);
				this.fetchMedicineDetails(medicineId);
			},
			fetchMedicineDetails(medicineId) {
				uni.request({
					url: 'https://vx.mikumikumi.xyz/api/v1/medicine/', // 后端接口地址
					method: 'GET',
					header: {
						'Authorization': `Bearer ${uni.getStorageSync('token')}`,
					},
					success: (res) => {
						console.log('数据接收成功:', res.data);
						this.medicine = res.data.find(a => a.id === Number(medicineId));
			
					},
					fail: (err) => {
						console.error('数据发送失败:', err);
					}
				});
			},
			change() {
				uni.navigateTo({
					url: `/pages/medicine/medicine_change?medicineId=${this.medicine.id}`
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
		border: 1px solid #fefbe0;
		padding: 10px;
		margin-bottom: 10px;
		background-color: #fefbe0;
	}

	.item-row {
		display: flex;
		align-items: center;
		margin-bottom: 5px;
	}

	.item-label {
		font-weight: bold;
		margin-right: 5px;
		font-size: 16px;
	}

	.item-value {
		flex: 1;
		font-size: 16px;
	}

	button {
		margin-top: 20px;
		padding: 10px 20px;
		background-color: #406B43;
		color: white;
		border: none;
		border-radius: 5px;
		cursor: pointer;
	}
</style>