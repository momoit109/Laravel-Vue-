<template>
  <transition props="fade-in">
    <router-view />
  </transition>
</template>

<script>
  import axios from 'axios'
  export default {
    created () {
      axios.interceptors.request.use(config => {
        const token = localStorage.getItem('token')
        if (token) {
          config.headers['Authorization'] = token
        }
        return config
      })

      axios.interceptors.response.use((response) => {
        // 判断一下响应中是否有 token，如果有就直接使用此 token 替换掉本地的 token。你可以根据你的业务需求自己编写更新 token 的逻辑
        var token = response.headers.authorization
        if (token) {
          // 如果 header 中存在 token，那么触发 refreshToken 方法，替换本地的 token
          this.$store.dispatch('refreshToken', token)
        }
        return response
      }, (error) => {
        switch (error.response.status) {
          // 如果响应中的 http code 为 401，那么则此用户可能 token 失效了之类的，我会触发 logout 方法，清除本地的数据并将用户重定向至登录页面
          case 401:
            return this.$store.dispatch('logout')
          // 如果响应中的 http code 为 400，那么就弹出一条错误提示给用户
          case 400:
            this.$Message.error(error.response.data.error)
        }
        return Promise.reject(error)
      })
    }
  }
</script>

<style>
html,
body {
  width: 100%;
  height: 100%;
}
</style>
