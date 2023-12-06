<template>
	<view class="post-detail">
		<!-- 头部 -->
		<view class="post-header">
			<view class="user-info">
				<image class="avatar" :src="user.avatar_url" mode="aspectFill"></image>
				<view class="user-details">
					<text class="username">{{ post.author}}</text>

					<!-- <text class="post-time">{{ post.created_at }}</text> -->
					<text class="post-time">{{ formatPostTime(post.created_at) }}</text>

				</view>
			</view>
			<button class="follow-button">关注</button>
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
			<view class="action-item" @click="likePost(post)">
				<text class="iconfont icon-dianzan"></text>
				<text class="action-count">{{ post.likes_count }}</text>
			</view>
			<view class="action-item" @click="dislikePost(post)">
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
						:src="commentAuthors[comment.author_uuid].avatar_url" mode="aspectFill" class="avatar"></image>
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
				user: {},
				post: {},
				post_image: [],
				showCommentInput: false,
				commentText: '',
				comments: [],
				commentAuthors: {}, // 保存评论作者的信息
			}
		},
		onLoad(query) {
			const id = query.id;
			this.get_all_post(id);
		},
		methods: {

			canDeleteComment(comment) {
				// 只有帖子作者和评论作者可以删除评论
				return comment.author_uuid === this.user.uuid || this.post.author_uuid === this.user.uuid;
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
						this.get_all_post(this.post.id);
					},
					fail: (err) => {
						console.error('评论提交失败:', err);
					}
				});

				// 提交完成后清空输入框并隐藏输入框
				this.commentText = '';
				this.showCommentInput = false;
			},

			get_all_post(postid) {
				// 获取帖子信息
				uni.request({
					url: 'http://82.157.244.44:8000/api/v1/forum/posts/', // 后端接口地址
					method: 'GET',
					header: {
						'Authorization': `Bearer ${uni.getStorageSync('token')}`,
					},
					success: (res) => {
						console.log('数据接收成功:', res.data);

						this.post = res.data.find(a => a.id === Number(postid));
						console.log(this.post);

						const image_addr = this.post.images;
						this.post_image = image_addr.split(',');
						console.log(this.post_image);

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
								this.comments = res.data.map(comment => ({
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
						this.get_all_post(this.post.id);
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

	.follow-button {
		padding: 8px 16px;
		border: none;
		border-radius: 15px;
		background-color: #3498db;
		color: #fff;
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