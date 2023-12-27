<template>
  <view class="container">
    <view class="image-container">
      <image :src="pic_url" mode="aspectFill" class="post-image"></image>
    </view>
    <text class="description">扫描上方二维码关注公众号以正常获取药物提醒</text>
  </view>
</template>

<script>
  export default {
    data() {
      return {
        pic_url: "",
      }
    },
    onLoad() {
      this.getpic();
    },
    methods: {
      getpic() {
        uni.request({
          url: 'https://vx.mikumikumi.xyz/api/v1/medicine/qrcode/',
          method: 'POST',
          header: {
            'Authorization': `Bearer ${uni.getStorageSync('token')}`,
          },
          success: (res) => {
            console.log('数据发送成功:', res.data);
            this.pic_url = res.data;
          },
          fail: (err) => {
            console.error('数据发送失败:', err);
            // Handle the failure case as needed
          }
        });
      }
    }
  }
</script>

<style>
  .container {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    height: 80vh; /* Adjust this according to your layout */
  }

  .image-container {
    margin-bottom: 20px; /* Adjust this spacing as needed */
  }

  .post-image {
    width: 200px; /* Set the width of your image */
    height: 200px; /* Set the height of your image */
    /* Add other styles as needed */
  }

  .description {
    font-size: 14px; /* Adjust the font size */
    color: #333; /* Adjust the color */
    /* Add other styles as needed */
  }
</style>
