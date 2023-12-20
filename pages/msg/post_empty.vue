<template>
	<view class="deleted-message">
		<text class="deleted-text">该帖子已被删除</text>
	</view>
</template>

<script>
	export default {
		data() {
			return {

			}
		},
		methods: {

		}
	}
</script>

<style>
	.deleted-message {
		/* 设置显示位置为页面中间靠上一点 */
		position: absolute;
		top: 40%;
		/* 调整垂直位置 */
		left: 50%;
		transform: translate(-50%, -50%);
		text-align: center;
	}

	.deleted-text {
		/* 设置文本样式，加粗、字号较大 */
		font-weight: bold;
		font-size: 24px;
		/* 其他样式如颜色等可根据需求自行调整 */
	}
</style>