<template>
	<view>
		<!-- <button @click="get_report">获取举报信息</button> -->

		<!-- 使用 v-if 来检查是否已经加载数据 -->
		<view v-if="reportLoaded && report.length > 0">
			<view v-for="(item, index) in report" :key="index" @click="goToDetailPage(item.id)" class="report-item">
				<text class="item-title">举报时间：{{ formatReportTime(item.created_at) }}</text>
				<text class="item-status">{{ item.report_status }}</text>
			</view>
		</view>

		<!-- 数据加载中的提示 -->
		<view v-else-if="!reportLoaded">
			<text>数据加载中...</text>
		</view>

		<!-- 没有数据的提示 -->
		<view v-else>
			<text>暂无数据</text>
		</view>
	</view>
</template>

<script>
	import reportVue from '../msg/report.vue';

	export default {
		data() {
			return {
				report: [],
				reportLoaded: false, // 用于标记数据是否已加载
			};
		},
		onShow() {
			this.get_report(); // 在页面每次显示时执行 get_report()
		},
		methods: {
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
					url: 'http://82.157.244.44:8000/api/v1/forum/reports/user/',
					method: 'GET',
					header: {
						'Authorization': `Bearer ${uni.getStorageSync('token')}`,
					},
					success: (res) => {
						console.log('res:::', res);
						this.report = res.data.data;
						console.log('this.report:::', this.report);
						this.reportLoaded = true; // 数据加载完成后设置为 true
					},
					fail: (err) => {
						console.log('数据加载失败');
					},
				});
			},
			goToDetailPage(reportId) {
				console.log('reportId:', reportId);
				uni.navigateTo({
					url: `/pages/my/report_detail_user?reportId=${reportId}`,
				});
			},
		},
	};
</script>

<style>
.report-item {
  border-radius: 8px; /* Rounded corners */
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1); /* Shadow effect */
  padding: 15px;
  margin-bottom: 15px; /* Spacing between items */
  background-color: #ffffff; /* Background color */
  transition: transform 0.3s ease-in-out; /* Smooth transition */
  margin-left: 10px;
  margin-right: 10px;
}

.report-item:hover {
  transform: translateY(-3px); /* Hover effect: Slightly lift the box on hover */
}

.item-title {
  font-size: 16px;
  color: #333; /* Text color */
}

.item-status {
	margin-left: 30px;
  font-size: 14px;
  color: #888;
}

</style>