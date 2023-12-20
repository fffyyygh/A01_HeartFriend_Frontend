<template>
	<view>
		<view  class="report-details">
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
		    </view>
		
		    <button class="post-btn" @click="toPost">查看被举报贴子详情</button>
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
			toPost(){
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
					url: 'http://82.157.244.44:8000/api/v1/forum/reports/manage/all/',
					method: 'GET',
					header: {
						'Authorization': `Bearer ${uni.getStorageSync('token')}`,
					},
					success: (res) => {
						console.log('res:::', res);
						this.report = res.data.data[1];
						console.log('this.report:::', this.report);
						this.reportLoaded = true; // 数据加载完成后设置为 true
					},
					fail: (err) => {
						console.log('数据加载失败');
					},
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

.report-details {
  padding: 20px;
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
  width: 100%;
  max-width: 300px;
  margin: 20px auto;
  padding: 12px 20px;
  border-radius: 5px;
  background-color: #3498db;
  color: #fff;
  text-align: center;
  font-size: 16px;
  font-weight: bold;
  text-decoration: none;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  transition: transform 0.3s ease, background-color 0.3s ease, box-shadow 0.3s ease;
}

</style>