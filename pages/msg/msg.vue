<template>
	<view>
		<!-- 顶部选项卡 -->
		<scroll-view scroll-x="true" class="scroll-row" :scroll-into-view="scrollInto" :scroll-with-animation="true"
			style="height: 100rpx;">
			<view v-for="(item, index) in tabBars" :key="index" class="scroll-row-item px-3 py-2 font-md"
				:id="'tab'+index" :class="tabIndex === index ? 'text-main font-lg font-weight-bold' : ''"
				@click="changeTab(index)">{{item.name}}</view>
		</scroll-view>
		<!-- 滑块 -->
		<swiper :duration="150" :current="tabIndex" @change="onChangeTab" :style="'height: '+scrollH+'px;'">
			<swiper-item v-for="(item, index) in newsList" :key="index">
				<scroll-view scroll-y="true" :style="'height: '+scrollH+'px;'" @scrolltolower="loadMore(index)">
					<!-- 有数据 -->
					<template v-if="item.list.length > 0">
						<!-- 列表 -->
						<block v-for="(item2, index2) in item.list" :key="index2">
							<!-- 列表组件 -->
							<common-list :item="item2" :index="index2" @follow="follow" @doSupport="doSupport"
								@openDetail="openDetail"></common-list>
							<!-- 全局分割线 -->
							<divider></divider>
						</block>
						<!-- 上拉加载 -->
						<load-more :loadmore="item.loadmore"></load-more>
					</template>
					<!-- 无数据 -->
					<template v-else>
						<no-thing></no-thing>
					</template>
				</scroll-view>
			</swiper-item>
		</swiper>
		<!-- 发贴入口 -->
		<add-post-tag></add-post-tag>
	</view>
</template>

<script>
	import commonList from '@/components/common/common-list.vue';
	import loadMore from '@/components/common/load-more.vue';
	import addPostTag from '@/components/add-post-tag/add-post-tag.vue';

	export default {
		components: {
			commonList,
			loadMore,
			addPostTag
		},
		data() {
			return {
				newsList: [],
				// 顶部选项卡
				tabBars: [{
						name: '关注'
					},
					{
						name: '热门'
					},
					{
						name: '推荐'
					},
					{
						name: '最新'
					},
					{
						name: '随机'
					},
					{
						name: '赞过'
					},
					{
						name: '踩过'
					}

				],
				tabIndex: 0,
				scrollInto: '',
				// 列表高度
				scrollH: 600,
				list: []
			}
		},
		onLoad() {
			uni.getSystemInfo({
				success: (res) => {
					console.log(res);
					this.scrollH = res.windowHeight - uni.upx2px(100);
				}
			});
			// 根据选项生成列表
			this.getData();
		},
		methods: {
			// 关注
			follow(e) {
				console.log('Index followed');
				console.log(e);
				this.list[e].isFollow = true;
				uni.showToast({
					title: '关注' + this.list[e].username + '成功'
				})
			},
			// 顶踩操作
			doSupport(e) {
				console.log(e);
				// 获取当前列表项
				let item = this.list[e.index];
				let msg = e.type === 'support' ? '顶' : '踩';
				// 之前未顶踩过
				if (item.support.type === '') {
					console.log(1);
					item.support[e.type + '_count']++;
				}
				// 之前已顶过并且现在的操作为踩，则顶-1、踩+1
				else if (item.support.type === 'support' && e.type === 'unsupport') {
					console.log(2);
					item.support.support_count--;
					item.support.unsupport_count++;
				}
				// 之前已踩过并且现在的操作为顶，则踩-1、顶+1
				else if (item.support.type === 'unsupport' && e.type === 'support') {
					console.log(3);
					item.support.unsupport_count--;
					item.support.support_count++;
				}
				item.support.type = e.type;
				uni.showToast({
					title: msg + '成功'
				})
			},
			// 切换选项
			changeTab(index) {
				if (this.tabIndex === index) {
					return;
				}
				this.tabIndex = index;
				this.list = this.newsList[this.tabIndex].list;
				// 滚动到指定元素
				this.scrollInto = 'tab' + index;
			},
			// 监听滑动
			onChangeTab(e) {
				console.log(e);
				this.changeTab(e.detail.current);
			},

			// 获取数据
			// getData() {
			// 	var arr = [];
			// 	for (let i = 0; i < this.tabBars.length; i++) {
			// 		// 生成列表模板
			// 		let obj = {
			// 			// 3种状态：1.上拉加载更多；2.加载中...；3.没有更多了。
			// 			loadmore: "上拉加载更多",
			// 			list: []
			// 		}
			// 		if (i < 2) {
			// 			obj.list = test_data;
			// 		}
			// 		arr.push(obj)
			// 	}
			// 	this.newsList = arr;
			// 	this.list = this.newsList[this.tabIndex].list;
			// },

			// 获取数据
			async getData() {
				try {
					const response = await uni.request({
						url: 'https://vx.mikumikumi.xyz/api/v1/forum/posts/',
						method: 'GET',
						header: {
							'Authorization': 'Bearer ' + uni.getStorageSync('token'),
						},
					});

					if (Array.isArray(response) && response[1] && Array.isArray(response[1].data)) {
						// 处理帖子列表数据
						this.newsList = response[1].data.map(item => ({
							loadmore: '上拉加载更多',
							list: item.map(post => ({
								...post,
								// 你可能需要调整这里的数据结构，以适应你的 common-list.vue 组件
							})),
						}));
					} else {
						console.error('Invalid response format:', response);
						console.log('response[1]:', response[1]);
					}
				} catch (error) {
					console.error('Failed to fetch posts', error);
				}
			},

			// 上拉加载更多
			loadMore(index) {
				// 获取当前列表
				let item = this.newsList[index];
				// 判断是否处于可加载状态
				if (item.loadmore !== '上拉加载更多') return;
				// 修改当前列表加载状态
				item.loadmore = '加载中...';
				// 模拟数据请求
				setTimeout(() => {
					// 加载数据
					item.list = [...item.list, ...item.list];
					// 恢复加载状态
					this.newsList[index].loadmore = '上拉加载更多';
				}, 2000)
			}
		}
	}
</script>

<style lang="scss">

</style>