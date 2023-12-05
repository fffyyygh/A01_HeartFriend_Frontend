<template>
	<view class="post-edit-container">
		<!-- 帖子标题输入框 -->
		<input v-model="postTitle" placeholder="输入标题" />

		<!-- 帖子内容输入框 -->
		<textarea v-model="postContent" placeholder="输入内容"></textarea>

		<!-- 图片上传 -->
		<view class="image-upload-container">
			<view v-for="(item, index) in imageList" :key="index" class="image-item">
				<image :src="item" mode="aspectFill" style="width: 100%; height: 100%;" @click="previewImage(index)" />
				<view class="delete-btn" @click="deleteImage(index)">×</view>
			</view>

			<view v-show="imageList.length < maxImages" class="add-image" @click="addImage">
				<text>+</text>
			</view>
		</view>

		<!-- 提交按钮 -->
		<button @click="submitPost">发布帖子</button>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				postTitle: "", // 帖子标题
				postContent: "", // 帖子内容
				imageList: [], // 存储已上传的图片地址
				maxImages: 9, // 最大上传图片数量
			};
		},
		methods: {
			addImage() {
				if (this.imageList.length >= this.maxImages) {
					uni.showToast({
						title: '最多只能上传' + this.maxImages + '张图片',
						icon: 'none',
					});
					return;
				}

				uni.chooseImage({
					count: this.maxImages - this.imageList.length,
					success: (res) => {
						const tempFilePaths = res.tempFilePaths;
						this.imageList = this.imageList.concat(tempFilePaths);
					},
				});
			},
			previewImage(index) {
				uni.previewImage({
					urls: this.imageList,
					current: this.imageList[index],
				});
			},
			deleteImage(index) {
				this.imageList.splice(index, 1);
			},
			submitPost() {
				// 在这里执行发布帖子的逻辑
				// 可以将帖子的标题、内容、图片地址等数据提交到后端进行保存

				// 发布成功后，可以跳转回列表页或其他需要的页面
				uni.showToast({
					title: '发布成功',
					icon: 'success',
					duration: 2000,
					success: () => {
						uni.navigateTo({
							url: '/pages/msg/msg.vue', // 这里填写你的消息列表页的路径
						});
					},
				});
			},
		},
	};
</script>

<style>
	.post-edit-container {
		padding: 20px;
	}

	input,
	textarea {
		width: 100%;
		margin-bottom: 10px;
	}

	.image-upload-container {
		display: flex;
		flex-wrap: wrap;
		margin: 10px 0;
	}

	.image-item {
		position: relative;
		width: 100px;
		height: 100px;
		margin-right: 10px;
		margin-bottom: 10px;
	}

	.delete-btn {
		position: absolute;
		top: 5px;
		right: 5px;
		width: 20px;
		height: 20px;
		line-height: 20px;
		text-align: center;
		background-color: rgba(0, 0, 0, 0.5);
		color: #fff;
		font-size: 14px;
		border-radius: 50%;
		cursor: pointer;
	}

	.add-image {
		width: 100px;
		height: 100px;
		background-color: #eee;
		display: flex;
		align-items: center;
		justify-content: center;
		font-size: 40px;
		cursor: pointer;
	}
</style>