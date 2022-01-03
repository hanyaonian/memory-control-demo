<template>
<div class="container">
  <label for="count"> 列表数 </label>
  <input name="count" v-model="dataCount" type="number"/>
  <list :listData="dataList"/>
</div>
</template>


<script>
import { ref, watch } from 'vue';
import List from './components/list.vue';

export default {
  components: { List },
  setup() {
    const dataList = ref([]);
    const dataCount = ref(0);
    
    watch(dataCount, (val) => {
      dataList.value = (new Array(val)).fill(0).map((_, i) => {
        return {
          name: i,
          key: i,
        }
      });
    })

    return {
      dataList,
      dataCount,
    }
  },
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
