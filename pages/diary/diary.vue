<template>
	<view>
		<button>日记</button>
		<uni-section title="食欲" subTitle="今天有没有好好吃饭？" type="line" padding>
			<uni-rate size="18" :value="10" max="10" />
		</uni-section>
		<uni-section title="情绪" subTitle="今天的心情怎么样？" type="line" padding>
			<uni-rate size="18" :value="10" max="10" />
		</uni-section>
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

<style lang="scss">

</style>