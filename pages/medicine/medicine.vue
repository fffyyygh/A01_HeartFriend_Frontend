<template>
	<view>
		<!-- 触发弹出输入框的按钮 -->
		<u-button @click="openForm" hair-line="false" :ripple="true" ripple-bg-color="#e86158"
			:custom-style="btnStyle">添加药物信息</u-button>

		<!-- 输入表单弹窗 -->
		<view v-if="showForm">
			<uni-section title="名称" type="line" padding>
				<uni-easyinput v-model="medicineName" focus placeholder="请输入药物名称"></uni-easyinput>
			</uni-section>
			<uni-section title="剂量" type="line" subTitle="单位:mg" padding>
				<uni-number-box v-model="medicineQuantity" />
			</uni-section>
			<uni-section title="次数" type="line" padding>
				<uni-number-box v-model="medicineFrequency" />
			</uni-section>
			<uni-section title="时间" type="line" padding>
				<uni-datetime-picker v-model="medicineTime" :mode="mode"
					@confirm="onDatetimeConfirm"></uni-datetime-picker>
			</uni-section>
			<u-button @click="submitForm" hair-line="false" :ripple="true" ripple-bg-color="#e86158"
				:custom-style="btnStyle">提交</u-button>
		</view>

		<!-- 用药记录显示 -->
		<view v-if="medicineRecords.length > 0">
			<uni-table class="medicine-table">
				<uni-tr>
					<uni-th width="150rpx" align="center">名称</uni-th>
					<uni-th width="100rpx" align="center">剂量</uni-th>
					<uni-th width="100rpx" align="center">次数</uni-th>
					<uni-th width="400rpx" align="center">时间</uni-th>
				</uni-tr>
				<uni-tr v-for="(record, index) in medicineRecords" :key="index">
					<uni-td>{{ record.name }}</uni-td>
					<uni-td>{{ record.quantity }}mg</uni-td>
					<uni-td>{{ record.frequency }}次</uni-td>
					<uni-td>{{ record.time }}</uni-td>
				</uni-tr>
			</uni-table>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				btnStyle: {
					color: "#f9f8e5",
					backgroundColor: '#ff8a89',
					marginTop: '20rpx',
					marginRight: '20rpx',
					marginLeft: '20rpx',
				},
				showForm: false, // 控制表单显示隐藏
				medicineName: '', // 药物名称
				medicineQuantity: 50, // 药物数量
				medicineFrequency: 1, // 药物次数
				medicineTime: '', // 用药时间
				mode: 'datetime', // 时间选择器模式
				medicineRecords: [] // 存储用药记录
			};
		},
		methods: {
			// 打开输入表单
			openForm() {
				this.showForm = true;
			},
			// 提交表单数据
			submitForm() {
				// 这里可以根据需求处理提交表单的逻辑，比如将数据保存到某个数组或对象中
				const medicineData = {
					name: this.medicineName,
					quantity: this.medicineQuantity,
					frequency: this.medicineFrequency,
					time: this.medicineTime
				};
				console.log('药物信息:', medicineData);

				// 将用药记录添加到数组中
				this.medicineRecords.push(medicineData);

				// 提交完数据后，隐藏表单
				this.showForm = false;
				// 清空表单数据，方便下一次使用
				// this.medicineName = '';
				// this.medicineQuantity = 50;
				// this.medicineFrequency = 1;
				this.medicineTime = '';
			},
			// 时间选择器确认事件处理
			onDatetimeConfirm(value) {
				this.medicineTime = value;
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
		border-radius: 5px;
		cursor: pointer;
		outline: none;
		/* 取消按钮点击时的默认边框样式 */
	}

	u-button {
		margin-top: 20px;
		background-color: #ff8a89;
	}

	uni-easyinput,
	uni-number-box,
	uni-datetime-picker {
		width: 100%;
	}

	.medicine-table {
		width: 100%;
		border-collapse: collapse;
		margin-top: 20px;
	}

	th,
	td {
		border: 1px solid #ddd;
		padding: 12px;
		text-align: left;
	}

	th {
		background-color: #f2f2f2;
	}
</style>