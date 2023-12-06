<template>
	<view class="post-list">
		<view v-for="(post, index) in posts" :key="index"  class="post-item">
			<!-- 用户信息 -->
			<view class="user-info" >
				<!-- 头像 -->
				<image class="user-avatar" :src="users[index].avatar_url" mode="aspectFill"></image>
				<!-- 用户名、发帖时间等信息 -->
				<view>
					<text class="user-name">{{ post.author }}</text>
					<text class="post-time">{{ formatPostTime(post.created_at) }}</text>
				</view>
				<button @click="deletePost(index)">删除</button>
			</view>
			
			<!-- 帖子内容 -->
			<view class="post-content" >
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
				<view class="action-item" @click="commentPost(post)">
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
				posts: [],
				users:[],
			};
		},
		onShow() {
			this.get_all_post();
		},
		methods: {
			deletePost(index){
				console.log("a");
				const post = this.posts[index];
				const id = post.id;
				console.log(id);
				uni.request({
					url: 'http://82.157.244.44:8000/api/v1/forum/posts/'+ id + "/",  // 后端接口地址
					method: 'DELETE',
					header: {
						'Authorization': `Bearer ${uni.getStorageSync('token')}`,
					},
					success: (res) => {
						console.log('数据接收成功:', res.data);	
						this.posts= [];
						this.users=[];
						this.get_all_post();
							
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
			
			async getUsers() {
			  try {
			    for (const post of this.posts) {
			      const response = await this.makeRequest(
			        'http://82.157.244.44:8000/api/v1/user/query-info/?uuid=' + post.author_uuid,
			        'GET',
			        {
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
			
			get_all_post() {
				this.users=[];
				this.posts=[];
				uni.request({
					url: 'http://82.157.244.44:8000/api/v1/forum/posts/', // 后端接口地址
					method: 'GET',
					header: {
						'Authorization': `Bearer ${uni.getStorageSync('token')}`,
					},
					success: (res) => {
						console.log('数据接收成功:', res.data);	
						const userInfo = uni.getStorageSync('userInfo');
						const user_uuid = userInfo.uuid;
						//this.post = res.data[7]; //
						res.data.forEach((post,index)=>{								
							if(post.author_uuid==user_uuid){
								
								this.posts.push(post);
							}
							
							
						})
						
						this.posts.forEach((post,index)=>{
							const image_addr = post.images;
							post.images = image_addr.split(',');
							
							
						})	
						this.getUsers();
			
					},
					fail: (err) => {
						console.error('数据发送失败:', err);
					}
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