<template>
  <div class="share">
    <p class="diggit" @click="praiseBlog(blogId)">
      <a href="javascript:void(0);">很赞哦！</a>
      <span v-if="praiseCount!== 0">
        (<b id="diggnum">{{praiseCount}}</b>)
      </span>
    </p>
    <p class="dasbox">
      <a href="javascript:void(0)" @click="dashangToggle()" class="dashang" title="打赏，支持一下">打赏我</a>
    </p>
    <div class="hide_box" v-if="showPay"></div>
    <div class="shang_box" v-if="showPay">
      <a class="shang_close" href="javascript:void(0)" @click="dashangToggle()" title="关闭">关闭</a>
      <div class="shang_tit">
        <p>感谢您的支持，我会继续努力的!</p>
      </div>
      <div class="shang_payimg">
        <img :src="payCode" alt="扫码支持" title="扫一扫">
      </div>
      <div class="pay_explain">扫码打赏，你说多少就多少</div>
      <div class="shang_payselect">
        <div
          class="pay_item"
          :class="[this.payMethod==='1'?'checked':'']"
          data-id="alipay"
          @click="choosePay('1')"
        >
          <span class="radiobox"></span>
          <span class="pay_logo">
            <img src="../../../public/images/alipay.jpg" alt="支付宝">
          </span>
        </div>
        <div
          class="pay_item"
          :class="[this.payMethod===2?'checked':'']"
          data-id="weipay"
          @click="choosePay('2')"
        >
          <span class="radiobox"></span>
          <span class="pay_logo">
            <img src="../../../public/images/wechat.jpg" alt="微信">
          </span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { getWebConfig } from "@/api/blog/home";
import {praiseBlogById, getBlogPraiseCount} from "@/api/blog/content";
import {mapMutations} from "vuex";
export default {
  name: "PayCode",
  props: {
    praiseCount: {
      type: Number,
      default: 0
    },
    blogId: {
      type: [Number, String]
    }
  },
  data() {
    return {
      praiseCountBak:0,
      webConfigData: [],
      showPay: false, //是否显示支付
      payMethod: 1, // 1: 支付宝  2：微信
      payCode: "", //支付码图片
    };
  },
  created() {
    getWebConfig().then(response => {
        this.webConfigData = response.data;
        this.payCode = this.webConfigData.aliPayPhoto;
    });
  },
  watch:{
    praiseCountBak(val){
      this.praiseCountBak = val
      this.$emit('addPraiseCount',this.praiseCountBak)
    }
  },
  methods: {
    //拿到vuex中的写的方法
    ...mapMutations(['setLoginMessage']),
    dashangToggle: function() {
      this.showPay = !this.showPay;
    },
    // 支付方式
    choosePay: function(type) {
      this.payMethod = type;
      if (type === '1') {
        this.payCode = this.webConfigData.aliPayPhoto;
      } else {
        this.payCode = this.webConfigData.weChatPayPhoto;
      }
    },
    //博客点赞
    praiseBlog: function(id) {
      // 判断用户是否登录
      let isLogin = this.$store.state.user.isLogin
      if(!isLogin) {
        this.$notify.error({
          title: '警告',
          message: '登录后才可以评论哦~',
          offset: 100
        });
        // 未登录，自动弹出登录框
        this.setLoginMessage(Math.random())
        return;
      }

      praiseBlogById(id).then(response => {

        this.$notify({
            title: '成功',
            message: "点赞成功",
            type: 'success',
            offset: 100}
            );
          this.praiseCountBak = response.data;
      });
    },
    //获取点赞数
    getPraiseCount: function(id) {
      params.append("id", id);
      getBlogPraiseCount(id).then(response => {
          this.praiseCount = response.data;
      });
    }
  }
};
</script>

<style>
  .diggit {
    cursor: pointer;
  }
</style>
