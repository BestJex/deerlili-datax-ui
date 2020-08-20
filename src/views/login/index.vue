<template>
  <div id="loginBackground" class="loginWrapper">
    <div class="formWrapper">
      <h1 class="loginTitle">DATAX-WEB</h1>
      <p class="loginSystem">离线数据同步平台</p>
      <el-form
        ref="loginForm"
        :model="loginForm"
        :rules="loginRules"
        label-width="100px"
        hide-required-asterisk
      >
        <el-form-item prop="username">
          <el-input
            v-model="loginForm.username"
            prefix-icon="el-icon-user"
            placeholder="用户名"
            style="width: 375px;margin-left: -100px;"
          />
        </el-form-item>
        <el-form-item prop="password">
          <el-input
            v-model="loginForm.password"
            prefix-icon="el-icon-lock"
            placeholder="密码"
            show-password
            style="width: 375px;margin-left: -100px;"
          />
        </el-form-item>
        <el-form-item>
          <drag-verify
            ref="dragVerifyRef"
            text="请按住滑块拖动解锁"
            success-text="验证通过"
            handler-icon="el-icon-d-arrow-right"
            success-icon="el-icon-circle-check"
            :width="375"
            style="margin-left: -100px;"
            handler-bg="#F5F7FA"
            :is-passing.sync="isPassing"
            @update:isPassing="updateIsPassing"
          />
        </el-form-item>
        <!-- <el-form-item class="forgetPassword">忘记密码</el-form-item> -->
        <el-form-item class="loginButtonWrapper">
          <el-button class="loginButton" style="width: 375px;margin-left: -100px;" type="primary" :disabled="submitDisabled" @click.native.prevent="handleLogin">登录</el-button>
        </el-form-item>
        <!-- <el-form-item>
          <el-alert type="success" description="如有问题请邮箱联系：deerlili@163.com" />
        </el-form-item> -->
        <!-- <el-form-item style="text-align:left;">
          其他账号登录：
          <a href="/api/user/login/qq">
            <img :src="qqIcon" style="width: 30px;" />
          </a>
        </el-form-item> -->
      </el-form>
    </div>
    <div style="position: absolute;bottom: 0;width: 100%;background: rgba(204, 204, 204, 0.3);height: 35px;">
      <p style="text-align: center;line-height: 5px;">版权所有©XX科技   2019-2020 技术支持QQ群：XXXXXX</p>
    </div>
  </div>
</template>

<script>
import CanvasNest from 'canvas-nest.js'
import DragVerify from '@/components/DragVerify/index.vue'
// 帆布窝配置
const config = {
  color: '51, 255, 51', // 线条颜色
  pointColor: '255, 62, 150', // 节点颜色
  opacity: 1, // 线条透明度
  count: 199, // 线条数量
  zIndex: 1 // 画面层级
}
export default {
  name: 'Login',
  components: { DragVerify },
  data() {
    const validatePassword = (rule, value, callback) => {
      if (value.length < 6) {
        callback(new Error('长度在 5 到 20 个字符'))
      } else {
        callback()
      }
    }
    return {
      loginForm: {
        username: '',
        password: ''
      },
      loginRules: {
        username: [{ required: true, message: '请输入用户名', trigger: 'blur' }],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 5, max: 20, message: '长度在 5 到 20 个字符', trigger: 'blur' },
          { required: true, trigger: 'blur', validator: validatePassword }
        ]
      },
      capsTooltip: false,
      loading: false,
      showDialog: false,
      isPassing: false,
      submitDisabled: true,
      redirect: undefined,
      otherQuery: {}
    }
  },
  watch: {
    $route: {
      handler: function(route) {
        const query = route.query
        if (query) {
          this.redirect = query.redirect
          this.otherQuery = this.getOtherQuery(query)
        }
      },
      immediate: true
    },
    //  已验证通过时，若重新输入用户名或密码，滑动解锁恢复原样
    'loginForm.username'() {
      this.isPassing = false
      this.$refs.dragVerifyRef.reset()
    },
    'loginForm.password'() {
      this.isPassing = false
      this.$refs.dragVerifyRef.reset()
    }
  },
  created() {
    this.$nextTick(() => {
      const element = document.getElementById('loginBackground')
      new CanvasNest(element, config)
    })
  },
  mounted() {
  },
  destroyed() {
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
      })
    }
  }
}
</script>

<style lang="scss" scoped>
.loginWrapper {
  height: 100%;
  min-height: 550px;
  padding-top: 100px;
  background-color: #2d3a4b;
  .formWrapper {
    width: 375px;
    margin: 0 auto;
    text-align: center;
    .loginTitle {
      margin-bottom: 10px;
      font-weight: 300;
      font-size: 30px;
      color: #7FFFD4;
    }
    .loginSystem {
      font-weight: 300;
      color: #999;
    }
   }
}
</style>
