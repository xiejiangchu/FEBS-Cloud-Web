<template>
  <i-frame
    :src="eureka"
    @refresh="refresh"
  />
</template>
<script>
import iFrame from '@/components/iFrame'
import { registerUrl } from '@/settings'

export default {
  components: { iFrame },
  data() {
    return {
      eureka: registerUrl
    }
  },
  mounted() {
    setTimeout(() => {
      this.$notify.info({
        title: this.$t('common.tips'),
        message: this.$t('tips.iframeGrant'),
        duration: 5000
      })
    }, 1000)
  },
  methods: {
    refresh(u) {
      this.eureka = u
    }
  }
}
</script>
