<template>
  <div id="loginBackground" class="loginWrapper">
    <div class="formWrapper">
      <h1 class="loginTitle">{{ loginTitle }}</h1>
      <p class="loginSystem">{{ loginSystem }}</p>
      <el-form
        ref="ruleForm"
        :model="ruleForm"
        :rules="rules"
        label-width="100px"
        class="demo-ruleForm"
        hide-required-asterisk
      >
        <!-- <el-form-item>
          <el-alert type="success" description="线上演示环境使用奇文社区账号进行登录，网址：www.qiwenshare.com" />
        </el-form-item> -->
        <el-form-item prop="username">
          <el-input
            v-model="ruleForm.username"
            prefix-icon="el-icon-user"
            placeholder="用户名"
          />
        </el-form-item>
        <el-form-item prop="password">
          <el-input
            v-model="ruleForm.password"
            prefix-icon="el-icon-lock"
            placeholder="密码"
            show-password
          />
        </el-form-item>
        <el-form-item>
          <drag-verify
            ref="dragVerifyRef"
            text="请按住滑块拖动解锁"
            success-text="验证通过"
            handler-icon="el-icon-d-arrow-right"
            success-icon="el-icon-circle-check"
            width="375"
            handler-bg="#F5F7FA"
            :is-passing.sync="isPassing"
            @update:isPassing="updateIsPassing"
          />
        </el-form-item>
        <!-- <el-form-item class="forgetPassword">忘记密码</el-form-item> -->
        <el-form-item class="loginButtonWrapper">
          <el-button :loading="loading" class="loginButton" type="primary" :disabled="submitDisabled" @click.native.prevent="handleLogin">登录</el-button>
        </el-form-item>
        <!-- <el-form-item style="text-align:left;">
          其他账号登录：
          <a href="/api/user/login/qq">
            <img :src="qqIcon" style="width: 30px;">
          </a>
        </el-form-item> -->
      </el-form>
    </div>
  </div>
</template>

<script>
import CanvasNest from 'canvas-nest.js'
import DragVerify from '@/components/DragVerify.vue'

// 配置
const config = {
  color: '64, 158, 255', // 线条颜色
  pointColor: '64, 158, 255', // 节点颜色
  opacity: 0.5, // 线条透明度
  count: 99, // 线条数量
  zIndex: -1 // 画面层级
}

export default {
  name: 'Login',
  components: { DragVerify },
  data() {
    return {
      loginTitle: '登录',
      loginSystem: '离线数据同步平台',
      ruleForm: {
        username: '',
        password: ''
      },
      rules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 5, max: 20, message: '长度在 5 到 20 个字符', trigger: 'blur' }
        ]
      },
      isPassing: false,
      submitDisabled: true,
      loading: false
      // qqIcon: require('@/assets/images/login/qqIcon.png')
    }
  },
  computed: {
    url() {
      const _url = this.$route.query.Rurl // 获取路由前置守卫中next函数的参数，即登录后要去的页面
      if (_url) {
        // 若登录之前有页面，则登录后仍然进入该页面
        return _url
      } else {
        // 若登录之前无页面，则登录后进入首页
        return '/'
      }
    }
  },
  watch: {
    //  已验证通过时，若重新输入用户名或密码，滑动解锁恢复原样
    'ruleForm.username'() {
      this.isPassing = false
      this.$refs.dragVerifyRef.reset()
    },
    'ruleForm.password'() {
      this.isPassing = false
      this.$refs.dragVerifyRef.reset()
    },
    $route: {
      handler: function(route) {
        const query = route.query
        if (query) {
          this.redirect = query.redirect
          this.otherQuery = this.getOtherQuery(query)
        }
      },
      immediate: true
    }
  },
  created() {
    if (this.$store.getters.isLogin) {
      // 用户若已登录，自动跳转到首页
      const username = this.$store.getters.username
      this.$message({
        message: username + ' 您已登录！已跳转到首页',
        center: true,
        type: 'success'
      })
      this.$router.replace({ name: 'Home' })
    }
    this.$nextTick(() => {
      const element = document.getElementById('loginBackground')
      new CanvasNest(element, config)
    })
  },
  mounted() {
    if (this.loginForm.username === '') {
      this.$refs.username.focus()
    } else if (this.loginForm.password === '') {
      this.$refs.password.focus()
    }
  },
  destroyed() {
    // window.removeEventListener('storage', this.afterQRScan)
  },
  methods: {
    //  滑动解锁完成
    updateIsPassing(isPassing) {
      if (isPassing) {
        this.submitDisabled = false
      } else {
        this.submitDisabled = true
      }
    },
    checkCapslock({ shiftKey, key } = {}) {
      if (key && key.length === 1) {
        if (shiftKey && (key >= 'a' && key <= 'z') || !shiftKey && (key >= 'A' && key <= 'Z')) {
          this.capsTooltip = true
        } else {
          this.capsTooltip = false
        }
      }
      if (key === 'CapsLock' && this.capsTooltip === true) {
        this.capsTooltip = false
      }
    },
    handleLogin() {
      this.$refs.loginForm.validate(valid => {
        if (valid) {
          this.loading = true
          this.$store.dispatch('user/login', this.loginForm)
            .then(() => {
              this.$router.push({ path: this.redirect || '/', query: this.otherQuery })
              this.loading = false
            })
            .catch(() => {
              this.loading = false
            })
        } else {
          console.log('error submit!!')
          return false
        }
      })
    },
    getOtherQuery(query) {
      return Object.keys(query).reduce((acc, cur) => {
        if (cur !== 'redirect') {
          acc[cur] = query[cur]
        }
        return acc
      }, {})
    }
  }
}
</script>

<style lang="stylus" scoped>
.loginWrapper
  background-color #2d3a4b
  height 550px !important
  min-height 550px !important
  padding-top 100px
  .formWrapper
    width 375px
    margin 0 auto
    text-align center
    .loginTitle
      margin-bottom 10px
      font-weight 300
      font-size 30px
      color #000
    .loginSystem
      font-weight 300
      color #999
    .demo-ruleForm
      width 100%
      margin-top 20px
      >>> .el-form-item__content
        margin-left 0 !important
      &>>> .el-input__inner
        font-size 16px
      .forgetPassword
        text-align right
        margin -22px 0 0 0
      .loginButtonWrapper
        .loginButton
          width 100%
        &>>> .el-button
          padding 10px 90px
          font-size 16px
    .tip
      width 70%
      margin-left 86px
</style>
