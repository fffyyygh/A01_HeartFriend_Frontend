<template>
	<view>
		<view class="container">
			<view class="rate-section">
				<text class="rate-title">食欲</text>
				<uni-rate :max="10" :value="10" v-model="appetite" :size="25" active-color="#FFD700"
					:void-color="'#ccc'"></uni-rate>
				<text class="rate-value">{{ appetite }}</text>
			</view>
			<view class="rate-section">
				<text class="rate-title">心情</text>
				<uni-rate :max="10" :value="10" v-model="mood" :size="25" active-color="#FFA07A"
					:void-color="'#ccc'"></uni-rate>
				<text class="rate-value">{{ mood }}</text>
			</view>
			<view class="rate-section">
				<text class="rate-title">睡眠</text>
				<uni-rate :max="10" :value="10" v-model="sleep" :size="25" active-color="#87CEFA"
					:void-color="'#ccc'"></uni-rate>
				<text class="rate-value">{{ sleep }}</text>
			</view>
		</view>
		<view class="input-section">
			<text class="input-title">标题</text>
			<uni-easyinput v-model="title" placeholder="请输入标题"></uni-easyinput>
		</view>
		<view class="input-section">
			<text class="input-title">内容</text>
			<textarea v-model="content" class="content-input" placeholder="请输入内容"></textarea>
		</view>

		<!-- 旧的方法 -->
<!-- 		<view>
			<view class="image-grid" v-if="imageUrls.length > 0">
				<image v-for="(imageUrl, index) in displayedImages" :key="index" :src="imageUrl" mode="aspectFit"
					class="grid-item" @longpress="showDeleteButton(index)"></image>
			</view>

			<button class="btn" @tap="chooseImage">
				<view class="button-content">
					<text class="btn-text">选择图片</text>
				</view>
			</button>
		</view> -->
		
		
		<!-- 图片改变方法 -->
		
		<view class="image-upload-container">
			<view v-for="(item, index) in imageList" :key="index" class="image-item">
				<image :src="item" mode="aspectFill" style="width: 100%; height: 100%;" @click="previewImage(index)" />
				<view class="delete-btn" @click="deleteImage(index)">×</view>
			</view>
		
			<view v-show="imageList.length < maxImages" class="add-image" @click="addImage">
				<text>+</text>
			</view>
		</view>
		
		
		<!-- 图片上传改变方法 -->

		<button class="btn" @click="uploadData">
			<view class="button-content">
				<text class="btn-text">提交</text>
			</view>
		</button>
		<view style="height: 60rpx;"></view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				appetite: 8,
				mood: 8,
				sleep: 8,
				title: '', // 保存用户输入的标题
				content: '', // 保存用户输入的内容
				deleteIndex: null, // 保存需要删除的图片索引
				
				//
				imageList: [], // 存储已上传的图片地址
				maxImages: 9, // 最大上传图片数量
			};
		},

		methods: {
			//
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
			

			
			deleteImage(index) {
				this.imageList.splice(index, 1);
			},
			previewImage(index) {
				uni.previewImage({
					urls: this.imageList,
					current: this.imageList[index],
				});
			},


			uploadData() {
				// 首先上传图片
				this.uploadImages().then((imageUrls) => {
					// 图片上传成功后，将图片地址和其他数据一起发送给后端
					const dataToSend = {
						title: this.title,
						content: this.content,
						mood_score: this.mood,
						eat_score: this.appetite,
						sleep_score: this.sleep,
						images: imageUrls, // 上传后的图片地址数组
					};
					console.log(dataToSend);
					// 发送数据给后端服务器
					uni.request({
						url: 'http://82.157.244.44:8000/api/v1/diary/', // 后端接口地址
						method: 'POST', // POST 或者适合你的请求方式
						header: {
							'Authorization': `Bearer ${uni.getStorageSync('token')}`,

						},
						data: dataToSend,
						success: (res) => {
							console.log('数据发送成功:', res.data);

							uni.redirectTo({
								url: "/pages/diary/diary_index"
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
								url: 'http://82.157.244.44:8000/api/v1/diary/upload-image/', // 后端上传图片接口地址
								method: 'POST',
								filePath: imageUrl,
								name: 'image',
								header: {
									'Authorization': `Bearer ${uni.getStorageSync('token')}`,
								},
								success: (uploadRes) => {
									console.log('上传成功:', uploadRes.data);
									const imageFinalPath =
										"http://82.157.244.44:8000" + JSON.parse(
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
			}
		},
	};
</script>
<style lang="scss">
	
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
	
	.btn {
		display: flex;
		justify-content: center;
		align-items: center;
		width: calc(100% - 60rpx);
		height: 120rpx;
		margin-top: 20rpx;
		padding: 12px 20px;
		border-radius: 5px;
		background-color: #527853;
		color: #fff;
		text-align: center;
		font-size: 18px;
		/* 调整字体大小 */
		font-weight: bold;
		text-decoration: none;
		box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
		transition: transform 0.3s ease, background-color 0.3s ease, box-shadow 0.3s ease;
	}

	.btn-text {
		color: #fff;
		font-size: 16px;
		/* 调整字体大小 */
		font-weight: bold;
	}

	.tip {
		color: #888;
		/* Set the font color to gray */
		text-align: left;
		margin-bottom: -20rpx;
		font-size: 25rpx;
	}

	.image-grid {
		display: flex;
		flex-wrap: wrap;
	}

	.grid-item {
		width: calc(33.33% - 10px);
		/* 以三列排布，减去间隔 */
		margin-bottom: 10px;
		border: 2px solid #ccc;
		/* 添加边框样式 */
		box-sizing: border-box;
		/* 边框不增加宽度 */
		height: 200rpx
	}

	.container {
		padding: 10px;
		/* 设置容器与页面边界的距离 */
		border: 1px solid #ddd;
		/* 添加边框样式 */
	}

	.rate-section {
		display: flex;
		align-items: center;
		margin-bottom: 10px;
		padding: 10px;
		/* 设置每个区块的内边距 */
		border: 1px solid #ccc;
		/* 添加边框样式 */
		border-radius: 5px;
		/* 圆角边框 */
	}

	.rate-title {
		font-size: 18px;
		margin-right: 10px;
		color: #333;
		font-weight: bold;
	}

	.rate-value {
		margin-left: 10px;
		font-size: 16px;
		color: #666;
	}

	.input-section {
		margin-bottom: 20px;
		margin-left: 20rpx;
		margin-right: 30rpx;
	}

	.input-title {
		font-size: 18px;
		margin-bottom: 0px;
		margin-left: 10rpx;
		margin-right: 20rpx;
		color: #333;
	}


	.content-input {
		width: 90%;
		height: 200px;
		padding: 5%;
		border: 3rpx solid #ccc;
		border-radius: 5px;
		resize: none;
		/* 禁止调整文本框大小 */
		font-size: 16px;
		line-height: 1.5;
		background-color: white;
	}
</style>