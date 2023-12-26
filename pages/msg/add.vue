<template>
	<view class="post-edit-container">
		<!-- 帖子标题输入框 -->
		<textarea v-model="postTitle" placeholder="输入标题" class="custom-input"></textarea>

		<!-- 帖子内容输入框 -->
		<textarea v-model="postContent" placeholder="输入内容" maxlength="-1" class="custom-textarea"></textarea>

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
		<button class="handlebtn" @click="uploadData">发布帖子</button>
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
				console.log(this.imageList);
				if (this.imageList.length >= this.maxImages) {
					uni.showToast({
						title: '最多只能上传' + this.maxImages + '张图片',
						icon: 'none',
					});
					return;
				}

				uni.chooseImage({
					count: this.maxImages - this.imageList.length,
					sizeType: ['compressed'], // 压缩图片
					success: (res) => {
						const tempFilePaths = res.tempFilePaths;
						tempFilePaths.forEach((item, index) => {
							wx.getFileSystemManager().saveFile({
								tempFilePath: item,
								success: async (saveRes) => {
									const savedFilePath = saveRes.savedFilePath;
									console.log(savedFilePath);
									this.imageList = this.imageList.concat(
										savedFilePath); // 将选择的图片添加到数组中
								},
								fail: (saveErr) => {
									console.error('Save file failed:', saveErr);
									// Handle the save failure
								},
							});


						});
					},
					fail: (err) => {
						console.log('选择图片失败:', err);
					},
				});
			},



			uploadData() {
				// 首先上传图片
				this.uploadImages().then((imageUrls) => {
					// 图片上传成功后，将图片地址和其他数据一起发送给后端
					const img = imageUrls.toString();

					const dataToSend = {
						title: this.postTitle,
						content: this.postContent,
						images: img, // 上传后的图片地址数组
					};
					console.log(dataToSend);
					// 发送数据给后端服务器
					uni.request({
						url: 'https://vx.mikumikumi.xyz/api/v1/forum/posts/', // 后端接口地址
						method: 'POST', // POST 或者适合你的请求方式
						header: {
							'Authorization': `Bearer ${uni.getStorageSync('token')}`,
						},
						data: dataToSend,
						success: (res) => {
							console.log('数据发送成功:', res.data);

							uni.switchTab({
								url: "/pages/msg/post-list"
							});
							console.log(dataToSend);
						},
						fail: (err) => {
							console.error('数据发送失败:', err);
						}
					});
				}).catch((error) => {
					console.error('上传图片失败:', error);
				});
			},


			// 上传图片方法
			uploadImages() {
				return new Promise((resolve, reject) => {
					const uploadedImageUrls = []; // 保存上传后的图片地址的数组

					// 循环上传图片
					const promises = this.imageList.map((imageUrl) => {
						return new Promise((resolve, reject) => {
							console.log("上传图片地址", imageUrl);

							uni.uploadFile({
								url: 'https://vx.mikumikumi.xyz/api/v1/forum/upload-image/', // 后端上传图片接口地址
								method: 'POST',
								filePath: imageUrl,
								name: 'image',
								header: {
									'Authorization': `Bearer ${uni.getStorageSync('token')}`,
								},
								success: (uploadRes) => {
									console.log('上传成功:', uploadRes.data);
									const imageFinalPath =
										"https://vx.mikumikumi.xyz" + JSON.parse(
											uploadRes.data)["image"];
									console.log(imageFinalPath);
									uploadedImageUrls.push(
										imageFinalPath); // 将上传后的图片地址保存到数组中
									console.log("a", );
									resolve();
								},
								fail: (err) => {
									console.error('上传失败:', err);
									reject(err);
								}
							});
						});
					});

					// 等待所有图片上传完成
					Promise.all(promises)
						.then(() => {
							resolve(uploadedImageUrls); // 将上传后的图片地址数组返回
						})
						.catch((error) => {
							reject(error);
						});
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
		},
	};
</script>

<style>
	.custom-input {
		width: 100%;
		height: 120rpx;
		padding: 12px;
		margin-bottom: 10px;
		border: 1px solid #ccc;
		border-radius: 5px;
		box-sizing: border-box;
		font-size: 16px;
		transition: border-color 0.3s ease;
	}

	.custom-textarea {
		border-radius: 8px;
		box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
		/* margin-left: 10px;
		margin-right: 10px; */
		height: 800rpx;
		padding: 12px;
		margin-top: 10px;
		border: 1px solid #ccc;
		border-radius: 5px;
		box-sizing: border-box;
		font-size: 16px;
		resize: none;
	}

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

	.handlebtn {
		display: block;
		width: 100%;
		/* 调整为所需的宽度值，确保左右边距一致 */
		/* margin: 20px auto; */
		padding: 12px 20px;
		border-radius: 5px;
		background-color: #527853;
		color: #fff;
		text-align: center;
		font-size: 16px;
		font-weight: bold;
		text-decoration: none;
		box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
		transition: transform 0.3s ease, background-color 0.3s ease, box-shadow 0.3s ease;
	}
</style>