<template>
	<view>
		<div v-for="(medicine, index) in medicines" :key="index" class="diary-item">
			<div @click="showDiaryContent(diary)" class="diary-content" @longpress="showDeleteDiaryButton(diary)">
				<p class="diary-title">{{ medicine.name }}</p>
				<p class="diary-date">服药周期：{{ medicine.start_date }}---{{ medicine.finish_date }}</p>
				<p class="diary-date">取药时间：{{ medicine.next_pick_date }}</p>
			</div>
		</div>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				medicines:[
					{
						amount: "1",
						finish_date: "2023-12-21",
						name: "药名",
						next_pick_date: "2023-12-13",
						note: "asd",
						select_time: (2) ["09:01", "11:02"],
						start_date: "2023-12-06",
						unit: "瓶",
					},
					{
						amount: "1",
						finish_date: "2023-12-21",
						name: "药名",
						next_pick_date: "2023-12-13",
						note: "asd",
						select_time: (2) ["09:01", "11:02"],
						start_date: "2023-12-06",
						unit: "瓶",
					}
				],
			}
		},
		methods: {
			
			get_all_medicine() {
				uni.request({
					url: 'http://82.157.244.44:8000/api/v1/diary/', // 后端接口地址
					method: 'GET',
					header: {
						'Authorization': `Bearer ${uni.getStorageSync('token')}`,
					},
					success: (res) => {
						console.log('数据接收成功:', res.data);
						this.medicines = res.data; // 将获取的日记信息存储到diaries数组中
					},
					fail: (err) => {
						console.error('数据发送失败:', err);
					}
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
