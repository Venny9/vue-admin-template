<template>
  <div class="app-container">
    <div>
      <button @click="showresult">结果</button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      id: '45180'
    }
  },
  methods: {
    showresult() {
      this.$router.push({
        path: '/result/show',
        query: {
          id: this.id
        }
      })
    }
  }
}
</script>
