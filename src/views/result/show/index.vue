<template>
  <div class="app-container">
    <div>name</div>
  </div>
</template>
<script>
import axios from 'axios'
axios.get('http://9.86.69.48:8081/callgraph')
  .then((response) => console.log(response))
  .catch(err => console.log(err))

export default {
  data() {
    return {
      data: ''
    }
  }
}
</script>
