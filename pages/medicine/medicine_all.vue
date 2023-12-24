<template>
	<view>
		<div v-for="(medicine, index) in medicines" :key="index" class="diary-item">
			<div @click="showMedicineContent(medicine)" class="diary-content"
				@longpress="showDeleteMedicineButton(medicine)">
				<p class="diary-title">{{ medicine.name }}</p>
				<p class="diary-date">服药周期：{{ medicine.start_date }}---{{ medicine.finish_date }}</p>
				<p class="diary-date">取药时间：{{ medicine.next_pick_date }}</p>
			</div>
		</div>

		<uni-popup ref="popup" type="bottom">
			<view class="popup-content">
				<button class="delete-button" @tap="deleteMedicine">删除这条提醒</button>
			</view>
		</uni-popup>
	</view>
</template>

<script>
	export default {
		onLoad() {
			this.get_all_medicine();
		},
		data() {
			return {
				deleteMedicineId: "",
				medicines: [],
			}
		},
		methods: {

			get_all_medicine() {
				uni.request({
					url: 'http://82.157.244.44:8000/api/v1/medicine/', // 后端接口地址
					method: 'GET',
					header: {
						'Authorization': `Bearer ${uni.getStorageSync('token')}`,
					},
					success: (res) => {
						console.log('数据接收成功:', res.data);
						this.medicines = res.data; // 将获取的日记信息存储到diaries数组中
						console.log(this.medicines);
					},
					fail: (err) => {
						console.error('数据发送失败:', err);
					}
				});


			},

			showDeleteMedicineButton(medicine) {
				this.deleteMedicineId = medicine.id;
				this.$refs.popup.open();
			},
			deleteMedicine() {
				if (this.deleteDiaryId !== null) {

					uni.request({
						url: 'http://82.157.244.44:8000/api/v1/medicine/' + String(this
						.deleteMedicineId), // 后端接口地址
						method: 'DELETE',
						header: {
							'Authorization': `Bearer ${uni.getStorageSync('token')}`,
						},
						success: (res) => {
							console.log("删除成功");
							this.get_all_medicine();
						},
						fail: (err) => {
							console.error('数据发送失败:', err);
						}
					});
					this.deleteDiaryId = null; // 重置删除索引
					this.$refs.popup.close();
				}
			},
			showMedicineContent(medicine) {
				uni.navigateTo({
					url: `/pages/medicine/medicine_detail?medicineId=${medicine.id}`,
				});
			},


		}
	}
</script>

<style>
	.diary-item {
		margin-top: 30rpx;
		margin-left: 10rpx;
		margin-right: 10rpx;
		border-radius: 10px;
		/* Rounded corners for a softer look */
		background: #fff;
		/* White background */
		box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
		/* Subtle box shadow for depth */
	}

	.diary-content {
		padding: 16px;
		/* Increase padding for better spacing */
		border-radius: 10px;
		background-color: #fefbe0;
		transition: background-color 0.3s ease;
		/* Add a smooth transition effect */
	}

	.diary-title {
		font-size: 18px;
		font-weight: bold;
		margin-bottom: 8px;
		/* Adjust the spacing between title and date */
	}

	.diary-date {
		color: #888;
		font-size: 14px;
	}
</style>