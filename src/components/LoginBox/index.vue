<template>
  <div>
    <div class="box loginBox" v-if="showLogin == true">
      <div class="title"  >
        <span class="t1">
          登录
        </span>
        <div class="t2" @click="closeLogin()">
          X
        </div>
      </div>
      <el-divider></el-divider>
      <el-form :label-position="labelPosition" :rules="loginRules" :model="loginForm" ref="loginForm">
        <el-form-item label="用户名" prop="userName">
          <el-input v-model="loginForm.userName" placeholder="请输入用户名或邮箱" :disabled="loginType.password"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input type="password" v-model="loginForm.password" placeholder="请输入密码" :disabled="loginType.password"></el-input>
        </el-form-item>
        <el-row class="btn">
          <el-button class="loginBtn" type="primary" @click="startLogin" :disabled="loginType.password">登录</el-button>
          <el-button class="registerBtn" type="info" @click="goRegister" :disabled="loginType.password">注册</el-button>
        </el-row>

        <el-row class="elRow">
          <el-tooltip content="码云" placement="bottom">
            <el-button type="danger" circle @click="goAuth('gitee')" :disabled="loginType.gitee">
              <span class="iconfont">&#xe602;</span>
            </el-button>
          </el-tooltip>

          <el-tooltip content="Github" placement="bottom">
            <el-button type="info" circle @click="goAuth('github')" :disabled="loginType.github">
              <span class="iconfont">&#xe64a;</span>
            </el-button>
          </el-tooltip>

          <el-tooltip content="QQ" placement="bottom">
            <el-button type="primary" circle @click="goAuth('qq')" :disabled="loginType.qq">
              <span class="iconfont">&#xe601;</span>
            </el-button>
          </el-tooltip>


        </el-row>
        <div class="loginTip">目前登录方式支持
          <span v-if="!loginType.password"> 账号密码 </span>
          <span v-if="!loginType.gitee"> 码云 </span>
          <span v-if="!loginType.github"> Github </span>
          <span v-if="!loginType.qq"> QQ </span>
          <span v-if="!loginType.weibo"> 微博 </span>
        </div>
      </el-form>
    </div>

    <!--    注册-->
    <div class="box registerBox" v-if="showLogin == false">
      <div class="title">
        <span class="t1">
          注册
        </span>
        <div class="t2" @click="closeLogin()">
          X
        </div>
      </div>
      <el-divider></el-divider>
      <el-form :rules="rules" :label-position="labelPosition" :model="registerForm" ref="registerForm">
        <el-form-item label="用户名" prop="userName">
          <el-input v-model="registerForm.userName" placeholder="用户名长度在5~20之间" :disabled="loginType.password"></el-input>
        </el-form-item>

        <el-form-item label="昵称" prop="nickName">
          <el-input v-model="registerForm.nickName" :disabled="loginType.password"></el-input>
        </el-form-item>

        <el-form-item label="密码" prop="password">
          <el-input type="password" v-model="registerForm.password" :disabled="loginType.password"></el-input>
        </el-form-item>

        <el-form-item label="重复密码" prop="password2">
          <el-input type="password" v-model="registerForm.password2" :disabled="loginType.password"></el-input>
        </el-form-item>

        <el-form-item label="邮箱" prop="email">
          <el-input v-model="registerForm.email" :disabled="loginType.password"></el-input>
        </el-form-item>

        <el-row class="btn">
          <el-button class="loginBtn" type="primary" @click="startRegister" :disabled="loginType.password">注册</el-button>
          <el-button class="registerBtn" type="info" @click="goLogin" :disabled="loginType.password">返回登录</el-button>
        </el-row>

        <div class="loginTip">注册后，需要到邮箱进行邮件认证~</div>
      </el-form>
    </div>

    <div class="mask"></div>

  </div>
</template>

