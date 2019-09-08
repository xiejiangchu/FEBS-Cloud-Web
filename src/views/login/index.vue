<template>
  <div class="login-container">
    <div class="login-info">
      <div class="title">FEBS Cloud</div>
      <div class="sub-title">{{ $t('common.system') }}</div>
      <div class="desc">1. {{ $t('common.desc.a') }}</div>
      <div class="desc">2. {{ $t('common.desc.b') }}</div>
      <div class="desc">3. {{ $t('common.desc.c') }}</div>
      <div class="desc">4. {{ $t('common.desc.d') }}</div>
      <div class="desc">5. {{ $t('common.desc.e') }}</div>
      <div class="desc">6. {{ $t('common.desc.f') }}</div>
    </div>
    <el-form ref="loginForm" :model="loginForm" :rules="loginRules" class="login-form" autocomplete="off" label-position="left">
      <div class="title-container">
        <h3 class="title">
          {{ $t('login.title') }}
        </h3>
        <lang-select class="set-language" />
      </div>
      <el-form-item prop="username">
        <span class="svg-container">
          <el-icon class="el-icon-user" />
        </span>
        <el-input
          ref="username"
          v-model="loginForm.username"
          :placeholder="$t('login.username')"
          name="username"
          type="text"
          tabindex="1"
          autocomplete="off"
          @keyup.enter.native="handleLogin"
        />
      </el-form-item>
      <el-form-item prop="password">
        <span class="svg-container">
          <el-icon class="el-icon-key" />
        </span>
        <el-input
          :key="passwordType"
          ref="password"
          v-model="loginForm.password"
          :type="passwordType"
          :placeholder="$t('login.password')"
          name="password"
          tabindex="1"
          autocomplete="off"
          @keyup.enter.native="handleLogin"
        />
        <span class="show-pwd" @click="showPwd">
          <svg-icon :icon-class="passwordType === 'password' ? 'eye' : 'eye-open'" />
        </span>
      </el-form-item>
      <el-form-item prop="code" class="code-input">
        <span class="svg-container">
          <el-icon class="el-icon-lock" />
        </span>
        <el-input
          ref="code"
          v-model="loginForm.code"
          :placeholder="$t('login.code')"
          name="code"
          type="text"
          tabindex="1"
          autocomplete="off"
          style="width: 70%"
          @keyup.enter.native="handleLogin"
        />
      </el-form-item>
      <img :src="imageCode" alt="codeImage" class="code-image" @click="getCodeImage">
      <el-button :loading="loading" type="primary" style="width:100%;margin-bottom:14px;" @click.native.prevent="handleLogin">
        {{ $t('login.logIn') }}
      </el-button>
      <span class="login-footer">
        Copyright © 2019 <a target="_blank" href="https://mrbird.cc">MrBird</a>
      </span>
    </el-form>
  </div>
</template>

<script>
import LangSelect from '@/components/LangSelect'
import db from '@/utils/localstorage'
import { randomNum } from '@/utils'
import axios from 'axios'

