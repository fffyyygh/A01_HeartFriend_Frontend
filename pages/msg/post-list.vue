<template>
	<view class="post-list">
		<view v-for="(post, index) in posts" :key="index" @click="goToPostDetail(post)" class="post-item">
			<!-- 用户信息 -->
			<view class="user-info">
				<!-- 头像 -->
				<image class="user-avatar" :src="getUserAvatar(post.author_uuid)" mode="aspectFill"></image>
				<!-- 用户名、发帖时间等信息 -->
				<view>
					<text class="user-name">{{ post.author }}</text>
					<text class="post-time">{{ formatPostTime(post.created_at) }}</text>
				</view>
			</view>

			<!-- 帖子内容 -->
			<view class="post-content">
				<text class="post-title">{{ post.title }}</text>
				<!-- 前三张图片 -->
				<view class="post-images">
					<image v-for="(image, imageIndex) in post.imagesArray.slice(0, 3)" :key="imageIndex" :src="image"
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
				<view class="action-item" @click="commentPost(post)">
					<text class="iconfont icon-pinglun"></text>
					<text class="action-count">{{ post.comments_count }}</text>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				posts: [],
			};
		},
		mounted() {
			this.getPosts();
		},
		methods: {
			async getPosts() {
				try {
					const response = await uni.request({
						url: 'http://82.157.244.44:8000/api/v1/forum/posts/',
						method: 'GET',
						header: {
							'Authorization': 'Bearer ' + uni.getStorageSync('token'),
						},
					});

					if (Array.isArray(response) && response[1] && Array.isArray(response[1].data)) {
						this.posts = response[1].data.map(post => ({
							...post,
							imagesArray: post.images.split(','),
						}));
					} else {
						console.error('Invalid response format:', response);
						console.log('response[1]:', response[1]);
					}
				} catch (error) {
					console.error('Failed to fetch posts', error);
				}
			},
			async getUserInfo(uuid) {
				try {
					const response = await uni.request({
						url: `http://82.157.244.44:8000/api/v1/user/query-info/?uuid=${uuid}`,
						method: 'GET',
						header: {
							'Authorization': 'Bearer ' + uni.getStorageSync('token'),
						},
					});

					if (response[1] && response[1].data) {
						return response[1].data;
					} else {
						console.error('Invalid user info response format:', response);
						return null;
					}
				} catch (error) {
					console.error('Failed to fetch user info', error);
					return null;
				}
			},
			async getUserAvatar(uuid) {
				const userInfo = await this.getUserInfo(uuid);
				console.log('userInfo:', userInfo);
				this.avatarUrl = userInfo ? `http://82.157.244.44:8000${userInfo.avatar_url}` :
					''; // Adjust the URL as needed
				console.log('this.avatarUrl:', this.avatarUrl);
				return this.avatarUrl;
			},
			formatPostTime(time) {
			    const dateTime = new Date(time);
			    const year = dateTime.getFullYear();
			    const month = String(dateTime.getMonth() + 1).padStart(2, '0');
			    const day = String(dateTime.getDate()).padStart(2, '0');
			    const hours = String(dateTime.getHours()).padStart(2, '0');
			    const minutes = String(dateTime.getMinutes()).padStart(2, '0');
			    const seconds = String(dateTime.getSeconds()).padStart(2, '0');
			
			    const formattedTime = `${year}-${month}-${day} ${hours}:${minutes}:${seconds}`;
			    return formattedTime;
			},
			// 模仿的点赞、点踩、评论操作
			likePost(post) {
				console.log('Liked post:', post);
			},
			dislikePost(post) {
				console.log('Disliked post:', post);
			},
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