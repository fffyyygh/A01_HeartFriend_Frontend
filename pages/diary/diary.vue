<template>
	<view>
		<view class="container">
			<view class="rate-section">
				<text class="rate-title">食欲</text>
				<uni-rate :max="10" :value="10" v-model="appetite"  :size="25"
					active-color="#FFD700" :void-color="'#ccc'"></uni-rate>
				<text class="rate-value">{{ appetite }}</text>
			</view>
			<view class="rate-section">
				<text class="rate-title">心情</text>
				<uni-rate :max="10" :value="10" v-model="mood"  :size="25" active-color="#FFA07A"
					:void-color="'#ccc'"></uni-rate>
				<text class="rate-value">{{ mood }}</text>
			</view>
			<view class="rate-section">
				<text class="rate-title">睡眠</text>
				<uni-rate :max="10" :value="10" v-model="sleep"  :size="25" active-color="#87CEFA"
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
		
		<!-- 在记录日记的时候选择和显示图片的功能，后端的接口还没有开发好先搁置一下 -->
		<view>
			<view class="image-grid" v-if="imageUrls.length > 0">
				<image v-for="(imageUrl, index) in displayedImages" :key="index" :src="imageUrl" mode="aspectFit"
					class="grid-item" @longpress="showDeleteButton(index)"></image>
			</view>

			
			<uni-popup ref="popup" type="bottom">
				<view class="popup-content">
					<button class="delete-button" @tap="deleteImage">删除图片</button>
				</view>
			</uni-popup>
			<button @tap="chooseImage">选择图片</button>
		</view>
		
		
		<button @click="uploadData">提交</button>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				imageUrls: [], // 存储选择的图片地址数组
				appetite: 8,
				mood: 8,
				sleep: 8,
				title: '', // 保存用户输入的标题
				content: '', // 保存用户输入的内容
				deleteIndex: null // 保存需要删除的图片索引
			};
		},
		computed: {
			displayedImages() {
				// 显示的图片数量最多为9张
				return this.imageUrls.slice(0, 9);
			},
			remainingImageCount() {
				// 剩余未显示的图片数量
				return Math.max(0, this.imageUrls.length - 9);
			},
		},

		methods: {
			chooseImage() {
				uni.chooseImage({
					count: 9 - this.imageUrls.length, // 最多选择未选择的图片数量
					sizeType: ['compressed'], // 压缩图片
					sourceType: ['album', 'camera'], // 从相册选择或拍照
					success: (res) => {
						const tempFilePaths = res.tempFilePaths;						
						tempFilePaths.forEach((item,index)=>{
							wx.getFileSystemManager().saveFile({
								tempFilePath: item,
								success: async (saveRes) => {
									const savedFilePath = saveRes.savedFilePath;
									console.log(savedFilePath);
									this.imageUrls = this.imageUrls.concat(savedFilePath); // 将选择的图片添加到数组中
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

			showDeleteButton(index) {
				// 点击图片时展示删除按钮
				
				this.deleteIndex = index;
				this.$refs.popup.open();
				
			},

			deleteImage() {
				// 点击删除按钮时删除对应图片
				if (this.deleteIndex !== null) {
					this.imageUrls.splice(this.deleteIndex, 1); // 从数组中删除图片
					this.deleteIndex = null; // 重置删除索引
					this.$refs.popup.close();
				}
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
								url:"/pages/diary/diary_index"
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
					const promises = this.imageUrls.map((imageUrl) => {
						return new Promise((resolve, reject) => {
							console.log("上传图片地址",imageUrl);
							
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
									const imageFinalPath = "http://82.157.244.44:8000" + JSON.parse(uploadRes.data)["image"];
									console.log(imageFinalPath);
									uploadedImageUrls.push(imageFinalPath); // 将上传后的图片地址保存到数组中
									console.log("a",);
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
		font-size: 18px;
		line-height: 1.5;
		background-color: antiquewhite;
	}
</style>