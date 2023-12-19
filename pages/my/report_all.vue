<template>
	<view>
		<button @click="get_report">获取举报信息</button>
	</view>
</template>

<script>
import reportVue from '../msg/report.vue';
	export default {
		data() {
			return {
				report: [] ,
			}
		},
		methods: {
			get_report(){
				uni.request({
					url: `http://82.157.244.44:8000/api/v1/forum/reports/manage/all/`,
					method: "GET",
					header: {
						'Authorization': `Bearer ${uni.getStorageSync('token')}`,
					},
							
					success: (res) => {
						this.report = res.data.data;
						console.log(this.report);
						
							
					},
					fail: (err) => {
						console.log("a");
					}
				})
			}
		}
	}
</script>

<style>

</style>
