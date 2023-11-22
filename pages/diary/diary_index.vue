<template>
	<view>
		<button @click="write_diary">写个日记</button>

		<div v-for="(diary, index) in diaries" :key="index" class="diary-item">
			<div @click="showDiaryContent(diary)" class="diary-content">
				<p class="diary-title">{{ diary.title }}</p>
				<p class="diary-date">{{ diary.create_time }}</p>
			</div>
		</div>
	</view>
</template>

<script>
export default {
	onLoad() {
		this.get_all_diary();
	},
	data() {
		return {
			diaries: [] //存储所有的日记
		}
	},
	methods: {
		write_diary() {
			uni.navigateTo({
				url: "/pages/diary/diary"
			})
		},
		get_all_diary() {
			uni.request({
				url: 'http://82.157.244.44:8000/api/v1/diary/', // 后端接口地址
				method: 'GET',
				header: {
					'Authorization': `Bearer ${uni.getStorageSync('token')}`,
				},
				success: (res) => {
					console.log('数据接收成功:', res.data);
					this.diaries = res.data; // 将获取的日记信息存储到diaries数组中
				},
				fail: (err) => {
					console.error('数据发送失败:', err);
				}
			});


		},
		showDiaryContent(diary) {

			console.log(diary);
			uni.navigateTo({
				url: `/pages/diary/diary_detail?diaryId=${diary.id}`,
			});
		}
	}
}
</script>

<style>
	.diary-item {
		margin-bottom: 20px;
		/* 添加间距，使日记组件之间有一定的空隙 */
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

	.diary-content:hover {
		background-color: #f9f9f9;
		/* 鼠标悬停时的背景色 */
	}

	.diary-title {
		font-size: 18px;
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