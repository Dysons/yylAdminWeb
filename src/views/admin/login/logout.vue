<template>
  <div v-loading.fullscreen.lock="loading" element-loading-text="正在退出" />
</template>

<script>
export default {
  name: 'Logout',
  components: {},
  data() {
    return { loading: true }
  },
  created() {
    this.logout()
  },
  methods: {
    async logout() {
      await this.$store.dispatch('user/logout')
      this.$router.push('/login')
      // this.$router.push(`/login?redirect=${this.$route.fullPath}`)
    }
  }
}
</script>
