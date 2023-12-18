<template>
	<view class="post-detail">
		<!-- 头部 -->
		<view class="post-header">
			<view class="user-info">
				<image class="avatar" :src="user.avatar_url" mode="aspectFill" @click="goToUserHome_post"></image>
				<view class="user-details">
					<text class="username">{{ post.author}}</text>

					<!-- <text class="post-time">{{ post.created_at }}</text> -->
					<text class="post-time">{{ formatPostTime(post.created_at) }}</text>

				</view>
			</view>
			<button :class="{'followed-style': isFollowed, 'unfollowed-style': !isFollowed}"
				@click="follow_click">{{ isFollowed ? '已关注' : '关注' }}</button>
		</view>
		<view class="divider"></view>
		<!-- 中间内容 -->
		<view class="post-content">
			<text class="content">{{ post.content }}</text>
			<view class="images">
				<image v-for="(image, index) in post_image" :key="index" :src="image" mode="aspectFill"
					class="post-image" @click="previewImage(index)"></image>
			</view>
		</view>

		<!-- 点赞、点踩、评论图标和数量 -->
		<view class="post-actions">
			<view class="action-item" @click="likePost(post)" :class="{ 'likeactive': isLiked }">
				<text class="iconfont icon-dianzan"></text>
				<text class="action-count">{{ post.likes_count }}</text>
			</view>
			<view class="action-item" @click="dislikePost(post)" :class="{ 'dislikeactive': isDisliked }">
				<text class="iconfont icon-cai"></text>
				<text class="action-count">{{ post.dislikes_count }}</text>
			</view>
			<view class="action-item" @click="toggleCommentInput">
				<text class="iconfont icon-pinglun"></text>
				<text class="action-count">{{ post.comments_count }}</text>
			</view>
		</view>

		<view v-if="showCommentInput" class="comment-input">
			<textarea v-model="commentText" class="input-textarea" placeholder="请输入评论"></textarea>
			<button class="submit-button" @click="submitComment">提交</button>
		</view>

		<!-- 显示评论列表 -->
		<view class="comments">
			<view v-for="(comment, index) in comments" :key="index" class="comment">
				<!-- 评论作者的头像和名称 -->
				<view class="comment-author">
					<image v-if="commentAuthors[comment.author_uuid]"
						:src="commentAuthors[comment.author_uuid].avatar_url" mode="aspectFill" class="avatar"
						@click="goToUserHome_comment(comment.author_uuid)"></image>
					<view class="author-details">
						<text class="author-name">{{ comment.author }}</text>
						<!-- 评论创建时间 -->
						<text class="comment-time">{{ formatPostTime(comment.created_at) }}</text>
					</view>
				</view>
				<!-- 评论内容 -->
				<view class="comment-details">
					<text class="comment-content">{{ comment.content }}</text>
				</view>
				<!-- 删除评论按钮 -->
				<button v-if="canDeleteComment(comment)" class="delete-button"
					@click="deleteComment(comment.id, comment.author_uuid)">删除</button>
			</view>
		</view>




	</view>
</template>

