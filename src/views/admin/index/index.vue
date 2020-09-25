<template>
  <div class="app-container">
    <div class="filter-container">
      <el-row :gutter="0" />
      <back-to-top transition-name="fade" />
    </div>
  </div>
</template>

<script>
import BackToTop from '@/components/BackToTop'

export default {
  name: 'Dashboard',
  components: { BackToTop },
  data() {
    return {}
  },
  computed: {},
  created() {}
}
</script>
