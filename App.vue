<script>
	export default {
		onLaunch: function() {
			console.warn('当前组件仅支持 uni_modules 目录结构 ，请升级 HBuilderX 到 3.1.0 版本以上！')
			console.log('App Launch')
		},
		onShow: function() {
			console.log('App Show')
		},
		onHide: function() {
			console.log('App Hide')
		}
	}
</script>

<style lang="scss">
	@import "uview-ui/index.scss";
	/*每个页面公共css */
	@import '@/uni_modules/uni-scss/index.scss';
	/* 官方CSS库 */
	@import url("./common/uni.css");
	/* 自定义图标库 */
	@import url("./common/icon.css");
	/* 动画库 */
	@import url("./common/animate.css");
	/* 自定义样式库 */
	@import url("./common/base.css");
	/* 全局样式 */
	@import url("./common/common.css");
	/* #ifndef APP-NVUE */
	@import '@/static/customicons.css';

	// 设置整个项目的背景色
	page {
		
	}

	/* #endif */
	.example-info {
		font-size: 14px;
		color: #333;
		padding: 10px;
	}
</style>