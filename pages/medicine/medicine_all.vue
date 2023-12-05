<template>
	<view>
		<div v-for="(medicine, index) in medicines" :key="index" class="diary-item">
			<div @click="showMedicineContent(medicine)" class="diary-content" @longpress="showDeleteMedicineButton(medicine)">
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
				deleteMedicineId:"",
				medicines:[],
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
			deleteMedicine(){
				if (this.deleteDiaryId !== null) {
					
					uni.request({
						url: 'http://82.157.244.44:8000/api/v1/medicine/' + String(this.deleteMedicineId), // 后端接口地址
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
		margin-bottom: 20px;
		margin-left: 10rpx;
		margin-right: 10rpx;
		/* 添加间距，使日记组件之间有一定的空隙 */
		border-radius: 44px;
		background: linear-gradient(145deg, #f0f0f0, #cacaca);
		box-shadow:  5px 5px 5px #797979,
		             -5px -5px 5px #ffffff;
	}

	.diary-content {
		border: 1px solid #ccc;
		/* 边框样式 */
		padding: 10px;
		/* 内边距，增加内容与边框之间的间隔 */
		border-radius: 5px;
		/* 边框圆角 */
		cursor: pointer;
		/* 鼠标指针样式为指示可点击 */
		background-color: #F0FFFF;
	}



	.diary-title {
		font-size: 15px;
		/* 标题字体大小 */
		font-weight: bold;
		/* 标题粗体 */
		margin-bottom: 5px;
		/* 标题与日期之间的间距 */
	}

	.diary-date {
		color: #666;
		/* 日期文字颜色 */
		font-size: 14px;
		/* 日期字体大小 */
	}

</style>
