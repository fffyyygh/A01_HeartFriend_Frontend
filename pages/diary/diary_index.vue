<template>
	<view class="diary_index">
		<!-- 写日记入口 -->
		<add-diary-tag></add-diary-tag>
		<view class="tip">日记长按可删除</view>
		<div v-for="(diary, index) in diaries" :key="index" class="diary-item">
			<div @click="showDiaryContent(diary)" class="diary-content" @longpress="showDeleteDiaryButton(diary)">
				<p class="diary-date">{{ diary.create_time }}</p>
				<p class="diary-title">{{ diary.title }}</p>
			</div>
		</div>

		<uni-popup ref="popup" type="bottom">
			<view class="popup-content">
				<button class="delete-button" @tap="deleteDiary">删除这篇日记</button>
			</view>
		</uni-popup>
	</view>
</template>

<script>
	import addDiaryTag from '@/components/add-diary-tag/add-diary-tag.vue';
	export default {
		components: {
			addDiaryTag
		},
		onLoad() {
			this.get_all_diary();
		},
		data() {
			return {
				diaries: [], //存储所有的日记
				deleteDiaryId: null
			}
		},
		methods: {
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
			},

			showDeleteDiaryButton(diary) {
				// 点击图片时展示删除按钮
				console.log("s");
				this.deleteDiaryId = diary.id;
				this.$refs.popup.open();
				console.log(this.deleteDiaryId);

			},
			deleteDiary() {

				if (this.deleteDiaryId !== null) {

					uni.request({
						url: 'http://82.157.244.44:8000/api/v1/diary/' + String(this.deleteDiaryId), // 后端接口地址
						method: 'DELETE',
						header: {
							'Authorization': `Bearer ${uni.getStorageSync('token')}`,
						},
						success: (res) => {
							console.log("删除成功");
							this.get_all_diary();
						},
						fail: (err) => {
							console.error('数据发送失败:', err);
						}
					});


					this.deleteDiaryId = null; // 重置删除索引
					this.$refs.popup.close();
				}


			}

		}
	}
</script>

<style scoped>
	.tip {
		color: #888;
		/* Set the font color to gray */
		text-align: left;
		margin-bottom: -20rpx;
		font-size: 25rpx;
	}

	.diary-item {
		margin-top: 30rpx;
		margin-left: 10rpx;
		margin-right: 10rpx;
		border-radius: 10px;
		/* Rounded corners for a softer look */
		background: #fff;
		/* White background */
		box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
		/* Subtle box shadow for depth */
	}

	.diary-content {
		padding: 16px;
		/* Increase padding for better spacing */
		border-radius: 10px;
		background-color: #fefbe0;
		transition: background-color 0.3s ease;
		/* Add a smooth transition effect */
	}

	.diary-title {
		font-size: 18px;
		font-weight: bold;
		margin-bottom: 8px;
		/* Adjust the spacing between title and date */
	}

	.diary-date {
		color: #888;
		font-size: 14px;
	}
</style>