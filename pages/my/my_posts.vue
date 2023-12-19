<template>
	<view class="post-list">
		<view v-for="(post, index) in posts" :key="index" class="post-item">
			<!-- 用户信息 -->
			<view class="user-info">
				<!-- 头像 -->
				<image class="user-avatar" :src="userInfo.avatar_url" mode="aspectFill"></image>
				<!-- 用户名、发帖时间等信息 -->
				<view class="user-details">
					<text class="user-name">{{ post.author }}</text>
					<text class="post-time">{{ formatPostTime(post.created_at) }}</text>
				</view>
				<button class="delete-button" @click="showDeleteConfirmation(index)">删除</button>
			</view>

			<!-- 帖子内容 -->
			<view class="post-content">
				<text class="post-title" @click="goToPostDetail(post)">{{ post.title }}</text>
				<!-- 前三张图片 -->
				<view class="post-images">
					<image v-for="(image, imageIndex) in post.images.slice(0, 3)" :key="imageIndex" :src="image"
						mode="aspectFill" class="post-image"></image>
				</view>
			</view>

			<!-- 点赞、点踩、评论图标和数量 -->
			<view class="post-actions">
				<view class="action-item" @click="likePost(post)">
					<text class="iconfont icon-dianzan"></text>
					<text class="action-count">{{ post.likes_count }}</text>
				</view>
				<view class="action-item" @click="dislikePost(post)">
					<text class="iconfont icon-cai"></text>
					<text class="action-count">{{ post.dislikes_count }}</text>
				</view>
				<view class="action-item" @click="goToPostDetail(post)">
					<text class="iconfont icon-pinglun"></text>
					<text class="action-count">{{ post.comments_count }}</text>
				</view>
			</view>
			<!-- 发贴入口 -->
			<add-post-tag></add-post-tag>
		</view>
	</view>
</template>

