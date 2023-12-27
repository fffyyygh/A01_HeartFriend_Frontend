<template>
	<view class="post-detail"  v-show="deleted">
		<!-- 头部 -->
		<view class="post-header">
			<view class="user-info">
				<image class="avatar" :src="user.avatar_url" mode="aspectFill" @click="goToUserHome_post"></image>
				<view class="user-details">
					<text class="username">{{ post.author}}</text>
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
			<button v-show="if_admin" @click="showDeleteConfirmation_post">删除帖子</button>
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

			<!-- 设置用户是否可以评论 -->

			<picker class="action-item" mode="selector" :range="units" @change="unitSelected">
				<view v-show="!not_my_post">设置</view>
			</picker>

			<picker class="action-item" mode="selector" :range="units2" @change="unitSelected2">
				<view v-show="!not_my_post">{{ visibilityText }}</view>
			</picker>

			<!-- 举报按钮 -->
			<button class="report-button" @click="reportPost">举报</button>
		</view>

		<view v-if="showCommentInput" class="comment-input">
			<textarea v-model="commentText" class="input-textarea" focus placeholder="请输入评论"></textarea>
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
					<!-- 删除评论按钮 -->
					<button v-if="canDeleteComment(comment)" class="delete-button"
						@click="showDeleteConfirmation(comment)">删除</button>
				</view>

				<!-- 评论内容 -->
				<view class="comment-details">
					<text class="comment-content">{{ comment.content }}</text>
				</view>
				<!-- 分割线 -->
				<view class="divider"></view>
			</view>
		</view>
		<view style="height: 100rpx;"></view>

	</view>
</template>

