<template>
  <view>
    <!-- 帖子详情页 -->
    <common-list
      :item="info"
      isdetail
      @doComment="doComment"
      @doShare="doShare"
      @follow="follow"
      @doSupport="doSupport"
    >
      <!-- <view>{{info.content}}</view>
      <view class="">
        <block v-for="(item, index) in info.images" :key="index">
          <image
            :src="item.url"
            class="w-100"
            mode="widthFix"
            @click="preview(index)"
          ></image>
        </block>
      </view> -->
    </common-list>
    <divider></divider>
    <!-- 评论 -->
    <view class="p-2 font-md font-weight-bold">最新评论 {{ info.comments.length }}</view>
    <view class="uni-comment-list px-2">
      <block v-for="(comment, index) in info.comments" :key="index">
        <view class="uni-comment-face">
          <image :src="comment.userpic" mode="widthFix"></image>
        </view>
        <view class="uni-comment-body">
          <view class="uni-comment-top">
            <text>{{ comment.username }}</text>
          </view>
          <view class="uni-comment-content">{{ comment.content }}</view>
          <view class="uni-comment-date">
            <view>{{ comment.date }}</view>
          </view>
        </view>
      </block>
    </view>
    <!-- 占位和评论输入框 -->
    <view style="height: 100rpx;"></view>
    <bottom-input @submit="submit"></bottom-input>
  </view>
</template>

<script>
import commonList from "@/components/common/common-list.vue";
import bottomInput from "@/components/common/bottom-input.vue";
export default {
  data() {
    return {
      // 当前帖子信息
      info: {},
    };
  },
  components: {
    commonList,
    bottomInput,
  },
  // computed: {
  // 	imagesList() {
  // 		return this.info.images.map(item => item.url);
  // 	}
  // },
  onLoad(e) {
    console.log("e.detail", e.detail);
    // 初始化操作
    if (e.detail) {
      this.__init(JSON.parse(e.detail));
    }
  },
  onNavigationBarButtonTap() {
    this.$refs.share.open();
  },
  onBackPress() {
    this.$refs.share.close();
  },
  methods: {
    __init(data) {
      // 修改标题
      uni.setNavigationBarTitle({
        title: data.title,
      });

      // 请求API
      this.info = {
        ...data,
        comments: [
          {
            username: "Alice",
            userpic: "/static/img/userpic/1.jpeg",
            content: "这是一条测试评论1",
            date: "1天前",
          },
          {
            username: "Bob",
            userpic: "/static/img/userpic/2.jpeg",
            content: "这是一条测试评论2",
            date: "2天前",
          },
          // 可以添加更多的测试评论
        ],
      };
    },
    // 关注
    follow() {
      this.info.isFollow = true;
      uni.showToast({
        title: "关注成功",
      });
    },
    // 顶踩操作
    doSupport(e) {
      let msg = e.type === "support" ? "顶" : "踩";
      // 之前操作过
      if (this.info.support.type === e.type) {
        return uni.showToast({
          title: "您已经" + msg + "过了",
        });
      }
      // 之前未操作过
      if (this.info.support.type === "") {
        this.info.support[e.type + "_count"]++;
      }
      // 之前已顶过并且现在的操作为踩，则顶-1、踩+1
      else if (
        this.info.support.type === "support" &&
        e.type === "unsupport"
      ) {
        this.info.support.support_count--;
        this.info.support.unsupport_count++;
      }
      // 之前已踩过并且现在的操作为顶，则踩-1、顶+1
      else if (
        this.info.support.type === "unsupport" &&
        e.type === "support"
      ) {
        this.info.support.unsupport_count--;
        this.info.support.support_count++;
      }
      this.info.support.type = e.type;
      uni.showToast({
        title: msg + "成功",
      });
    },
    // 点击评论
    doComment() {
      console.log("Commenting...");
    },
    // 点击分享
    doShare() {
      console.log("Sharing...");
    },
    // 预览图片
    preview(index) {
      uni.previewImage({
        urls: this.imagesList,
        current: index,
      });
    },
    // 提交评论
    submit(data) {
      console.log(data);
    },
  },
};
</script>

<style></style>
