<template>
	<view class="main-container">
	    <div class="title-box">
	      <p class="diary-title">{{ diary.title }}</p>
	    </div>
	    <p class="diary-date">{{ diary.create_time }}</p>
	
	    <div class="diary-content-box">
	      <p class="diary-content">{{ diary.content }}</p>
	    </div>
	
	    <div class="diary-details">
	      <div class="detail-container">
	        <p class="detail-label">Eat Score:</p>
	        <p class="detail-value">{{ diary.eat_score }}</p>
	      </div>
	      <div class="detail-container">
	        <p class="detail-label">Mood Score:</p>
	        <p class="detail-value">{{ diary.mood_score }}</p>
	      </div>
	      <div class="detail-container">
	        <p class="detail-label">Sleep Score:</p>
	        <p class="detail-value">{{ diary.sleep_score }}</p>
	      </div>
	    </div>
		<view class="image-grid" v-if="diary.images.length > 0">
			<image v-for="(imageUrl, index) in displayedImages" :key="index" :src="imageUrl" mode="aspectFit"
				class="grid-item" @longpress="showDeleteButton(index)"></image>
		</view>
	  </view>
</template>

<script>
	export default {
		data() {
			return {
				diary: {}, // 存储选定的日记数据
			};
		},
		onLoad(query) {
			const diaryId = query.diaryId;
			console.log(diaryId);
			this.fetchDiaryDetails(diaryId);
		},
		computed: {
			displayedImages() {
				// 显示的图片数量最多为9张
				return this.diary.images.slice(0, 9);
			},
			remainingImageCount() {
				// 剩余未显示的图片数量
				return Math.max(0, this.diary.images.length - 9);
			},
		},
		
		methods: {
			fetchDiaryDetails(diaryId) {
				// 使用 diaryId 获取特定日记条目的详细信息
				// 向后端 API 发送请求，获取基于 diaryId 的日记详情
				// 使用获取的日记详情更新 this.diary
				uni.request({
					url: 'http://82.157.244.44:8000/api/v1/diary/', // 后端接口地址
					method: 'GET',
					header: {
						'Authorization': `Bearer ${uni.getStorageSync('token')}`,
					},
					success: (res) => {
						console.log('数据接收成功:', res.data);
						console.log(diaryId);
						const diaries = res.data; // 将获取的日记信息存储到diaries数组中
						this.diary = res.data.find(a => a.id === Number(diaryId));
						console.log("这个日记的信息",this.diary);

					},
					fail: (err) => {
						console.error('数据发送失败:', err);
					}
				});
			}
		},
	};
</script>

<style>
.main-container {
    max-width: 800px;
    margin: 0 auto;
    padding: 20px;
    background-color: #f2f2f2;
    color: #333;
  }

  .title-box {
    border: 2px solid #f2f2f2;
    border-radius: 8px;
    padding: 15px;
    background-color: #ffffff;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    margin-bottom: 20px;
  }

  .diary-title {
    font-size: 28px;
    font-weight: bold;
    color: #0066cc;
    margin: 0;
  }

  .diary-date {
    color: #777;
    margin-bottom: 20px;
  }

  .diary-content-box {
    border: 2px solid #ff9900;
    border-radius: 8px;
    padding: 20px;
    background-color: #fff7e6;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    margin-bottom: 20px;
    overflow-wrap: break-word;
  }

  .diary-details {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
    gap: 20px;
    margin-bottom: 20px;
  }

  .detail-item {
    border: 2px solid #ffcc00;
    border-radius: 8px;
    padding: 15px;
    text-align: center;
    background-color: #fff9e6;
  }

  .detail-label {
    font-weight: bold;
    margin-bottom: 8px;
    color: #ff9900;
  }

  .detail-value {
    font-size: 18px;
    color: #333;
  }
  
    .diary-details {
      display: flex;
      justify-content: space-between;
      margin-bottom: 20px;
    }
  
    .detail-container {
      flex: 1;
      border: 2px solid #ffcc00;
      border-radius: 8px;
      padding: 15px;
      text-align: center;
      background-color: #fff9e6;
      margin-right: 10px;
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
</style>