<script>

  import {getWebConfig} from "../../api/blog/home";
  import {authLogin} from "../../api/blog/user";
  import { Loading } from 'element-ui';
  export default {
    name: "loginBox",
    data() {
      return {
        loading: null,
        option: {
          fullscreen: true,
          lock: true
        },
        // 显示登录页面
        showLogin: true,
        isLogin: false,
        table: false,
        dialog: false,
        labelPosition: "right",
        loginForm: {
          userName: "",
          password: ""
        },
        registerForm: {
          userName: "",
          password: "",
          password2: "",
          email: ""
        },
        // 登录类别
        loginType: {
          password: true,
          gitee: true,
          github: true,
          qq: true,
          weibo: true
        },
        loginRules: {
          userName: [
            {required: true, message: '请输入用户名', trigger: 'blur'},
            { min: 5, message: "用户名长度大于等于 5 个字符", trigger: "blur" },
            { max: 20, message: "用户名长度不能大于 20 个字符", trigger: "blur" }
          ],
          password: [
            { required: true, message: "请输入密码", trigger: "blur" },
            { min: 5, message: "密码长度需要大于等于 5 个字符", trigger: "blur" },
            { max: 20, message: "密码长度不能大于 20 个字符", trigger: "blur" }
          ]
        },
        rules: {
          userName: [
            {required: true, message: '请输入用户名', trigger: 'blur'},
            { min: 5, message: "用户名长度大于等于 5 个字符", trigger: "blur" },
            { max: 20, message: "用户名长度不能大于 20 个字符", trigger: "blur" }
          ],
          nickName: [
            {required: true, message: '请输入昵称', trigger: 'blur'}
          ],
          password: [
            { required: true, message: "请输入密码", trigger: "blur" },
            { min: 5, message: "密码长度需要大于等于 5 个字符", trigger: "blur" },
            { max: 20, message: "密码长度不能大于 20 个字符", trigger: "blur" }
          ],
          password2: [
            { required: true, message: "请再次输入密码", trigger: "blur" },
            { min: 5, message: "密码长度需要大于等于 5 个字符", trigger: "blur" },
            { max: 20, message: "密码长度不能大于 20 个字符", trigger: "blur" }
          ],
          email: [
            { required: true, message: "邮箱不能为空", trigger: "blur" },
            {pattern: /\w[-\w.+]*@([A-Za-z0-9][-A-Za-z0-9]+\.)+[A-Za-z]{2,14}/, message: '请输入正确的邮箱'},
          ]
        }
      };
    },
    components: {},
    created() {
      this.setLoginTypeList()
    },
    methods: {
      setLoginTypeList: function() {
        //获取登录方式
        getWebConfig().then(response => {
          let webConfigData  = response.data;
          if(webConfigData.loginTypeList !== undefined) {
            let loginTypeList = JSON.parse(webConfigData.loginTypeList)
            for(let a=0; a<loginTypeList.length; a++) {
              switch (loginTypeList[a]) {
                case "1": {
                  this.loginType.password = false
                } break;
                case "2": {
                  this.loginType.gitee = false
                } break;
                case "3": {
                  this.loginType.github = false
                } break;
                case "4": {
                  this.loginType.qq = false
                } break;
                case "5": {
                  this.loginType.weibo = false
                } break;
                default: {
                  console.log("登录方式设置有误！！")
                }
              }
            }
          }
        });
      },
      startLogin: function () {
        //TODO: 登录
      },
      startRegister: function () {
        //TODO 注册
      },
      goLogin: function () {
        this.showLogin = true;
      },
      goRegister: function () {
        this.showLogin = false;
      },
      goAuth: function (source) {
        this.loading = Loading.service({
          lock: true,
          text: '加载中……',
          background: 'rgba(0, 0, 0, 0.7)'
        })
        authLogin(source).then(response => {
          window.location.href = response.data.url
        });
      },
      closeLogin: function() {
        this.$emit("closeLoginBox", "");
      }
    }
  };
</script>


<style>
  .box {
    width: 400px;
    height: 420px;
    background: white;
    position: fixed;
    margin: auto;
    left: 0;
    right: 0;
    top: 0;
    bottom: 0;
    z-index: 101; /* 要比遮罩层大 */
  }

  .registerBox {
    height: 660px;
  }

  .box .title {
    height: 48px;
    font-size: 22px;
    font-weight: bold;
    text-align: center;
    line-height: 48px;
  }
  .box .title .t2 {
    font-size: 16px;
    float: right;
    margin-right: 6px;
    margin-top: -6px;
    cursor: pointer;
  }

  .box .el-divider--horizontal {
    margin: 12px 0;
  }

  .box .el-form-item__label {
    margin-left: 10px;
    font-size: 16px;
  }

  .box .el-input__inner {
    margin-left: 10px;
    width: 90%;
  }

  .box .btn {
    text-align: center;
  }

  .box .loginBtn {
    width: 40%;
  }

  .box .registerBtn {
    width: 40%;
  }

  .elRow {
    margin-top: 15px;
    text-align: center;
  }

  .loginTip {
    margin-top: 10px;
    font-size: 14px;
    text-align: center;
    color: #bababa;
  }

  .remarksBox {
    position: fixed;
    left: 50%;
    margin-left: -100px;
    top: 50%;
    margin-top: -50px;
    border: 1px solid red;
    width: 200px;
    height: 100px;
    text-align: center;
    z-index: 101; /* 要比遮罩层大 */
  }

  /* 遮罩层 */
  .mask {
    position: fixed;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.5);
    z-index: 100;
  }
</style>
