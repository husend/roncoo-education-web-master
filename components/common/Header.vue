<template>
  <div class="h_header">
<!--    <div class="h_top" v-if="!hideTop">-->
<!--      <div class="h_top_body">-->
<!--        <ul class="top_list clearfix" v-if="userInfo">-->
<!--          <div v-if="isOpenLecture">-->
<!--            <li><a :href="mainUrl+'/recruit'">讲师招募</a></li>-->
<!--            <li v-if="isTeacher"><nuxt-link :to="{name: 'account-teacher'}">讲师中心</nuxt-link></li>-->
<!--          </div>-->
<!--          <li class="s_left"><nuxt-link :to="{name: 'account-order'}">我的订单</nuxt-link></li>-->
<!--          <li class="s_left">-->

<!--            <nuxt-link :to="{name: 'account'}">{{userInfo.mobile}}</nuxt-link>-->
<!--          </li>-->
<!--          <li><a href="javascript:" @click="signOut">退出</a></li>-->
<!--        </ul>-->
<!--        <ul class="top_list clearfix" v-else>-->
<!--          <li class="s_left"><a href="javascript:" @click="login">登录</a></li>-->
<!--          <li><a href="javascript:" @click="register">注册</a></li>-->
<!--        </ul>-->
<!--      </div>-->
<!--    </div>-->
    <div class="h_nav">
      <div class="h_logo">
        <a :href="mainUrl">
          <img :src="webInfo.logoImg" alt="" v-if="webInfo">
        </a>
      </div>
      <ul class="h_nav_ul clearfix" v-if="!hideTop && navList">
        <li v-for="(item, index) in navList" :key="index">
          <nuxt-link :class="{active: isNow === item.navUrl}" :to="item.navUrl" :target="item.target">{{item.navTitle}}</nuxt-link>
        </li>
      </ul>
      <nuxt-link v-if="!hideTop && userInfo" class="h_nav_link" :to="{name: 'account-course'}">
        <span class="personal">个人中心</span>
      </nuxt-link>
      <a v-if="!hideTop && userInfo" class="h_nav_a" href="javascript:" @click="signOut">
        <span class="singOut">退出登录</span>
      </a>
      <a v-if="!hideTop && !userInfo" class="h_nav_a" @click="login">
        登录/注册
      </a>
      <nuxt-link v-if="hideTop" :to="{name: 'index'}" class="go_index font_14 c_blue">返回首页</nuxt-link>
    </div>
  </div>
</template>
<script>
  import {serviceInfo, getNav} from '~/api/main.js'
export default {
  props: {
    active: {
      type: String
    },
    hideTop: {
      type: Boolean
    },
    hideSearch: {
      type: Boolean
    }
  },
  data () {
    return {
      webInfo: this.$store.state.webInfo,
      mainUrl: this.$store.state.clientData.mainUrl,
      userInfo: '',
      name: '',
      isTeacher: false,
      navList: this.$store.state.navList.list,
      isNow: '',
      isOpenLecture: false
    }
  },
  methods: {
    signOut () {
      this.$store.commit('SIGN_OUT');
      this.userInfo = '';
      console.log(this.$route.path)
      if (this.$route.path.includes('account')) {
        this.$router.push({name: 'login'})
      } else {
        window.location.reload()
      }
      console.log('退出登录')
    },
    login () {
      this.$store.commit('SET_TEMPORARYURL');
      this.$router.push({name: 'login'});
    },
    register () {
      this.$router.push({name: 'login', query: {tab: 2}});
    }
  },
  mounted () {
    this.isNow = this.$route.path;
    this.userInfo = this.$store.state.userInfo;
    if (this.$store.state.tokenInfo && this.userInfo) {
      this.name = this.userInfo.mobile
      if (this.userInfo.userType === 2 || this.userInfo.userType === 4) {
        this.isTeacher = true
      }
    }
  }
}
</script>
<style lang="scss" rel="stylesheet/scss" scoped>
  .h_header {
    background: #fff;
  }
  .h_top {
    background: rgb(51, 51, 51);
    .h_top_body {
      width: 1200px;
      height: 30px;
      margin: 0px auto;
    }
  }
  .top_list {
    float: right;
    li {
      float: left;
      color: #ccc;
      font-size: 12px;
      margin-top: 9px;
      padding: 0px 10px 0px 0px;
      position: relative;
      img {
        width: 14px;
        height: 13px;
        position: absolute;
        left: -20px;
      }
      .vip_icon {
        position: relative;
        left: 0;
        top: 1px;
        cursor: pointer;
      }
      a {
        color: #ccc;
        &:hover {
          text-decoration: none;
          color: #fff;
        }
        &.c_gold {
          color: #CA9E70;
        }
      }
      &.s_left {
        border-left: 1px solid #ccc;
        padding-left: 10px;
      }
    }
  }
  .h_nav {
    width: 1200px;
    margin: 0 auto;
    height: 110px;
    position: relative;
    .go_index {
      position: absolute;
      right: 10px;
      bottom: 20px;
    }
  }
  .h_logo {
    display: inline-block;
    position: absolute;
    top: 25px;
    // left: 20px;
    img {
      height: 52px;
    }
  }
  .h_nav_ul {
    display: inline-block;
    margin-left: 280px;
    li {
      float: left;
      height: 110px;
      line-height: 110px;
      font-size: 16px;
      padding: 0 15px;
      .active {
        color: red;
      }
      a:hover {
        color: red;
        text-decoration: none;
      }
    }
  }
  .h_nav_link {
    display: inline-block;
    margin-left: 174px;
    width: 106px;
    height: 36px;
    line-height: 36px;
    text-align: center;
    border-radius: 6px;
    font-size: 14px;
    border: 1px solid #999;
    color: #999;
    background: #fff;
    position: absolute;
    top: 40px;
    right: 112px;
    cursor: pointer;
    &:hover {
      text-decoration: none;
      color: #D51423;
      border-color: #D51423;
    }
  }
  .h_nav_a {
    display: inline-block;
    margin-left: 280px;
    width: 106px;
    height: 36px;
    line-height: 36px;
    text-align: center;
    border-radius: 6px;
    font-size: 14px;
    border: 1px solid #999;
    color: #999;
    background: #fff;
    position: absolute;
    top: 40px;
    right: 0;
    cursor: pointer;
    &:hover {
      text-decoration: none;
      color: #D51423;
      border-color: #D51423;
    }
  }
</style>
