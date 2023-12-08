<template>
	<view class="post-list">
		<view v-for="(post, index) in posts" :key="index" class="post-item">
			<!-- 用户信息 -->
			<view class="user-info" @click="goToPostDetail(post)">
				<!-- 头像 -->
				<image class="user-avatar" :src="users[index].avatar_url" mode="aspectFill"></image>
				<!-- 用户名、发帖时间等信息 -->
				<view>
					<text class="user-name">{{ post.author }}</text>
					<text class="post-time">{{ formatPostTime(post.created_at) }}</text>
				</view>
			</view>

			<!-- 帖子内容 -->
			<view class="post-content" @click="goToPostDetail(post)">
				<text class="post-title">{{ post.title }}</text>
				<!-- 前三张图片 -->
				<view class="post-images">
					<image v-for="(image, imageIndex) in post.images.slice(0, 3)" :key="imageIndex" :src="image"
						mode="aspectFill" class="post-image"></image>
				</view>
			</view>

			<!-- 点赞、点踩、评论图标和数量 -->
			<view class="post-actions">
				<view class="action-item" @click="likePost(post, index)">
					<text class="iconfont icon-dianzan"></text>
					<text class="action-count">{{ post.likes_count }}</text>
				</view>
				<view class="action-item" @click="dislikePost(post, index)">
					<text class="iconfont icon-cai"></text>
					<text class="action-count">{{ post.dislikes_count }}</text>
				</view>
				<view class="action-item" @click="commentPost(post)">
					<text class="iconfont icon-pinglun"></text>
					<text class="action-count">{{ post.comments_count }}</text>
				</view>
			</view>
		</view>
		<!-- 发贴入口 -->
		<add-post-tag></add-post-tag>
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
				posts: [],
				users: [],
				isLiked: [],
				isDisliked: [],
			};
		},
		// onShow() {
		// 	this.get_all_post();
		// 	this.getLikeDislikeStatus();
		// },
		onShow() {
			this.loadPostData();
		},



		methods: {
			async loadPostData() {
				await this.get_all_post();
				this.getLikeDislikeStatus();
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

			async getUsers() {
				try {
					for (const post of this.posts) {
						const response = await this.makeRequest(
							'http://82.157.244.44:8000/api/v1/user/query-info/?uuid=' + post.author_uuid,
							'GET', {
								'Authorization': `Bearer ${uni.getStorageSync('token')}`,
							}
						);

						console.log('数据接收成功:', response);
						const user = response;
						user.avatar_url = "http://82.157.244.44:8000" + user.avatar_url;
						this.users.push(user);
					}
				} catch (error) {
					console.error('数据发送失败:', error);
				}
			},

			async get_all_post() {
				console.log('get_all_post() is called');
				this.users = [];
				this.posts = [];

				// 包装 uni.request 在 Promise 中
				return new Promise((resolve, reject) => {
					uni.request({
						url: 'http://82.157.244.44:8000/api/v1/forum/posts/',
						method: 'GET',
						header: {
							'Authorization': `Bearer ${uni.getStorageSync('token')}`,
						},
						success: (res) => {
							this.posts = res.data;
							this.posts.forEach((post, index) => {
								const image_addr = post.images;
								post.images = image_addr.split(',');
							});
							this.getUsers();
							resolve(); // 请求成功时调用 resolve
						},
						fail: (err) => {
							console.error('数据发送失败:', err);
							reject(err); // 请求失败时调用 reject
						}
					});
				});
			},


			formatPostTime(time) {
				// 检查 time 是否为 undefined 或者 null
				if (time == null) {
					return ''; // 或者你想要返回的默认时间字符串
				}

				// 去掉毫秒部分和末尾的 "Z"
				const timeWithoutMilliseconds = time.replace(/\.\d+Z$/, '');

				// 将 "T" 替换为空格，确保日期字符串格式是 "yyyy-MM-dd HH:mm:ss"
				const formattedTime = timeWithoutMilliseconds.replace('T', ' ');

				return formattedTime;
			},

			// async getLikeDislikeStatus() {
			// 	try {
			// 		console.log('getLikeDislikeStatus is called');
			// 		console.log('Posts in getLikeDislikeStatus:', this.posts);
			// 		for (const post of this.posts) {
			// 			console.log('every post::::',post);
			// 			console.log('every post.isliked::::',post.is_liked);
			// 			console.log('Fetching like and dislike status for post:', post.id);
			// 			const likeStatus = await this.fetchLikeStatus(post.id);
			// 			const dislikeStatus = await this.fetchDislikeStatus(post.id);
			// 			console.log('Like status:', likeStatus);
			// 			console.log('Dislike status:', dislikeStatus);
			// 			this.isLiked.push(likeStatus.isLiked);
			// 			this.isDisliked.push(dislikeStatus.isDisliked);
			// 		}
			// 	} catch (error) {
			// 		console.error('Failed to fetch like/dislike status:', error);
			// 	}
			// },

			async getLikeDislikeStatus() {
				try {
					console.log('getLikeDislikeStatus is called');
					console.log('Posts in getLikeDislikeStatus:', this.posts);
					for (const post of this.posts) {
						console.log('every post::::',post);
						console.log('every post.is_liked::::',post.is_liked);
						console.log('every post.is_disliked::::',post.is_disliked);
						this.isLiked.push(post.is_liked);
						this.isDisliked.push(post.is_disliked);
					}
				} catch (error) {
					console.error('Failed to fetch like/dislike status:', error);
				}
			},

			// async fetchLikeStatus(postId) {
			// 	const url = `http://82.157.244.44:8000/api/v1/forum/posts/${postId}`;
			// 	const method = 'GET';
			// 	const header = {
			// 		'Authorization': `Bearer ${uni.getStorageSync('token')}`,
			// 	};
			// 	try {
			// 		const response = await this.makeRequest(url, method, header);
			// 		console.log('Fetched like status for post', postId, ':', response);
			// 		console.log('response.is_liked', response.is_liked);
			// 		return {
			// 			postId: postId,
			// 			isLiked: response.is_liked,
			// 		};
			// 	} catch (error) {
			// 		console.error('Failed to fetch like status for post', postId, ':', error);
			// 		return {
			// 			postId: postId,
			// 			isLiked: false,
			// 		};
			// 	}
			// },

			// async fetchDislikeStatus(postId) {
			// 	const url = `http://82.157.244.44:8000/api/v1/forum/posts/${postId}`;
			// 	const method = 'GET';
			// 	const header = {
			// 		'Authorization': `Bearer ${uni.getStorageSync('token')}`,
			// 	};
			// 	try {
			// 		const response = await this.makeRequest(url, method, header);
			// 		console.log('Fetched dislike status for post', postId, ':', response);
			// 		return {
			// 			postId: postId,
			// 			isDisliked: response.is_disliked,
			// 		};
			// 	} catch (error) {
			// 		console.error('Failed to fetch dislike status for post', postId, ':', error);
			// 		return {
			// 			postId: postId,
			// 			isDisliked: false,
			// 		};
			// 	}
			// },






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




			// 模仿的点赞、点踩、评论操作
			// likePost(post) {
			// 	console.log('Liked post:', post);
			// },
			// dislikePost(post) {
			// 	console.log('Disliked post:', post);
			// },
			commentPost(post) {
				console.log('Commented on post:', post);
			},
			goToPostDetail(post) {
				console.log('Navigating to post detail:', post);
				console.log('传递的id:', post.id);
				// 使用 uni.navigateTo 进行页面跳转，并通过 query 参数传递帖子的ID
				uni.navigateTo({
					url: '/pages/msg/post_detail?id=' + post.id,
				});
			},
		},
	};
</script>

<style scoped>
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
	}

	.user-avatar {
		width: 40px;
		height: 40px;
		border-radius: 50%;
		margin-right: 10px;
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