<script>
	import addPostTag from '@/components/add-post-tag/add-post-tag.vue';
	export default {
		components: {
			addPostTag
		},
		data() {
			return {
				userInfo: uni.getStorageSync('userInfo'),
				posts: [],
				users: [],
				newposts: [],
				isLiked: [],
				isDisliked: [],
			};
		},
		onShow() {
			this.get_user_posts();
			console.log(this.userInfo);
		},
		onReachBottom() {
			this.get_other_post();

		},
		methods: {
			showDeleteConfirmation(index) {
				const that = this; // Store the reference to the component

				uni.showModal({
					title: '确认删除',
					content: '确认删除本条帖子？',
					success(res) {
						if (res.confirm) {
							// User clicked the confirm button
							that.deletePost(index); // Call the deletePost method if confirmed
						} else if (res.cancel) {
							// User clicked the cancel button
							console.log('用户点击取消');
						}
					}
				});
			},
			deletePost(index) {
				console.log("a");
				const post = this.posts[index];
				const id = post.id;
				console.log(id);
				uni.request({
					url: 'http://82.157.244.44:8000/api/v1/forum/posts/' + id + "/", // 后端接口地址
					method: 'DELETE',
					header: {
						'Authorization': `Bearer ${uni.getStorageSync('token')}`,
					},
					success: (res) => {
						console.log('数据接收成功:', res.data);
						this.posts = [];
						this.users = [];
						this.get_user_posts();

					},
					fail: (err) => {
						console.error('数据发送失败:', err);
					}
				});

			},

			makeRequest(url, method, header) {
				return new Promise((resolve, reject) => {
					uni.request({
						url: url,
						method: method,
						header: header,
						success: (res) => {
							resolve(res.data);
						},
						fail: (err) => {
							reject(err);
						},
					});
				});
			},
			getFullAvatarUrl(relativeUrl) {
				return `http://82.157.244.44:8000${relativeUrl}`;
			},

			get_user_posts() {
				this.posts = [];
				this.users = [];
				uni.request({
					url: `http://82.157.244.44:8000/api/v1/forum/posts/getUserPosts/?offset=0&limit=20`,
					method: "GET",
					header: {
						'Authorization': `Bearer ${uni.getStorageSync('token')}`,
					},
					data: {
						"uuid": this.userInfo.uuid,
					},
					success: (res) => {
						console.log("获取到了贴子列表", res.data);
						this.posts = res.data.data;
						this.posts.forEach((post, index) => {
							const image_addr = post.images;
							post.images = image_addr.split(',');
						});
						this.getLikeDislikeStatus();
					}

				});
			},

			formatPostTime(time) {
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

			// 模仿的点赞、点踩、评论操作

			goToPostDetail(post) {
				console.log('Navigating to post detail:', post);
				console.log('传递的id:', post.id);
				// 使用 uni.navigateTo 进行页面跳转，并通过 query 参数传递帖子的ID
				uni.navigateTo({
					url: '/pages/msg/post_detail?id=' + post.id,
				});
			},

			async getLikeDislikeStatus() {
				try {
					for (const post of this.posts) {
						this.isLiked.push(post.is_liked);
						this.isDisliked.push(post.is_disliked);
					}
				} catch (error) {
					console.error('Failed to fetch like/dislike status:', error);
				}
			},

			async likePost(post, index) {
				try {
					// 发送点赞请求
					const response = await uni.request({
						url: `http://82.157.244.44:8000/api/v1/forum/posts/${post.id}/like/`,
						method: 'POST',
						header: {
							'Authorization': `Bearer ${uni.getStorageSync('token')}`,
						},
					});

					console.log('response内容', response);

					// 根据后端返回的数据更新点赞状态
					if (response[1].statusCode === 200) {
						console.log('this.isLiked[index]:::', this.isLiked[index]);
						if (!this.isLiked[index]) {
							this.isLiked[index] = true;
							post.likes_count += 1; // 增加点赞数量

							// 如果之前点踩了，取消点踩状态
							if (this.isDisliked[index]) {
								this.isDisliked[index] = false;
								post.dislikes_count -= 1; // 减少点踩数量
							}
						} else {
							// 如果之前点赞了，取消点赞状态
							this.isLiked[index] = false;
							post.likes_count -= 1; // 减少点赞数量
						}
					} else {
						console.error('点赞失败:', response[1].data);
					}
				} catch (error) {
					console.error('点赞失败:', error);
				}
			},

			async dislikePost(post, index) {
				try {
					// 发送点踩请求
					const response = await uni.request({
						url: `http://82.157.244.44:8000/api/v1/forum/posts/${post.id}/dislike/`,
						method: 'POST',
						header: {
							'Authorization': `Bearer ${uni.getStorageSync('token')}`,
						},
					});

					// 根据后端返回的数据更新点踩状态
					if (response[1].statusCode === 200) {
						if (!this.isDisliked[index]) {
							this.isDisliked[index] = true;
							post.dislikes_count += 1; // 增加点踩数量

							// 如果之前点赞了，取消点赞状态
							if (this.isLiked[index]) {
								this.isLiked[index] = false;
								post.likes_count -= 1; // 减少点赞数量
							}
						} else {
							// 如果之前点踩了，取消点踩状态
							this.isDisliked[index] = false;
							post.dislikes_count -= 1; // 减少点踩数量
						}
					} else {
						console.error('点踩失败:', response[1].data);
					}
				} catch (error) {
					console.error('点踩失败:', error);
				}
			},

			async getNewLikeDislikeStatus() {
				try {
					for (const post of this.newposts) {
						this.isLiked.push(post.is_liked);
						this.isDisliked.push(post.is_disliked);
					}
				} catch (error) {
					console.error('Failed to fetch like/dislike status:', error);
				}
			},

			async get_other_post() {
				// 包装 uni.request 在 Promise 中
				return new Promise((resolve, reject) => {
					let a = this.posts.length;
					console.log(a);
					uni.request({
						url: `http://82.157.244.44:8000/api/v1/forum/posts/getUserPosts/?offset=${a}&limit=20`,
						method: 'GET',
						header: {
							'Authorization': `Bearer ${uni.getStorageSync('token')}`,
						},
						data: {
							"uuid": this.userInfo.uuid,
						},
						success: (res) => {
							this.newposts = res.data.data;
							//this.posts = this.posts.slice().reverse();
							this.newposts.forEach((post, index) => {
								const image_addr = post.images;
								post.images = image_addr.split(',');
								this.posts.push(post);
							});

							this.getNewLikeDislikeStatus();
							resolve(); // 请求成功时调用 resolve
						},
						fail: (err) => {
							console.error('数据发送失败:', err);
							reject(err); // 请求失败时调用 reject
						}
					});
				});
			},


		},
	};
</script>

<style scoped>
	.delete-button {
		align-self: flex-start;
		/* 设置按钮在垂直方向上靠上显示 */
		margin-left: 8rpx;
		color: black;
		height: 60rpx;
		width: 120rpx;
		font-size: 25rpx;
		margin-right: 20rpx;
	}
	.post-list {
		padding: 20rpx;
	}

	.post-item {
		background-color: #fff;
		margin-bottom: 20rpx;
		padding: 20rpx;
		border-radius: 10rpx;
		box-shadow: 0 2rpx 4rpx rgba(0, 0, 0, 0.1);
	}

	.user-info {
		display: flex;
		align-items: center;
		justify-content: space-between;
		margin-bottom: 8rpx;
	}

	.user-avatar {
		width: 80rpx;
		height: 80rpx;
		border-radius: 50%;
		margin-right: 8rpx;
	}

	.user-details {
		flex: 1;
		display: flex;
		flex-direction: column;
	}
	
	.user-name {
		font-weight: bold;
	}

	.post-time {
		/* Style for post time */
		color: #888;
	}

	.post-content {
		/* Add styles for post content section */
		margin-top: 10px;
	}

	.post-title {
		/* Style for post title */
		font-weight: bold;
	}

	.post-images {
		/* Add styles for post images section */
		display: flex;
		margin-top: 10px;
	}

	.post-image {
		/* Style for each post image */
		width: 80px;
		height: 80px;
		margin-right: 10px;
	}

	.post-actions {
		/* Add styles for post actions section */
		display: flex;
		margin-top: 10px;
	}

	.action-item {
		/* Style for each action item (like, dislike, comment) */
		display: flex;
		align-items: center;
		margin-right: 20px;
		cursor: pointer;
	}

	.iconfont {
		/* Style for action icons */
		font-size: 24px;
		margin-right: 5px;
	}

	.action-count {
		/* Style for action count (likes, dislikes, comments) */
		color: #888;
	}
</style>