<script>
	export default {
		data() {
			return {
				deleted: false,
				units2: ["公开帖子", "隐藏帖子"],
				visibilityText: "公开",
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
				units: ["允许评论", "禁止所有人评论", "仅自己可评论"],
				units2: ["公开帖子", "隐藏帖子"],
				if_admin: false,

			}
		},
		onLoad(query) {
			this.getIfAdmin();
			const id = query.id;
			if( id=== "null" ){
				
				
				console.log("aaaaaaa");
				uni.redirectTo({
					url:"/pages/msg/post_empty",
				})
			}
			
			else
			{
				this.deleted = true
				this.get_id_post(id);
			}
			


		},
		onShow() {
			this.get_if_followed();
		},
		methods: {
			showDeleteConfirmation_post() {
			        uni.showModal({
			            title: '确认删除',
			            content: '确认删除本条帖子？',
			            success: (res) => {
			                if (res.confirm) {
			                    this.deletePost(); // Call deletePost if user confirms
			                }
			            },
			        });
			    },

			deletePost() {
				const id = this.post.id;
				uni.request({
					url: 'https://vx.mikumikumi.xyz/api/v1/forum/posts/' + id + "/", // 后端接口地址
					method: 'DELETE',
					header: {
						'Authorization': `Bearer ${uni.getStorageSync('token')}`,
					},
					success: (res) => {
						console.log('帖子已删除');
						uni.redirectTo({
							url:"/pages/msg/post_empty",
						})

					},
					fail: (err) => {
						console.error('数据发送失败:', err);
					}
				});

			},


			getIfAdmin() {
				uni.request({
					url: `https://vx.mikumikumi.xyz/api/v1/user/info/`,
					method: "GET",
					header: {
						'Authorization': `Bearer ${uni.getStorageSync('token')}`,
					},

					success: (res) => {
						if (res.data.is_forum_admin)

						{
							this.if_admin = true;
						} else {
							this.if_admin = false;
						}

					},
					fail: (err) => {
						console.log("a");
					}
				})
			},

			reportPost() {
				// Add logic to navigate to the report page
				uni.navigateTo({
					url: '/pages/msg/report?id=' + this.post.id,
				});
			},
			showDeleteConfirmation(comment) {
				uni.showModal({
					title: '确认删除',
					content: '确认删除该评论？',
					success: (res) => {
						if (res.confirm) {
							this.deleteComment(comment.id, comment.author_uuid);
						}
					},
				});
			},
			unitSelected: function(event) {
				const index = event.detail.value;
				if (index == 0) {

					uni.request({
						url: `https://vx.mikumikumi.xyz/api/v1/forum/posts/${this.post.id}/allowComment/`,
						method: "POST",
						header: {
							'Authorization': `Bearer ${uni.getStorageSync('token')}`,
						},
						data: {
							allow_comment: "OP",
						},
						success: (res) => {
							this.showCommentInput = false;
							console.log("aaaaa");
							uni.showToast({
								title: '权限设置成功',
								duration: 1000,
								icon: "success",
							});

							this.get_id_post(this.post.id);

						},
						fail: (err) => {
							console.log("a");
						}
					})
				} else if (index == 1) {
					uni.request({
						url: `https://vx.mikumikumi.xyz/api/v1/forum/posts/${this.post.id}/allowComment/`,
						method: "POST",
						header: {
							'Authorization': `Bearer ${uni.getStorageSync('token')}`,
						},
						data: {
							allow_comment: "CL",
						},
						success: (res) => {
							this.showCommentInput = false;
							uni.showToast({
								title: '权限设置成功',
								duration: 1000,
								icon: "success",
							});

							this.get_id_post(this.post.id);
						},
						fail: (err) => {
							console.log("a");
						}
					})
				} else {
					uni.request({
						url: `https://vx.mikumikumi.xyz/api/v1/forum/posts/${this.post.id}/allowComment/`,
						method: "POST",
						header: {
							'Authorization': `Bearer ${uni.getStorageSync('token')}`,
						},
						data: {
							allow_comment: "AU",
						},
						success: (res) => {
							this.showCommentInput = false;
							uni.showToast({
								title: '权限设置成功',
								duration: 1000,
								icon: "success",
							});

							this.get_id_post(this.post.id);
						},
						fail: (err) => {
							console.log("a");
						}
					})
				}


			},


			unitSelected2: function(event) {
				const index = event.detail.value;
				if (index == 0) {
					uni.request({
						url: `https://vx.mikumikumi.xyz/api/v1/forum/posts/${this.post.id}/setVisibility/`,
						method: "POST",
						header: {
							'Authorization': `Bearer ${uni.getStorageSync('token')}`,
						},
						data: {
							visibility: "PU",
						},
						success: (res) => {
							console.log("aaaaa");
							uni.showToast({
								title: '公开设置成功',
								duration: 1000,
								icon: "success",
							});

							this.get_id_post(this.post.id);
							this.visibilityText = "公开";
						},
						fail: (err) => {
							console.log("a");
						}
					})
				} else {
					uni.request({
						url: `https://vx.mikumikumi.xyz/api/v1/forum/posts/${this.post.id}/setVisibility/`,
						method: "POST",
						header: {
							'Authorization': `Bearer ${uni.getStorageSync('token')}`,
						},
						data: {
							visibility: "PR",
						},
						success: (res) => {

							console.log("aaaaa");
							uni.showToast({
								title: '隐藏设置成功',
								duration: 1000,
								icon: "success",
							});

							this.get_id_post(this.post.id);
							this.visibilityText = "隐藏";
						},
						fail: (err) => {
							console.log("a");
						}
					})
				}


			},

			goToUserHome_post() {
				// const uuid = this.user.uuid;
				// // 跳转到 user-home 页面
				// uni.navigateTo({
				// 	url: `/pages/user-home/user-home?uuid=${uuid}`,
				// });
				uni.navigateBack();
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
					url: "https://vx.mikumikumi.xyz/api/v1/user/following/",
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
					url: "https://vx.mikumikumi.xyz/api/v1/user/follow-unfollow/",
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
						url: `https://vx.mikumikumi.xyz/api/v1/forum/posts/${post.id}/like/`,
						method: 'POST',
						header: {
							'Authorization': `Bearer ${uni.getStorageSync('token')}`,
						},

					});


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
						url: `https://vx.mikumikumi.xyz/api/v1/forum/posts/${post.id}/dislike/`,
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
				return comment.author_uuid === userInfo.uuid || this.not_my_post === false;
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


			toggleCommentInput() {
				console.log(this.post.allowed_comment);
				if (this.post.allowed_comment === "OP") {
					this.showCommentInput = !this.showCommentInput;
				} else if (this.post.allowed_comment === "CL")

				{
					uni.showToast({
						title: '用户已禁止评论',
						duration: 1000,
						icon: "error",
					});
				} else {
					if (this.not_my_post) {
						uni.showToast({
							title: '用户已禁止评论',
							duration: 1000,
							icon: "error",
						});
					} else {
						this.showCommentInput = !this.showCommentInput;
					}

				}
			},
			submitComment() {
				// 处理提交评论的逻辑，可以在这里发送评论内容到后端或者做其他处理
				console.log('提交评论:', this.commentText);

				// 向后端发送评论请求
				uni.request({
					url: 'https://vx.mikumikumi.xyz/api/v1/forum/comments/',
					method: 'POST',
					header: {
						'Authorization': `Bearer ${uni.getStorageSync('token')}`,
					},
					data: {
						content: this.commentText,
						post_id: this.post.id,
					},
					success: (res) => {


						// 更新评论的作者、头像、创建时间
						const newComment = {
							id: res.data.id,
							post: res.data.post,
							content: res.data.content,
							author: res.data.author,
							author_avatar: "https://vx.mikumikumi.xyz" + res.data
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
					url: `https://vx.mikumikumi.xyz/api/v1/forum/posts/${id}/`, // 后端接口地址
					method: 'GET',
					header: {
						'Authorization': `Bearer ${uni.getStorageSync('token')}`,
					},
					success: (res) => {
						this.post = res.data;
						//this.post = res.data.data.find(a => a.id === Number(postid));
						// console.log(this.post);

						const image_addr = this.post.images;
						this.post_image = image_addr.split(',');
						// console.log(this.post_image);

						// 更新点赞和点踩状态
						this.isLiked = this.post.is_liked; // 假设后端返回的字段为 is_liked
						this.isDisliked = this.post.is_disliked; // 假设后端返回的字段为 is_disliked


						// 获取公开状态

						if (this.post.visibility === "PR") {
							this.visibilityText = "隐藏";
						}

						// 获取评论列表
						uni.request({
							url: `https://vx.mikumikumi.xyz/api/v1/forum/comments/?post_id=${this.post.id}`, // 根据帖子ID获取评论列表
							method: 'GET',
							header: {
								'Authorization': `Bearer ${uni.getStorageSync('token')}`,
							},
							success: (res) => {
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
							url: `https://vx.mikumikumi.xyz/api/v1/user/query-info/?uuid=${this.post.author_uuid}`, // 后端接口地址
							method: 'GET',
							header: {
								'Authorization': `Bearer ${uni.getStorageSync('token')}`,
							},
							success: (res) => {
								console.log('数据接收成功:', res.data);
								this.user = res.data;
								this.user.avatar_url = "https://vx.mikumikumi.xyz" + this.user
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


			// 获取评论用户信息
			getCommentAuthorInfo(authorUuid) {
				uni.request({
					url: `https://vx.mikumikumi.xyz/api/v1/user/query-info/?uuid=${authorUuid}`,
					method: 'GET',
					header: {
						'Authorization': `Bearer ${uni.getStorageSync('token')}`,
					},
					success: (res) => {
						//console.log('评论用户信息接收成功:', res.data);
						// 将评论用户的头像信息保存到commentAuthors对象中
						this.$set(this.commentAuthors, authorUuid, {
							avatar_url: "https://vx.mikumikumi.xyz" + res.data.avatar_url,
						});
					},
					fail: (err) => {
						console.error('评论用户信息获取失败:', err);
					}
				});
			},

			deleteComment(commentId, authorUuid) {
				uni.request({
					url: `https://vx.mikumikumi.xyz/api/v1/forum/comments/${commentId}/`, // 删除评论的后端接口
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
	.report-button {
		align-self: center;
		margin-left: 8rpx;
		color: black;
		height: 60rpx;
		width: 120rpx;
		font-size: 25rpx;
	}

	.comment {
		display: flex;
		flex-direction: column;
		padding: 10rpx;
	}

	.comment-author {
		display: flex;
		align-items: center;
		justify-content: space-between;
		margin-bottom: 8rpx;
	}

	.avatar {
		width: 80rpx;
		height: 80rpx;
		border-radius: 50%;
		margin-right: 8rpx;
	}

	.author-details {
		flex: 1;
		display: flex;
		flex-direction: column;
	}

	.author-name {
		/* font-size: 14px; */
		font-weight: bold;
	}

	.comment-details {
		/* flex-grow: 1; */
		margin-bottom: 8rpx;
	}

	.comment-content {
		/* font-size: 14px; */
		/* margin-bottom: 5px; */
	}

	.comment-time {
		color: #888;
	}

	.delete-button {
		align-self: flex-start;
		/* 设置按钮在垂直方向上靠上显示 */
		margin-left: 8rpx;
		color: black;
		height: 60rpx;
		width: 120rpx;
		font-size: 25rpx;
	}

	.divider {
		height: 1px;
		background-color: #eee;
		margin: 8rpx 0;
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
		justify-content: space-between;
		margin-bottom: 8rpx;
	}

	.user-details {
		flex: 1;
		display: flex;
		flex-direction: column;
	}

	.username {
		font-weight: bold;
	}

	.post-time {
		color: #888;
	}

	.unfollowed-style {
		display: flex;
		align-items: center;
		justify-content: center;
		align-self: flex-start;
		padding: 8px 16px;
		border: none;
		background-color: #527853;
		color: #fff;
		font-size: 30rpx;
		height: 80rpx;
		width: 200rpx;
		margin-right: 20rpx;
	}

	.followed-style {
		display: flex;
		align-items: center;
		justify-content: center;
		align-self: flex-start;
		padding: 8px 16px;
		border: none;
		background-color: darkgray;
		color: white;
		font-size: 30rpx;
		height: 80rpx;
		width: 200rpx;
		margin-right: 20rpx;
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
		color: #EF4040;
		/* 设置激活状态下的颜色，可以根据需要调整 */
	}

	.action-item.dislikeactive {
		color: #527853;
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
		background-color: #527853;
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
		background-color: #527853;
		color: #fff;
		font-size: 14px;
	}
</style>
