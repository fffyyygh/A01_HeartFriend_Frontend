<template>
	<view>
		<view class="report-details">
			<view class="report-item">
				<text class="item-title">举报时间：</text>
				<text>{{ formatReportTime(report.created_at) }}</text>
			</view>
			<view class="report-item">
				<text class="item-title">详情：</text>
				<text>{{ report.details }}</text>
			</view>
			<view class="report-item">
				<text class="item-title">状态：</text>
				<text>{{ report.report_status }}</text>
			</view>
			<view class="report-item">
				<text class="item-title">举报类型：</text>
				<text>{{ report.report_type }}</text>
			</view>
			<view class="report-item" v-show="!canHandleReport">
				<text class="item-title">处理详情：</text>
				<text>{{ report.resolution_details }}</text>
			</view>
			<view class="report-item" v-show="!canHandleReport">
				<text class="item-title">处理时间：</text>
				<text>{{ formatReportTime(report.resolved_at) }}</text>
			</view>
			<picker v-if="canHandleReport" v-model="selectedStatus" :range="reportStatusOptions"
				@change="handleStatusChange">
				<view class="picker">
					{{ selectedStatus }}
				</view>
			</picker>
			<textarea v-if="canHandleReport" v-model="resolutionDetails" placeholder="处理详情"
				class="resolution-input" maxlength="-1"></textarea>
			<button class="post-btn" @click="toPost">查看被举报贴子详情</button>
			<button class="handle-btn" v-if="canHandleReport" @click="handleReport">处理举报</button>
		</view>
	</view>
</template>

<script>
	import reportVue from '../msg/report.vue';

	export default {
		data() {
			return {
				report: {},
				reportLoaded: false, // 用于标记数据是否已加载
				report_id: "",
				canHandleReport: false, // Flag to determine if the report can be handled
				selectedStatus: "pending", // Default status
				resolutionDetails: "", // Input for resolution_details
				reportStatusOptions: ["pending", "accepted", "rejected"], // Options for report status
			};
		},
		onLoad(query) {
			this.report_id = query.reportId;
			console.log(this.report_id);
		},
		onShow() {
			this.get_report(); // 在页面每次显示时执行 get_report()
		},
		methods: {
			handleStatusChange(e) {
				this.selectedStatus = this.reportStatusOptions[e.detail.value];
			},
			toPost() {
				uni.navigateTo({
					url: '/pages/msg/post_detail?id=' + this.report.post,
				});
			},
			formatReportTime(time) {
				// 检查 time 是否为 undefined 或者 null
				if (time == null) {
					return ''; // 或者你想要返回的默认时间字符串
				}
				const newTime = time.replace(/\.\d+Z$/, '');
				// 将时间字符串转换为 Date 对象
				const dateObj = new Date(newTime);

				// 添加8个小时
				dateObj.setHours(dateObj.getHours() + 8);
				// console.log('dateObj:::', dateObj);
				// 使用 toLocaleString 来格式化日期，设置参数为24小时制
				const options = {
					hour12: false
				};
				// 获取格式化后的日期字符串
				const formattedTime = dateObj.toLocaleString(undefined, options).replace('T', ' ').replace(/\.\d+Z$/, '');
				// console.log('formattedTime:::', formattedTime);
				return formattedTime;
			},
			get_report() {
				uni.request({
					url: `https://vx.mikumikumi.xyz/api/v1/forum/reports/${this.report_id}/`,
					method: 'GET',
					header: {
						'Authorization': `Bearer ${uni.getStorageSync('token')}`,
					},
					success: (res) => {
						console.log('res:::', res);
						this.report = res.data;
						console.log('this.report:::', this.report);
						this.canHandleReport = this.report.report_status === "pending";
						this.reportLoaded = true; // 数据加载完成后设置为 true
					},
					fail: (err) => {
						console.log('数据加载失败');
					},
				});
			},
			handleReport() {
				// Validate if resolutionDetails is not empty
				if (this.resolutionDetails.trim() === "") {
					uni.showToast({
						title: '处理详情不能为空',
						icon: 'none',
					});
					return;
				}
				const reportId = this.report_id; // Use the actual report ID
				// Send PUT request to handle the report
				uni.request({
					url: `https://vx.mikumikumi.xyz/api/v1/forum/reports/manage/${reportId}/`,
					method: 'PUT',
					header: {
						'Authorization': `Bearer ${uni.getStorageSync('token')}`,
					},
					data: {
						resolution_details: this.resolutionDetails,
						report_status: this.selectedStatus,
					},
					success: (res) => {
						console.log('Report handled successfully:', res);
						uni.showToast({
							title: '举报处理成功',
							icon: 'success',
						});
						uni.navigateBack();
					},
					fail: (err) => {
						console.log('举报处理失败:', err);
					},
				});
			},
		},
	};
</script>

<style>
	.picker {
		border-radius: 8px;
		box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
		margin-left: 10px;
		margin-right: 10px;
		border: 1px solid #ccc;
		border-radius: 5px;
		padding: 12px;
		box-sizing: border-box;
		font-size: 16px;
		color: forestgreen;
	}

	.resolution-input {
		border-radius: 8px;
		box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
		margin-left: 10px;
		margin-right: 10px;
		width: calc(100% - 20px);
		height: 200rpx;
		padding: 12px;
		margin-top: 10px;
		border: 1px solid #ccc;
		border-radius: 5px;
		box-sizing: border-box;
		font-size: 16px;
		resize: none;
	}

	.report-item {
		border-radius: 8px;
		/* Rounded corners */
		box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
		/* Shadow effect */
		padding: 15px;
		margin-bottom: 15px;
		/* Spacing between items */
		background-color: #ffffff;
		/* Background color */
		transition: transform 0.3s ease-in-out;
		/* Smooth transition */
		margin-left: 10px;
		margin-right: 10px;
	}

	.report-details {
		padding: 20px;
	}

	.report-item:hover {
		transform: translateY(-3px);
		/* Hover effect: Slightly lift the box on hover */
	}

	.item-title {
		font-size: 16px;
		color: #333;
		/* Text color */
	}

	.item-status {
		margin-left: 30px;
		font-size: 14px;
		color: #888;
	}

	.item-title {
		font-weight: bold;
		margin-right: 5px;
	}

	.item-title,
	.item-content {
		color: #333;
	}

	.item-content {
		flex: 1;
		word-break: break-word;
	}

	.post-btn {
		display: block;
		width: calc(100% - 20px);
		/* 调整为所需的宽度值，确保左右边距一致 */
		max-width: 300px;
		margin-top: 30rpx;
		padding: 12px 20px;
		border-radius: 5px;
		background-color: #527853;
		color: #fff;
		text-align: center;
		font-size: 16px;
		font-weight: bold;
		text-decoration: none;
		box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
		transition: transform 0.3s ease, background-color 0.3s ease, box-shadow 0.3s ease;
	}

	.handle-btn {
		display: block;
		width: calc(100% - 20px);
		/* 调整为所需的宽度值，确保左右边距一致 */
		max-width: 300px;
		margin: 20px auto;
		padding: 12px 20px;
		border-radius: 5px;
		background-color: #527853;
		color: #fff;
		text-align: center;
		font-size: 16px;
		font-weight: bold;
		text-decoration: none;
		box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
		transition: transform 0.3s ease, background-color 0.3s ease, box-shadow 0.3s ease;
	}
</style>