export default {
  name: 'Login',
  components: { LangSelect },
  data() {
    return {
      loginForm: {
        username: '',
        password: ''
      },
      loginRules: {
        username: { required: true, message: this.$t('rules.require'), trigger: 'blur' },
        password: { required: true, message: this.$t('rules.require'), trigger: 'blur' },
        code: { required: true, message: this.$t('rules.require'), trigger: 'blur' }
      },
      passwordType: 'password',
      loading: false,
      showDialog: false,
      redirect: undefined,
      otherQuery: {},
      randomId: randomNum(24, 16),
      imageCode: ''
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
    }
  },
  mounted() {
    db.clear()
    this.getCodeImage()
  },
  methods: {
    showPwd() {
      if (this.passwordType === 'password') {
        this.passwordType = ''
      } else {
        this.passwordType = 'password'
      }
      this.$nextTick(() => {
        this.$refs.password.focus()
      })
    },
    getCodeImage() {
      axios({
        method: 'GET',
        url: `${process.env.VUE_APP_BASE_API}auth/captcha?key=${this.randomId}`,
        responseType: 'arraybuffer'
      }).then(res => {
        return 'data:image/png;base64,' + btoa(
          new Uint8Array(res.data)
            .reduce((data, byte) => data + String.fromCharCode(byte), '')
        )
      }).then((res) => {
        this.imageCode = res
      }).catch((e) => {
        if (e.toString().indexOf('429') !== -1) {
          this.$message({
            message: this.$t('tips.tooManyRequest'),
            type: 'error'
          })
        } else {
          this.$message({
            message: this.$t('tips.getCodeImageFailed'),
            type: 'error'
          })
        }
      })
    },
    handleLogin() {
      this.$refs.loginForm.validate(valid => {
        if (valid) {
          this.loading = true
          const that = this
          this.$login('auth/oauth/token', {
            ...that.loginForm,
            key: this.randomId
          }).then((r) => {
            const data = r.data
            this.saveLoginData(data)
            this.$get('auth/user').then((r) => {
              this.$store.commit('account/setUser', r.data.principal)
              this.$message({
                message: this.$t('tips.loginSuccess'),
                type: 'success'
              })
              that.loading = false
              this.$router.push('/')
            }).catch((error) => {
              this.$message({
                message: this.$t('tips.loginFail'),
                type: 'error'
              })
              console.error(error)
              that.loading = false
            })
            this.$get(`system/user/success/${that.loginForm.username}`).catch((e) => { console.log(r) })
          }).catch((error) => {
            console.error(error)
            that.loading = false
            this.changeCodeImage()
          })
        } else {
          return false
        }
      })
    },
    saveLoginData(data) {
      this.$store.commit('account/setAccessToken', data.access_token)
      this.$store.commit('account/setRefreshToken', data.refresh_token)
      const current = new Date()
      const expireTime = current.setTime(current.getTime() + 1000 * data.expires_in)
      this.$store.commit('account/setExpireTime', expireTime)
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

<style lang="scss">
$bg:#283443;
$light_gray:#fff;
$cursor: #555;

@supports (-webkit-mask: none) and (not (cater-color: $cursor)) {
  .login-container .el-input input {
    color: $cursor;
  }
}
/* reset element-ui css */
.login-container {
  .el-form-item__content {
    line-height: 27px !important;
  }
  .el-input {
    display: inline-block;
    width: 85%;

    input {
      background: transparent;
      border: 0;
      -webkit-appearance: none;
      border-radius: 0;
      padding: 12px 5px 12px 15px;
      color: #000000;
      height: 32px;
      caret-color: $cursor;

      &:-webkit-autofill {
        box-shadow: 0 0 0 1000px $bg inset !important;
        -webkit-text-fill-color: $cursor !important;
      }
    }
  }

  .el-form-item {
    border: 1px solid #dedede;
    border-radius: 2px;
    color: #454545;
    transition: all .3s;
    &:hover {
      border-color: #57a3f3;
    }
  }
}
</style>
<style lang="scss" scoped>
$bg:#2d3a4b;
$dark_gray:#aaa;
$light_gray:#eee;

.login-container {
  background: url(../../assets/background.jpg) 50% no-repeat;
  background-size: cover;
  width: 100%;
  height: 100vh;
  .login-info {
    position: absolute;
    left: 15%;
    top: 44%;
    margin-top: -100px;
    color: #fff;
    .title {
      font-size: 1.8rem;
      font-weight: 600;
    }
    .sub-title {
      font-size: 1.5rem;
      margin: .3rem 0 .7rem 1rem;
    }
    .desc {
      font-size: .96rem;
      line-height: 1.9rem;
    }
  }
  .login-form {
    position: absolute;
    top: 50%;
    left: 70%;
    margin: -180px 0 0 -160px;
    width: 320px;
    height: 380px;
    padding: 36px;
    background: #fff;
    border-radius: 3px;
    .code-input {
      width: 50%;
      display: inline-block;
      vertical-align: middle;
    }
    .code-image {
      display: inline-block;
      vertical-align: top;
      cursor: pointer;
    }
    .login-footer {
      text-align: center;
      display: inline-block;
      width: 100%;
      font-size: 12px;
      color: #818181;
    }
  }
  .tips {
    font-size: 14px;
    color: #fff;
    margin-bottom: 10px;

    span {
      &:first-of-type {
        margin-right: 16px;
      }
    }
  }

  .svg-container {
    padding: 6px 5px 6px 15px;
    color: $dark_gray;
    font-size: 1.04rem;
    vertical-align: middle;
    margin-bottom: 1px;
    width: 30px;
    display: inline-block;
  }

  .title-container {
    position: relative;

    .title {
      font-size: 20px;
      color: rgba(0,0,0,.85);
      margin: 0 auto 40px auto;
      text-align: center;
      font-weight: bold;
    }

    .set-language {
      color: #aaa;
      position: absolute;
      top: 3px;
      font-size: 18px;
      right: 0;
      cursor: pointer;
    }
  }

  .show-pwd {
    position: absolute;
    right: 10px;
    top: 7px;
    font-size: 16px;
    color: $dark_gray;
    cursor: pointer;
    user-select: none;
  }

  .thirdparty-button {
    position: absolute;
    right: 0;
    bottom: 6px;
  }

  @media only screen and (max-width: 470px) {
    .thirdparty-button {
      display: none;
    }
  }

  @media screen and (max-width: 1100px)
  {
    .login-info {
      left: 8%;
    }
  }

  @media screen and (max-width: 970px)
  {
    .login-form {
      left: 50%;
    }
    .login-info {
      display: none;
    }
  }
}
</style>
