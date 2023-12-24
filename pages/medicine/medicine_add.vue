<template>
	<view>
		<view>
			<uni-section title="药物名称" type="line" padding>
				<uni-easyinput v-model="medicineName" focus placeholder="请输入药物名称"></uni-easyinput>
			</uni-section>
			<uni-section title="剂量" type="line" padding>
				<view class="input-container">
					<input class="input-left" type="number" v-model="quantity" placeholder="输入数量">
					<picker class="input-right" mode="selector" :range="units" @change="unitSelected">
						<view>{{ selectedUnit }}</view>
					</picker>
				</view>
			</uni-section>
			<uni-section title="提醒时间" type="line" padding>
				<view>
					<picker class="time-picker" mode="time" :value="time" start="09:01" end="21:01"
						@change="bindTimeChange">
						选择提醒时间
					</picker>



					<view v-for="(time, index) in selectedTimes" :key="index" class="time">
						<text class="medicine_time">{{ time }}</text>
						<image class="delete_img" @click="deleteTime(index)" src="/static/medicine/叉号.png"></image>
					</view>
				</view>
			</uni-section>
			<uni-section title="服药周期" type="line" padding>
				<uni-datetime-picker v-model="range" type="daterange"></uni-datetime-picker>
			</uni-section>

			<uni-section title="下次取药时间" type="line" padding>
				<uni-datetime-picker v-model="nextPickTime" type="date"></uni-datetime-picker>
			</uni-section>

			<uni-section title="服药备注" type="line" padding>
				<uni-easyinput v-model="note" focus placeholder="请输入服药提醒事项"></uni-easyinput>
			</uni-section>

			<button class="submit_medicine" @click="submitForm">提交</button>
			<view style="height: 60rpx;"></view>

		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				medicineName: "", //药名

				quantity: '', // 左侧输入的数字
				units: ['粒', '瓶', '毫升', '毫克'], // 可选择的单位列表
				selectedUnit: '单位', // 初始显示的单位

				selectedTimes: [],

				range: [],
				nextPickTime: "",

				note: "",

			}
		},
		methods: {

			bindTimeChange: function(e) {
				this.selectedTimes.push(e.detail.value);
			},
			unitSelected(event) {
				const index = event.detail.value;
				this.selectedUnit = this.units[index];
			},

			deleteTime(index) {
				this.selectedTimes.splice(index, 1);
			},
			submitForm() {
				if (this.note === "") {
					this.note = "无";
				}

				const dataToSend = {
					name: this.medicineName, //名称
					amount: this.quantity, // 数量
					unit: this.selectedUnit, //单位
					select_time: this.selectedTimes, //时间         //周期
					start_date: this.range[0],
					finish_date: this.range[1],
					next_pick_date: this.nextPickTime, // 取药时间
					note: this.note, //用药备注					
				};

				const requiredFields = ['name', 'amount', 'unit', 'select_time', "start_date"];
				const text = {
					name: "药物名称",
					amount: "剂量",
					unit: "单位",
					select_time: "用药时间",
					start_date: "用药周期"

				}
				for (const field of requiredFields) {
					if (!dataToSend[field]) {
						uni.showToast({
							title: `请填写${text[field]}`,
							icon: 'none',
							duration: 2000
						});
						return; // 停止提交
					}
				}

				console.log(dataToSend);
				uni.request({
					url: 'http://82.157.244.44:8000/api/v1/medicine/', // 后端接口地址
					method: 'POST',
					header: {
						'Authorization': `Bearer ${uni.getStorageSync('token')}`,
					},
					data: dataToSend,
					success: (res) => {
						console.log('数据发送成功:', res.data);
						console.log(dataToSend);
						//跳转不了 很奇怪  等后端开发好之后试一下;
						uni.switchTab({
							url: "/pages/medicine/medicine"
						})


					},
					fail: (err) => {
						console.error('数据发送失败:', err);
						console.log(dataToSend);
						// uni.redirectTo({
						// 	url:"/pages/medicine/medicine"
						// })
					}
				});
				uni.navigateBack();
			}
		}
	}
</script>

<style>
	.time_picker {
		height: 100rpx;
	}

	.input-container {
		display: flex;
		align-items: center;
		border: 1px solid #ccc;
		border-radius: 5px;
		padding: 5px;
	}

	.input-left {
		flex: 1;
		padding: 10px;
		border: none;
		outline: none;
	}

	.input-right {
		flex-basis: 80px;
		text-align: center;
		color: #333;
	}

	.time-picker {
		width: 80%;
		margin-bottom: 20px;
		border: 1px solid #ccc;
		border-radius: 5px;
		padding: 10px;
		text-align: center;
	}

	.selected-times {
		display: flex;
		flex-direction: column;
		align-items: center;
	}




	.submit_medicine {
		color: white;
		margin-top: 20rpx;
		margin-bottom: 60rpx;
		width: 60%;
		border-radius: 12px;
		background-color: #527853;
		box-shadow: 0 8px 16px 0 rgba(0, 0, 0, 0.2), 0 6px 20px 0 rgba(0, 0, 0, 0.19);

	}

	.time {
		display: flex;
		justify-content: space-between;
		align-items: center;
		margin-bottom: 10px;
		margin-right: 280rpx;
		margin-left: 10rpx;
		border: 2px solid #527853;
		/* 边框样式 */
		border-radius: 5px;
		/* 可以添加圆角 */
	}

	.medicine_time {
		margin-left: 10rpx;
	}



	.delete_img {
		width: 20px;
		height: 20px;
		margin-right: 10rpx;
	}
</style>