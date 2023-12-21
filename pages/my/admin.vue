<template>
	<view>
		<view>
			<button class="btn" @click="all_report">
				<view class="button-content">
					<text class="btn-text">所有举报信息</text>
				</view>
			</button>

			<button class="btn" @click="handel_report">
				<view class="button-content">
					<text class="btn-text">已处理举报</text>
				</view>
			</button>

			<button class="btn" @click="not_handel_report">
				<view class="button-content">
					<text class="btn-text">未处理举报</text>
				</view>
			</button>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {

			};
		},
		methods: {

			all_report() {
				uni.navigateTo({
					url: "report_all"
				})
			},
			handel_report() {
				uni.navigateTo({
					url: "report_handel"
				})
			},
			not_handel_report() {
				uni.navigateTo({
					url: "report_unhandel"
				})
			},
		}
	};
</script>

<style>
	.btn {
		display: flex;
		justify-content: center;
		align-items: center;
		width: calc(100% - 60rpx);
		height: 120rpx;
		margin: 20px auto;
		padding: 12px 20px;
		border-radius: 5px;
		background-color: #3498db;
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
</style>