<script>
	export default {
		data() {
			return {
				my_follow: [],
				user: {},
				post: {},
				post_image: [],
				showCommentInput: false,
				commentText: '',
				comments: [],
				commentAuthors: {}, // 保存评论作者的信息
				isLiked: false, // 是否已点赞
				isDisliked: false, // 是否已点踩
				isFollowed: false, //是否已关注
				not_my_post: true, //是否是我发的贴子 用来决定是否显示关注按钮
			}
		},
		onLoad(query) {
			const id = query.id;
			this.get_id_post(id);
		},
		onShow() {
			this.get_if_followed();
		},
		methods: {
			goToUserHome_post() {
				const uuid = this.user.uuid;
				// 跳转到 user-home 页面
				uni.navigateTo({
					url: `/pages/user-home/user-home?uuid=${uuid}`,
				});
			},
			goToUserHome_comment(authorUuid) {
				// 跳转到 user-home 页面，并传递评论作者的 uuid
				uni.navigateTo({
					url: `/pages/user-home/user-home?uuid=${authorUuid}`,
				});
			},
			if_my_post() {
				const userInfo = uni.getStorageSync('userInfo');
				const user_uuid = userInfo.uuid;

				if (this.user.uuid === user_uuid)

				{
					this.not_my_post = false;
				} else {
					this.not_my_post = true;
				}


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
						let uuid = this.user.uuid;
						this.isFollowed = follow.some(item => item.uuid === uuid);
					},
					fail: (err) => {
						console.log("a");
					}
				})

			},


			follow_click() {
				console.log("a");

				this.isFollowed = !this.isFollowed;
				uni.request({
					url: "http://82.157.244.44:8000/api/v1/user/follow-unfollow/",
					method: "POST",
					header: {
						'Authorization': `Bearer ${uni.getStorageSync('token')}`,
					},
					data: {
						"uuid": this.user.uuid,
					},
					success: (res) => {
						console.log("关注成功", res.data);
					}

				});
			},

			async likePost(post) {
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
						this.isLiked = !this.isLiked;
						if (this.isLiked) {
							post.likes_count += 1; // 增加点赞数量
							// 如果之前点踩了，取消点踩状态
							if (this.isDisliked) {
								this.isDisliked = false;
								post.dislikes_count -= 1; // 减少点踩数量
							}
						} else {
							post.likes_count -= 1; // 减少点赞数量
						}
					} else {
						console.error('点赞失败:', response[1].data);
					}
				} catch (error) {
					console.error('点赞失败:', error);
				}
			},

			async dislikePost(post) {
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
						this.isDisliked = !this.isDisliked;
						if (this.isDisliked) {
							post.dislikes_count += 1; // 增加点踩数量
							// 如果之前点赞了，取消点赞状态
							if (this.isLiked) {
								this.isLiked = false;
								post.likes_count -= 1; // 减少点赞数量
							}
						} else {
							post.dislikes_count -= 1; // 减少点踩数量
						}
					} else {
						console.error('点踩失败:', response[1].data);
					}
				} catch (error) {
					console.error('点踩失败:', error);
				}
			},

			canDeleteComment(comment) {
				// 只有评论的作者可以删除评论
				const userInfo = uni.getStorageSync('userInfo');
				return comment.author_uuid === userInfo.uuid;
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

				// 获取格式化后的日期字符串
				const formattedTime = dateObj.toISOString().replace('T', ' ').replace(/\.\d+Z$/, '');

				return formattedTime;
			},


			toggleCommentInput() {
				this.showCommentInput = !this.showCommentInput;
			},
			submitComment() {
				// 处理提交评论的逻辑，可以在这里发送评论内容到后端或者做其他处理
				console.log('提交评论:', this.commentText);

				// 向后端发送评论请求
				uni.request({
					url: 'http://82.157.244.44:8000/api/v1/forum/comments/',
					method: 'POST',
					header: {
						'Authorization': `Bearer ${uni.getStorageSync('token')}`,
					},
					data: {
						content: this.commentText,
						post_id: this.post.id,
					},
					success: (res) => {
						console.log('评论提交成功:', res.data);

						// 更新评论的作者、头像、创建时间
						const newComment = {
							id: res.data.id,
							post: res.data.post,
							content: res.data.content,
							author: res.data.author,
							author_avatar: "http://82.157.244.44:8000" + res.data
								.author_avatar, // Update with the correct property
							created_at: res.data.created_at,
						};

						// 将新评论添加到本地评论列表
						this.comments.push(newComment);

						// 提交完成后清空输入框并隐藏输入框
						this.commentText = '';
						this.showCommentInput = false;

						// 重新加载帖子详情页，刷新整个页面
						this.get_id_post(this.post.id);
					},
					fail: (err) => {
						console.error('评论提交失败:', err);
					}
				});

				// 提交完成后清空输入框并隐藏输入框
				this.commentText = '';
				this.showCommentInput = false;
			},

			get_id_post(id) {
				uni.request({
					url: `http://82.157.244.44:8000/api/v1/forum/posts/${id}/`, // 后端接口地址
					method: 'GET',
					header: {
						'Authorization': `Bearer ${uni.getStorageSync('token')}`,
					},
					success: (res) => {
						console.log('数据接收成功:', res.data);
						this.post = res.data;
						//this.post = res.data.data.find(a => a.id === Number(postid));
						// console.log(this.post);

						const image_addr = this.post.images;
						this.post_image = image_addr.split(',');
						// console.log(this.post_image);

						// 更新点赞和点踩状态
						this.isLiked = this.post.is_liked; // 假设后端返回的字段为 is_liked
						this.isDisliked = this.post.is_disliked; // 假设后端返回的字段为 is_disliked

						// 获取评论列表
						uni.request({
							url: `http://82.157.244.44:8000/api/v1/forum/comments/?post_id=${this.post.id}`, // 根据帖子ID获取评论列表
							method: 'GET',
							header: {
								'Authorization': `Bearer ${uni.getStorageSync('token')}`,
							},
							success: (res) => {
								console.log('评论数据接收成功:', res.data);
								// 更新本地评论列表
								this.comments = res.data.data.map(comment => ({
									id: comment.id,
									post: comment.post,
									content: comment.content,
									author: comment.author,
									author_uuid: comment.author_uuid,
									created_at: comment.created_at,
								}));

								// 获取评论用户信息
								this.comments.forEach(comment => {
									this.getCommentAuthorInfo(comment.author_uuid);
								});
							},
							fail: (err) => {
								console.error('评论数据发送失败:', err);
							}
						});

						// 获取帖子作者信息
						uni.request({
							url: `http://82.157.244.44:8000/api/v1/user/query-info/?uuid=${this.post.author_uuid}`, // 后端接口地址
							method: 'GET',
							header: {
								'Authorization': `Bearer ${uni.getStorageSync('token')}`,
							},
							success: (res) => {
								console.log('数据接收成功:', res.data);
								this.user = res.data;
								this.user.avatar_url = "http://82.157.244.44:8000" + this.user
									.avatar_url;
								console.log(this.user.avatar_url);
								this.get_if_followed();
								this.if_my_post();
							},
							fail: (err) => {
								console.error('数据发送失败:', err);
							}
						});
					},
					fail: (err) => {
						console.error('数据发送失败:', err);
					}
				});
			},

			// get_all_post(postid) {
			// 	// 获取帖子信息
			// 	uni.request({
			// 		url: 'http://82.157.244.44:8000/api/v1/forum/posts/', // 后端接口地址
			// 		method: 'GET',
			// 		header: {
			// 			'Authorization': `Bearer ${uni.getStorageSync('token')}`,
			// 		},
			// 		success: (res) => {
			// 			console.log('数据接收成功:', res.data);

			// 			this.post = res.data.data.find(a => a.id === Number(postid));
			// 			console.log(this.post);

			// 			const image_addr = this.post.images;
			// 			this.post_image = image_addr.split(',');
			// 			console.log(this.post_image);

			// 			// 更新点赞和点踩状态
			// 			this.isLiked = this.post.is_liked; // 假设后端返回的字段为 is_liked
			// 			this.isDisliked = this.post.is_disliked; // 假设后端返回的字段为 is_disliked

			// 			// 获取评论列表
			// 			uni.request({
			// 				url: `http://82.157.244.44:8000/api/v1/forum/comments/?post_id=${this.post.id}`, // 根据帖子ID获取评论列表
			// 				method: 'GET',
			// 				header: {
			// 					'Authorization': `Bearer ${uni.getStorageSync('token')}`,
			// 				},
			// 				success: (res) => {
			// 					console.log('评论数据接收成功:', res.data);
			// 					// 更新本地评论列表
			// 					this.comments = res.data.map(comment => ({
			// 						id: comment.id,
			// 						post: comment.post,
			// 						content: comment.content,
			// 						author: comment.author,
			// 						author_uuid: comment.author_uuid,
			// 						created_at: comment.created_at,
			// 					}));

			// 					// 获取评论用户信息
			// 					this.comments.forEach(comment => {
			// 						this.getCommentAuthorInfo(comment.author_uuid);
			// 					});
			// 				},
			// 				fail: (err) => {
			// 					console.error('评论数据发送失败:', err);
			// 				}
			// 			});

			// 			// 获取帖子作者信息
			// 			uni.request({
			// 				url: `http://82.157.244.44:8000/api/v1/user/query-info/?uuid=${this.post.author_uuid}`, // 后端接口地址
			// 				method: 'GET',
			// 				header: {
			// 					'Authorization': `Bearer ${uni.getStorageSync('token')}`,
			// 				},
			// 				success: (res) => {
			// 					console.log('数据接收成功:', res.data);
			// 					this.user = res.data;
			// 					this.user.avatar_url = "http://82.157.244.44:8000" + this.user
			// 						.avatar_url;
			// 					console.log(this.user.avatar_url);
			// 					this.get_if_followed();
			// 					this.if_my_post();
			// 				},
			// 				fail: (err) => {
			// 					console.error('数据发送失败:', err);
			// 				}
			// 			});
			// 		},
			// 		fail: (err) => {
			// 			console.error('数据发送失败:', err);
			// 		}
			// 	});
			// },

			// 获取评论用户信息
			getCommentAuthorInfo(authorUuid) {
				uni.request({
					url: `http://82.157.244.44:8000/api/v1/user/query-info/?uuid=${authorUuid}`,
					method: 'GET',
					header: {
						'Authorization': `Bearer ${uni.getStorageSync('token')}`,
					},
					success: (res) => {
						console.log('评论用户信息接收成功:', res.data);
						// 将评论用户的头像信息保存到commentAuthors对象中
						this.$set(this.commentAuthors, authorUuid, {
							avatar_url: "http://82.157.244.44:8000" + res.data.avatar_url,
						});
					},
					fail: (err) => {
						console.error('评论用户信息获取失败:', err);
					}
				});
			},

			deleteComment(commentId, authorUuid) {
				uni.request({
					url: `http://82.157.244.44:8000/api/v1/forum/comments/${commentId}/`, // 删除评论的后端接口
					method: 'DELETE',
					header: {
						'Authorization': `Bearer ${uni.getStorageSync('token')}`,
					},
					success: (res) => {
						console.log('评论删除成功:', res.data);
						// 在本地评论列表中移除被删除的评论
						this.comments = this.comments.filter(comment => comment.id !== commentId);

						// 移除评论作者的头像信息
						this.$delete(this.commentAuthors, authorUuid);

						// 重新加载帖子详情页，刷新整个页面
						this.get_id_post(this.post.id);
					},
					fail: (err) => {
						console.error('评论删除失败:', err);
					}
				});
			},

			previewImage(index) {
				uni.previewImage({
					urls: this.post_image,
					current: this.post_image[index],
				});
			},

		}
	}
