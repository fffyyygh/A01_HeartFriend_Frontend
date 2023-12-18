<template>
	<view class="user-profile">
		<view class="user-info">
			<image class="user-avatar" :src="getFullAvatarUrl(userInfo.avatar_url)" mode="aspectFill"></image>
			<text class="user-name">{{ userInfo.username }}</text>
			<!-- 根据性别显示不同图标 -->
			<image v-if="userInfo.gender === 'male'" src="/static/my/male.png" class="gender-icon"></image>
			<image v-else-if="userInfo.gender === 'female'" src="/static/my/female.png" class="gender-icon"></image>
			<image v-else-if="userInfo.gender === 'other'" src="/static/my/other.png" class="gender-icon"></image>
			<image v-else-if="userInfo.gender === 'secret'" src="/static/my/secret.png" class="gender-icon"></image>
			<!-- 添加关注按钮 -->
			<button :class="{'followed-style': isFollowed, 'unfollowed-style': !isFollowed}"
				@click="follow_click">{{ isFollowed ? '已关注' : '关注' }}</button>
		</view>
		<view class="info-section">
			<view>
				<text class="info-label">年龄：</text>
				<text class="user-age">{{ userInfo.age }}岁</text>
			</view>
			<view>
				<text class="info-label">个人介绍：</text>
				<text class="user-intro">{{ userInfo.self_intro }}</text>
			</view>
		</view>

		<!-- TODO：加一个显示该用户发的所有帖子的列表 -->
		<view v-for="(post, index) in posts" :key="index" class="post-item">
			<!-- 用户信息 -->
			<view class="user-info" @click="goToPostDetail(post)">
				<!-- 头像 -->
				<image class="user-avatar" :src="getFullAvatarUrl(userInfo.avatar_url)" mode="aspectFill"></image>
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
				<view class="action-item" @click="likePost(post, index)" :class="{ 'liked': isLiked[index] }">
					<text class="iconfont icon-dianzan"></text>
					<text class="action-count">{{ post.likes_count }}</text>
				</view>
				<view class="action-item" @click="dislikePost(post, index)" :class="{ 'disliked': isDisliked[index] }">
					<text class="iconfont icon-cai"></text>
					<text class="action-count">{{ post.dislikes_count }}</text>
				</view>
				<view class="action-item" @click="goToPostDetail(post)">
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
				userInfo: {},
				uuid: '',
				isFollowed: false,
				posts: [],
				isLiked: [],
				isDisliked: [],
				newposts: [],
			};
		},
		onLoad(query) {
			this.uuid = query.uuid;
			console.log('this.uuid::::', this.uuid);
			this.getUserInfo(this.uuid);
			this.get_user_posts();
		},
		onShow() {},
		onReachBottom() {
			this.get_other_post();

		},

		methods: {
			follow_click() {
				console.log("a");


				uni.request({
					url: "http://82.157.244.44:8000/api/v1/user/follow-unfollow/",
					method: "POST",
					header: {
						'Authorization': `Bearer ${uni.getStorageSync('token')}`,
					},
					data: {
						"uuid": this.uuid,
					},
					success: (res) => {
						console.log("关注成功", res.data);
						this.isFollowed = !this.isFollowed;
					}

				});
			},

			get_if_followed() {
				uni.request({
					url: "http://82.157.244.44:8000/api/v1/user/following/",
					method: "GET",
					header: {
						'Authorization': `Bearer ${uni.getStorageSync('token')}`,
					},
					success: (res) => {
						let follow = res.data.following;
						let uuid = this.userInfo.uuid;
						this.isFollowed = follow.some(item => item.uuid === uuid);
					},
					fail: (err) => {
						console.log("a");
					}
				})

			},

			getUserInfo(uuid) {
				uni.request({
					url: `http://82.157.244.44:8000/api/v1/user/query-info/?uuid=${uuid}`,
					method: 'GET',
					header: {
						'Authorization': `Bearer ${uni.getStorageSync('token')}`,
					},
					success: (res) => {
						console.log('用户信息请求成功:', res.data);
						this.userInfo = res.data;
						this.get_if_followed();
					},
					fail: (err) => {
						console.error('用户信息请求失败:', err);
					},
				});
			},
			getFullAvatarUrl(relativeUrl) {
				return `http://82.157.244.44:8000${relativeUrl}`;
			},
			// 添加关注按钮点击事件
			followUser() {
				// 实现关注逻辑，你可以在这里调用后端接口等
				console.log('关注用户');
			},
			get_user_posts() {
				uni.request({
					url: `http://82.157.244.44:8000/api/v1/forum/posts/getUserPosts/?offset=0&limit=20`,
					method: "GET",
					header: {
						'Authorization': `Bearer ${uni.getStorageSync('token')}`,
					},
					data: {
						"uuid": this.uuid,
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

				// 将时间字符串转换为 Date 对象
				const dateObj = new Date(time);

				// 添加8个小时
				dateObj.setHours(dateObj.getHours() + 8);

				// 获取格式化后的日期字符串
				const formattedTime = dateObj.toISOString().replace('T', ' ').replace(/\.\d+Z$/, '');

				return formattedTime;
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

			goToPostDetail(post) {
				console.log('Navigating to post detail:', post);
				console.log('传递的id:', post.id);
				// 使用 uni.navigateTo 进行页面跳转，并通过 query 参数传递帖子的ID
				uni.navigateTo({
					url: '/pages/msg/post_detail?id=' + post.id,
				});
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
							"uuid": this.uuid,
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
	.user-profile {
		padding: 20rpx;
	}

	.user-info {
		display: flex;
		align-items: center;
	}

	.user-avatar {
		width: 80px;
		height: 80px;
		border-radius: 50%;
		margin-right: 20rpx;
	}

	.user-name {
		font-size: 18px;
		font-weight: bold;
	}

	.info-section {
		margin-top: 20rpx;
	}

	.user-age,
	.user-intro {
		margin-top: 10rpx;
		color: #666;
		font-size: 14px;
	}

	.gender-icon {
		width: 30rpx;
		height: 30rpx;
		margin-left: 15rpx;
	}

	.unfollowed-style {
		margin-left: auto; // 将关注按钮推到右侧
		background-color: #ff8a89;
		color: #f9f8e5;
		padding: 5px 10px;
		border-radius: 15px;
		cursor: pointer;
		width: 160rpx;
		font-size: 10;
		text-align: center;
	}

	.followed-style {
		margin-left: auto; // 将关注按钮推到右侧
		background-color: white;
		color: #ff8a89;
		padding: 5px 10px;
		border-radius: 15px;
		border: 2px solid #ff8a89;
		cursor: pointer;
		width: 160rpx;
		text-align: center;
	}


	.liked {
		color: #ff6347;
		/* 设置喜欢状态的图标颜色为红色 */
	}

	.disliked {
		color: #ff6347;
		/* 设置不喜欢状态的图标颜色为蓝色 */
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