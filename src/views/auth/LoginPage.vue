<template>
  <div>
    <el-container id="auth-container">
      <el-main>
        <div id="logo-name" class="animated slideInLeft">
          {{ $store.state.website_name }}
        </div>
        <div id="login-box">
          <div class="header">快捷登录</div>
          <div class="main">
            <el-form ref="form" :model="form" :rules="rules">
              <el-form-item prop="username">
                <el-input
                  v-model="form.username"
                  placeholder="手机号"
                  class="cuborder-radius"
                  maxlength="11"
                  @keyup.enter.native="onSubmit('form')"
                />
              </el-form-item>
              <el-form-item prop="password">
                <el-input
                  v-model="form.password"
                  type="password"
                  placeholder="密码"
                  class="cuborder-radius"
                  @keyup.enter.native="onSubmit('form')"
                />
              </el-form-item>
              <el-form-item>
                <el-button
                  type="primary"
                  @click="onSubmit('form')"
                  class="submit-btn"
                  :loading="loginLoading"
                  >立即登录
                </el-button>
              </el-form-item>
              <el-form-item>
                <div class="links">
                  <el-link
                    type="primary"
                    @click="toLink('/forget')"
                    :underline="false"
                    >找回密码</el-link
                  >
                  <el-link
                    type="primary"
                    @click="toLink('/register')"
                    :underline="false"
                    >还没有账号？立即注册</el-link
                  >
                </div>
              </el-form-item>

              <p style="margin-top: 50px">
                <el-divider>
                  <span style="color: rgb(181, 176, 176); font-weight: 200">
                    <i class="el-icon-mobile-phone"></i> 预览账号</span
                  >
                </el-divider>
              </p>
              <el-form-item class="preview-account">
                <p>预览账号:18798272054 / 密码: admin123</p>
                <p>预览账号:18798272055 / 密码: admin123</p>
              </el-form-item>
            </el-form>
          </div>
        </div>
        <div class="copyright" v-html="$store.state.copyright"></div>
      </el-main>
    </el-container>

    <div class="fly-box">
      <div class="fly bg-fly-circle1"></div>
      <div class="fly bg-fly-circle2"></div>
      <div class="fly bg-fly-circle3"></div>
      <div class="fly bg-fly-circle4"></div>
    </div>
  </div>
</template>
<script>
import { setToken } from "@/utils/auth";
import { isMobile } from "@/utils/validate";
import { ServeLogin } from "@/api/user";

export default {
  name: "LoginPage",
  data() {
    let validateMobile = (rule, value, callback) => {
      if (value === "") {
        callback(new Error("登录手机号不能为空！"));
      } else {
        if (!isMobile(value)) {
          callback(new Error("登录手机号格式不正确！"));
        } else {
          callback();
        }
      }
    };

    return {
      loginLoading: false,
      form: {
        username: "",
        password: "",
      },
      rules: {
        username: [
          {
            validator: validateMobile,
            trigger: "blur",
          },
          {
            min: 11,
            max: 11,
            message: "手机号格式不正确!",
            trigger: "blur",
          },
        ],
        password: [
          {
            required: true,
            message: "登录密码不能为空!",
            trigger: "blur",
          },
        ],
      },
    };
  },
  methods: {
    onSubmit(formName) {
      if (this.loginLoading) {
        return false;
      }

      this.$refs[formName].validate((valid) => {
        if (!valid) return false;
        this.loginLoading = true;
        this.login();
      });
    },

    login() {
      ServeLogin({
        mobile: this.form.username,
        password: this.form.password,
        platform: "web",
      })
        .then((res) => {
          this.loginLoading = false;
          if (res.code == 200) {
            let result = res.data;

            // 保存授权信息到本地缓存
            setToken(
              result.authorize.access_token,
              result.authorize.expires_in
            );

            this.$store.commit("UPDATE_USER_INFO", result.userInfo);
            this.$store.commit("UPDATE_LOGIN_STATUS");

            this.$root.initialize();
            this.$router.push({
              path: "/",
            });

            setTimeout(() => {
              this.$notify({
                title: "友情提示",
                message:
                  "此站点仅供演示、学习所用，请勿进行非法操作、上传或发布违法资讯。",
                duration: 0,
              });
            }, 3000);
          } else {
            this.$notify.info({
              title: "提示",
              message: "登录密码不正确或账号不存在...",
            });
          }
        })
        .catch((err) => {
          this.loginLoading = false;
        });
    },

    toLink(url) {
      this.$router.push({
        path: url,
      });
    },
  },
};
</script>
<style lang="less" scoped>
@import "~@/assets/css/page/login-auth.less";
</style>