</script>


<style scoped>
	.comment {
		display: flex;
		margin-bottom: 10px;
	}

	.comment-author {
		display: flex;
		align-items: center;
		margin-right: 10px;
	}

	.avatar {
		width: 30px;
		height: 30px;
		border-radius: 50%;
		margin-right: 5px;
	}

	.author-details {
		display: flex;
		flex-direction: column;
	}

	.author-name {
		font-size: 14px;
		font-weight: bold;
	}

	.comment-details {
		flex-grow: 1;
	}

	.comment-content {
		font-size: 14px;
		margin-bottom: 5px;
	}

	.comment-time {
		font-size: 12px;
		color: #999;
	}

	.delete-button {
		align-self: flex-start;
		/* 设置按钮在垂直方向上靠上显示 */
	}


	/* 样式可以根据需要进行修改 */
	.post-detail {
		padding: 20px;
	}

	.post-header {
		display: flex;
		align-items: center;
		justify-content: space-between;
		margin-bottom: 20px;
	}

	.user-info {
		display: flex;
		align-items: center;
	}

	.avatar {
		width: 50px;
		height: 50px;
		border-radius: 50%;
	}

	.user-details {
		margin-left: 10px;
	}

	.username {
		font-size: 20px;
		font-weight: bold;
	}

	.post-time {
		font-size: 16px;
		color: #999;
	}

	.unfollowed-style {
		padding: 8px 16px;
		border: none;
		border-radius: 15px;
		background-color: #3498db;
		color: #fff;
		font-size: 12px;
		height: 80rpx;
		width: 200rpx;
	}

	.followed-style {
		padding: 8px 16px;
		border: none;
		border-radius: 15px;
		background-color: #f5f5f5;
		color: red;
		font-size: 12px;
		height: 80rpx;
		width: 200rpx;
	}

	.divider {
		height: 1px;
		background-color: #ccc;
		margin-bottom: 20px;
	}

	.post-content {
		line-height: 1.6;
	}

	.content {
		font-size: 18px;
		margin-bottom: 10px;
	}

	.post-image {
		width: 100px;
		height: 100px;
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

	.action-item.likeactive {
		color: #ff6347;
		/* 设置激活状态下的颜色，可以根据需要调整 */
	}

	.action-item.dislikeactive {
		color: #ff6347;
		/* 设置激活状态下的颜色，可以根据需要调整 */
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

	.comment-button {
		padding: 8px 16px;
		border: none;
		border-radius: 20px;
		background-color: #3498db;
		color: #fff;
		font-size: 14px;
		margin-top: 20px;
	}

	.comment-input {
		margin-top: 20px;
	}

	.input-textarea {
		width: 90%;
		min-height: 80px;
		padding: 10px;
		margin-bottom: 10px;
		border: 1px solid #ccc;
		border-radius: 5px;
	}

	.submit-button {
		padding: 8px 16px;
		border: none;
		border-radius: 20px;
		background-color: #3498db;
		color: #fff;
		font-size: 14px;
	}
</style>