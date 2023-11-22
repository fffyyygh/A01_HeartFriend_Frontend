<template>
	<view>
		<!-- 触发弹出输入框的按钮 -->
		<button @click="openForm">添加药物信息</button>

		<!-- 输入表单弹窗 -->
		<view v-if="showForm">
			<uni-section title="名称" subTitle="使用 focus 属性自动获取输入框焦点" type="line" padding>
				<uni-easyinput v-model="medicineName" focus placeholder="请输入内容"></uni-easyinput>
			</uni-section>
			<uni-section title="剂量" type="line" subTitle="使用 background 属性设置自定义背景色" padding>
				<uni-number-box v-model="medicineQuantity" background="#2979FF" color="#fff" />
			</uni-section>
			<uni-section title="次数" subTitle="使用 disabled 属性设置组件禁用" type="line" padding>
				<uni-number-box v-model="medicineFrequency" :disabled="true" />
			</uni-section>
			<button @click="submitForm">提交</button>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				showForm: false, // 控制表单显示隐藏
				medicineName: '', // 药物名称
				medicineQuantity: 0, // 药物数量
				medicineFrequency: 1 // 药物次数
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
					frequency: this.medicineFrequency
				};
				console.log('药物信息:', medicineData);

				// 提交完数据后，隐藏表单
				this.showForm = false;
				// 清空表单数据，方便下一次使用
				this.medicineName = '';
				this.medicineQuantity = 0;
				this.medicineFrequency = 0;
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

	/* 输入框样式 */

	/* 提交按钮样式 */
	button[type="submit"] {
		background-color: #2ecc71;
	}
</style>