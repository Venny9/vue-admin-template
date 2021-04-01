<template>
  <div class="app-container">
    <div>id:{{ id }}</div>
    <keep-alive>
      <Graph :url="url" />
    </keep-alive>
  </div>
</template>
<script>
import Graph from '../../../components/Graph/index'
export default {
  components: {
    Graph
  },
  data() {
    return {
      id: '',
      isAll: false,
      url: 'http://9.86.69.48:8081/modifygraph'
    }
  },
  mounted() {
    this.id = this.$route.query.id
  }
}
</script>
