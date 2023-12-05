<template>
	<view>
		<!-- 头部 -->
		<view class="flex align-center p-3 border-bottom">
			<image src="/static/img/userpic/1.jpeg" style="width: 180rpx; height: 180rpx;" class="rounded-circle">
			</image>
			<view class="pl-3 flex flex-column flex-1">
				<view class="flex align-center">
					<view class="flex-1 flex flex-column align-center justify-center">
						<text class="font-lg font-weight-bold">2</text>
						<text class="font text-muted">获赞</text>
					</view>
					<view class="flex-1 flex flex-column align-center justify-center">
						<text class="font-lg font-weight-bold">1</text>
						<text class="font text-muted">关注</text>
					</view>
					<view class="flex-1 flex flex-column align-center justify-center">
						<text class="font-lg font-weight-bold">6</text>
						<text class="font text-muted">粉丝</text>
					</view>
				</view>
				<view class="flex align-center justify-center">
					<button type="primary" size="mini" class="bg-main" style="width: 400rpx;">关注</button>
				</view>
			</view>
		</view>

		<!-- 选项卡 -->
		<view class="flex align-center" style="height: 100rpx;">
			<view class="flex-1 flex align-center justify-center" v-for="(item, index) in tabBars" :key="index"
			    :class="index === tabIndex ? 'font-lg font-weight-bold text-main' : 'font-md'"
			    @click="changeTab(index)">{{item.name}}</view>
		</view>

		<template v-if="tabIndex === 0">
			<view class="animate__animated animate__fast animate__fadeIn">
				<view class="p-3 border-bottom">
					<view class="font-md">账号信息</view>
					<view class="font">账号年龄：3个月</view>
					<view class="font">账号id：1</view>
				</view>
				<view class="p-3 border-bottom">
					<view class="font-md">个人信息</view>
					<view class="font">星座：天蝎座</view>
					<view class="font">职业：IT</view>
					<view class="font">家乡：江苏苏州</view>
					<view class="font">情感：保密</view>
				</view>
			</view>
		</template>
		<template v-else>
			<view class="animate__animated animate__fast animate__fadeIn">
				<block v-for="(item, index) in list" :key="index">
					<common-list :item="item" :index="index" @follow="follow" @doSupport="doSupport"></common-list>
					<divider></divider>
				</block>
				<load-more :loadmore="loadmore"></load-more>
			</view>
		</template>

		<!-- 弹出层 -->
		<!-- 		<uni-popup ref="popup" type="top">
			<view class="popup-content flex align-center justify-center font-md border-bottom" hover-class="bg-light">
				<text class="iconfont icon-heimingdan mr-2"></text> 加入黑名单
			</view>
			<view class="popup-content flex align-center justify-center font-md" hover-class="bg-light">
				<text class="iconfont icon-conversation_icon mr-2"></text> 聊天
			</view>
		</uni-popup> -->
	</view>
</template>

<script>
	const test_data = [{
			username: "Carol",
			userpic: "/static/img/userpic/1.jpeg",
			newstime: "2021-01-24 上午11:30",
			isFollow: false,
			title: "绿意 希望 光",
			titlepic: "/static/img/datapic/1.jpeg",
			support: {
				type: "support", // 顶
				support_count: 1,
				unsupport_count: 2
			},
			comment_count: 2,
			share_count: 2
		},
		{
			username: "Bob",
			userpic: "/static/img/userpic/2.jpeg",
			newstime: "2021-01-24 下午14:00",
			isFollow: false,
			title: "一哭就停不下来，甚至需要伤害自己才可以冷静，这是为什么？",
			support: {
				type: "unsupport", // 踩
				support_count: 2,
				unsupport_count: 3
			},
			comment_count: 5,
			share_count: 1
		},
		{
			username: "Alice",
			userpic: "/static/img/userpic/3.jpeg",
			newstime: "2021-01-24 下午14:44",
			isFollow: true,
			title: "或许明天会更好，我只需一步一步地前进，直到再次看到阳光。",
			titlepic: "/static/img/datapic/3.jpeg",
			support: {
				type: "", // 未操作
				support_count: 2,
				unsupport_count: 7
			},
			comment_count: 0,
			share_count: 2
		},
		{
			username: "Dave",
			userpic: "/static/img/userpic/4.jpeg",
			newstime: "2021-01-24 下午18:20",
			isFollow: true,
			title: "好困不想起床不想起床不想起床不想起床不想起床",
			titlepic: "/static/img/datapic/4.jpeg",
			support: {
				type: "support",
				support_count: 5,
				unsupport_count: 1
			},
			comment_count: 3,
			share_count: 0
		}
	];
	import commonList from '@/components/common/common-list.vue';
	import loadMore from '@/components/common/load-more.vue';
	// import uniPopup from '@/components/uni-ui/uni-popup/uni-popup.vue';
	export default {
		data() {
			return {
				tabIndex: 0,
				tabBars: [{
						name: '主页'
					},
					{
						name: '帖子',
						list: test_data,
						loadmore: '上拉加载更多'
					},
					{
						name: '动态',
						list: test_data,
						loadmore: '上拉加载更多'
					}
				],
			}
		},
		components: {
			commonList,
			loadMore
			// uniPopup
		},
		computed: {
			list() {
				return this.tabBars[this.tabIndex].list;
			},
			loadmore() {
				return this.tabBars[this.tabIndex].loadmore;
			}
		},
		// onNavigationBarButtonTap() {
		// 	this.$refs.popup.open();
		// },
		methods: {
			changeTab(index) {
				this.tabIndex = index;
			},
			// 关注
			follow(e) {
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
				console.log(0);
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
		}
	}
</script>

<style>

</style>