<template>
	<view>
		<view class="report-form">
			<view class="form-group">
				<text class="label">举报类型：</text>
				<picker v-model="reportType" mode="selector" :range="reportTypeszh" @change="handleTypeChange">
					<view class="picker">
						{{ reportTypeText }}
					</view>
				</picker>
			</view>

			<view class="form-group">
				<text class="label">举报详情：</text>
				<textarea v-model="details" placeholder="请输入举报详情" maxlength="-1"></textarea>
			</view>

			<button class="submit-button" @tap="submitReport">提交举报</button>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				postID: null,
				reportType: 0, // 默认选择第一个类型
				details: '',
				reportTypes: ['spam', 'abuse', 'offensive', 'false_info', 'illegal_info', 'other'],
				reportTypeszh: ['垃圾信息', '滥用或骚扰', '冒犯性内容', '虚假信息', '违法活动', '其他'],
			};
		},
		onLoad(query) {
			this.postID = query.id;
		},
		computed: {
			reportTypeText() {
				return this.reportTypeszh[this.reportType];
			},
		},
		methods: {
			handleTypeChange(event) {
				// 更新选择的举报类型
				this.reportType = event.mp.detail.value;
			},
			async submitReport() {
				// 检查举报详情是否为空
				if (!this.details.trim()) {
					uni.showToast({
						title: '举报信息不能为空',
						icon: 'none',
						duration: 2000,
					});
					return; // 阻止继续执行后续的提交逻辑
				}

				try {
					const response = await uni.request({
						url: 'https://vx.mikumikumi.xyz/api/v1/forum/reports/',
						method: 'POST',
						header: {
							'Authorization': `Bearer ${uni.getStorageSync('token')}`,
						},
						data: {
							post: this.postID,
							report_type: this.reportTypes[this.reportType],
							details: this.details,
						},
					});

					if (response[1].statusCode === 201) {
						uni.showModal({
							title: '举报成功',
							content: '感谢您的举报，我们会尽快处理。',
							showCancel: false, // 不显示取消按钮
							success: () => {
								uni.navigateBack(); // 或者 uni.navigateTo({url: '你的目标页面路径'});
							},
						});
					} else {
						console.error('举报失败:', response[1].data);
					}
				} catch (error) {
					console.error('举报失败:', error);
				}
			},
		},
	};
</script>

<style lang="scss">
	.report-form {
		margin-top: 20px;
		padding: 20px;
		display: flex; // 使用 Flex 布局
		flex-direction: column; // 垂直方向布局
	}

	.form-group {
		margin-bottom: 20px;
		align-items: center; // 垂直居中
	}

	.label {
		font-size: 16px;
		margin-right: 10px;
	}

	.picker,
	textarea,
	.submit-button {
		width: calc(100% - 20px); // 考虑到左右各边距 10px
		margin-top: 5px;
	}

	.picker {
		display: inline-block;
		padding: 10px;
		border: 1px solid #007BFF;
		border-radius: 5px;
		color: #007BFF;
		background-color: #fff;
		box-sizing: border-box;
	}

	textarea {
		height: 300px;
		padding: 10px;
		border: 1px solid #ccc;
		border-radius: 5px;
		box-sizing: border-box;
	}

	.submit-button {
		padding: 10px;
		background-color: #007BFF;
		color: #fff;
		border: none;
		border-radius: 5px;
		cursor: pointer;
		margin-left: 0; // 确保左边无额外间距
	}
</style>