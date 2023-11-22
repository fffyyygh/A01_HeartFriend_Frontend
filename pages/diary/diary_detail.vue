<template>
	<view>
		<p class="diary-title">{{ diary.title }}</p>
		<p class="diary-date">{{ diary.create_time }}</p>
	    <p class="diary-content">{{ diary.content }}</p>
	</view>
</template>

<script>
	export default {
			data() {
				return {
					diary:{}, // 存储选定的日记数据
				};
			},
			onLoad(query) {
				const diaryId = query.diaryId;
				console.log(diaryId);
				this.fetchDiaryDetails(diaryId);
			},
			methods: {
				fetchDiaryDetails(diaryId) {
					// 使用 diaryId 获取特定日记条目的详细信息
					// 向后端 API 发送请求，获取基于 diaryId 的日记详情
					// 使用获取的日记详情更新 this.diary
					uni.request({
						url: 'http://82.157.244.44:8000/api/v1/diary/', // 后端接口地址
						method: 'GET',
						header: {
							'Authorization': `Bearer ${uni.getStorageSync('token')}`,
						},
						success: (res) => {
							console.log('数据接收成功:', res.data);
							console.log(diaryId);
							const diaries = res.data; // 将获取的日记信息存储到diaries数组中
							//this.diary = res.data.find(a => a.id === diaryId);
							this.diary=diaries[1];
							console.log(this.diary)
						},
						fail: (err) => {
							console.error('数据发送失败:', err);
						}
					});
				},
			},
		};
</script>

<style>

</style>
