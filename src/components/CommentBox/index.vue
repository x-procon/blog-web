<template>
  <div>
    <div class="commentBox">
    <span class="left">
      <img :src="getUserPhoto" onerror="onerror=null;src='https://typora-photo.oss-cn-shenzhen.aliyuncs.com/%E5%A4%B4%E5%83%8F/g.jpg'"/>
    </span>
      <span class="right">
      <textarea id="textpanel" class="textArea" placeholder="既然来了，那就留下些什么吧~" v-model="value" @click="hideEmojiPanel" @input="validCount"></textarea>
    </span>
    </div>
    <div class="bottom">
      <el-button class="submit p2" type="primary"  @click="handleSubmit">发送评论</el-button>
      <el-button class="cancel p2" type="info" @click="handleCancel">取消评论</el-button>
      <div class="emoji-panel-btn p2" @click="showEmojiPanel">
        <img src="../../assets/img/face_logo.png" />
      </div>
      <span class="allow p2">还能输入{{count}}个字符</span>

      <emoji-panel class="emojiPanel" @emojiClick="appendEmoji" v-if="isShowEmojiPanel"></emoji-panel>

    </div>
  </div>
</template>

<script>
import {dateFormat} from '../../utils/webUtils';
import {mapGetters, mapMutations} from 'vuex';
import EmojiPanel from "@/components/EmojiPanel/EmojiPanel";
export default {
  name: "CommentBox",
  props: {
    userInfo: {
      type: Object
    },
    // 回复的对象
    toInfo: {
      type: Object
    },
    // 博客信息
    commentInfo: {
      type: Object
    },
    showCancel: {
      type: Boolean,
      default: true
    }
  },
  components: {
    EmojiPanel
  },

  data() {
    return {
      comments: [],
      submitting: false,
      value: '',
      user: {},
      count: 1024,
      isShowEmojiPanel: false, // 是否显示表情面板
    }
  },
  computed: {
    ...mapGetters(['getUserPhoto'])
  },
  mounted() {
    // 页面加载的时候调用监听
    this.hideEmojiPanel()
  },
  methods: {
    //拿到vuex中的写的方法
    ...mapMutations(['setLoginMessage']),
    validCount: function() {
      let count = 1024 - this.value.length;
      if(count <= 0) {
        this.count = 0
      } else {
        this.count = count;
      }
    },
    handleSubmit() {

      let info = this.$store.state.user.userInfo
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

      if(this.value ==="") {
        this.$notify.error({
          title: '警告',
          message: '评论内容不能为空哦~',
          offset: 100
        });
        return;
      }

      let userId = info.id;
      let toUserId = "";
      let toCommentId = "";
      let blogId = "";

      // 评论来源： MESSAGE_BOARD，ABOUT，BLOG_INFO 等 代表来自某些页面的评论
      let source = "";
      // 替换表情
      console.log("替换后", this.value.replace(/:.*?:/g, this.emoji))
      let content = this.value.replace(/:.*?:/g, this.emoji);
      if(this.toInfo) {
        toUserId = this.toInfo.id;
        toCommentId = this.toInfo.commentId;
      }
      if(this.commentInfo) {
        blogId = this.commentInfo.blogId;
        source = this.commentInfo.source;
      }

      this.comments = {
        userId: userId,
        toCommentId: toCommentId,
        toUserId: toUserId,
        content: content,
        blogId: blogId,
        source: source,
        reply: [],
      }
      this.value = '';
      this.count = 1024;
      this.comments.createTime = dateFormat("YYYY-mm-dd HH:MM:SS", new Date());
      this.hideEmojiPanel()
      this.$emit("submit-box", this.comments)
    },
    emoji(word) {
      // 生成html
      const type = word.substring(1, word.length - 1);
      return `<span class="emoji-item-common emoji-${type} emoji-size-small"></span>`;
    },
    handleCancel() {
      this.value = '';
      this.count = 1024;
      this.isShowEmojiPanel = false
      if(this.toInfo) {
        this.$emit("cancel-box", this.toInfo.commentId)
      }
      this.hideEmojiPanel()
    },
    showEmojiPanel() {
      this.isShowEmojiPanel = !this.isShowEmojiPanel;
    },
    hideEmojiPanel() {
      this.isShowEmojiPanel = false
    },
    appendEmoji(text) {
      this.value = this.value + ":" + text + ":";
    }
  },
}
</script>

<style scoped>
@import "../../assets/css/emoji.css";
.emoji-panel-wrap {
  box-sizing: border-box;
  border: 1px solid #cccccc;
  border-radius: 5px;
  background-color: #ffffff;
  width: 650px;
  height: 135px;
  position: absolute;
  z-index: 999;
  left: 50px;
  top: 10px;
}
.emoji-size-small {
  zoom: 0.3;
  margin: 5px;
  vertical-align: middle;
}
.emoji-size-large {
  zoom: 0.5;
  margin: 5px;
}
.commentBox {
  min-width: 700px;
  width: 100%;
  height: 100px;
  margin: 0 auto;
}
.commentBox .left {
  display: inline-block;
  width: 4%;
  height: 100%;
  padding-top: 3px;
}
.commentBox .left img {
  cursor: pointer;
  margin: 0 auto;
  width: 90%;
  border-radius: 50%;
}
.commentBox .right {
  display: inline-block;
  margin: 5px 2px 0 0;
  width: 95%;
  height: 100%;
}
textarea::-webkit-input-placeholder {
  color: #909399;
}
.commentBox .right textarea {
  color: #606266;
  padding:10px 5px 5px 10px;
  resize: none;
  width: 95%;
  height: 100%;
}
.bottom {
  position: relative;
  min-width: 690px;
  width: 98%;
  height: 60px;
  line-height: 40px;
  margin-top: 20px;
}
.bottom .p2 {
  float: right;
  margin-top: 5px;
  margin-right: 10px;
}
.emoji-panel-btn img{
  height: 35px;
  width: 35px;
}
.emoji-panel-btn:hover {
  cursor: pointer;
  opacity: 0.8;
}
.emoji-item-common {
  background: url("../../assets/img/emoji_sprite.png");
  display: inline-block;
}
.emoji-item-common:hover {
  cursor: pointer;
}
.emoji-size-small {
  zoom: 0.3;
}
</style>
